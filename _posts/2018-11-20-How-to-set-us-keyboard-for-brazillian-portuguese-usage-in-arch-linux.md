---
layout: post
title: "How to set US keyboard for brazillian portuguese usage in Arch Linux"
categories: linux
tags: [archlinux]
comments: true
---
Configuring keyboard for pt-br.

- Table of Contents
{:toc .large-only}

## US Keyboards and other languages

It is a very common practice to adopt US keyboards worldwide, but the layout of
these keyboards are far from ideal for many languages, which have characters sets
different from the english language.
The goal of this article is to explain how to configure an US keyboard to support
brazillian portuguese keymap in Arch Linux.

## Two main ways

There are two different sets of configurations: for console and for XOrg (X Window System).
Let's cover both.

### Console configuration

The following command must be executed in order to change keyboard layout in console:
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard(0)"/>

```bash
$ localectl --no-convert set-keymap br-latin1-us
```
This will add the following entry in `/etc/vconsole.conf`:

```bash
KEYMAP=br-latin1-us
```

This is enough to allow brazillian keymap, including cedilla, in console.
But another different configuration is required for XOrg, since it does not inherit
the console configuration.

### XOrg configuration

The first step is to set the keyboard. The following command get the job done
in a persistent way:

```bash
$ localectl --no-convert set-x11-keymap us_intl
```
This will add the following entry in `/etc/X11/xorg.conf.d/00-keyboard.conf`:

```bash
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "us_intl"
EndSection
```

It will work, with one (important) exception: for `dead_acute + C key` combination,
instead of cedilla, the character U0106 (ć) is presented.

#### Problem: Locale
The problem is related to different locale. In order to change this, it is
necessary to change the default US locale (en_US.UTF-8) to brazillian portuguese
locale.
The following command does the trick:

```bash
$ localectl set-locale LANG=pt_BR.UTF8
```

This will add the following in `/etc/locale.conf`:

```bash
LANG=pt_BR.UTF-8
```

In Arch, `pt_BR.UTF8` layout details can be found in `/usr/share/X11/locale/pt_BR.UTF-8/Compose` file.
The cedilla behavior in each locale can be perceived comparing both locale files:

```bash
$ cat /usr/share/X11/locale/en_US.UTF-8/Compose | grep ccedil -i

<dead_cedilla> <C>                    : "Ç"   Ccedilla # LATIN CAPITAL LETTER C WITH CEDILLA
<Multi_key> <comma> <C>               : "Ç"   Ccedilla # LATIN CAPITAL LETTER C WITH CEDILLA
<Multi_key> <C> <comma>               : "Ç"   Ccedilla # LATIN CAPITAL LETTER C WITH CEDILLA
<Multi_key> <cedilla> <C>             : "Ç"   Ccedilla # LATIN CAPITAL LETTER C WITH CEDILLA
<dead_cedilla> <c>                    : "ç"   ccedilla # LATIN SMALL LETTER C WITH CEDILLA
<Multi_key> <comma> <c>               : "ç"   ccedilla # LATIN SMALL LETTER C WITH CEDILLA
<Multi_key> <c> <comma>               : "ç"   ccedilla # LATIN SMALL LETTER C WITH CEDILLA
<Multi_key> <cedilla> <c>             : "ç"   ccedilla # LATIN SMALL LETTER C WITH CEDILLA
<dead_acute> <Ccedilla>               : "Ḉ"   U1E08 # LATIN CAPITAL LETTER C WITH CEDILLA AND ACUTE
<Multi_key> <acute> <Ccedilla>        : "Ḉ"   U1E08 # LATIN CAPITAL LETTER C WITH CEDILLA AND ACUTE
<Multi_key> <apostrophe> <Ccedilla>   : "Ḉ"   U1E08 # LATIN CAPITAL LETTER C WITH CEDILLA AND ACUTE
<dead_acute> <ccedilla>               : "ḉ"   U1E09 # LATIN SMALL LETTER C WITH CEDILLA AND ACUTE
<Multi_key> <acute> <ccedilla>        : "ḉ"   U1E09 # LATIN SMALL LETTER C WITH CEDILLA AND ACUTE
<Multi_key> <apostrophe> <ccedilla>   : "ḉ"   U1E09 # LATIN SMALL LETTER C WITH CEDILLA AND ACUTE
<dead_currency> <Ccedilla>            : "₵"   U20B5               # CEDI SIGN
<dead_currency> <ccedilla>            : "₵"   U20B5               # CEDI SIGN
```

```bash
$ cat /usr/share/X11/locale/pt_BR.UTF-8/Compose | grep ccedil -i

<dead_acute> <C> 			: "Ç" Ccedilla	# LATIN CAPITAL LETTER C WITH CEDILLA
<dead_acute> <c> 			: "ç" ccedilla	# LATIN SMALL LETTER C WITH CEDILLA
```

## Final notes

It is also possible to configure keyboard layout on the fly, without persisting
configuration, using `setxkbmap -layout us -variant intl`;
however, the cedilla character will also only work with the brazillian-portuguese locale previously set.

## References

[Xorg/Keyboard configuration](https://wiki.archlinux.org/index.php/Xorg/Keyboard_configuration)

[Locale](https://wiki.archlinux.org/index.php/Locale)

[Xorg](https://wiki.archlinux.org/index.php/Xorg)
