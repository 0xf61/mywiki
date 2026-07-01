---
title: GitHub URL Hacks
draft: false
date: 2025-06-18
tags:
  - tips
  - github
---

GitHub's user interface has gotten much better over the years, but sometimes you just need quick access without clicking. Here are a few GitHub URL tips to get you the data you want faster. One cool thing is that all of these tips give raw text output, so they work great with `curl` and other CLI tools.

## Public SSH keys

If you want to get a user's public SSH keys, you can add `.keys` to the end of their user profile URL. Here's mine.

<https://github.com/0xf61.keys>

## Profile image

If you want to get a user's profile picture, you can add `.png` to the end of their user profile URL.

<https://github.com/0xf61.png>

## Public GPG keys

If you want to get GPG public keys, you can add `.gpg` to the end of their user profile URL.

<https://github.com/0xf61.gpg>

## RSS feeds

There are lots of different feeds you can subscribe to.

Repo commits

<https://github.com/$USER/$REPO/commits.atom>

Repo releases

<https://github.com/$USER/$REPO/releases.atom>

Repo tags

<https://github.com/$USER/$REPO/tags.atom>

### User feeds

Public RSS feed will show public user activity, like repo stars and releases.

<https://github.com/0xf61.atom>

There's also a private user feed, which is great if you don't log into GitHub often.

It requires you to click in the user interface, but I still find it incredibly useful. Log in to your account and on your dashboard, scroll all the way to the bottom and click "Subscribe to your news feed". This will generate a private token automatically and send you to `https://github.com/$USER.private.atom?token=...`

You can plug this directly into an RSS reader and it'll include everything that normally shows up on your private dashboard feed, like repos and users you follow, and project releases.

### Security advisories

This is a public RSS feed for GitHub security advisories.

<https://github.com/security-advisories.atom>

### Global timeline

<https://github.com/timeline>

## Diffs

You can compare branches in a repo by adding `/compare/[fork-user:]$BRANCH...$BRANCH` to the end of a repo URL.

If you want to compare a `ai` branch to the `main` branch for my mywiki, you can check out:

<https://github.com/0xf61/mywiki/compare/main...ai>

If you had a fork of the repo, you could add your username before `main` like this:

<https://github.com/0xf61/mywiki/compare/$USER:main...ai>

The cool thing is you can get a raw patch or diff output using the same URL and adding `.patch` and `.diff` to the end.

<https://github.com/0xf61/mywiki/compare/main...ai.patch>

<https://github.com/0xf61/mywiki/compare/main...ai.diff>
