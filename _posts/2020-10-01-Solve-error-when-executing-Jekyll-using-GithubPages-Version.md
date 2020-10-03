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

## First Problem: cannot run locally

According to the [GitHub Pages' Dependencies Version](https://pages.github.com/versions/), the supported Jekyll version for GitHub Pages is 3.9.0. However, the Hydejack PRO
Theme was using a 4.1 version, which is incompatible with GitHub.

It was necessary to downgrade the Jekyll version. In order to do so, the
Gemfile was changed, as below:

```
gem "jekyll", "~> 3.9.0"
```

When trying to run locally, the following error started to happen:

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

## Another issue: cannot run remotely

It's now working perfectly locally, but for some reason it
was not working on GitHub. The strange behavior was: the pages were blank.
No errors, response HTTP Status Code=200 - but blank.

I added TravisCI to be able to see the internal erros - and no errors were
found after compile at this point, but the pages were still not rendered
properly.
If I hit `/blog/`, the HTTP Status code was 200,
but the page was blank. If I hit the main page (`/`), it only showed the text
that I set in `index.md` file, but no other information was rendered.

## The solution: stop using themes on GitHub Pages

I suspected that the problem was related to the local theme `#jekyll-theme-hydejack`
directory (because I changed to the "remote_theme: hydecorp/hydejack@v9" in
`_config.yml` file and it worked - however, using a non-PRO remote theme).
I believed that, for some reason, Github was denying the use of `#jekyll-theme-hydejack` as a gem.

After getting some help from [LazyRen](https://github.com/LazyRen) on the
[issue 170](https://github.com/hydecorp/hydejack/issues/170), I was able to
make it work.

In the end, the `theme:` attribute seems not to work in GitHub pages, and,
instead of using it, it was removed and the necessary content from theme
directory was them moved to the repository root directory.

In summary, I had to perform the following actions for `Hydejack PRO 9.0.4` to work on GitHub Pages:

- `_config.yaml` file:
  - remove the attributes `#theme: jekyll-theme-hydejack` and
  `#remote_theme: hydecorp/hydejack@v9` (ssems not to work with GitHub Pages)
- `Gemfile` file:
   - change the jekyll version from `~> 4.1` to `~> 3.9.0` (to be compatible with https://pages.github.com/versions/)
   - remove the line `gem "jekyll-theme-hydejack", path: "./#jekyll-theme-hydejack"` (it is unnecessary, since the theme is no longer used as a referenced gem file)
- `#jekyll-theme-hydejack` directory:
  - move the following subdirectories to the root of the repository:
    -  _layouts
    - _includes
    - _sass
    - assets
  - `jekyll-theme-hydejack.gemspec` file
    - change `rake` version to ~> `12.3.3` (GitHub pointed previous version as security vulnerability)

## References

- [GitHub Pages' Dependencies Version](https://pages.github.com/versions/)
- [TIL: SASS & locale-gen](http://hollybecker.net/blog/2017-10-22-til-sass-locale-gen/)
- [Invalid US-ASCII character "\xE2" thread](https://github.com/csswizardry/inuit.css/issues/270#issuecomment-56056606)
- [Hydejack GitHub Issue 170](https://github.com/hydecorp/hydejack/issues/170)
