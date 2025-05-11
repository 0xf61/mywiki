---
title: Block Stringray attack on Android
draft: false
date: 2025-05-11
tags:
  - android
  - security
  - privacy
---

## What are "stingrays"?

A "stingray" is a fake cell tower. It tricks phones into connecting to it.

Sometimes, the police use stingrays to find people but, stingrays have also been used to spy on people at protests.
Regular people have also used stingrays to spy on others. This is because 2G connections have security problems.
Stingrays can force phones to use 2G, even if the phone can use 4G. This lets them use the old security problems to spy on people.

Being able to turn off 2G is a good thing. It doesn't solve every security problem, but it's a good start.

When a phone connects to a stingray, the people controlling the stingray can see personal information, like:

- The phone's IMSI (a special ID number)
- Phone call details (who you called, how long you talked)
- Text messages and call content
- What websites you visit

![Stingray intercepting people communications](https://www.bleepstatic.com/images/news/u/1220909/Diagrams/stingray.jpg)
_Source: EFF_

Open your phone app, then type:

## How to turn off 2G on Android

`*#*#4636#*#*`

Then enter **Phone Information**.

"Set Preferred Network Type" to "LTE-only" (4g), LTE/WCDMA (4G/3G), NR/LTE/WCDMA (5G/4G/3G)

It should be noted that LTE-only will also disable 3G - which may seriously affect your coverage - and your device's Preferred Roaming List (PRL), which may prevent your device from roaming.

If roaming doesn't matter to you, and you want to disable **only 2G**, select "LTE/WCDMA".

## Google now lets you turn off 2G on your Android phone. But, it's turned on by default.

To turn it off, go to ' **Settings → Network & Internet → SIMs → Allow 2G**'. The exact steps might be different depending on your phone.

![Android option to disable 2G](https://www.bleepstatic.com/images/news/u/1220909/devices/allow-option.png)
_Source: EFF_

This option is only available on Android 12 for now. It's not available on Android 11 or 10.

Google says that your phone needs to have special hardware to use this option.

Also, 2G will still be used for emergency calls, even if you turn it off.
