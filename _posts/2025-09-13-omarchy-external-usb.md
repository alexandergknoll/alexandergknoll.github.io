---
title: Installing Omarchy to an external USB device & troubleshooting boot issues
date: 2025-09-13 14:57:00
categories: [Omarchy, Linux]
tags: [omarchy, arch-linux, usb-installation, bootloader, limine, troubleshooting]
---

## Introduction

Over the past week or so, I've been test driving [Omarchy](https://omarchy.org/) as my primary OS. Omarchy is an opinionated Arch Linux distribution that ships with the modern tiling window manager [Hyprland](https://hypr.land/), something I had been interested in trying for some time. I wanted to install Omarchy to an external USB drive so that I could fall back to my existing Arch Linux installation on my internal drive if I ran into any issues.  The installation process seemed to complete successfully, however I encountered boot failures that prevented the system from starting properly.

This post describes the problem in more detail and provides step-by-step instructions to resolve the issue.

## The Problem: Failed Root Mount on Boot

After successfully installing Omarchy to an external USB device, I encountered the following error during boot:

```terminal
ERROR: Failed to mount ' ' on real root.
You are now being dropped into an emergency shell.
sh: can't access tty; job control turned off
```

The issue in this case was that the bootloader cannot locate or mount the encrypted root partition. I needed to do the following to fix the issue:

1. Fix the kernel command line parameters in the Limine bootloader configuration so that I could mount and boot to the partition on my USB device
2. Update the Limine configuration so that future updates to the kernel/OS do not break the bootloader configuration

## Prerequisites

To fix the issue, you will need the following:

* Access to the emergency shell or another Linux system
* Root privileges on the system
* Basic understanding of Linux block devices and mounting

In my case, I was able to perform the steps below by booting to my Arch installation on my internal disk, but you could also do this by booting to a live CD environment.

## Step-by-Step Solution

### 1. Identify Block Devices

First, identify your USB device and its partitions using the `lsblk` command:

```terminal
lsblk
```

Expected output should look similar to:

```terminal
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                                             8:0    0 232.9G  0 disk  
├─sda1                                          8:1    0     2G  0 part  
└─sda2                                          8:2    0 230.9G  0 part  
nvme0n1                                       259:0    0 465.8G  0 disk  
├─nvme0n1p1                                   259:1    0   300M  0 part  /boot/efi
├─nvme0n1p2                                   259:2    0 456.7G  0 part  
```

In this example, `sda` represents the external USB device with:
* `sda1`: Boot partition (2GB)
* `sda2`: Encrypted root partition (230.9GB)

### 2. Mount the Boot Partition

Mount the boot partition to access the Limine bootloader configuration:

```terminal
sudo mount /dev/sda1 /mnt/
```

Replace `/dev/sda1` with your actual boot partition device path if different.

### 3. Retrieve the Root Partition UUID

Get the UUID of your encrypted root partition:

```terminal
sudo blkid /dev/sda2
```

This command will output something like:
```terminal
/dev/sda2: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTUUID="..."
```

Copy the UUID value as you'll need it for the next step.

### 4. Update Limine Configuration

Edit the Limine bootloader configuration file, `/mnt/limine.conf`, and locate the boot entry for Omarchy.  Update the `kernel_cmdline` parameter with the following configuration:

```terminal
kernel_cmdline: cryptdevice=UUID=YOUR_UUID_HERE:root root=/dev/mapper/root rootflags=subvol=@ rootfstype=btrfs rw quiet splash
```

Replace `YOUR_UUID_HERE` with the actual UUID you obtained in step 3.

#### Explanation of Kernel Parameters

* `cryptdevice=UUID=...:root`: Specifies the encrypted device and maps it to `/dev/mapper/root`
* `root=/dev/mapper/root`: Sets the root filesystem location after decryption
* `rootflags=subvol=@`: Mounts the Btrfs subvolume named "@" as root
* `rootfstype=btrfs`: Specifies Btrfs as the root filesystem type
* `rw`: Mounts the root filesystem as read-write
* `quiet splash`: Suppresses verbose boot messages and shows a splash screen

### 5. Configure Persistent Boot Parameters

To ensure future kernel updates maintain the correct boot parameters, edit the Limine entry tool configuration file `/etc/limine-entry-tool.conf`.  Add or modify the following line:

```terminal
KERNEL_CMDLINE[default]+="cryptdevice=UUID=YOUR_UUID_HERE:root root=/dev/mapper/root rootflags=subvol=@ rootfstype=btrfs rw quiet splash"
```

Replace `YOUR_UUID_HERE` with the actual UUID you obtained in step 3.

This configuration ensures that whenever the kernel is updated, the correct boot parameters are automatically applied to new Limine entries.

## Verification and Testing

After making these changes:

1. Unmount the boot partition:
   ```terminal
   sudo umount /mnt
   ```

2. Reboot your system and select the USB device from your BIOS/UEFI boot menu

3. The system should now boot successfully and prompt for your encryption password

## Troubleshooting

If the system doesn't prompt for the encryption password:

* Verify the UUID in your configuration matches the output of `blkid`
* Ensure the `cryptdevice` parameter syntax is correct
* Check that the necessary kernel modules for encryption are loaded

## Conclusion

This was a minor stumbling block when setting up Omarchy for the first time; once I was able to fix the bootloader I was able to get up and running with Omarchy with no issues on my [Framework 13](https://frame.work/) laptop.

**Questions or comments?**  Sign in with GitHub to comment below!