---
title: Broken Access Control
draft: false
date: 2025-01-01
tags:
  - owasp
  - access_control
  - rbac
---

Broken Access Control is a critical security vulnerability that occurs when applications fail to properly enforce authorization policies, allowing users to access resources, perform actions, or view data beyond their intended privileges. This vulnerability manifests when access control mechanisms are improperly implemented, missing, or can be bypassed through manipulation of requests, URLs, or application state.

**Impact Severity:** Broken access control can lead to unauthorized data disclosure, data modification, privilege escalation, account takeover, and complete system compromise. According to OWASP, this vulnerability consistently ranks as the #1 web application security risk.

**Attack Vectors:** Attackers exploit these vulnerabilities through URL manipulation, parameter tampering, token modification, forced browsing, privilege escalation techniques, and session hijacking.

## Common Vulnerability Patterns:

### 1. **Insecure Direct Object References (IDOR)**
```http
GET /api/users/1234/profile HTTP/1.1
Authorization: Bearer user_token_5678
```
Attacker changes user ID to access other users' data:
```http
GET /api/users/9999/profile HTTP/1.1
Authorization: Bearer user_token_5678
```

### 2. **Vertical Privilege Escalation**
```http
POST /admin/users HTTP/1.1
Content-Type: application/json
Authorization: Bearer regular_user_token

{"username": "newadmin", "role": "administrator"}
```

### 3. **Horizontal Privilege Escalation**
```http
GET /dashboard?user_id=victim_user_id HTTP/1.1
Cookie: session=attacker_session_id
```

### 4. **Forced Browsing (Directory Traversal)**
```http
GET /admin/panel HTTP/1.1
GET /backup/database.sql HTTP/1.1
GET /../../../etc/passwd HTTP/1.1
```

### 5. **Method-based Bypass**
```http
# If POST is protected but PUT isn't
PUT /api/users/1234 HTTP/1.1
Content-Type: application/json

{"role": "admin", "permissions": ["all"]}
```

### 6. **Parameter Tampering**
```html
<!-- Hidden form fields manipulation -->
<input type="hidden" name="user_role" value="admin">
<input type="hidden" name="price" value="0.01">
```

### 7. **JSON/XML Parameter Injection**
```json
{
  "user_id": 1234,
  "role": "user",
  "admin": true,
  "permissions": ["read", "write", "delete"]
}
```

## Comprehensive Prevention Strategies

### 1. **Implement Robust Authorization Frameworks**

**Role-Based Access Control (RBAC) Implementation:**
```python
from enum import Enum
from typing import List, Dict, Any

class Role(Enum):
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"
    GUEST = "guest"

class Permission(Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"
    ADMIN = "admin"

class AccessControl:
    def __init__(self):
        self.role_permissions = {
            Role.ADMIN: [Permission.READ, Permission.WRITE, Permission.DELETE, Permission.ADMIN],
            Role.MODERATOR: [Permission.READ, Permission.WRITE, Permission.DELETE],
            Role.USER: [Permission.READ, Permission.WRITE],
            Role.GUEST: [Permission.READ]
        }

    def check_permission(self, user_role: Role, required_permission: Permission) -> bool:
        """Check if user role has required permission."""
        return required_permission in self.role_permissions.get(user_role, [])

    def authorize_resource_access(self, user_id: int, resource_owner_id: int,
                                user_role: Role, required_permission: Permission) -> bool:
        """Authorize access to a specific resource."""

        # Check if user has the required permission
        if not self.check_permission(user_role, required_permission):
            return False

        # For non-admin users, ensure they can only access their own resources
        if user_role != Role.ADMIN and user_id != resource_owner_id:
            return False

        return True

# Usage example
access_control = AccessControl()

def get_user_profile(requesting_user_id: int, target_user_id: int, user_role: Role):
    """Secure endpoint for getting user profile."""

    if not access_control.authorize_resource_access(
        requesting_user_id, target_user_id, user_role, Permission.READ
    ):
        raise PermissionError("Access denied: Insufficient privileges")

    return fetch_user_profile(target_user_id)
```

