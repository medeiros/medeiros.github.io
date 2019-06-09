---
layout: post
title: "Scan Documents with Epson XP-241 and Archlinux"
categories: snippets
tags: [linux]
comments: true
---

## What is this about?

The following shell script was applied to scan documents using Epson XP-241 multifunctional in Arch Linux operational system.

If no param is informed, the *usage* screen appears. The user must inform the name of output file without extension (*mandatory*) and the path to save the output file (*optional*).  

The DEVICE variable was set based on the result of `scanimage -L` command, and varies for each environment. More information about scanimage and SANE can be found at <a href="https://wiki.archlinux.org/index.php/SANE" target="_blank">Arch Linux Wiki about SANE</a>.

The script requires **scanimage** *(to convert the digital document to image)* and **convert** *(to convert the output image to a PDF file)*.

## Snippet
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard()"/>

```bash
if [[ $# -eq 0 ]]; then
	echo
	echo 'Usage: scan-to-pdf <file name (omit extension)> <path (optional)>'
	echo ' file name: the PDF file name to be generated.'
	echo ' path: the path to save pdf. If not informed, it is set to current directory.'
	echo
	exit 0
fi

A4_X=210
A4_Y=297
#DEVICE="utsushi:esci:usb:/sys/devices/pci0000:00/0000:00:14.0/usb1/1-3/1-3.1/1-3.1:1.0"
DEVICE=$(scanimage -L | grep -i 'xp-240_series' | cut -d"'" -f1 | cut -d'`' -f2)
USER_PATH="$2"
DEFAULT_PATH='.'
PDF_PATH="${USER_PATH:-$DEFAULT_PATH}"/$1.pdf

scanimage -x $A4_X -y $A4_Y --device $DEVICE --format=png | convert png:- $PDF_PATH
```
