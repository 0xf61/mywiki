---
title: Restoring Android Signal DB from Desktop Files
draft: false
date: 2025-05-25
tags:
  - signal
  - android
  - backup
---

I recently upgraded my phone and tried to import my Signal backup but accidentally created an empty one, preventing me from restoring my old backup. Fortunately, I had a linked Signal desktop app, which allowed me to recover parts of my chat history.

It's goofy how asymmetric the signal environment is. It's heavily based on having access to a cellular phone and nominally that is the 2FA authentication source. Signal desktop on its own has no functionality without being paired to a phone.

## Step 1: Take a minute and collect files

You have all your chats in the desktop app. Close the application and make a copy of the files. These files are your source material. Ensure the application is fully closed, and confirm this using the task manager.

## Step 2: Generate a valid backup on Android

You don't have any messages, so there's nothing to lose on your Android phone. Create a new backup, which should only take a moment and be under 5MB. Save the backup recovery passphrase somewhere safe, as you'll need it later. This is a 30-character string used to restore your backup.

## Step 3: Copy the backup file and new passkey

Transfer the backup file that previously generated on android phone to PC. The file will be named something like `signal-2025-05-25-11-09-26.backup`, where the date and time reflect when the backup was created.

Next let's grab signalbackup tools from [signal Backup tools on github](https://github.com/bepaald/signalbackup-tools)].

Nix users can use this comamnd instead

```bash
nix-shell --packages signalbackup-tools
```

## Step 4: Command Prompt

In the command prompt, replace the 30 zeros with your Signal recovery passphrase. This is the passphrase you saved earlier. The command will look like this:

**Notice:** The name on the output file is slightly ahead of the input file.

```bash
signalbackup-tools ~/signal-2025-05-25-11-09-26.backup 000000000000000000000000000000 --importfromdesktop --output ~/signal-2025-05-25-11-09-27.backup
```

If you're running the normal signal desktop path, signal backup tools will grab the files to rebuild a backup file, using the basically empty backup file as a reference. If your signal install is somewhere else, make you life easier and just copy those files to that location. This can take a bit of time, as shown below, so be patient.

## Step 5: Copy and import

Delete the Signal app storage and its data, or simply reinstall the app. Open Signal, select "Restore from Backup," and use the backup file you generated earlier. You can use this command to download a larger Signal backup if needed.

```bash
python3 -m http.server 1234
```

Restore your signal backup with the downloaded backup and enter your backup recovery passphrase when prompted. This will restore your messages from the desktop files.

Old messages are back and visible without an issue. Group messages are spotty, some imported without issues and some did not show up correctly, but individual messages were visible without issue.

[Reference](https://transistor-man.com/restoring_android_signal_from_desktop.html)
