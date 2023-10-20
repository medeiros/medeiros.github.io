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

## Hardware and approach

This is the laptop hardware spec I will adopt in this article:

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

My approach for this setup considers the following:

- **Security at boot**: Dell Security Password applied at BIOS, in order to 
password validation to be prompted at every boot. It is an additional layer 
to protect against unauthorized access (theft prevention, mostly)
- **Dual boot**: with previous Windows 11 system (Windows is still a 
requirement in some particular scenarios - regarding compatibility, mostly)
- **External data files**: my data files reside on an external drive, so the 
local disk only have OS and application files 
- **Minimalism is the key**: the idea is to have a simple and clean 
environment, without distractions in order to maximize productivity.

## Key concepts

These are key concepts that are important to know beforehand:

- In dual boot: Windows 11 only supports x86_64 and a boot in UEFI mode 
from GPT disk [^5]: this is important to know, so all analysis must 
consider that the boot mode must be UEFI;
- Windows cannot recognize other OS than itself on boot - but Linux can; 
- Dell SecureBoot is supported only by Windows, and not by ArchLinux. So, 
it will be disabled;
- Once Dell SecureBoot is disabled, Windows will activate BitLocker. So, 
it is mandatory to know/define your Bitlocker PIN;

## Preparation: Windows

### Windows 11 previously installed

I still have the urge to maintain a Windows system, mostly because of 
compatibility issues.

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

Since the ISO is downloaded, the next step is to write it to an valid USB 
flash drive.

For this scenario (UEFI-only booting, since we're using Windows 11), it 
is enough to extract the ISO contents onto a FAT-formatted USB flash 
drive. [^3]

So, execute the following steps to write the ISO in the USB flash drive: [^3]:

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
- In the _"Boot Sequence"_ section,, locate your USB flash drive. 
- Move the USB flash device using the arrows - make sure that it will be 
above "Windows Boot Manager" 

## Installing Arch Linux

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

TODO: explain how to configure rEFInd

#### In Dell BIOS

These are final actions in Dell BIOS:

- Remove your USB flash drive
- Enter into Dell BIOS
- Click on the menu _"Boot Configuration"_ 
- In the _"Boot Sequence"_ section, make sure that
  - "rEFInd Boot Manager" is the top item
  - "Windows Boot Manager" is disabled

That's it! Your laptop is dual boot ready, with Windows 11 and Arch Linux!

## Post-Install actions

TODO: add setup details (configure external drive, etc)

## References

[^1]: [Change Bitlocker PIN in Windows10](https://www.thewindowsclub.com/change-bitlocker-pin-in-windows-10))
[^2]:[Arch Wiki: Dual boot With Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "2.1.2 - Installation / Windows Before Linux / BIOS Systems / UEFI Systems" 
[^3]: [ArchWiki: USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium): see section "2.3.2 - Using manual formatting/UEFI only/In Windows" 
[^4]: [Microsoft Recovery Key Page](https://account.microsoft.com/devices/recoverykey)
[^5]: [Arch Wiki: Dual boot with Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "1.1 - Important Information/Windows UEFI vs BIOS limitations"
