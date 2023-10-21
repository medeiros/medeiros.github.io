---
layout: post
title: "Arch Linux: Installation and Minimalistic Setup"
description: >
  Description of steps to setup a brand new Arch Linux installation with 
  minimalistic approach.
categories: linux
tags: [archlinux]
comments: true
image: /assets/img/blog/arch/archlinux.png
---
> The new setup of an Arch Linux environment is a demanding task, and 
also a very personal experience. 
In this page, I'll describe a step-by-step method to setup a brand new 
Arch Linux environment with my personal minimalistic approach.
{:.lead}

- Table of Contents
{:toc}

## Project: Minimalistic Arch Linux

### Hardware 

This is the laptop hardware spec I adopted in this article:

- Dell Latitude 3420
  - BIOS version 1.29.0
- Processor
  - 11th Gen Intel(R) Core(TM) i7-1165G7 @2.80GHz
  - Clock Speed: min 0.4 GHz - max 4.7 GHz
  - Cache: L2 5120kB; L3 12288 kB
  - Microcode Version: A4
  - 4 Core count (Hyper-Threading Capable)
  - 64-bit technology
- Memory  
  - Installed: 16384 MB 
  - Speed: 3200 Mhz
  - Channel Mode: Single
  - Technology: DDR4 SDRAM
  - DIMM Slot 1: 16384 MB SO-DIMM
  - DIMM Slot 2: Empty
- Device: Video
  - Panel type: 14'' FHD
  - Video Controller: Intel(R) Iris(R) Xe Graphics
  - Video Memory: 64 MB
  - Native Resolution: 1920 by 1080
  - Video BIOS Version: 17.0.1061
- Other Devices
  - Audio Controller: Realtek ALC3204
  - WiFi Device: Intel Wireless
  - Bluetooth Device: installed
  - Keyboard layout: Portuguese Brazilian (ABNT-2)
  - Time Zone: America/Sao Paulo (UTC-3)

Also, an USB flash drive will be required (to install Arch Linux), along with 
Internet access. 

I already have a external drive (Linux ext4) that will be referred in this 
article as well.

### System Requirements

The requirements for this system are the following:

- **Security at boot**: Dell Security Password should applied at BIOS, in 
order to password validation to be prompted at every boot. It is an additional 
layer to protect against unauthorized access (theft prevention, mostly);
- **Dual boot**: with previous Windows 11 system (Windows is still a 
requirement in some particular scenarios - mostly related to data 
compatibility);
- **External data files**: my data files will reside on an external drive, so 
the local disk will only maintain OS and application files; 
- **Minimalism is the key**: the idea is to have a simple and clean 
environment, without distractions in order to maximize productivity.

### Key concepts

These are key concepts that are important to know beforehand:

- **Dual boot**: Windows 11 only supports x86_64 and a boot in UEFI mode from 
GPT disk [^5]: this is important to know, so all analysis must consider that 
the boot mode must be UEFI;
- **Dual boot**: Windows cannot recognize other OS than itself on boot - but 
Linux can; 
- **SecureBoot**: Dell SecureBoot is supported only by Windows, and not by 
ArchLinux. So, it must be disabled;
- **BitLocker**: Once Dell SecureBoot is disabled, Windows will activate 
BitLocker. So, it is mandatory to know your Bitlocker PIN.

## Preparation: Windows

### Windows 11 previously installed

I still need to maintain a Windows system, mostly because of 
compatibility issues in Windows application files.

Windows is not able to perform dual boot with Linux systems (but Linux is able
to detect and work with Windows systems at boot). So, the best approach in 
terms of dual boot is to configure Arch Linux dual boot only after a Windows 
installation has already took place.

This article assumes that Windows 11 is installed in the laptop before Arch
Linux.

### Make sure you have your Windows Bitlocker PIN

In the next steps, the Dell SecureBoot will be disabled, what will enable the 
Windows Bitlocker. So, you will need to make sure that you know your 
Bitlocker PIN in order to keep accessing Windows.

If you don't know your Bitlocker PIN, you never used it before (or if you want 
to change it for some reason), please follow the steps below in Windows [^1]:

