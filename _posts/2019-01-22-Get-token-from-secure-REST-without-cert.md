---
layout: post
title: "Get token from secure REST service without cert"
categories: snippets
tags: [linux]
comments: true
---

## What is this about?

The following snippet performs a POST request sending JSON in a SSL connection
(secure) but does no verify for security (the certificate is not required).  

## Bash snippet
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard()"/>

```bash
curl –k –d ‘{ "clientId": "x", "secret": "y" }’ –X POST https://<some cool domain> -H ”Content-Type: application/json”
```
