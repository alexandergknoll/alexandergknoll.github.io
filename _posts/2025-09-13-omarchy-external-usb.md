---
title: Installing Omarchy to an external USB device & troubleshooting boot issues
date: 2025-09-13 14:57:00
categories: [Omarchy, Linux]
tags: [omarchy, arch-linux, usb-installation, bootloader, limine, troubleshooting]
---

## Introduction

Over the past week or so, I've been test driving [Omarchy](https://omarchy.org/) as my primary OS. Omarchy is an opinionated Arch Linux distribution that ships with the modern tiling window manager [Hyprland](https://hypr.land/), something I had been interested in trying for some time. I wanted to install Omarchy to an external USB drive so that I could fall back to my existing Arch Linux installation on my internal drive if I ran into any issues.  The installation process seemed to complete successfully, however I encountered boot failures that prevented the system from starting properly.

This guide walks through my process of solving the boot issue so that I could boot into my Omarchy installation located on an external USB device. It also shows how to configure Omarchy's bootloader Limine to ensure future kernel updates maintain the correct boot parameters.

## The Problem: Failed Root Mount on Boot

After successfully installing Omarchy to an external USB device, you may encounter the following error during boot:

```terminal
ERROR: Failed to mount ' ' on real root.
You are now being dropped into an emergency shell.
sh: can't access tty; job control turned off
```

This error typically occurs because the bootloader cannot locate or mount the encrypted root partition. The issue stems from incorrect or missing kernel command line parameters in the Limine bootloader configuration.

## Prerequisites

Before proceeding with the fix, ensure you have:

- Access to the emergency shell or another Linux system (I was able to perform the steps below by booting to my Arch installation on my internal disk, but you could also do this by booting to a live CD environment or similar)
- Root privileges on the system
- Basic understanding of Linux block devices and mounting

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
- `sda1`: Boot partition (2GB)
- `sda2`: Encrypted root partition (230.9GB)

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

- `cryptdevice=UUID=...:root`: Specifies the encrypted device and maps it to `/dev/mapper/root`
- `root=/dev/mapper/root`: Sets the root filesystem location after decryption
- `rootflags=subvol=@`: Mounts the Btrfs subvolume named "@" as root
- `rootfstype=btrfs`: Specifies Btrfs as the root filesystem type
- `rw`: Mounts the root filesystem as read-write
- `quiet splash`: Suppresses verbose boot messages and shows a splash screen

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

## Troubleshooting Additional Issues

### USB Device Not Detected

If your system doesn't detect the USB device during boot:
- Ensure your BIOS/UEFI settings allow USB booting
- Try different USB ports (prefer USB 3.0 ports)

### Encryption Password Not Prompted

If the system doesn't prompt for the encryption password:
- Verify the UUID in your configuration matches the output of `blkid`
- Ensure the `cryptdevice` parameter syntax is correct
- Check that the necessary kernel modules for encryption are loaded

## Best Practices for External USB Installations

1. **Use High-Quality USB Drives**: Invest in a fast, reliable USB 3.0 or USB-C drive with good write endurance
2. **Regular Backups**: External drives are more prone to failure; maintain regular system backups
3. **Monitor Drive Health**: Use tools like [`smartctl`](https://man.archlinux.org/man/smartctl.8.en) to monitor the USB drive's health
4. **Keep Configuration Files Backed Up**: Save copies of your bootloader configurations

## Conclusion

Installing Omarchy to an external USB device provides excellent portability for your custom Linux environment. While boot issues can occur due to the unique challenges of USB-based installations, they're typically resolved by properly configuring the bootloader with the correct kernel command line parameters.

The key to a successful external USB installation is ensuring the bootloader can locate and mount the encrypted root partition with appropriate delays for USB device initialization. By following this guide and implementing the persistent configuration changes, your Omarchy installation should boot reliably across different systems.

Remember to test your USB installation on multiple machines to ensure compatibility and consider keeping a backup configuration in case of drive failure or corruption.

Have questions or comments?  Sign in with GitHub to comment below!