---
layout: post
title: "Docker without sudo"
categories: journal
tags: [docker,devops]
comments: true
image:
  feature: docker.png
  teaser: docker-teaser.png
  credit:
  creditlink: ""
---

To run docker commands, it is necessary to prefix them with `sudo`. This linux command allows the current user to perform actions as `root`. Out of the box, to run docker commands you must have `root` privileges.

However, this can be a pain in the ass. Isn't practical to keep informing `sudo` every time. There is a simpler way to run docker commands, without the hassle of keep using `sudo`.

### Concept and solution

The concept is this: if your user is in a specific linux group (called "docker"), automatically you will be granted to run docker commands. Simple like that.

So, it is necessary to create this group and then add your own user to it. To create the "docker" linux group, you can use the command: <a name="footnote"><sup>1</sup></a>

```bash
sudo groupadd docker
```

And then you have to bind your user to this group, using the command: <a name="footnote"><sup>1</sup></a>

```bash
sudo usermod -a -G docker [your user]
```

You can now restart docker service. In arch linux, the command is: <a name="footnote"><sup>1</sup></a>

```bash
sudo systemctl start docker
```

And now we can test, using a very simple docker command to print a message in the screen using an Debian container. The command will work without sudo:
```bash
docker run debian echo "hello world"
```

### Footnotes

<sup>1</sup> comands here presents were tested in Arch Linux, and may not work correctly in other distros. But the general ideia is the same and Google is always for the rescue.
