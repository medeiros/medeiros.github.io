---
layout: post
title: "Binding Godaddy subdomains to Github pages"
categories: journal
tags: [github]
comments: true
---

The procedure to bind your Godaddy subdomain to Github pages is very simple and brings a great cost/benefit. In this article, I'm going to show how to bind `[user].github.io` Github Pages URL to `test.[domain-name].com` subdomain.

## Godaddy

In Godaddy administration area, follow the steps:

- Go to your domain configuration, and then to DNS Management.
- Add a new Record, as below:

| Type      | Host     | Points To | TTL      |
|:---------:|:--------:|:---------:|:--------:|
| `CNAME`   | `test`   | `[user].github.io`   | `1 hour` |


## Github

In Github, follow the single step:

- In your Github Page repository (typically `[user].github.io` repo), add a text file called `CNAME` in the root folder. The file must contains just one line, with the content `[user].github.io`

That's it!

## Why not to forward?

Godaddy has a feature that allows you to forward your domain to another, and that results in HTTP 301 and 302 status codes. This solution, however, will not really bind your domain to Github pages, but just forward it. It will not allow the domain name from Gadaddy to wrap the Github pages URL. You can mask the domain in Godaddy, but the outcome is a fixed domain that will not change when navigating between pages, so permalinks will be not possible.

Use forward only if you really want to forward (i.e: site changes its location), and not for binding.