### 2. **Server-Side Authorization Enforcement**

**Middleware-based Authorization:**
```python
from functools import wraps
from flask import request, jsonify, g

def require_permission(permission: Permission):
    """Decorator to enforce permission requirements."""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Extract user from authenticated session
            user = g.current_user

            if not user:
                return jsonify({"error": "Authentication required"}), 401

            if not access_control.check_permission(user.role, permission):
                return jsonify({"error": "Insufficient privileges"}), 403

            return f(*args, **kwargs)
        return decorated_function
    return decorator

def require_resource_ownership():
    """Decorator to ensure user can only access their own resources."""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            user = g.current_user
            resource_id = kwargs.get('user_id') or request.view_args.get('user_id')

            # Admins can access any resource
            if user.role == Role.ADMIN:
                return f(*args, **kwargs)

            # Regular users can only access their own resources
            if str(user.id) != str(resource_id):
                return jsonify({"error": "Access denied"}), 403

            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Usage in Flask routes
@app.route('/api/users/<int:user_id>/profile', methods=['GET'])
@require_permission(Permission.READ)
@require_resource_ownership()
def get_user_profile(user_id):
    return jsonify(get_user_data(user_id))
```

### 3. **Secure Object Reference Implementation**

**Indirect Object References:**
```python
import uuid
from typing import Dict, Optional

class SecureObjectReference:
    def __init__(self):
        self.reference_map: Dict[str, int] = {}
        self.reverse_map: Dict[int, str] = {}

    def create_reference(self, actual_id: int, user_id: int) -> str:
        """Create secure reference for an object."""
        # Generate UUID-based reference
        reference = str(uuid.uuid4())

        # Store mapping with user context
        key = f"{user_id}:{reference}"
        self.reference_map[key] = actual_id
        self.reverse_map[actual_id] = reference

        return reference

    def resolve_reference(self, reference: str, user_id: int) -> Optional[int]:
        """Resolve secure reference to actual ID."""
        key = f"{user_id}:{reference}"
        return self.reference_map.get(key)

# Usage example
secure_refs = SecureObjectReference()

@app.route('/api/documents/<reference>')
@require_permission(Permission.READ)
def get_document(reference: str):
    user = g.current_user

    # Resolve secure reference to actual document ID
    document_id = secure_refs.resolve_reference(reference, user.id)

    if not document_id:
        return jsonify({"error": "Document not found"}), 404

    # Additional authorization check
    if not user_can_access_document(user.id, document_id):
        return jsonify({"error": "Access denied"}), 403

    return jsonify(get_document_data(document_id))
```

### 4. **Input Validation and Sanitization**

```python
from marshmallow import Schema, fields, validate, ValidationError

class UserUpdateSchema(Schema):
    """Schema for validating user update requests."""
    username = fields.Str(validate=validate.Length(min=3, max=50))
    email = fields.Email()
    # Explicitly exclude sensitive fields
    role = fields.Str(dump_only=True)  # Read-only field
    permissions = fields.List(fields.Str(), dump_only=True)
    is_admin = fields.Bool(dump_only=True)

def validate_user_input(json_data: dict, user_role: Role) -> dict:
    """Validate and sanitize user input based on role."""

    # Regular users cannot modify role-related fields
    if user_role != Role.ADMIN:
        sensitive_fields = ['role', 'permissions', 'is_admin', 'account_status']
        for field in sensitive_fields:
            if field in json_data:
                raise ValidationError(f"Field '{field}' cannot be modified")

    schema = UserUpdateSchema()
    try:
        return schema.load(json_data)
    except ValidationError as err:
        raise ValidationError(f"Validation failed: {err.messages}")
```

### 5. **Session Security and Token Management**

