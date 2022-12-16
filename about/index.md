---
layout: layouts/page.njk
title: About Me
templateClass: page
eleventyNavigation:
  key: About Me
  order: 3
---

Hey, my name is Timothée “Tim” Pillard ([@tpillard on Twitter](https://twitter.com/tpillard), [@ziir on GitHub](https://github.com/ziir)) and this is my Tech Blog.
Formerly Senior Front-End Engineer / Tech Lead at [Dashlane](https://www.dashlane.com/) / [GANDI](https://www.gandi.net/), I write stuff about Web Development, Web Performance, JavaScript, nonsense.

## I'm Looking For A Job

I'm a Front-end Engineer with 10 years of professional experience building business-critical & public-facing web applications and websites in various environments.
Passionate about the Web, I am proficient with JavaScript, Web technologies, the modern Front-end ecosystem and related tooling, with a strong focus on performance and security.

[Download my CV (pdf)](/public/Timoth%C3%A9e%20Pillard's%20CV%202023.pdf)

### Get In Touch

- [Send me a DM on Twitter](https://twitter.com/tpillard)
- [Send me an email](mailto:tim+about@timtech.blog)

## Recent Projects

In 2021/2022, I have analyzed the JavaScript bundles & Babel configuration of two important Mozilla projects, and contributed to bringing their JavaScript build to the then current state of the art, leading to modest bundle size & runtime performance improvements:

- [firefox-devtools / profiler - Babel improvements #3195](https://github.com/firefox-devtools/profiler/pull/3195)
- [mozilla / addons-frontend -  Babel improvements: reloaded #11593](https://github.com/mozilla/addons-frontend/pull/11593)

## Past Projects

### [parisweb.app (2018)](https://parisweb.app)

Demo apps for the React/Redux performance workshop [“Petits trucs pour rendre vos applications React plus performantes”](https://www.paris-web.fr/2018/ateliers/petits-trucs-pour-rendre-vos-applications-react-plus-performantes.php) performed with [Julien Wajsberg](https://twitter.com/jwajsberg) at a meetup hosted by [Gandi](https://www.gandi.net) and then at [Paris Web](https://www.paris-web.fr) in 2018.
The materials produced at this occasion (slides, code, examples) are still somewhat relevant resources to understand and improve performance of React apps, with or without Redux, in 2020.

### [soundcut/app](https://github.com/soundcut/app)

Soundcut is a [Progressive Web App](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps) which enables you to extract, share, save, download or simply listen to specific moments (a slice or cut) of a song or any audio source.
Built using JavaScript (ES2017) with [hyper/viperHTML](https://viperhtml.js.org/hyper.html) and a strong focus on performance, it leverages a SPA/SSR/SW/Shell Architecture.

### [soundcut/decode-audio-data-fast](https://github.com/soundcut/decode-audio-data-fast)

**decode-audio-data-fast** is a small JavaScript utility allowing to decode mp3 audio file data in the browser from a File/Blob into a AudioBuffer using [AudioContext.decodeAudioData()](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/decodeAudioData), but consistently faster than the native method.
