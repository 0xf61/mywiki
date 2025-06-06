---
title: Android SSL Certificate - Burp Setup
draft: false
date: 2025-05-11
tags:
  - android
  - proxy
  - ssl
  - security
---

If you see the error `The Client failed to negotiate an SSL Connection` on the Burp Dashboard, you can fix it by installing the Burp CA certificate on the device at the **SYSTEM** level.

With Nougat and later systems, it is no longer possible to capture application requests by installing the certificate at the traditional **USER** level. In Android, system certificates are stored in `/system/etc/security/cacerts`.

```bash
openssl x509 -inform DER -in cacert.der -out cacert.pem
openssl x509 -inform PEM -subject_hash_old -in cacert.pem |head -1
mv cacert.pem <hash>.0
```

```bash
adb root
adb remount
adb push <cert>.0 /sdcard/
```

```bash
mv /sdcard/<cert>.0 /system/etc/security/cacerts/
chmod 644 /system/etc/security/cacerts/<cert>.0
```

TL;DR

```bash
openssl x509 -inform DER -in cacert.der -out cacert.pem
export CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1)
adb root && adb remount
adb push cacert.pem "/sdcard/${CERT_HASH}.0"
adb shell su -c "mv /sdcard/${CERT_HASH}.0 /system/etc/security/cacerts"
adb shell su -c "chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0"
rm cacert.*
```

You probably also want to configure the proxy settings on the phone. You can change it with the command below instead of the ridiculous Android interface.

```bash
adb shell settings put global http_proxy 192.168.56.1:8080
```

[References](https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/)
