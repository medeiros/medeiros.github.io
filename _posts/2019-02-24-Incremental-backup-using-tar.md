---
layout: post
title: "Incremental Backup using TAR"
categories: snippets
tags: [linux]
comments: true
---

## What is this about?

The following shell script perform incremental backup using TAR.

It mounts a pendrive and then backs up all subdirectories under `/home/daniel`, as a compressed **.tar.gz** file, to the target pendrive (excluding some directories which do not require backup, such as cache, dropbox, etc). Finally, the pedrive is unmounted.

The `backup-snapshot.file` is the file used to control the increments, and it is managed by TAR itself.

## Snippet
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard()"/>

```bash
#!/bin/bash

SRC_PATHS='/home/daniel'
DIRS_TO_EXCLUDE='--exclude=Dropbox --exclude=tmp --exclude=.cache --exclude=.mozilla --exclude=.IdeaIC2018.2 --exclude=.m2 --exclude=.npm --exclude=.Trash-1000 --exclude=.v8* --exclude=.vagrant.d'
DEST_PATH=/home/daniel/tmp/pendrive

mount /dev/sdb1 $DEST_PATH
if grep -qs $DEST_PATH /proc/mounts; then
	tar --listed-incremental=/home/daniel/data/backup-snapshot.file $DIRS_TO_EXCLUDE -cvpzf $DEST_PATH/backup-`date +%y-%m-%d-%H-%M-%S`.tar.gz $SRC_PATHS
	umount $DEST_PATH
fi
```

The script above can be added to **crontab** to perform daily incremental backups, with the following entry (to backup everyday at 21pm, for instance): `00 21 * * * /home/daniel/code/script/backup.sh`.


## Final notes

To extract (and preserve the final state) of overall backup, for each backup file in order, use the param `--listed-incremental=/dev/null`. e.g.:

`tar --listed-incremental=/dev/null -xvf backup-19-02-24.tar.gz`
`tar --listed-incremental=/dev/null -xvf backup-19-02-25.tar.gz`

and so forth.
