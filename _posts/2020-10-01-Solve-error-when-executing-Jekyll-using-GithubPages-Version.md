---
layout: post
title: "Solve error when executing Jekyll using GitHubPages Version"
description: >
  There are some issues when running Jekyll in different version of supported
  by GitHub Pages.
categories: misc
tags: [jekyll]
comments: true
---
There are some issues when running Jekyll in different version of supported
by Github Pages.

{:.lead}

- Table of Contents
{:toc .large-only}

## The problem

According to the [GitHub Pages' Dependencies Version](https://pages.github.com/versions/), the supported Jekyll version for GitHub Pages is 3.9.0. However, my theme was
using a 4.1 version, which is incompatible.

It was necessary to downgrade the Jekyll version. In order to do so, the
Gemfile was changed, as below:

```
gem "jekyll", "~> 3.9.0"
```

When trying to run locally, the following error start to happen:

```
Invalid US-ASCII character "\\xE2
```

## The solution: Set proper environment variables

In my particular situation (Arch Linux), it was necessary to add the following
environment variables:

```bash
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
```

In order to the locale 'en_US.UTF-8' to be propery set in these ENV variables,
it must already be set into the system. Use `locale -a` to verify. If the
locale is not on the list, it must be set:

- uncomment the line `en_US.UTF-8 UTF-8` in the `/etc/locale.gen` file
- run `locale-gen` to add it
- set the system locale in `/etc/locale.conf` to `LANG=en_US.UTF-8` as well

After that, when running:

```bash
./bundle exec jekyll serve
```

It worked properly.

## References

- [GitHub Pages' Dependencies Version](https://pages.github.com/versions/)
- [TIL: SASS & locale-gen](http://hollybecker.net/blog/2017-10-22-til-sass-locale-gen/)
- [Invalid US-ASCII character "\xE2" thread](https://github.com/csswizardry/inuit.css/issues/270#issuecomment-56056606)
