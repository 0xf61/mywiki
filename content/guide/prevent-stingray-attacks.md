---
title: Prevent Stingray Attacks on Android
draft: false
date: 2025-05-12
tags:
  - android
  - security
  - privacy
---

## Introduction

In this guide, we will explore what stingrays are, how they pose a threat to your privacy, and the steps you can take to protect yourself by disabling 2G on your Android device.

## What are "stingrays"?

A "stingray" is a fake cell tower designed to trick phones into connecting to it. While law enforcement agencies sometimes use stingrays for legitimate purposes, such as locating individuals, they have also been misused to spy on people at protests or by malicious actors exploiting 2G security vulnerabilities.

### Why are stingrays dangerous?

When a phone connects to a stingray, the operator can access sensitive information, including:

- The phone's IMSI (a unique identifier)
- Details of phone calls (e.g., who you called and call duration)
- Text messages and call content
- Websites you visit

![Stingray intercepting people communications](https://www.bleepstatic.com/images/news/u/1220909/Diagrams/stingray.jpg)
_Source: EFF_

## How to protect yourself

### Disable 2G on Android

#### Method 1: Using the Phone App

1. Open your phone app and dial `*#*#4636#*#*`.
2. Navigate to **Phone Information**.
3. Set "Preferred Network Type" to one of the following options:

   - `LTE-only` (4G)
   - `LTE/WCDMA` (4G/3G)
   - `NR/LTE/WCDMA` (5G/4G/3G)

   > **Note:** Choosing `LTE-only` will also disable 3G, which may impact your coverage and roaming capabilities. If you want to disable only 2G, select `LTE/WCDMA`.

#### Method 2: Android Settings (Android 12 and above)

1. Go to **Settings → Network & Internet → SIMs → Allow 2G**.
2. Toggle the option to disable 2G.

   ![Android option to disable 2G](https://www.bleepstatic.com/images/news/u/1220909/devices/allow-option.png)
   _Source: EFF_

   > **Note:** This feature is available only on Android 12 and newer versions. Additionally, 2G will still be used for emergency calls, even if disabled.

## Conclusion

Disabling 2G is a simple yet effective step to enhance your privacy and security. While it won't solve all security issues, it significantly reduces the risk of falling victim to stingray attacks. Stay informed and take control of your digital safety!
