---
title: Setting up this blog (quickly) with Eleventy.
description: How I've come to finally build my tech blog, after years of frustration and a few failed attempts.
date: 2020-08-06
tags:
  - this-blog
  - tooling
  - 11ty
layout: layouts/post.njk
---
### Hey, welcome!

I've been wanting to have a tech blog of my own I could write stuff on for a while now...
And so I have tried to set one up a few times over the years, always with the same goals:
- not rely on a third party platform such as Medium and the like
- not code it all myself, because who's got time for that

All my previous attempts failed, roughly at the same stage, whether it was full-features CMSs such as Wordpress or Ghost, or static site generators such as Gatsby. Every time, something felt off: it would require too much work for me to get what I wanted, or, more accurately: it would require too much work for me **to get rid of all the stuff that I did not want**.

Here are a few examples of things I do not want:

- `node_modules` bloat
- JavaScript / markup bloat
- client-side rendering
- complicated webserver runtime, database engine
- overly complex APIs & tooling
- etc...

Paradoxically (or, is it?), it is now that I have much more time on my hands that I have found the way forward.

### Eleventy (11ty), a simpler static site generator

I have been hearing about [Eleventy (11ty), a simpler static site generator](https://11ty.dev) for quite some time now and decided to give it a spin.
Immediately upon opening Eleventy Documentation, I stumbled upon the [Starter Projects](https://www.11ty.dev/docs/starter/), and more specifically, the [eleventy-base-blog starter project](https://github.com/11ty/eleventy-base-blog). Skipping further reading of the documentation, let's take a look at the steps I followed that led me to having this blog ready for ~production within a few hours.

### This is a not a tutorial

- [Register a domain name](https://www.gandi.net/domain) first, as any good procrastinator with a side-project idea would
- Clone [eleventy-base-blog starter project](https://github.com/11ty/eleventy-base-blog):

```
$ cd Repositories
$ git clone git@github.com:11ty/eleventy-base-blog.git timtech.blog
```

- Start the development server:

```
$ cd timtech.blog
$ npm install
$ npm start
```

- Fiddle with the Nunjucks templates, the CSS file
- Add a webfont: [Fira Sans](https://mozilla.github.io/Fira/)
  - downloaded as `.tff` from [Google Fonts](https://fonts.google.com/specimen/Fira+Sans)
  - exported to `.woff2` (+ subsetting, etc...) using [Font Squirrel](https://www.fontsquirrel.com/tools/webfont-generator)
- Fiddle some more `.eleventy.js`, `package.json` files and other stuff
- Write this post in Markdown
- Create a GitHub repository
- Set git remote, commit, push:

```
$ rm -rf .git
$ git init
$ git remote add git@github.com:ziir/timtech.blog.git
$ git add .
$ git commit -m "initial commit"
$ git push origin master
```

- And... voila! (conveniently leaving out the deploy & hosting part)

![timtech.blog scoring 100s on Lighthouse](/img/timtech-blog-lighthouse-100-eleventy.png)

### Now, is this blog perfect?

Absolutely not, it's just the bare minimum for me to push my content out there.

### What about Eleventy, is it good?

The fact that I was able to hack together a decent blog out of it without being familiar with it or reading any documentation puts it in a good position I believe.
There's one thing in particular that I appreciate of `Eleventy`: I mentionned earlier that the other options I had tried previously didn't satisfy me and, in some way, would be forcing too much work on me to remove all of the stuff that I did not want, and so far, `Eleventy` hasn't given me any of those.

### What next?

Now begins the journey of this blog, built with `Eleventy`. There's a bunch of things that this blog is currently missing and that I am fairly confident I will be able to implement with ease using `Eleventy`:
- a search feature, even as a static site
- pagination
- some sort of a comments system
- an original webdesign
- [Tailwind CSS](https://tailwindcss.com/), because _I love utility-first_
- offline-first PWA
- more stuff I'll make up along the way

### What if Eleventy is not up to the task?

If, for some reason, I am not satisfied by `Eleventy` for the purposes of this blog, moving away from it to some other tool should be pretty easy.
Let's take a look of the things I would want to keep in such a scenario:
- `/posts/:slug`, `/tags/:tag`, `/about`, etc... URLs & associated content
- JSON / Atom feeds
- a sitemap
- and... that's it!

### Wrapping things up

While it's usually best to weigh the pros and cons of available options before committing to either of them, sometimes it really is fine to just try it, move forward and adapt along the way, even if the first results are far from perfection.

_Note: please do not build your startup's app the same way I built this blog._
