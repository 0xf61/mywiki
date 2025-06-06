---
title: IIS Tilde Enumeration
draft: false
date: 2025-01-01
tags:
  - owasp
  - iis
  - tilde_enumeration
  - information_disclosure
  - directory_enumeration
  - microsoft_iis
---

IIS Tilde Enumeration (sometimes called the IIS Short Filename Vulnerability) uses the way old Windows systems used to support 8.3 short filenames.

This problem comes from old DOS-style naming in Windows. If short filename creation is turned on, each long filename also has a short 8.3 name. IIS, depending on how it is set up, might respond differently when a correct or incorrect short name is requested, showing directory or file structures that are not meant to be seen.

If the real folder on the server is `SecretAdmin`, the short name might be `SECRE~1`. An attacker might try URLs like:

```http
GET /SECRE~1/ HTTP/1.1
Host: example.com
```

- If the server responds with a `200 OK` (or a `403/401`, meaning it exists but is not allowed), the attacker knows the folder probably exists.
- If it responds with a `404 Not Found`, the guess was wrong, and they try another short name.

Similarly, if a file is named `ImportantConfig.txt` in the `Config` directory, the attacker might test requests for `IMPOR~1.TXT` in that directory:

```http
GET /Config/IMPOR~1.TXT HTTP/1.1
Host: example.com
```

Different server responses or error messages can show that the file is there, even if it is not linked anywhere on the site.

1. **Disable 8.3 Filename Creation**

- If your Windows version and application allow it, you can turn off 8.3 short file name creation on new drives using registry settings or system policies.
- (Remember that changing this setting might affect older applications.)
- For example, on some Windows systems, you can change:

```regedit
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
NtfsDisable8dot3NameCreation = 1
```

1. **Apply Security Patches and Updates**
- Make sure you are using the latest version of IIS and Windows.
- Microsoft has released updates that make it harder to find file or directory information using the short name method.
1. **Restrict Folder and File Access**
- Use Access Control Lists (ACLs) to protect important directories and files, preventing unauthorized access even if short filename enumeration shows they exist.
- Set up strong authorization checks in IIS to make sure only the right users can access important resources.
