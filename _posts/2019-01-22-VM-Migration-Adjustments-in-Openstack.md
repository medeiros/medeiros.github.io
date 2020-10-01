---
layout: post
title: "VM migration adjustments (Openstack)"
categories: snippets
tags: [virtualization, linux]
comments: true
---
A little information about virtualization.

- Table of Contents
{:toc .large-only}

## VM migration adjustments (Openstack)

The following snippet was used to configure virtual machine in Openstack
(Red Hat 6.8).

## Snippet

```bash
vim /etc/resolv.conf
	nameserver <ip>

vim /etc/hosts
  <configure>

vim /etc/sysconfig/network-scripts/ifcfg-eth0
	DEVICE=eth0
	IPADDR=<some ip>
	BOOTPROTO=dhcp
	ONBOOT=yes
	USERCTL=no

if 70-persistent-net.rules present
	rm /etc/udev/rules.d/70-persistent-net.rules
	reboot

ifconfig
service network restart
service iptables restart
netstat -tanp | grep LISTEN | grep 22
```
