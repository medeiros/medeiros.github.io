---
layout: post
title: "How to enable remote access to MySQL"
description: >
  How to enable remote access to MySQL.
categories: distributedarchitecture
tags: [mysql]
comments: true
image: /assets/img/blog/mysql/mysql.png
---
> MySQL is one of the most known relational databases in the market.
In this page, its remote access perspective will be covered.
{:.lead}

- Table of Contents
{:toc}

## MySQL and Remote Access

In order to access MySQL remotely, it is necessary to allow the origin IP, and
then to grant permission in the database. It is also important to know which
port MySQL is running in the server.

## Check which port MySQL is running  

The following command does the trick:

```bash
netstat -tlpn | grep mysql
```

The usual port is `3306`.


## Allowing Remote Access

Let's assume that the remote user is `root` and the related IP is `192.168.56.1`.

First of all, it is necessary to add the client IP in the `/etc/my.cnf` file:

```bash
[mysqld]
bind-address=192.168.56.1
```

Then, it is required to execute the following command to grant access:

```bash
GRANT ALL PRIVILEGES ON . TO root@192.168.56.1 IDENTIFIED BY 'root' WITH GRANT OPTION;
```

This command inserts a record in the `user` table.

## Accessing remotely

Execute the following command:

```bash
mysql -u root -p'root' -h [remote host] -P 3306 -D [dbname]
```

## References

[Configuring database connection results in Error: Host 'xxxxxxx' is not allowed to connect to this MySQL server](https://confluence.atlassian.com/jirakb/configuring-database-connection-results-in-error-host-xxxxxxx-is-not-allowed-to-connect-to-this-mysql-server-358908249.html)
