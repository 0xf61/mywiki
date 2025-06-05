---
title: Data Tampering
draft: false
date: 2025-01-01
tags:
  - owasp
  - data_tampering
  - integrity
  - validation
  - server_side_validation
---

Data tampering is a critical security vulnerability where unauthorized modifications are made to application data, either during transmission (man-in-the-middle attacks) or at rest (direct storage manipulation). This attack compromises data integrity and can lead to privilege escalation, financial fraud, business logic bypass, and system compromise.

Tampering attacks exploit vulnerabilities in input validation, client-side controls, insecure data transmission, inadequate server-side verification, and weak session management. Attackers manipulate these weaknesses to alter application state, circumvent security controls, and achieve unauthorized objectives.

**Primary Attack Vectors:**
- **HTTP Request Manipulation**: Modifying parameters, headers, and payloads
- **Session Token Tampering**: Altering authentication and authorization tokens
- **Client-Side Logic Bypass**: Circumventing JavaScript validation and business rules
- **Database Injection**: Direct manipulation of stored data through injection attacks
- **API Parameter Modification**: Tampering with REST/GraphQL API requests

```html
<input type="hidden" name="price" value="9.99">
<input type="hidden" name="product_id" value="123">
```

An attacker can use browser developer tools or proxy tools like Burp Suite to modify the price parameter:

```http
POST /checkout HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

price=0.01&product_id=123
```

If the system doesn't double-check the price, the attacker can buy the item for much less.

If authentication or authorization data is stored in unprotected cookies, attackers can manipulate these values:

```http
Cookie: role=user; sessionId=abc123def456; privileges=basic
```

Using browser developer tools or cookie editors, attackers can escalate privileges:

```http
Cookie: role=admin; sessionId=abc123def456; privileges=superuser
```

can give them access to administrator features if the system trusts the cookie without checking.

REST APIs accepting JSON payloads are vulnerable when server-side validation is insufficient:

```json
{
  "user_id": 1001,
  "amount": 100.00
}
```

Attackers can intercept and modify the request to perform unauthorized transfers:

```json
{
  "user_id": 1002,
  "amount": 10000.00
}
```

If important data is only checked on the website itself, attackers can bypass these checks by changing the website's code or using tools to send altered values directly.

## Comprehensive Prevention Strategies

### 1. **Server-Side Validation and Sanitization**
- **Input Validation**: Implement strict whitelisting, regex validation, and data type checking
- **Parameter Binding**: Use parameterized queries and prepared statements for database operations
- **Business Logic Enforcement**: Re-validate all business rules server-side regardless of client checks
- **Schema Validation**: Use JSON Schema or similar tools for API request validation

### 2. **Data Integrity Protection**
- **HMAC Signatures**: Implement HMAC-SHA256 for sensitive cookie and token data
- **JWT Security**: Use properly signed JSON Web Tokens with rotating secrets
- **Digital Signatures**: Apply RSA/ECDSA signatures for critical data integrity
- **Merkle Trees**: Implement for tamper-proof verification of large datasets

### 3. **Secure Architecture Design**
- **Zero Trust Principle**: Never trust client-side data; always verify server-side
- **Stateless Design**: Minimize server state; recalculate critical values per request
- **Defense in Depth**: Implement multiple validation layers
- **Fail-Secure Design**: Default to secure state when validation fails

### 4. **Transport and Communication Security**
- **TLS 1.3**: Use latest transport encryption with perfect forward secrecy
- **Certificate Pinning**: Implement for mobile applications and critical services
- **HSTS Implementation**: Enable HTTP Strict Transport Security headers
- **Content Security Policy**: Use CSP headers to prevent client-side tampering

### 5. **Monitoring and Anomaly Detection**
- **Real-time Monitoring**: Implement immediate detection of suspicious patterns
- **Behavioral Analytics**: Monitor for unusual user behavior and transaction patterns
- **Audit Logging**: Maintain comprehensive logs with user actions, timestamps, IP addresses
- **SIEM Integration**: Use Security Information and Event Management systems

### 6. **Access Control and Authorization**
- **Principle of Least Privilege**: Grant minimal necessary permissions
- **RBAC Implementation**: Use role-based access control with fine-grained permissions
- **ABAC Systems**: Implement attribute-based access control for complex scenarios
- **Session Management**: Secure session handling with proper timeout and regeneration

### 7. **Cryptographic Controls**
- **Hash Functions**: Use SHA-256 or stronger for data integrity verification
- **Encryption at Rest**: Encrypt sensitive data in databases and storage systems
- **Key Management**: Implement proper cryptographic key lifecycle management
- **Salt and Pepper**: Use unique salts and global peppers for password hashing

*Reference: OWASP Top 10 Security Risks*
