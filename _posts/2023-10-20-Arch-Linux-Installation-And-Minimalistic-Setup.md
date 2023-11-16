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

I already have a external drive (Linux ext4) and a external monitor, that will 
be referred in this article as well.

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
- **Different workmodes**: in the _home_ workmode, my laptop will be connected 
to an HDMI external monitor and I will be using a external `us-intl` keyboard; 
in the _remote_ workmode, I will be using laptop directly, without external 
monitor or keyboard. My laptop keyboard layout is `br-abnt2`. 
- **Minimalism is the key**: my minimalistic drive is to create a simple and 
productive environment, so I will choose my setup carefully, considering 
that new programs and configurations:
  - should provide me with a sense of aesthetics;
  - should have a clear purpose;
  - should be funcional;
  - should improve my productivity;
  - should be logical. 

These minimalist items are in tune with one definition of minimalism that I 
like: _**"all the things that I have must be clearly funcional or give me 
joy"**_.

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
- _section 3.4. Localization_: 
  - in `/etc/vconsole.conf`, for Brazil, set `KEYMAP=br-abnt2`;
  - in `/etc/locale.conf`, change the `LANG=en_US.UTF-8` entry to 
  `LANG=pt_BR.UTF-8` (so the us-intl keyboard maps the `<dead_cedilla> <C>` 
  combination to 'ç' instead of the character U0106 ('ć') [^19] . 

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

### X Window System 

In order to have a productive environment, it is mandatory to install a 
X Window System, that contains a Display Server and a Window Manager. 
Those are required for any GUI applications.

The first step is to install a display server. My choice is for 
[Xorg](https://wiki.archlinux.org/title/Xorg).

#### Display Server: Xorg

[Xorg](https://wiki.archlinux.org/title/Xorg) the most traditional 
display server in Linux. 

To install Xorg, run as follows:

```
sudo pacman -S xorg xorg-xinit
```

> Explanation: xorg is a package group—it contains the Xorg display server 
and a collection of other useful X-related packages; xorg-init is used to 
start the X Window System [^11]

#### Window Manager: i3wm

My choice for a window manager is [i3](https://i3wm.org/). It is a tiling
window manager, apropriate for my minimalistic approach.

```
sudo pacman -S i3-wm i3status i3blocks
```

> Explanation: i3-wm is the i3 window manager; i3status and i3blocks 
provide the i3 status bar. [^11]


#### X Launcher: startx

Once X is installed, now the next step is to configure the `~/.xinitrc` file. 
This file is the configuration file for the `startx` program (which is the 
one who actually starts the X Window System and launches the window 
manager).

So, create a `.xinitrc` with the following content:

```
vim ~/.xinitrc
```

```bash
#!/bin/bash

setxkbmap -model pc105 -layout br -variant abnt2

exec i3
```

The `setxkbmap` line just define the pt_BR keyboard of my laptop (to be 
used in the X system). After that, the `exec i3` command will execute 
the i3 window manager.

Finally, let's add `startx` to `.bash_profile` file, so that it runs at every
login:


```
vim ~/.bash_profile
```

```bash
#
# ~/.bash_profile
#

if [[ -z $DISPLAY ]]; then
  startx
fi

[[ -f ~/.bashrc ]] && . ~/.bashrc
```

This way, at each login, if the login is performed by an user (if there is 
a display), then executes `startx` to load the i3 window manager.

Now you can run `startx` at command line, or login again.

A very basic i3 GUI interface will appear. 

### Git

Just install git. It will be required many, many times.

```
sudo pacman -S git
```

### Terminal and shell

The most important thing now is to setup a shell and terminal.
My choices in that regard are:

- **Fish**: for Linux shell
- **Tmux**: for terminal multiplexer
- **Alacritty**: for terminal manager

Those three programs must be installed and properly configured to work 
together. 

#### Linux Shell: Fish

Fish is a linux shell with more practical features than xterm.

```
sudo pacman -S fish
```

Let's extend the basic feature here by adding a simpler approach to the 
common `ls -l` command. 
Instead of create an alias, we will create a fish function to `ll`:

```
$ vim ~/.config/fish/functions/ll.fish
```

```bash
function ll
    ls -lah $argv
end
```

Now, when you execute `ll` in terminal, the `ls -lah` will run, 
instead - like an alias, but different.

#### Terminal Multiplexer: tmux

Tmux is a terminal multiplexer. It is a program that can open multiple 
panes in the terminal interface. In fact, there are more related concepts, 
such as Sessions, Windows and Panes, that can be better understand 
[in this article](https://haseebmajid.dev/posts/2023-05-02-my-development-workflow-with-alacritty-fish-tmux-nvim/).

In order to install tmux, execute as below:

```
sudo pacman -S tmux
```

Regarding configuration, I want to change some default bind keys to make 
easier for me to create vertical and horizontal panes, and I want to use 
fish shell every time I use tmux. I also want to configure a nice visual 
interface. So, my config file is as below:

```
mkdir -p ~/.config/tmux

vim ~/.config/tmux/tmux.conf
```

```bash
# Remap prefix keys
unbind C-b
set-option -g prefix M-a
bind-key M-a send-prefix

# Terminal quality
set -g history-limit 100000
set-option -g base-index 1
set-option -g renumber-windows on
set-option -g automatic-rename on

# Screen spliting
unbind '"'
unbind %
#bind v split-window -v
#bind h split-window -h

# Joining Windows
bind-key j command-prompt -p "join pane from: "  "join-pane -s '%%'"
bind-key s command-prompt -p "send pane to: "  "join-pane -t '%%'"

# Awitch panes using Alt-arrow without prefix
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Window navigation
bind -n M-0 select-window -t :0
bind -n M-1 select-window -t :1
bind -n M-2 select-window -t :2
bind -n M-3 select-window -t :3
bind -n M-4 select-window -t :4
bind -n M-5 select-window -t :5
bind -n M-6 select-window -t :6
bind -n M-7 select-window -t :7
bind -n M-8 select-window -t :8
bind -n M-9 select-window -t :9

# Syntronize panes - send command to all panes
bind-key g set-window-option synchronize-panes\; display-message "synchronize-panes is now #{?pane_synchronized,on,off}"

# Resize panes with VIM nav keys
bind -n M-S-Left resize-pane -L
bind -n M-S-Down resize-pane -D
bind -n M-S-Up resize-pane -U
bind -n M-S-Right resize-pane -R

# Move panes inside the same windows
unbind-key '{'
unbind-key '}'
bind-key S-Up swap-pane -U
bind-key S-Down swap-pane -D

# Set the layout
bind-key l select-layout main-vertical
bind-key V select-layout even-vertical
bind-key H select-layout even-horizontal

### Status bar customization
set-option -g status-style bg=color234,fg=color244
set-option -g status-left 'windows: '
set-option -g status-right 'session: [#{session_name}]'
set-option -g window-status-format '#{window_index}-#{window_name}'
set-option -g window-status-current-format '#[bold, fg=white]#{window_index}-#{window_name}'

# set vi copy commands
#setw -g mode-keys vi

# Changes in terminal borders
set -g pane-active-border-style fg="cyan"

# set  right status bar lenght to 200
# set-option -g status-right-length 200

### Misc

# Reload config file (change file location to your the tmux.conf you want to use)
bind r source-file ~/.config/tmux/tmux.conf \; display-message " Config updated successfully!"

# set mouse on 
unbind m
bind-key m set-option mouse \; display-message "mouse is now  #{?mouse,on,off}"

# Refer: # https://gist.github.com/mzmonsour/8791835

set-option -g default-shell /bin/fish
#bind-key l run-shell "develop"
#bind-key / command-prompt "split-window 'exec man %%'"

# change from ctrl-b to ctrl-a
#set-option -g prefix C-a
#unbind-key C-b
#bind-key C-a send-prefix

#change splits to be more natural
#unbind %
#unbind '"'
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
```

#### Terminal Emulator: Alacritty

Alacritty is a very useful and efficient terminal emulator.
It will adopt fish (as shell) along with tmux (terminal multiplex) in a 
unified solution.

Execute the following to install Alacritty:

```
sudo pacman -S alacritty
```

Then, download some nice visual themes [^12] :

```
mkdir -p ~/.config/alacritty/themes
git clone https://github.com/alacritty/alacritty-theme ~/.config/alacritty/themes
```

Once the program and themes are in place, configure Alacritty as below:

```
vim ~/.config/alacritty/alacritty.yml
```

```bash
# from https://github.com/alacritty/alacritty-theme/tree/4cb179606c3dfc7501b32b6f011f9549cee949d3
import:
  - ~/.config/alacritty/themes/themes/darcula.yaml
...
shell:
  program: /usr/bin/fish
  args:
    - -l
    - -c
    - "tmux attach || tmux"
```

So, now Alacritty is configure to use `fish` shell with `tmux`, and the `Darcula` 
theme to make things prettier. 

##### i3 bind key for Alacritty

The last step is to configure i3 to run Alacritty. Edit i3 config file and 
replace the existing `i3-sensible-terminal` by `alacritty`, as below:

```
vim ~/.config/i3/config
```

```bash
# start a terminal
#bindsym $mod+Return exec i3-sensible-terminal
bindsym $mod+Return exec /usr/bin/alacritty
```

Press `$mod+Enter` to open Alacritty window.

Now, the terminal configuration is complete.

### Wireless interaction: iwd

[Iwd](https://wiki.archlinux.org/title/Iwd) is a minimal wireless daemon. 
I like to use the iwctl interactive prompt to handle wireless connections.

Install it as below:

```
sudo pacman -S iwd
```

After that, you can execute general commands, such as:

```
$ sudo iwctl help
$ sudo iwctl station list
$ sudo iwctl station wlan0 show
$ sudo iwctl station wlan0 connect isengard pwdIseng
```

### Run Dialog: rofi

[Rofi](https://wiki.archlinux.org/title/Rofi) is a nice run dialog.

Install it as below:

```
sudo pacman -S rofi
```

After that, edit i3 config file to bind it to $mod+d:

```
vim ~/.config/i3/config
```

```bash
# start dmenu (a program launcher)
#bindsym $mod+d exec --no-startup-id dmenu_run
bindsym $mod+d exec --no-startup-id rofi -show drun
```

### Browser: qutebrowser

[Qutebrowser](https://wiki.archlinux.org/title/Qutebrowser) is a 
keyboard-focused web browser. It supports the vim keymap, which is 
a very nice feature in terms of productivity.

Install it as below:

```
sudo pacman -S qutebrowser
```

After that, just use `rofi` run dialog to start it.

### Backlight: Linux configuration 

to do: finalize it - https://www.ejmastnak.com/tutorials/arch/backlight/

### Copy and Paste: Linux configuration

to do: finalize it - https://www.ejmastnak.com/tutorials/arch/copy-paste/


### Sound: Pulseaudio and ALSA

[Pulseaudio](https://wiki.archlinux.org/title/PulseAudio) is a sound 
server that works between application and hardware device. It uses 
[ALSA](https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture), 
a set of drivers for sound.

It is necessary to install some programs:

First, install `alsamixer`:

```
$ sudo pacman -S alsa-utils
```

Then, install `pulseaudio`:

```
$ sudo pacman -S pulseaudio
```

After that, reboot. Pulseaudio will start automatically, since it is a 
[Systemd/User](https://wiki.archlinux.org/title/Systemd/User).

You can check if it's running by entering the following command:

```
$ systemctl --user status pulseaudio.service

● pulseaudio.service - Sound Service
     Loaded: loaded (/usr/lib/systemd/user/pulseaudio.service; disabled; preset: enabled)
     Active: active (running) since Sat 2023-10-21 08:31:04 -03; 9h ago
TriggeredBy: ● pulseaudio.socket
   Main PID: 4518 (pulseaudio)
      Tasks: 12 (limit: 18864)
     Memory: 12.6M
        CPU: 67ms
     CGroup: /user.slice/user-1000.slice/user@1000.service/session.slice/pulseaudio.service
             ├─4518 /usr/bin/pulseaudio --daemonize=no --log-target=journal
             └─4544 /usr/lib/pulse/gsettings-helper

Oct 21 08:31:04 ataraxia systemd[4434]: Starting Sound Service...
Oct 21 08:31:04 ataraxia pulseaudio[4518]: stat('/etc/pulse/default.pa.d'): No such file or directory
Oct 21 08:31:04 ataraxia systemd[4434]: Started Sound Service.
```

And you can also test the sound by starting some audio (access Youtube 
or similar) and type `alsamixer` in the terminal to enable/test sound.

### Image conversion: imagick

The urge to convert images types is most common that it looks.

The [ImageMagick](https://wiki.archlinux.org/title/ImageMagick) program is 
valuable in that sense. It can convert a lot of image types.


```
$ sudo pacman -S imagemagick
```

After that, to convert a single file from different types is as simple as 
executing the following command:

```
$ convert image.png image.jpg
```

Very handy when you want to use some image but the format is not 
supported.

But you can also convert all files in a directory from one type to 
another (the originals will be retained):

```
mogrify -format png *.jpg
```

### Background image: feh

[Feh](https://wiki.archlinux.org/title/Feh) is a highly configurable program 
that allows you, among other things, to set your wallpaper.

In order to install it, execute the command as following:

```
$ sudo pacman -S feh
```

You can set a single wallpaper or define a randomize mode, to random select
one between several files. In the next sections of this page, `feh` will be 
used to define a random background.

### Control bar: polybar

[Polybar](https://wiki.archlinux.org/title/Polybar) is a very handy and 
essential tool. It allow us to get real-time information about the system 
(CPU, RAM, time, disk usage, battery, etc) and also to work in conjunction 
with i3 to define bind keys to control backlight and audio volume levels.

#### Installation

As expected:

```
$ sudo pacman -S polybar
```

#### Configuration

Execute the following command to create a config file:

```
$ cp /etc/polybar/config.ini ~/.config/polybar
```

Then edit this file and change the existing attributes as below:

```
$ vim ~/.config/polybar/config.ini
```

```bash
modules-left = xworkspaces date xwindow
modules-right = cpu memory filesystem pulseaudio xkeyboard wlan backlight battery
```
> These modules will be configured one by one, in the next sections. 

Also, create a `launch.sh` file to call polybar:

```
$ vim ~/.config/polybar/launch.sh
```

```bash
#!/usr/bin/env bash

polybar-msg cmd quit

echo "---" | tee -a /tmp/polybar.log #/tmp/polybar2.log
polybar example 2>&1 | tee -a /tmp/polybar.log & disown
#polybar bar2 2>&1 | tee -a /tmp/polybar2.log & disown

echo "Bars launched..."
```

> In this config, only one bar will be used. However, The commented code makes 
clear that multiple bars could be configured, if desired.

#### date module

This module is for date/time presentation.

Configure it as follows:

```
$ vim ~/.config/polybar/config.ini
```
```bash
[module/date]
type = internal/date
interval = 1

date = %H:%M
date-alt = %d/%m/%Y %H:%M:%S

label = %date%
label-foreground = ${colors.primary}
```

#### pulseaudio module

This module is for volume control.

Configure it as follows:

```
$ vim ~/.config/polybar/config.ini
```
```bash
[module/pulseaudio]
type = internal/pulseaudio
use-ui-max = true
interval = 5

; Available tags:
;   <label-volume> (default)
;   <ramp-volume>
;   <bar-volume>
format-volume = <label-volume>
format-volume-prefix = "VOL "
format-volume-prefix-foreground = ${colors.primary}

; Available tags:
;   <label-muted> (default)
;   <ramp-volume>
;   <bar-volume>
;format-muted = <label-muted>

; Available tokens:
;   %percentage% (default)
;   %decibels%
;label-volume = %percentage%%

; Available tokens:
;   %percentage% (default)
;   %decibels%
label-volume = %percentage%%
label-muted = muted
label-muted-foreground = ${colors.disabled}

; Only applies if <ramp-volume> is used
ramp-volume-0 = 0
ramp-volume-1 = 1
ramp-volume-2 = 2

; Right and Middle click
click-right = pavucontrol
; click-middle = 
```

#### backlight module

This module is for monitor backlight.

Configure it as follows:

```
$ vim ~/.config/polybar/config.ini
```
```bash
[module/backlight]
type = internal/backlight
format-prefix = "BCL "
format-prefix-foreground = ${colors.primary}

; Use the following command to list available cards:
; $ ls -1 /sys/class/backlight/
card = intel_backlight

; Use the `/sys/class/backlight/.../actual-brightness` file
; rather than the regular `brightness` file.
; Defaults to true unless the specified card is an amdgpu backlight.
; New in version 3.6.0
use-actual-brightness = true

; Enable changing the backlight with the scroll wheel
; NOTE: This may require additional configuration on some systems. Polybar will
; write to `/sys/class/backlight/${self.card}/brightness` which requires polybar
; to have write access to that file.
; DO NOT RUN POLYBAR AS ROOT. 
; The recommended way is to add the user to the
; `video` group and give that group write-privileges for the `brightness` file.
; See the ArchWiki for more information:
; https://wiki.archlinux.org/index.php/Backlight#ACPI
; Default: false
enable-scroll = true

; Available tags:
;   <label> (default)
;   <ramp>
;   <bar>
format = <label>

; Available tokens:
;   %percentage% (default)
label = %percentage%%

; Only applies if <ramp> is used
ramp-0 = 🌕
ramp-1 = 🌔
ramp-2 = 🌓
ramp-3 = 🌒
ramp-4 = 🌑

; Only applies if <bar> is used
bar-width = 10
bar-indicator = |
bar-fill = ─
bar-empty = ─
```

#### battery module

This module is for battery.

Configure it as follows:

```
$ vim ~/.config/polybar/config.ini
```
```bash
[module/battery]
type = internal/battery

; This is useful in case the battery never reports 100% charge
; Default: 100
full-at = 99

; format-low once this charge percentage is reached
; Default: 10
; New in version 3.6.0
low-at = 5

; Use the following command to list batteries and adapters:
; $ ls -1 /sys/class/power_supply/
battery = BAT0
adapter = ADP1

; If an inotify event haven't been reported in this many
; seconds, manually poll for new values.
;
; Needed as a fallback for systems that don't report events
; on sysfs/procfs.
;
; Disable polling by setting the interval to 0.
;
; Default: 5
poll-interval = 5

; see "man date" for details on how to format the time string
; NOTE: if you want to use syntax tags here you need to use %%{...}
; Default: %H:%M:%S
time-format = %H:%M

; Available tags:
;   <label-charging> (default)
;   <bar-capacity>
;   <ramp-capacity>
;   <animation-charging>
format-charging = <animation-charging> <label-charging>

; Available tags:
;   <label-discharging> (default)
;   <bar-capacity>
;   <ramp-capacity>
;   <animation-discharging>
format-discharging = <ramp-capacity> <label-discharging>

; Available tags:
;   <label-full> (default)
;   <bar-capacity>
;   <ramp-capacity>
;format-full = <ramp-capacity> <label-full>

; Format used when battery level drops to low-at
; If not defined, format-discharging is used instead.
; Available tags:
;   <label-low>
;   <animation-low>
;   <bar-capacity>
;   <ramp-capacity>
; New in version 3.6.0
;format-low = <label-low> <animation-low>

; Available tokens:
;   %percentage% (default) - is set to 100 if full-at is reached
;   %percentage_raw%
;   %time%
;   %consumption% (shows current charge rate in watts)
label-charging = Charging %percentage%%

; Available tokens:
;   %percentage% (default) - is set to 100 if full-at is reached
;   %percentage_raw%
;   %time%
;   %consumption% (shows current discharge rate in watts)
label-discharging = Discharging %percentage%%

; Available tokens:
;   %percentage% (default) - is set to 100 if full-at is reached
;   %percentage_raw%
label-full = Fully charged

; Available tokens:
;   %percentage% (default) - is set to 100 if full-at is reached
;   %percentage_raw%
;   %time%
;   %consumption% (shows current discharge rate in watts)
; New in version 3.6.0
label-low = BATTERY LOW

; Only applies if <ramp-capacity> is used
ramp-capacity-0 = 
ramp-capacity-1 = 
ramp-capacity-2 = 
ramp-capacity-3 = 
ramp-capacity-4 = 

; Only applies if <bar-capacity> is used
bar-capacity-width = 10

; Only applies if <animation-charging> is used
animation-charging-0 = 
animation-charging-1 = 
animation-charging-2 = 
animation-charging-3 = 
animation-charging-4 = 
; Framerate in milliseconds
animation-charging-framerate = 750

; Only applies if <animation-discharging> is used
animation-discharging-0 = 
animation-discharging-1 = 
animation-discharging-2 = 
animation-discharging-3 = 
animation-discharging-4 = 
; Framerate in milliseconds
animation-discharging-framerate = 500

; Only applies if <animation-low> is used
; New in version 3.6.0
animation-low-0 = !
animation-low-1 = 
animation-low-framerate = 200

```

#### Config i3 to launch polybar

The last step is to config i3 to launch polybar and disable the 
previous i3 status bar. Make sure the end of `config` file is as below: 

```
$ vim ~/.config/i3/config
```

```
exec_always --no-startup-id $HOME/.config/polybar/launch.sh

#make sure to remove or comment this section
#bar {
#        status_command i3status
#}
```

### Random Wallpaper: Systemd/User & scripting

This solution uses a 
[Systemd/User](https://wiki.archlinux.org/title/Systemd/User) to change the 
wallpaper from time to time. It chooses a random wallpaper and apply it.


First, it is necessary to create directories for script files:

```
$ mkdir -p ~/.config/scripts ~/.config/systemd/user
```

Then, it is necessary to create the shell script to set the wallpaper:

```
$ vim ~/.config/scripts/change-wallpaper.sh 
```
```bash
#!/bin/sh
# This file lives at `~/.config/scripts/change-wallpaper.sh`
# Sets background wallpaper of X display :0 to a random JPG file chosen from
# the directory `~/.config/wallpapers`
DISPLAY=:0 feh --no-fehbg --bg-fill --randomize ~/.config/wallpapers/1920x1080/*.jpg
```

After that, the service file should be created: 

```
$ vim ~/.config/systemd/user/change-wallpaper.service 
```
```bash
# The ~/.config/systemd/user directory is the standard location for user units.

[Unit]
Description=Change the wallpaper on X display :0
Wants=change-wallpaper.timer

[Service]
# Type=oneshot is standard practice for units that start short-running shell scripts. 
Type=oneshot
# Adjust path to script as needed
ExecStart=/bin/sh /home/daniel/.config/scripts/change-wallpaper.sh

[Install]
WantedBy=graphical.target
```

And the, the timer file should be created:

```
$ vim ~/.config/systemd/user/change-wallpaper.timer 
```
```bash
[Unit]
Description=Change the wallpaper on X display :0 every few minutes
Requires=change-wallpaper.service

[Timer]
# Changes wallpaper every x minutes; adjusts as needed
# will run change-wallpaper service x minutes after the timer first activates
OnActiveSec=2s
# and then periodically every x minutes after that 
OnUnitActiveSec=10m

[Install]
WantedBy=timers.target
```

Then, add some wallpapers (JPG files) in the 
`~/.config/wallpapers/1920x1080` dir.

And finally, register the timer and the service:

```
systemctl --user enable --now ~/.config/systemd/user/change-wallpaper.service
systemctl --user enable --now ~/.config/systemd/user/change-wallpaper.timer
```

For validation, execute:

```
$ systemctl --user list-timers
$ systemctl --user status change-wallpaper.service
$ systemctl --user status change-wallpaper.timer
```

At this point, if the services are running well, one of the wallpaper 
files' in the specified directory will be randomly selected and applied
at each 10 minutes.
- wallpaper.service must be loaded but inactive
- wallpaper.timer must be loaded and active (waiting)
- list-timers must show something like the following:

```
NEXT                            LEFT LAST                          PASSED UNIT                   ACTIVATES
Sat 2023-10-21 18:07:22 -03 1min 13s Sat 2023-10-21 17:57:22 -03 8min ago change-wallpaper.timer change-wallpaper.service
```

### Lock screen: i3lock & xautolock

The i3 window manager does not have lock screen functionality by default. 
What we want here is to be able to lock/unlock the screen, and also that 
the system executes an autolock after some minutes of inactivity. 

So, the following utilities must be installed:

- i3lock: for locking funcionality
- xautolock: to execute i3lock based on time frame

> There is a very informative [Githib Gist in that regard](https://gist.github.com/rometsch/6b35524bcc123deb7cd30b293f2088d8). This section uses some of the concepts explained there. 

#### Installing lock programs

Let's install those in a basic way:

```
$ sudo pacman -S i3lock xautolock
```

#### Configuring i3lock

Now, we need to setup i3wm keybinding. The `Ctrl+Alt+l` (last one is a 
lowercase L) combination will be set to lock the screen. It would also be 
great if, instead of locking the screen in plain black color, it would be 
possible to lock using some predefined wallpaper. Let's configure this way:

```
$ vim ~/.config/i3/config
```

```bash
# keybinding to lock screen
# -c 000000 makes the screen turn black instead of the default white after the 
#   screen is locked.
# -i uses a background image when locking the screen
bindsym Control+Mod1+l exec "i3lock -c 000000 -i /home/daniel/.config/wallpapers/1920x1080/ships_sea_light_69192_1920x1080.png"
```

> It seems like the entire path of the wallpaper image should be used in the 
`-i` param. The relative path (~/.config...) doesn't seem to work properly.

At this point, if we reload i3wm and try to lock the screen, it will not be 
possible to unlock. That is because the lock application is owned by the 
`root:root`, and when you try to unlock with your user password, it will not 
be understood.

```
$ ls -la $(which i3lock)
-rwxr-xr-x 1 root root 51712 Jun 22  2022 /usr/bin/i3lock*
```

We need to unlock using your own user's password, but it doesn't feel 
right to change the ownership of the root user for the i3lock program because 
of it. After some digging, the solution I found for this issue was to change 
the group ownership of `i3lock` to the `users` group, and then to add my own 
user to that group.

```
$ sudo chown root:users /usr/bin/i3lock

$ sudo usermod -aG users $USER
```

This can be validated in some ways:

```
$ ls -la $(which i3lock)
-rwxr-xr-x 1 root users 51712 Jun 22  2022 /usr/bin/i3lock*
`
$ groups $USER
video users daniel

$ cat /etc/group | grep -i users
users:x:984:daniel
```

> The line of `/etc/group` are divided in 4 parts [^10]: 
> - group_name: It is the name of group. If you run ls -l command, you will see this name printed in the group field.
> - Password: Generally password is not used, hence it is empty/blank. It can store encrypted password. This is useful to implement privileged groups.
> - Group ID (GID): Each user must be assigned a group ID. You can see this number in your /etc/passwd file.
> - Group List: It is a list of user names of users who are members of the group. The user names, must be separated by commas.

After that, one can reload i3wm (via `$mod+Shift+r`) and try. When pressing 
`Ctrl+Alt+l`, the screen will lock with a specific backgroung image. Then type 
your user's password and press ENTER to unlock.

#### Configuring xautolock

Since the i3lock it properly set, it is time to configure xautolock.
We'll just add a new line in the i3wm config file:

```
$ vim ~/.config/i3/config
```

```bash
# auto lock the screen
# -detectsleep locks the screen properly when the computer goes to sleep,
# -time 3 sets the time after which to lock to 3 minutes and
# -locker "..." sets up i3lock as the command to lock.
exec "xautolock -detectsleep -time 3 -locker \"i3lock -c 000000 -i /home/daniel/.config/wallpapers/1920x1080/ships_sea_light_69192_1920x1080.png\""
```

The comments are self-explanatory. The `autolock` is just a wrapper to run 
`i3lock` after some period of time.

> Improvement needed: This configuration locks the screen if keyboard or mouse 
are not used after a period of time. It works great for that purpose, but 
it also locks the screen during Netflix movies (which is not cool); so this 
solution must be evolved to not to lock when streaming video is running in the 
browser (even if mouse or keyboard are not used).

### Env Beautifier: Picom

[Picom](https://wiki.archlinux.org/title/Picom) is a tool to make the 
interfaces and applications prettier.

One can install picom as usual:

```
$ sudo pacman -S picom
```

After that, the config file must be created:

```
$ mkdir ~/.config/picom

$ cp /etc/xdg/picom.conf ~/.config/picom/picom.conf
```

And then an opacity rule must be added for Alacritty terminal:

```
$ vim ~/.config/picom/picom.conf
```

```bash
opacity-rule = [
  # Makes Alacritty 95% opaque when focused...
  "95:class_g = 'Alacritty' && focused",
  # ... and 40% opaque when not focused.
  "40:class_g = 'Alacritty' && !focused",
];
```

### Image viewer: Viewnior

[Viewnior](https://siyanpanayotov.com/project/viewnior/download/) is a 
simple image viewer program that can be used to show several images in 
a directory.

One can install it in Arch as usual:

```
$ sudo pacman -S viewnior
```

And that's it. Use it at will.

### Text editor: NeoVim

to do: finalize it

### Creating a dev environment with Fish, tmux and NeoVim

Since tmux, fish anbd nvim are already installed, let's add a few functions 
to create a layout environment for development.

The functions below are fish functions that, when triggered, will execute 
tmux commands to setup a layout for development of my github personal site:

```
vim ~/.config/fish/functions/tmux_layout_mysite.fish 
```
```bash
function tmux_layout_mysite

        if test -z (findmnt /home/daniel/data|head -1)
                echo data device not mounted
                return 0
        end

        set session s0
        set session_name (tmux ls -F "#{session_name}" 2>/dev/null | grep "^$session\$" )

        set window1 mysite
        set window1path "~/data/code/opensource/medeiros.github.io"
        set window2 cmd

        if test -z $session_name
                tmux $TMUX_OPTS new-session -s s0 -d -n $window1
                #tmux new-window -a -t $session -n $window1
                tmux split-window -t $session:$window1
                tmux resize-pane -t $session:$window1.1 -y 20%
                tmux send-keys -t $session:$window1.0 "nvim" Enter
                tmux send-keys -t $session:$window1.1 "cd $window1path; git status" Enter

                tmux new-window -a -t $session -n $window2
                tmux send-keys -t $session:$window2 "cd ~; ls -lah" Enter

                tmux switch -t $session:$window1.0
        else
                tmux $TMUX_OPTS attach -t s0
                tmux switch -t $session
                return 0
        end

end
```

And now the function to kill dev tmux session:

```
vim ~/.config/fish/functions/tmux_layout_mysite_kill.fish 
```
```bash
function tmux_layout_mysite_kill
        set session s0
        set session_name (tmux ls -F "#{session_name}" 2>/dev/null | grep "^$session\$" )

        set window1 mysite
        set window2 cmd

        if ! test -z $session_name
                tmux switch -t 0
                tmux kill-window -t $session:$window2
                tmux kill-window -t $session:$window1
        else
                return 0
        end
end
```

The configuration is done. After that, when typying at fish shell: 

```
tmux_layout_mysite
```

The development environment will be created. More specifically, a tmux session 
will be created, with two windows: one window with two panes (one for nvim and 
one for git), and a second window for general fish shell.

When done, one can type at fish shell:

```
tmux_layout_mysite_kill
```

To kill the development session.

> These are simple examples, but very powerful. One can use them as reference 
to create its own specific automations.

### Premium streaming content: chromium-widevine

`qutebrowser` is not able to run premium streaming content out of the box. 
So, for that reason it is necessary to install `chromium-widevine`.

The installation is a little bit different: we need to use AUR.

- Create a directory for this build: 
```
$ mkdir -p /home/daniel/.config/AUR/chromium-widevine
```
- Access https://aur.archlinux.org/packages/chromium-widevine ;
- Download the [PKGBUILD](https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=chromium-widevine) file and save it in the previously created dir;
- Execute the following commands (at the same directory of PKGBUILD) to 
create package and then install:

```
$ makepkg -s

$ sudo pacman -U chromium-widevine-1:4.10.2710.0-1-x86_64.pkg.tar.zst

$ makepkg --clean
```

That's it. Now you can reload qutebrowser and it will be able to open 
and run content from Netflix, Spotify and etc.

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

### Set i3 startup programs in proper workspaces

In my setup, when i3 starts, I want it to open `alacritty` in my first 
workspace and `qutebrowser` in my second workspace. The "tradicional" way of 
doing this is something like below (in `~/.config/i3/config` file):

```bash 
exec --no-startup-id i3-msg 'workspace 1; exec alacritty; workspace 2; exec qutebrowser'
```
However, it won't work as expected for all programs. In this case, for both  
`alacritty` and `qutebrowser`, it seems to always start in the Workspace 1, 
even when workspace 2 was explicitly set.

[As explained here](https://i3wm.org/docs/userguide.html#assign_workspace), the 
problem is that the map might be executed when the application is not mapped to 
the window yet. So, it is recommended that _"you match on window classes (and 
instances, when appropriate) instead of window titles whenever possible because 
some applications first create their window, and then worry about setting the 
correct title."_ [^14].

Execute `xprop` at command line and then click on `alacritty` and `qutebrowser` 
windows. It will give you an output similiar to this:

```
...
WM_NAME(COMPOUND_TEXT) = "i3: i3 User’s Guide - qutebrowser"
_NET_WM_NAME(UTF8_STRING) = "i3: i3 User’s Guide - qutebrowser"
_MOTIF_WM_HINTS(_MOTIF_WM_HINTS) = 0x3, 0x3e, 0x7e, 0x0, 0x0
_NET_WM_WINDOW_TYPE(ATOM) = _NET_WM_WINDOW_TYPE_NORMAL
_XEMBED_INFO(_XEMBED_INFO) = 0x0, 0x1
WM_CLIENT_LEADER(WINDOW): window id # 0x1a00024
WM_HINTS(WM_HINTS):
                Client accepts input or input focus: True
                window id # of group leader: 0x1a00024
WM_CLIENT_MACHINE(STRING) = "ataraxia"
_NET_WM_PID(CARDINAL) = 22683
_NET_WM_SYNC_REQUEST_COUNTER(CARDINAL) = 27263011
WM_CLASS(STRING) = "qutebrowser", "qutebrowser"
WM_PROTOCOLS(ATOM): protocols  WM_DELETE_WINDOW, WM_TAKE_FOCUS, _NET_WM_PING, _NET_WM_SYNC_REQUEST
WM_NORMAL_HINTS(WM_SIZE_HINTS):
                user specified location: 2, 58
                user specified size: 1916 by 1020
                program specified minimum size: 127 by 32
                window gravity: NorthWest 
```

Note the entry `WM_CLASS(STRING)`, with two params: the first param 
corresponds to the instance and the second param corresponds to the class. 
The class is important here, because will be used below.

Now, edit the `~/.config/i3/config` file and add the following lines: 

```bash
assign [class="Alacritty"] 1
assign [class="qutebrowser"] 2
exec --no-startup-id i3-msg 'exec qutebrowser; exec alacritty; workspace 1'
```

What happens now (when i3 starts) is:

- The `assign` command will make sure that `alacritty` will always open on 
Workspace 1, and `qutebrowser` will always open on Workspace 2;
- The `exec` command will run `alacritty` (that will be properly opened in 
Workspace 1, as assigned) and qutebrowser` (that will be properly opened in 
Workspace 2, as assigned); then, will change to Workspace 1.

### Make shell fun with cowsay and fortune

This section is just for fun in your terminal. Install the following 
programs:

```
$ sudo pacman -S cowsay fortune-mod 
```

The, create a fish function `~/.config/fish/functions/fish_greeting.fish` 
to call these programs when the fish shell starts (greeting message) [^13] , 
as below:

```
vim ~/.config/fish/functions/fish_greeting.fish
```

```bash
function fish_greeting
        fortune | cowsay
end
```

Now, every time you open your terminal, a cow give you some words of 
wisdom:

```
 _________________________________________
/ The more complex the mind, the greater  \
| the need for the simplicity of play.    |
|                                         |
\ -- Kirk, "Shore Leave", stardate 3025.8 /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

daniel@ataraxia ~>
```

### Work Modes: home and remote

As previously said (in the 'System Requirements' section of this post), my 
way of working with Arch Linux demands two different "work modes": 

- **At home**: In this scenario, I connect an additional HDMI monitor and an 
external `us-intl` keyboard in my laptop, for better experience while working 
for long hours;
- **Remote**: in this mode, my laptop is not connected with keyboard or 
monitor external devices. My laptop's keyboard layout is `br-abnt2`.

So, it is important to me to have a simple and funcional system that can 
change my 'modes' depending on the situation.

To do so, my approach is as below:

#### Configure i3 workspaces

I want to configure i3 so that, if an external monitor (HDMI-1) is connected, 
it will be used for all my workspaces. Otherwise, it should use the laptop 
monitor (eDP-1) for all my workspaces. [^17][^18]. For that, add the following 
entries:

```
$ vim ~/.config/i3/config
```
```bash
# Define output monitor for workspaces
workspace $ws1 output HDMI-1 eDP-1
workspace $ws2 output HDMI-1 eDP-1
workspace $ws3 output HDMI-1 eDP-1
workspace $ws4 output HDMI-1 eDP-1
workspace $ws5 output HDMI-1 eDP-1
workspace $ws6 output HDMI-1 eDP-1
workspace $ws7 output HDMI-1 eDP-1
workspace $ws8 output HDMI-1 eDP-1
workspace $ws9 output HDMI-1 eDP-1
workspace $ws10 output eDP-1
```

> If an external monitor is connected, all workspaces will be bound to that 
monitor. But the laptop monitor still exists and it needs a workspace as well. 
That is why I configured the last workspace (10) to bind only to the laptop 
monitor (eDP-1), even if the external monitor is connected. But if I choose 
to bind workspace 10 to the HDMI-1 monitor, i3 will automatically create a 
workspace 11 and bind it to the laptop monitor (_there should be at least 
one workspace per monitor_).

#### Install autorandr

According to 
[Arch Wiki page](https://wiki.archlinux.org/title/multihead#Dynamic_display_configuration), 
`autorandr` _"allow you to automatically detect when a new display is connected 
and then change the layout based on that. This can be useful for laptop users 
who frequently work in multiple different environments that require different 
setups"_ [^16] . 

It should be installed as below:

```
$ sudo pacman -S autorandr 
```

And that's it. More information about `autorandr` can be found on the 
[Github repository](https://github.com/phillipberndt/autorandr).

#### Create fish functions for multihead and keyboard mapping

The following function define my monitor usage using `xrandr`:

```
vim ~/.config/fish/functions/multihead.fish 
```
```bash
function multihead
    xrandr --output eDP-1 --auto --output HDMI-1 --auto --right-of eDP-1
    echo 'multihead set.: output eDP-1; output HDMI-1 right-of eDP-1 \
            (see: xrandr -q)'
end
```

The following functions set keyboard mapping:

```
vim ~/.config/fish/functions/kb_map.fish 
```
```bash
function kb_map
    set kbmap $argv[1]

    if [ "$kbmap" = "us" ]
        _apply us intl
    else if [ "$kbmap" = "br" ]
        _apply br abnt2
    else
        echo "invalid param. options: us|br"
    end
end

function _apply
    set layout $argv[1]
    set variant $argv[2]

    setxkbmap -model pc105 -layout $layout -variant $variant

    echo -n "kb_map set.........: "; _print model; _print layout; \
            _print variant; echo ' '
end

function _print
    echo -n $argv[1]: (setxkbmap -print -verbose 10 | grep -i $argv[1] \
            | tr -s ' ' | cut -d ' ' -f2)
    echo -n '; '
end
```

And the following functions are responsible to apply changes based on the 
current mode. The criteria to define it is to identify if there is a connected 
HDMI monitor or not:

```
vim ~/.config/fish/functions/set_current_workmode.fish 
```
```bash
function set_current_workmode    
    _margin
    if [ (xrandr -q | grep ' connected' | wc -l) = 2 ]
        _detected_workmode 'home'
        _prefix; multihead
        _prefix; kb_map 'us'
    else
        _detected_workmode 'remote'
        _prefix; echo 'single monitor set'
        _prefix; kb_map 'br'
    end
    _margin
end

function _margin
    echo ' '
end

function _detected_workmode
    echo "[$(date +'%Y-%m-%d %H:%M:%S' )]: Detected work mode: $argv[1]"
end

function _prefix
    echo -n '   - '
end
```

After that, the functions responsible to identify the current workmode 
and to set monitor and keyboard accordingly are all set. Now it is 
necessary to add the trigger.

#### Configure function execution at fish startup

I choose to run the 'work mode identification' function everytime a new 
fish shell is started. To do so, it is just necessary to edit the 
`config.fish` file and make it call the `set_current_workmode` 
function, as below:

```
vim ~/.config/fish/config.fish
```
```bash
if status is-interactive    
    # Commands to run in interactive sessions can go here

    set_current_workmode
end
```

Now, when a new shell is open, fish will apply the work modes 
depending on if the additional monitor is connected or not.

#### Set Alacritty font size on multiple monitors

This is an additional item. I realized that, when using an external monitor, 
Alacritty font size renders really small in that monitor, and it requires me to 
always adjust manually (which is annoying).

[According to this page](https://wiki.archlinux.org/title/Alacritty#Different_font_size_on_multiple_monitors), 
_"by default, Alacritty attempts to scale fonts to the appropriate point size 
on each monitor based on the Device pixel ratio. On some setups with multiple 
displays, this behavior can result in vastly different physical sizes"_ [^15].
So, the ideal for me is to force a constant pixel ratio, instead.

For this, edit the Alacritty configuration file, as below:

```
vim ~/.config/alacritty/alacritty.yml
```
```bash 
# Any items in the `env` entry below will be added as
# environment variables. Some entries may override variables
# set by alacritty itself.
#env:
  # TERM variable
  #
  # This value is used to set the `$TERM` environment variable for
  # each instance of Alacritty. If it is not present, alacritty will
  # check the local terminfo database and use `alacritty` if it is
  # available, otherwise `xterm-256color` is used.
  #TERM: alacritty
env:
  WINIT_X11_SCALE_FACTOR: "1.3"
```

This ratio size of 1.3 suits me well. Different values can be used at will.

### Configure Dropbox process 

Dropbox is not available in Arch repository. There is an AUR alternative 
([^20]), but I prefer to follow the procedure described in the Dropbox 
website ([^21]).

So, execute the following command to download and install Dropbox 
(64-bit)([^21]):

```
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" \
    | tar xzf -
```

After that, the `~/.dropbox-dist` directory will be created. In order to 
run Dropbox process, one can simply call `. ~/.dropbox-dist/dropboxd`.
So, an authorization for access will be presented and, after confirmation, 
a `~/Dropbox` directory will be created, and the remote data will be 
properly downloaded.

To my setup, there are two key points to consider in that regard, as 
can be seen in the next sections.

#### I don't want my Dropbox data in the ~/Dropbox directory

My setup requirement states that all data must be in the external drive. 
Dropbox also does not allow us to configure the directory for data. 

The solution is to create an symbolic link, as below:

```
ln -s ~/data/Dropbox ~/Dropbox
```

So, now the Dropbox data can reside in the external drive.

> But what if the external drive is not mounted? This is the second key 
point to consider.

#### I can only run Dropbox if data dir is mounted

If is the case in which Dropbox should keep running all the time, one should 
configure Dropbox as a `systemd` service ([^22]). However, this is not my 
particular case. I want to run Dropbox process only when my `~/data` dir is 
mounted; that is because my Dropbox directory may reside inside `data` dir.

So, the following fish file was created to manage this. A `mountdata` command 
allows me to keep data mount point and Dropbox process synchronized:

```bash
# ~/.config/fish/functions/mountdata.fish

function mountdata 
	set opt $argv[1]

	if [ "$opt" = "start" ]
		_mount_data
	else if [ "$opt" = "stop" ]
		_umount_data
	else if [ "$opt" = "startdropbox" ]
		_start_dropbox_process
	else if [ "$opt" = "stopdropbox" ]
		_stop_dropbox_process
	else if  [ "$opt" = "status" ]
		_status_mount
	else
		echo "no valid param informed."
		echo "  options: "
		echo "    start........: mount data dir (if not mounted yet)"
		echo "                   and start dropbox "
		echo "    stop.........: umount data dir (if mounted "
		echo "                   and stop dropbox"
		echo "    startdropbox.: start dropbox (if data is mounted)"
		echo "    stopdropbox..: stop dropbox"
		echo "    status.......: general status"
	end
end

function _mount_data
	if test (mountpoint -q ~/data; echo $status) -eq 0
		echo "data already mounted"
		return 1
	else
		sudo mount /home/daniel/data
		_start_dropbox_process
		echo "==> ~/data" && ls ~/data
		echo "==> ~/Dropbox symlink (to ~/data/Dropbox)" && ls ~/Dropbox
		echo "==> ~/Dropbox/" && ls ~/Dropbox/
		return 0
	end
end

function _umount_data
	if test (mountpoint -q ~/data; echo $status) -eq 0
		_stop_dropbox_process
		sudo umount /home/daniel/data
		return 0
	else
		echo "data already unmounted"
		return 1
	end
end

function _start_dropbox_process
	if test -z (ps -ef | grep -i 'dropbox' | grep -v 'grep')
		nohup ~/.dropbox-dist/dropboxd &
	else
		echo "dropbox already running"
	end
end

function _stop_dropbox_process
	killall dropbox
end

function _status_mount
	if test (mountpoint -q ~/data; echo $status) -eq 0
		echo "[mounted] data is mounted"
	else 
		echo "[unmounted] data is not mounted"
	end

	if test -z (ps -ef | grep -i 'dropbox' | grep -v 'grep')
		echo "[stopped] dropbox is not running"
	else
		echo "[running] dropbox is running"
	end
end

```

Every time I plug my external drive, I run `mountdata start` command to 
mount the `~/data` directory and then start Dropbox process (that will
refer for Dropbox files inside of it). The `mountdata status` command can 
be used at any time to check the conditions of both `~/data` mount point 
and Dropbox process.

## Conclusions

This article was an attempt to document a practical procedure to install Arch 
Linux from scratch. My idea, while writing this, was to stop to reinvent the 
wheel every time I install Arch Linux, reducind the downtime - and also to make 
clear to myself, after a while, why I'm using some particular set of softwares.

I hope this can somehow be useful, in some level, for people configuring their 
own Arch Linux environment.

## References

- [Find Your Footing After Installing Arch Linux](https://www.ejmastnak.com/tutorials/arch/about/)

[^1]: [Change Bitlocker PIN in Windows10](https://www.thewindowsclub.com/change-bitlocker-pin-in-windows-10))
[^2]:[Arch Wiki: Dual boot With Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "2.1.2 - Installation / Windows Before Linux / BIOS Systems / UEFI Systems" 
[^3]: [ArchWiki: USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium): see section "2.3.2 - Using manual formatting/UEFI only/In Windows" 
[^4]: [Microsoft Recovery Key Page](https://account.microsoft.com/devices/recoverykey)
[^5]: [Arch Wiki: Dual boot with Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows), section "1.1 - Important Information/Windows UEFI vs BIOS limitations"
[^6]: [Arch boot process: Boot loader](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader)
[^7]: [Arch Linux: rEFInd - Beginning](https://wiki.archlinux.org/title/REFInd#)
[^8]: [Arch Wiki: rEFInd - Manual Installation](https://wiki.archlinux.org/title/REFInd#Manual_installation)
[^9]: [Red Hat: An introduction to the Linux /etc/fstab file](https://www.redhat.com/sysadmin/etc-fstab)
[^10]: [Understanding /etc/group File in Linux](https://www.cyberciti.biz/faq/understanding-etcgroup-file/)
[^11]: [Install and start Xorg](https://www.ejmastnak.com/tutorials/arch/startx/)
[^12]: [Github: Alacritty Themes](https://github.com/alacritty/alacritty-theme)
[^13]: [Fish - Configurable Greeting](https://fishshell.com/docs/current/interactive.html#configurable-greeting)
[^14]: [i3wm: Automatically putting clients on specific workspaces](https://i3wm.org/docs/userguide.html#assign_workspace)
[^15]: [Alacritty: different font size on multiple monitors](https://wiki.archlinux.org/title/Alacritty#Different_font_size_on_multiple_monitors)
[^16]: [Arch Multihead: dynamic display configuration](https://wiki.archlinux.org/title/multihead#Dynamic_display_configuration)
[^17]: [i3wm: Workspaces screen](https://i3wm.org/docs/userguide.html#workspace_screen)
[^18]: [i3wm: Assign workspaces on i3 to multiple displays](https://unix.stackexchange.com/questions/344329/assign-workspaces-on-i3-to-multiple-displays)
[^19]: [How to setup keyboard for Brazillian Portuguese usage in Arch Linux](https://daniel.arneam.com/blog/linux/2018-11-20-How-to-set-us-keyboard-for-brazillian-portuguese-usage-in-arch-linux/#problem-locale)
[^20]: [Dropbox AUR](https://aur.archlinux.org/packages/dropbox)
[^21]: [Dropbox: Install Linux](https://www.dropbox.com/install-linux)
[^22]: [Dropbox as a systemd service](https://www.bbkane.com/blog/dropbox-as-a-systemd-service/)
