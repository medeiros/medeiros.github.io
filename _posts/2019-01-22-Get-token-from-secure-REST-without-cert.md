---
layout: post
title: "Get token from secure REST service without cert"
categories: snippets
tags: [linux]
comments: true
---
Using curl with a k.

- Table of Contents
{:toc .large-only}

## Get token from secure REST service without cert

The following snippet performs a POST request sending JSON in a SSL connection
(secure) but does no verify for security (the certificate is not required).  

## Bash snippet
{:.lead}

- Table of Contents
{:toc .large-only}

```bash
curl –k –d ‘{ "clientId": "x", "secret": "y" }’ –X POST https://<some cool domain> -H ”Content-Type: application/json”
```
