---
layout: post
title: "Github commit emojis"
categories: misc
tags: [emoji,github]
image:
  #feature: mountains.jpg
  #teaser: mountains-teaser.jpg
  #credit: Death to Stock Photo
  #creditlink: ""
---
It's nice to know how to commit using emojis.

- Table of Contents
{:toc .large-only}

Github APIv3 provide a [list of emojis](https://api.github.com/emojis)[^1] that can be used in the commit messages. The names must be used between colon punctuation marks in any point of the the comment message.

For instance, the following value (returned from the api):

```json
{
  "beer": "https://github.githubassets.com/images/icons/emoji/unicode/1f37a.png?v8"
}
```

Returns the following image:

![Beer](https://github.githubassets.com/images/icons/emoji/unicode/1f37a.png?v8)

And this image can be used in a commit message, as following:

```bash
git commit -m ":beer: adding post that explains emoji usage in github commits"
```

The result is the following:

![Emoji Commit](https://github.githubassets.com/images/icons/emoji/unicode/1f37a.png?v8){: width="32px" height="32px"} adding post that explains emoji usage in github commits

[^1]: There is also a cheat sheet that [presents all the emojis](http://www.webpagefx.com/tools/emoji-cheat-sheet/), pointed by the [GitHub Guide "Mastering Markdown" page](https://guides.github.com/features/mastering-markdown/)