- Press Windows key + R to invoke the Run dialog;
- In the Run dialog box, type control and hit Enter to open Control Panel;
- Now set the panel view to Large icons;
- Click on Bitlocker Drive Encryption;
- Now click on Change PIN;
- If you know the Old PIN, enter it, then enter the New PIN and click Change 
PIN button. If you don’t know the Old PIN, then click on the "Reset a 
Forgotten PIN", enter the new PIN and Confirm it. Click on Set PIN and 
restart the system once and check.

Save this 6-character code for later.

### Create partitions in hard drive for Arch through Windows

In Windows, use the Disk Management Utility to create the following new empty 
partitions in the laptop hard drive:

- **a 1GB partition**: that will be used for Linux swap
- **a 49GB partition**: that will be used for Arch OS files and general 
applications (since my data files will reside on external drive, additional 
space is not required).

> Please make sure to not removing existing partitions while allocating space 
for these new partitions - the page [Arch Wiki: Dual boot With Windows
](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "2.1.2 - 
Installation / Windows Before Linux / BIOS Systems / UEFI Systems" [^2] 
give you a better clue.

### Download ArchLinux image and write it a USB flash drive

The next step is to create a bootable USB flash drive with ArchLinux system 
for installation. We will do this through Windows. 

#### Downloading Arch Linux image

One can find the Arch linux ISO image at [Arch Linux download 
page](https://archlinux.org/download/). At this page, find your closest 
mirror and download the *x86_64.iso* file (usually 808MB size).

> The HTTP direct URL may be something like 
> http://archlinux.c3sl.ufpr.br/iso/2023.10.14/archlinux-2023.10.14-x86_64.iso

#### Writing Arch ISO in a USB flash drive

Since the ISO is downloaded, the next step is to write it to a valid USB 
flash drive.

For this scenario (UEFI-only booting, since we're using Windows 11), it 
is enough to extract the ISO contents onto a FAT-formatted USB flash 
drive. [^3]

So, execute the following steps in order to write the ISO in the USB flash 
drive: [^3]:

- Partition the USB flash drive and format it to FAT32.
- Right click on archlinux-version-x86_64.iso and select Mount.
- Navigate to the newly created DVD drive and copy all files and folders to 
the USB flash drive.
- When done copying, right click on the DVD drive and select Eject.
- Eject the USB flash drive.

And that's it. All Windows steps were made and now it's time to work on the 
BIOS.

## Preparation: Dell BIOS

### Accessing Dell BIOS

Since we are dealing with dual boot and hardware security, some changes may 
be made in Dell BIOS. 

In order to get access to your laptop BIOS:

- Boot your Dell Laptop
- Press F12 multiple times (in order to get access to BIOS)

Once in the BIOS, some actions may be performed, as below.

### Disable Secure Boot

Dell laptop have Secure Boot enabled by default, which is compatible with 
Windows but not with Arch Linux. So, this feature must be disabled for 
dual boot to work.

To disable Dell Secure Boot in BIOS:

- Click on the menu _"Boot Configuration"_ 
- In the _"Secure Boot / Enable Secure boot"_ section, make sure that the 
option is **OFF** 

> After that, when booting your computer, Windows will detect that SecureBoot 
is no longer active and will start prompting for Bitlocker's PIN. At this 
point, hopefully you have your Bitlocker PIN 6-character code at hand. If you 
don't know your PIN, reboot and enable the BIOS Secure Boot back again, so you 
can reach Windows and set your PIN (as above described, in this article). Then, 
with Bitlocker PIN at hand, disable secure boot again to get back to this 
point. Now, we can proceed.

> If, for some reason, you lost your PIN and you can't change SecureBoot, you 
may be stuck in the Bitlocker blue screen. In order to recover your Bitlocker 
PIN, press ESC and then enter the Bitlocker Recovery Key (you can get this key 
in "Microsoft Recovery Key" page [^4]).

### Prepare boot from USB flash drive

- Connect your USB flash drive
- Re-enter into Dell BIOS
- Click on the menu _"Boot Configuration"_ 
- In the _"Boot Sequence"_ section, locate your USB flash drive. 
- Move the USB flash device using the arrows - make sure that it will be 
above "Windows Boot Manager" 

## Installation: Arch Linux

### Basic installation

The best way to install Arch Linux in my opinion is to follow the excellent
Arch Wiki on that regard: 
[Installation Guide](https://wiki.archlinux.org/title/Installation_guide). 
Since the USB flash drive is ready to go, you can skip steps previous to 
1.4 and go directly to the section "_1.4 - Boot the live environment_".

From that section till the end of page, there are some little adjustments 
that I consider important to mention:

- _section 1.5. Set the console keyboard layout and font_: for Brazil, use 
`loadkeys br-abnt2` 
- _section 1.9.1. Example layouts_: make sure that the partition layout is 
similar but slightly different - as below:

Mount point | Partition | Partition type | Size
--- | --- | --- | ---
/mnt/boot | /dev/efi_system_partition | EFI system partition | 320 MB
[SWAP] | /dev/swap_partition | Linux swap | 1 GB
/mnt | /dev/root_partition | Linux x86-64 root (/) | Remainder of the device

- _section 3.3. Time Zone_: for Brazil, use `ln -sf 
/usr/share/zoneinfo/America/Sao_Paulo /etc/localtime`
- _sectiion 3.4. Localization_: in `/etc/vconsole.conf`, for Brazil, set 
`KEYMAP=br-abnt2`. No need to change the locale.

### Configuring Dual Boot with rEFInd

#### Defining a boot loader

In order to configure Dual Boot, the first step is to select a UEFI Boot 
Loader. [Arch Linux provides a list of available boot loaders
](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader), which 
can be defined as a "a piece of software started by the firmware 
(BIOS or UEFI)" ... that "is responsible for loading the kernel with 
the wanted kernel parameters and any external initramfs images." [^6]

Out of all this options, my choice is for 
[rEFInd](https://wiki.archlinux.org/title/REFInd).

#### Locating ESP: Boot Loader partition

Please note that _esp_ denotes the mountpoint of the EFI system 
partition [^7]. It will probably be **/boot**. In order to verify, 
execute as below:

```
$ findmnt /boot
TARGET SOURCE         FSTYPE OPTIONS
/boot  /dev/nvme0n1p1 vfat   rw,relatime,fmask=0022,...

$ lsblk | grep boot
├─nvme0n1p1 259:1    0   320M  0 part /boot
```

It appears that `/boot` is a mount point that maps to the EFI partition 
(vfat, 320MB). But let's investigate deeper.

Since Windows is already in place in this laptop, this _esp_ directory 
should not be empty. There should be an EFI directory, with vendor subdirs, 
as below:

```
daniel@ataraxia /boot> pwd
/boot

daniel@ataraxia /boot> ll
total 99M
drwxr-xr-x  4 root root 4.0K Dec 31  1969  ./
drwxr-xr-x 17 root root 4.0K Sep 16 19:02  ../
drwxr-xr-x  7 root root 4.0K Oct 20 21:29  EFI/
-rwxr-xr-x  1 root root  72M Sep 16 19:57  initramfs-linux-fallback.img*
-rwxr-xr-x  1 root root  15M Sep 16 19:56  initramfs-linux.img*
drwxr-xr-x  2 root root 4.0K Jun 28  2022 'System Volume Information'/
-rwxr-xr-x  1 root root  13M Sep 16 19:56  vmlinuz-linux*

daniel@ataraxia /boot> ll EFI
total 28K
drwxr-xr-x 7 root root 4.0K Oct 20 21:02 ./
drwxr-xr-x 4 root root 4.0K Dec 31  1969 ../
drwxr-xr-x 2 root root 4.0K Jun 28  2022 Boot/
drwxr-xr-x 5 root root 4.0K Sep 16 15:04 dell/
drwxr-xr-x 4 root root 4.0K Jun 28  2022 Microsoft/
drwxr-xr-x 2 root root 4.0K Sep 16 19:28 tools/
```

#### Locating ArchOS disk partuuid

You need to discover the partuuid of the disk device where Arch OS will be 
installed. In order to do so, follow as below:

```
# findmnt /
TARGET
  SOURCE         FSTYPE OPTIONS
/ /dev/nvme0n1p4 ext4   rw,relatime

# ls -la /dev/disk/by-partuuid | grep nvme0n1p4 | cut -d' ' -f 10
<it will give you an uuid, like 1c234579-21g1-48aa-7d1u-111x4555d123>
```

This partuuid code will be required later on, during rEFInd configuration.

> It is not mandatory that partuuid may be used in that case. Other options 
may be adopted as well - using partuuid is my particular approach.

#### Installing rEFInd boot loader

Just install the refind package:

```
sudo pacman -S refind
```

After that, let's proceed with the manual installation.

> The Arch Linux Wiki Page [explains in 
depth](https://wiki.archlinux.org/title/REFInd#Manual_installation) how to 
perform this installation, but this can be hard to apply, since the 
definitions are broad and generic. This section aim to simplify this 
process by defining the exact steps to execute in this particular 
context [^8].

First, copy the executable file to the ESP directory:

```
# mkdir -p esp/EFI/refind
# cp /usr/share/refind/refind_x64.efi esp/EFI/refind/
```

Then use efibootmgr to create a boot entry in the UEFI NVRAM:

```
# pacman -S efibootmgr

# efibootmgr --create --disk /dev/nvme0n1 --part p1 \
  --loader /EFI/refind/refind_x64.efi \
  --label "rEFInd Boot Manager" --unicode
```

At this point, rEFInd is installed, but not configured.

#### Configuring rEFInd Boot Loader

Now, create a config file `refind.conf`, as below:

```
# cp /usr/share/refind/refind.conf-sample esp/EFI/refind/refind.conf
```

You can also copy icons, if you prefer (not required):

```
# cp -r /usr/share/refind/icons esp/EFI/refind/
```

Then, edit the refind.conf file as below:

```
# vim esp/EFI/refind/refind.conf
```

In this file, the customized parts are pointed below. Make sure that the 
following entries are set, so rEFInd boot manager will be configured to 
show a text option to choose between Windows and ArchLinux, and it will 
wait for 5 seconds until execute the boot for the selected option:

```bash
# Timeout in seconds for the main menu screen. Setting the timeout to 0
# disables automatic booting (i.e., no timeout). Setting it to -1 causes
# an immediate boot to the default OS *UNLESS* a keypress is in the buffer
# when rEFInd launches, in which case that keypress is interpreted as a
# shortcut key. If no matching shortcut is found, rEFInd displays its
# menu with no timeout.
#
timeout 5

# Whether to store rEFInd's rEFInd-specific variables in NVRAM (1, true,
# or on) or in files in the "vars" subdirectory of rEFInd's directory on
# disk (0, false, or off). Using NVRAM works well with most computers;
# however, it increases wear on the motherboard's NVRAM, and if the EFI
# is buggy or the NVRAM is old and worn out, it may not work at all.
# Storing variables on disk is a viable alternative in such cases, or
# if you want to minimize wear and tear on the NVRAM; however, it won't
# work if rEFInd is stored on a filesystem that's read-only to the EFI
# (such as an HFS+ volume), and it increases the risk of filesystem
# damage. Note that this option affects ONLY rEFInd's own variables,
# such as the PreviousBoot, HiddenTags, HiddenTools, and HiddenLegacy
# variables. It does NOT affect Secure Boot or other non-rEFInd
# variables.
# Default is true
#
use_nvram false

# Use text mode only. When enabled, this option forces rEFInd into text mode.
# Passing this option a "0" value causes graphics mode to be used. Pasing
# it no value or any non-0 value causes text mode to be used.
# Default is to use graphics mode.
#
textonly

# Which non-bootloader tools to show on the tools line, and in what
# order to display them:
#  shell            - the EFI shell (requires external program; see rEFInd
#                     documentation for details)
#  memtest          - the memtest86 program, in EFI/tools, EFI/memtest86,
#                     EFI/memtest, EFI/tools/memtest86, or EFI/tools/memtest
#  gptsync          - the (dangerous) gptsync.efi utility (requires external
#                     program; see rEFInd documentation for details)
#  gdisk            - the gdisk partitioning program
#  apple_recovery   - boots the Apple Recovery HD partition, if present
#  windows_recovery - boots an OEM Windows recovery tool, if present
#                     (see also the windows_recovery_files option)
#  mok_tool         - makes available the Machine Owner Key (MOK) maintenance
#                     tool, MokManager.efi, used on Secure Boot systems
#  csr_rotate       - adjusts Apple System Integrity Protection (SIP)
#                     policy. Requires "csr_values" to be set.
#  install          - an option to install rEFInd from the current location
#                     to another ESP
#  bootorder        - adjust the EFI's (NOT rEFInd's) boot order
#  about            - an "about this program" option
#  hidden_tags      - manage hidden tags
#  exit             - a tag to exit from rEFInd
#  shutdown         - shuts down the computer (a bug causes this to reboot
#                     many UEFI systems)
#  reboot           - a tag to reboot the computer
#  firmware         - a tag to reboot the computer into the firmware's
#                     user interface (ignored on older computers)
#  fwupdate         - a tag to update the firmware; launches the fwupx64.efi
#                     (or similar) program
#  netboot          - launch the ipxe.efi tool for network (PXE) booting
# Default is shell,memtest,gdisk,apple_recovery,windows_recovery,mok_tool,about,hidden_tags,shutdown,reboot,firmware,fwupdate
# To completely disable scanning for all tools, provide a showtools line
# with no options.
#
#showtools shell, bootorder, gdisk, memtest, mok_tool, apple_recovery, windows_recovery, about, hidden_tags, reboot, exit, firmware, fwupdate
showtools 

# Set the default menu selection.  The available arguments match the
# keyboard accelerators available within rEFInd.  You may select the
# default loader using:
#  - A digit between 1 and 9, in which case the Nth loader in the menu
#    will be the default.
#  - A "+" symbol at the start of the string, which refers to the most
#    recently booted loader.
#  - Any substring that corresponds to a portion of the loader's title
#    (usually the OS's name, boot loader's path, or a volume or
#    filesystem title).
# You may also specify multiple selectors by separating them with commas
# and enclosing the list in quotes. (The "+" option is only meaningful in
# this context.)
# If you follow the selector(s) with two times, in 24-hour format, the
# default will apply only between those times. The times are in the
# motherboard's time standard, whether that's UTC or local time, so if
# you use UTC, you'll need to adjust this from local time manually.
# Times may span midnight as in "23:30 00:30", which applies to 11:30 PM
# to 12:30 AM. You may specify multiple default_selection lines, in which
# case the last one to match takes precedence. Thus, you can set a main
# option without a time followed by one or more that include times to
# set different defaults for different times of day.
# The default behavior is to boot the previously-booted OS.
#
#default_selection 1
#default_selection Microsoft
#default_selection "+,bzImage,vmlinuz"
#default_selection Maintenance 23:30 2:00
#default_selection "Maintenance,macOS" 1:00 2:30
default_selection Arch

# Below is a more complex Linux example, specifically for Arch Linux.
# This example MUST be modified for your specific installation; if nothing
# else, the PARTUUID code must be changed for your disk. Because Arch Linux
# does not include version numbers in its kernel and initrd filenames, you
# may need to use manual boot stanzas when using fallback initrds or
# multiple kernels with Arch. This example is modified from one in the Arch
# wiki page on rEFInd (https://wiki.archlinux.org/index.php/rEFInd).
menuentry "Arch Linux" {
    icon     /EFI/refind/icons/os_arch.png
    volume   "Arch Linux"
    loader   /vmlinuz-linux
    initrd   /initramfs-linux.img
    options "rw root=/dev/disk/by-partuuid/<add here the Arch OS partuuid>"
    submenuentry "Boot using fallback initramfs" {
        initrd /initramfs-linux-fallback.img
    }
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
    #disabled
}

```

At this point, rEFInd is properly set. Now, restart the computer and 
configure the last steps at BIOS.

### Finishing configuration in Dell BIOS

These are final actions in Dell BIOS:

- Remove your USB flash drive (if plugged)
- Enter into Dell BIOS
- Click on the menu _"Boot Configuration"_ 
- In the _"Boot Sequence"_ section, make sure that
  - "rEFInd Boot Manager" is the top item
  - "Windows Boot Manager" is disabled

That's it! Your laptop is dual boot ready, with Windows 11 and Arch Linux!

## Post-Installation

The following actions are related to the configuration of Arch Linux 
for personal usage. 

### Mount an external drive for data

In my setup, my personal data will be located in an external drive, plugged 
in a USB port. 
I want to always mount this drive at startup if the external drive is 
plugged, but ignore at startup otherwise.

The solution is to add a custom entry in `/etc/fstab` file.

First, let's plug the device and check how it is mapped:

```
$ sudo fdisk -l

Disk /dev/sda: 931.48 GiB, 1000170586112 bytes, 1953458176 sectors
Disk model: Elements SE 25FE
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0112dc8c

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1        2048 1953458175 1953456128 931.5G 83 Linux
```

My external drive is mapped as `/dev/sda1`.

Now, I want to obtain the UUID related to this device:

```
$ ls -l /dev/disk/by-uuid/ | grep sda1

lrwxrwxrwx 1 root root 10 Oct 21 08:57 11e22d6a-11b1-2a2b-3b30-dd1412d1221x -> ../../sda1
```

I want to map this device to an entry point `~/data` (inside my home directory).
So, an entry must be added to the `/etc/fstab`, as following: 

```
$ sudo vim /etc/fstab
```

```bash
# /dev/sda1
UUID=<add-uuid> /home/daniel/data ext4 rw,nofail,x-systemd.device-timeout=3  0 2
```
Understanding this entry in depth:
- the external device is mapped as a `data` subdir, inside my home dir. I like 
this kind of abstraction in Linux, where "everything is a file", and also it 
can give me the pleasant illusion that an external drive can be treated as part 
of the home filesystem;
- the external drive that I own is an ext4 Linux filesystem, hence it must 
be mounted as ext4;
- the option `rw` mounts the drive with read/write permission;
- the option `nofail` ignores mounting if the device is not plugged;
- the option `x-systemd.device-timeout` works with `nofail`, and waits for 3 
seconds for the external drive to be plugged in before give up;
- the param `0` indicates that is a (deprecated) backup operation, which 
means "no backup" (if 1, it means "dump utility backup") [^9]
- the param `2` indicates file system check order: the value 2 means that 
the check will occur after 1 (the root filesystem) [^9]

And that's it. You can restart to validate the solution. You can also plug 
the external drive at any time and type `sudo mount -a`; this command will 
read the `/etc/fstab` file and apply the mount action immediately.



TODO: add setup details (configure external drive, etc)

## References

[^1]: [Change Bitlocker PIN in Windows10](https://www.thewindowsclub.com/change-bitlocker-pin-in-windows-10))
[^2]:[Arch Wiki: Dual boot With Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "2.1.2 - Installation / Windows Before Linux / BIOS Systems / UEFI Systems" 
[^3]: [ArchWiki: USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium): see section "2.3.2 - Using manual formatting/UEFI only/In Windows" 
[^4]: [Microsoft Recovery Key Page](https://account.microsoft.com/devices/recoverykey)
[^5]: [Arch Wiki: Dual boot with Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "1.1 - Important Information/Windows UEFI vs BIOS limitations"
[^6]: [Arch boot process: Boot loader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader)
[^7]: [Arch Linux: rEFInd - Beginning](https://wiki.archlinux.org/title/REFInd#)
[^8]: [Arch Wiki: rEFInd - Manual Installation](https://wiki.archlinux.org/title/REFInd#Manual_installation)
[^9]: [Red Hat: An introduction to the Linux /etc/fstab file](https://www.redhat.com/sysadmin/etc-fstab)

