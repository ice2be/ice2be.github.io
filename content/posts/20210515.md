+++
author = "Matthew Zegar"
title = "Hugo, Another Static Site Generator"
date = "2021-05-15"
description = "Why use a static site generators?"
tags = [
    "static",
    "site",
    "generators"
]
+++

I always took pride in creating my personal websites basically from scratch. But, what's the point of jumping through hoops
when other developers already have for me. At this current time, static site generators are the new hip thing, so let's
try one out.

[Hugo](https://gohugo.io/) is a static site generator that basically wraps a bunch of markdown files. This got me excited
as previously updating my website was a pain. I had to go through this typical process...

1. Open up webstorm (and wait for it to load...)
2. Edit .html and .ts files
3. Use a special tool to serve my React page statically to GitHub pages

What I wanted is to be able to open a Markdown file in GitHub's web interface, change a few lines, and have those changes
automatically populate to my GitHub pages website. This is what Hugo provides (alongside many other static web page generators).

The content in this blog post you are currently reading was done in [Markdown](https://en.wikipedia.org/wiki/Markdown). 
It supports extremely easy ways to format text. For example...

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Example HTML5 Document</title>
</head>
<body>
  <p>Pretty neat stuff.</p>
</body>
</html>
```

No more jumping through hoops.