```python
import jwt
import hashlib
from datetime import datetime, timedelta
from typing import Dict, Any

class SecureSessionManager:
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        self.active_sessions: Dict[str, Dict] = {}

    def create_session_token(self, user_id: int, user_role: str,
                           session_data: Dict[str, Any]) -> str:
        """Create secure JWT session token."""

        payload = {
            'user_id': user_id,
            'role': user_role,
            'session_id': self.generate_session_id(user_id),
            'issued_at': datetime.utcnow().isoformat(),
            'expires_at': (datetime.utcnow() + timedelta(hours=8)).isoformat(),
            'permissions': self.get_user_permissions(user_role)
        }

        token = jwt.encode(payload, self.secret_key, algorithm='HS256')

        # Store session metadata
        self.active_sessions[payload['session_id']] = {
            'user_id': user_id,
            'created_at': datetime.utcnow(),
            'last_activity': datetime.utcnow(),
            'ip_address': session_data.get('ip_address'),
            'user_agent': session_data.get('user_agent')
        }

        return token

    def validate_session_token(self, token: str, request_ip: str) -> Dict[str, Any]:
        """Validate session token and check security constraints."""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=['HS256'])

            # Check expiration
            expires_at = datetime.fromisoformat(payload['expires_at'])
            if datetime.utcnow() > expires_at:
                raise jwt.ExpiredSignatureError("Session expired")

            # Validate active session
            session_id = payload['session_id']
            if session_id not in self.active_sessions:
                raise jwt.InvalidTokenError("Session invalidated")

            session_data = self.active_sessions[session_id]

            # Check IP consistency (optional security measure)
            if session_data['ip_address'] != request_ip:
                self.log_security_event('ip_mismatch', session_id, request_ip)

            # Update last activity
            session_data['last_activity'] = datetime.utcnow()

            return payload

        except jwt.InvalidTokenError as e:
            raise PermissionError(f"Invalid session: {str(e)}")

    def generate_session_id(self, user_id: int) -> str:
        """Generate unique session identifier."""
        data = f"{user_id}:{datetime.utcnow().isoformat()}:{uuid.uuid4()}"
        return hashlib.sha256(data.encode()).hexdigest()
```

### 6. **Real-time Security Monitoring**

```python
import logging
from datetime import datetime, timedelta
from collections import defaultdict

class AccessControlMonitor:
    def __init__(self):
        self.failed_access_attempts = defaultdict(list)
        self.suspicious_patterns = defaultdict(int)

    def log_access_attempt(self, user_id: int, resource: str,
                          action: str, success: bool, ip_address: str):
        """Log all access attempts for monitoring."""

        log_entry = {
            'timestamp': datetime.utcnow(),
            'user_id': user_id,
            'resource': resource,
            'action': action,
            'success': success,
            'ip_address': ip_address
        }

        # Log to security system
        logging.info(f"Access attempt: {log_entry}")

        if not success:
            self.failed_access_attempts[user_id].append(log_entry)
            self.detect_suspicious_activity(user_id, ip_address)

    def detect_suspicious_activity(self, user_id: int, ip_address: str):
        """Detect patterns indicating potential attacks."""

        current_time = datetime.utcnow()
        recent_failures = [
            attempt for attempt in self.failed_access_attempts[user_id]
            if current_time - attempt['timestamp'] < timedelta(minutes=15)
        ]

        # Alert on multiple failed access attempts
        if len(recent_failures) > 5:
            self.alert_security_team(
                f"User {user_id} has {len(recent_failures)} failed access attempts from {ip_address}"
            )

        # Detect privilege escalation attempts
        escalation_patterns = ['admin', 'root', 'superuser', 'elevated']
        for attempt in recent_failures:
            if any(pattern in attempt['resource'].lower() for pattern in escalation_patterns):
                self.alert_security_team(
                    f"Potential privilege escalation attempt by user {user_id}"
                )
                break

    def alert_security_team(self, message: str):
        """Send security alerts to monitoring systems."""
        logging.critical(f"SECURITY ALERT: {message}")
        # Integrate with SIEM, Slack, email, etc.
```
