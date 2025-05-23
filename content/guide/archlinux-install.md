---
title: Hardened Arch Linux Installation
draft: false
date: 2025-05-11
tags:
  - archlinux
  - systemd-boot
  - btrfs
  - luks
---

## Writing the ISO File

Download the ISO file from [Link](https://archlinux.org/download/).

Identify the connected USB drive using `lsblk` as /dev/sd\[a\|b\|c\], and write the ISO file to the drive.

```bash
dd bs=4M if=archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress && sync
```

## UEFI Settings

Enter the UEFI/BIOS settings by pressing the F13 key several times while the computer is starting up.

Under the `Security` category, first define a `Supervisor Password`. If you don't want to enter a BIOS/UEFI password every time the system starts and easily get **Evil Maid**ed, don't do this.

Again, under the same `Security` category, activate `Secure Boot` and delete all saved secure boot keys, then open `Setup mode`.

Save the settings and exit.

## Boot Me Baby

Restart the system and press the F13 key again to boot the system from the USB drive. Hopefully, you remembered your UEFI password :).

When GRUB appears, select "Arch Linux install medium ..." (the top one) and continue.

After a series of realistic-looking texts that can't be used in hack movies, a `root@archiso` prompt appears, and our action begins.

**For those who don't have elf eyes, you may want to change the font**

```bash
setfont /usr/share/kbd/consolefonts/ter-132n.psf.gz # Be lazy, use TAB
```

## No Ethernet Port/Cable, Let's Solve it with WiFi

It's easier to connect to WiFi with iwctl :)

```bash
iwctl
device list # Powered on :)
station wlan0 scan
station wlan0 get-networks
station wlan0 connect $SSID
# A bunch of unnecessary password questions
exit
```

Let's see if it worked, use ICMP to control both DNS and internet access.

```bash
ip a                    # An important detail for those who will SSH to learn the IP address
ping -c 1 archlinux.org # Checking that Internet and DNS packets are working
```

## TTY sucks, SSH rocks

After providing the Internet connection, change the password with `passwd` in order to provide an SSH connection to the device.

You can now continue the installation by connecting via SSH from another system using `ssh root@192.168.x.x`.

## Partitioning

I am finding the disk that has not been mounted yet with `lsblk`. Because I have an nvme disk, this disk appears as `nvme0n1` on my computer. At this point, you need to adjust the disk name according to yourself.

I'm doing a clean pass over the disk. (I recommend you do it even for a short time)

```bash
dd if=/dev/urandom of=/dev/nvme0n1
```

Now I don't see a partition under nvme when `lsblk` is done.

I don't find it appropriate to divide the disk into more than 2 partitions. I will create partitions for Boot and ROOT.

```bash
gdisk /dev/nvme0n1
# At this stage, if it asks whether I found MBR and want to fix it, pass with Enter
# Command (? for help):        # This means prompt for gdisk
o # Create GPT table.
y # Yes create my last decision

# /boot
n
1
<Enter>
+200M
EF00

# /
n
<Enter>
<Enter>
<Enter>
<Enter>

w # we are recording
Y # Yes, I realize that I have deleted the entire disk table at this stage

# The operation has completed successfully.
```

## LUKS

Now we will encrypt the disk with cryptsetup, but this stage is completely up to you. If desired, you can directly continue with `cryptsetup -v -y luksFormat /dev/nvme0n1p2`.

**Additions for those who want to suffer**

The default parameters are considered in the bottom line with `cryptsetup --help`. The expected write speeds and supported algorithms from the disk are determined with `cryptsetup benchmark`.

The preferred parameters are replaced with the places in the example below and a passphrase is defined.

```fallback
cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha256 --iter-time 2000 --key-size 256 --pbkdf argon2id --use-urandom --verify-passphrase luksFormat /dev/nvme0n1p2
YES      # Last exit before the bridge, proceed by entering the passphrase.
```

This step may take some time.

To avoid confusion, this disk is re-opened and mounted.

```bash
cryptsetup open /dev/nvme0n1p2 luks
```

## Format Partitions

Partitions are formatted with the desired file system. fat32 is ideal for boot.

```bash
mkfs.vfat -F32 /dev/nvme0n1p1
mkfs.btrfs /dev/mapper/luks
```

BTRFS subvolumes are created and unmounted.

```bash
mount /dev/mapper/luks /mnt
btrfs sub create /mnt/@
btrfs sub create /mnt/@home
umount /mnt
```

