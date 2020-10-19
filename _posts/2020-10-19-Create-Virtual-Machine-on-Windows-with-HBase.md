---
layout: post
title: "Create CentOS Virtual Machine on Windows with HBase"
description: >
  How to create a CentOS Virtual Machine and configure HBase on in.
categories: bigdata, linux
tags: [centos, hbase, linux, bigdata]
comments: true
---
> This page will show you how to configure a CentOS Virtual Machine using
VirtualBox and then to configure HBase.
{:.lead}

- Table of Contents
{:toc}

Linux is a _de facto_ server environment. However, developers usually have
to perform their work in Windows workstations, due to companies' policies.
Hence, we often need a way to simulate Linux environments in our Windows
boxes.

Is this post, I'll explain how to configure a Virtual Machine to do exactly
this. In addition, an HBase server will be installed, to validate Linux port
opening - and also because it's fun.

## Virtual Machine on Windows

### Download Oracle VirtualBox

First of all, [download and install Oracle VirtualBox](https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html).

## Download CentOS

You will need to download an ISO of a Linux distro. I'll adopt CentOS,
because is very easy to get it done and compatible to Red Hat (a common
distro amongst enterprises).

A CentOS 7 ISO can be found [here](http://ftp.unicamp.br/pub/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-DVD-2003.iso).

## Installation

Open VirtualBox and create a new Virtual Machine with the following properties:
- Type: Linux
- Version: Red Hat (64-bit)
- Memory size: 4GB
- Hard disk: create a virtual hard disk
  - VDI: Virtual Box DIsk Image
  - Dinamically Allocated
  - Size: 8GB

After that, start a new machine
- An ISO will be asked. Select the ISO file downloaded in previous section
- Select "Install CentOS"
- Select Language (English)
- Confirm the Warnings and click in "Begin Installation"
- Set root password
- Reboot

Your CentOS installation is done.

Now, power off the machine for further configurations in Oracle VirtualBox.


## Configure VirtualBox to use the Virtual Machine

In Virtual Box, make sure that the brand new machine is powered off.
Go to its settings, and make sure, in Network Section, that both NAT and
Host-Only Adapter are defined.

NAT interface will give you internet access,
and Host-Only Interface will give you the possibility to connect to other
machines in the same network.
{:.note}

It is also necessary to access VirtualBox Host Network Manager dialog (ctrl+H)
and make sure that DHCP is enable and properly configured. For instance, one
can configure it like below:

- Adapter
  - Configure Adapter Manually
    - IPV4 Address: 192.168.56.1
    - IPV4 Network Mask: 255.255.255.0
- DHCP Server
  - Enable Server
  - Server Address: 192.168.56.100
  - Server Mask: 255.255.255.0
  - Lower Address Bound: 192.168.56.101
  - Upper Address Bound: 192.168.56.254

Your Virtual machine will get an IP in the range of 192.168.56.101-254.

## Configuring the New Machine

The first step is to start the new machine. Then, inform your root credentials
for logging in.

### Preparing Network Interfaces

After being logged in the brand new machine, the network interfaces must be
checked:

```bash
ip addr
```

You must have two interfaces:
- Host-Only interface: with a DHCP IP
- NAT interface: with no IP

Once you know the DHCP IP, you may prefer to switch to a better terminal than
VirtualBox term, to perform Linux actions. It is highly recommended to use
[MobaXTerm](https://mobaxterm.mobatek.net/download.html) terminal.
{:.note}

You may also want to restart your virtual machine in headless mode. That way,
virtual machine will start normally but with no additional terminal window.
Then, you can connect using MobaXterm without any additional, unnecessary,
annoying terminal.
{:.note}

The NAT interface must get a class-A private network address IP (range
10.0.0.0 to 10.255.255.255). The easiest way to ensure this is to edit the
existing network file as below:
```
vi /etc/sysconfig/network-scripts/ifcfg-<interface name>
```

And change the property `ONBOOT=no` to `ONBOOT=yes`.

After that, restart the machine. After new login, execute `ip addr` again to
check that the NAT interface now has a private IP, as expected.

### Testing Internet Access

Since the NAT interface have a private IP, the internet access should be
available. You can validate it with:

```bash
ping google.com -c2
```

### Install Required Packages

With proper internet access, let's download and install some required
applications:

```bash
yum install -y vim java-1.8.0-openjdk nc net-tools lsof wget
```

`nc` package brings us `netcat`; `net-tools` brings us `lsof`. Both commands
will be important when testing Linux ports (more on next sections).

## Downloading and Starting HBase

HBase can be downloaded as following:

```bash
wget --no-check-certificate https://downloads.apache.org/hbase/2.3.2/hbase-2.3.2-bin.tar.gz
```

After that, just extract, configure and start the process:
```bash
tar xvf hbase-2.3.2-bin.tar.gz
```
```bash
vim hbase-2.3.2/conf/hbase-env.sh
# uncomment the JAVA_HOME line and update with the following:
export JAVA_HOME=/etc/alternatives/jre_1.8.0
```
```bash
./hbase-2.3.2/bin/start-hbase.sh
```
You can now use a shell to create a test table with some values:

```shell
./hbase-2.3.2/bin/hbase shell

hbase(main):001:0> list
hbase(main):001:0> create 'test', 'd'    # d = column family
hbase(main):001:0> scan 'test'
hbase(main):001:0> put 'test', 'rowkey1', 'd:a1', 'value1'
hbase(main):001:0> put 'test', 'rowkey1', 'd:a2', 'value2'
hbase(main):001:0> put 'test', 'rowkey1', 'd:a3', 'value3'
hbase(main):001:0> get 'test', 'rowkey1'
hbase(main):001:0> delete 'test', 'rowkey1', 'd:a1'
hbase(main):001:0> scan 'test'
hbase(main):001:0> scan 'exit'
```

## Opening Ports for HBase

In order to be used by external clients, port `2181` (Zookeeper embedded in
  HBase) must be accessible from the outside.

Let's enable port 2181 in the firewall:

```bash
iptables-save | grep 2181  # should find no entry

vim /etc/services
  -> add entry: zookeeper 2181/tcp
  <esc>:wq

firewall-cmd --zone=public --add-port=2181/tcp --permanent
firewall-cmd --reload

iptables-save | grep 2181  # should find a new entry
```

Let's check port status:

```bash
netstat -na | grep 2181  # should be LISTEN
lsof -i -P  | grep 2181  # should be LISTEN
```

Edit /etc/hostname and declare a host name (for instance, "bigdata")
as a single line in this file:

```bash
vim /etc/hostname
bigdata
<esc>:wq
```

Now, edit `/etc/hosts` and set the domain before localhost in definition
(as a single line in this file):

```bash
vim /etc/hosts
<your DHCP IP> big-data localhost
<esc>:wq
```

## Last configuration: client machine

Now, in the client machine (that one who will connect to HBase), add the
following line to the end of hosts file:

```property
<hbase host machine ip> big-data
```

Now, your client machine is able to connect to HBase.