It is remounted using BTRFS subvolumes.

```bash
mount -o noatime,nodiratime,compress=zstd:1,space_cache,ssd,subvol=@ /dev/mapper/luks /mnt
# if it cries cannot disable free space tree
# btrfs check --clear-space-cache v2 /dev/mapper/luks
# then repeat step 1
mkdir -p /mnt/{boot,home}
mount -o noatime,nodiratime,compress=zstd:1,space_cache,ssd,subvol=@home /dev/mapper/luks /mnt/home
mount /dev/nvme0n1p1 /mnt/boot
```

## Pacstrap

Before downloading the necessary tools, let's use some new features.

In `vim /etc/pacman.conf`, correct the line under `[options]` as `ParallelDownloads = 42`. It might be a bit too much, 17 is not a bad number either :).

```bash
pacstrap /mnt linux-hardened{,-headers} linux-firmware base base-devel btrfs-progs intel-ucode vim
```

Let's create machine-id [;)](https://github.com/Whonix/dist-base-files/blob/master/etc/machine-id)

```bash
echo b08dfa6083e7567a1921a715000001fb > /mnt/etc/machine-id
```

## Chroot Time

Now we can edit our system as if we have booted by chrooting.

```bash
arch-chroot /mnt/
```

We may want to log in after reboot, so we change the root password with `passwd`.

Let's create our user and make basic settings.

```bash
useradd -mG wheel user
passwd user
visudo # %wheel ALL=(ALL) ALL
```

Let's make standard system settings

```bash
echo host > /etc/hostname
echo en_US.UTF-8 UTF-8  >> /etc/locale.gen
locale-gen
ln -sf /usr/share/zoneinfo/UTC /etc/localtime # Add 3 hours basic math
hwclock --systohc
```

/etc/hosts

```bash
127.0.0.1 localhost
::1 localhost
127.0.1.1 <YOUR-HOSTNAME>.localdomain <YOUR-HOSTNAME>
```

/etc/pacman.conf

```bash
Color
CheckSpace
VerbosePkgLists
ILoveCandy
ParallelDownloads = 17
```

vim /etc/mkinitcpio.conf # I'm only writing the lines to be changed

```bash
MODULES=(btrfs)
HOOKS=(base udev systemd autodetect keyboard modconf block sd-encrypt filesystems)
```

Let's create all our kernel initramfs with `mkinitcpio -P`.

Let's also install basic userspace packages.

```bash
pacman -S alsa-utils apparmor dialog dosfstools fish git mtools networkmanager pipewire pipewire-alsa pipewire-pulse sbctl hyprland wpa_supplicant xdg-user-dirs xdg-utils
```

After watching a pacman video for a while, the downloaded services are activated.

```bash
systemctl enable NetworkManager apparmor
```

## Bootloader (systemd-boot)

I use systemd-boot, which is simple to install and runs fast. (no hate grub)

I am performing the installation with `bootctl --path=/boot install`.

Since I will use UUID in its configuration, I am putting it into the file beforehand so that I don't have to write it manually.

```bash
blkid -s UUID -o value /dev/nvme0n1p2 > /boot/loader/entries/arch.conf
```

vim /boot/loader/entries/arch.conf

```bash
title Arch Linux
linux /vmlinuz-linux-hardened
initrd /intel-ucode.img
initrd /initramfs-linux-hardened.img
options rd.luks.name=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX=luks root=/dev/mapper/luks rootflags=subvol=@ rd.luks.options=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX=discard rw quiet lsm=lockdown,yama,apparmor,bpf
```

vim /boot/loader/loader.conf

```bash
default arch.conf
editor no
```

## Secureboot

We have previously downloaded the tools required for secure boot.

We can look at the situation with `sbctl status`. We need to part ways with 3 crosses.

```bash
Installed:      ✗ Sbctl is not installed
Setup Mode:     ✗ Enabled
Secure Boot:    ✗ Disabled
```

First, let's create and write our secureboot keys.

```bash
sbctl create-keys
sbctl enroll-keys
sbctl status
```

Let's sign the kernel images we created.

```bash
sbctl sign /boot/EFI/BOOT/BOOTX64.EFI
sbctl sign /boot/EFI/systemd/systemd-bootx64.efi
sbctl sign /boot/vmlinuz-linux-hardened
```

All images must be signed.

```bash
sbctl verify
```

## Final Touch

Time to test our one-shot bullet. We exit Chroot, unmount the disks and restart.

```bash
exit
umount -a
reboot
```
