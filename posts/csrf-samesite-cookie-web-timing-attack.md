---
title: CSRF When You Least Expect It.
description: Fun with Cross-Site Request Forgery (CSRF) in a creative Web Timing Attack scenario, highlighting the risks inherent to SameSite=None session cookies.
image: /img/seasurf-kaitlyn-jackson-LVbfbs0BPNY-unsplash.jpg
date: 2022-12-21
tags:
  - security
  - cookies
  - CORS
  - CSRF
layout: layouts/post.njk
---

### CSRF

<img src="/img/seasurf-kaitlyn-jackson-LVbfbs0BPNY-unsplash.jpg" alt="Black & white photo of a surfer falling off their board" width="480" />

[_Photo by Kaitlyn Jackson - Unsplash_](https://unsplash.com/photos/LVbfbs0BPNY)

[Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf) - a type of attack where the user's browser is tricked into **performing unwanted actions** on a target website, on behalf of the user - are among the **most common web application security risks**.
As such, CSRF is usually well known among web developers, and most of today's websites are sufficiently protected against its most severe forms by implementing a set of [best practices & defense mechanisms (OWASP)](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) (ie: CSRF tokens, request origin checks, not using GET requests for state changing operations, ...).

By the way, feel free to take a look at my [Cross-Origin Resource Sharing For The Web (Extension)](https://timtech.blog/posts/presentation-dashlane-web-extension-security-cross-origin-resource-sharing-CORS-OPTIONS-preflight-same-origin/) presentation for a refresher on the topic.

### SameSite Cookies?

[SameSite Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite) for session cookies are the [best defense-in-depth mitigation against CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#samesite-cookie-attribute), that's what they were created for.
In fact, in modern browsers, `SameSite=Lax` is actually the default value if the `SameSite` attribute is not used in the `Set-Cookie` HTTP response header.
With either `Strict` or `Lax` values, cookies will _not be forwarded in cross-origin subrequests_ (and even in first-party cross-origin requests in the case of `Strict`).

### SameSite=None

However, there are **still many websites** out there which rely on `SameSite=None` as they:

- already existed _before the introduction of SameSite cookies_ (from around 2016 to 2020 depending on the browser and stage of implementation), and thus already had _prior controls in place against CSRF_
- still need to include cookies in cross-site requests, probably for some _obscure tracking purposes_
- [aren't necessarily sure what they're doing](https://duckduckgo.com/?q=site%3Astackoverflow.com+"SameSite%3DNone")

But since these sites are using other anti-CSRF mechanisms & SameSite cookies are “defense in depth”, this should be fine, right? **RIGHT?**
Well, mostly, I mean, kinda ... it depends ... **ok not really.**

### Introducing Timing Attacks

A timing attack is a **side-channel attack** in which an attacker is able to gather information by **measuring the time various operations take to run**.
It is usually referred to in the context of _cryptographic algorithms_, or in the case of [transient execution CPU vulnerabilities](https://en.wikipedia.org/wiki/Transient_execution_CPU_vulnerability) such as the famous [Spectre](<https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)>) & [Meltdown](<https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability)>).

### Timing Attacks ... on the Web ?

However, per Wikipedia, “timing attacks can be applied to any algorithm that has data-dependent timing variation” and, I ask you, dear reader, what more than today's websites that, gated behind authentication & user accounts, querying a myriad of unequally optimized microservices in order to display user-specific content, are subject to ”data-dependent timing variation” ?
It is fair to assume the server handling requests to pages for user dashboards or social media timelines for example, behave _slightly differently_ whether or not the user is _logged-in or not_.
Indeed, in a lot of cases, those requests will take **significantly longer** to be answered compared to the login page that is served (or redirected to) in its place.

_Note: this assumption may not be true for some websites, nowadays even personalized content can be heavily cached, and public content may not be faster to retrieve either. However, as mentioned earlier, things are rarely equally optimized, which means that even though the main personalized content of a given websites may be very fast to retrieve, it may not be not the case for more niche content, located at another URL._

Ok that's cool and all, but what does it have to do with CSRF & SameSite Cookies?

### CSRF Based Web Timing Attacks

Even with some _safeguards_ against most common & severe forms of CSRF, we can still perform attacks that fall into the CSRF category, in fact, in an actually much simpler way than with usual forged cross-site requests.
The twist resides in actually sending out **two simple GET requests, with & without credentials**:

- one cross-origin request _omitting credentials (cookies)_, which will yield a _public version of the page_
- one cross-origin request _including credentials (cookies)_, which will yield the _private version of the page_ if the victim is logged-in

We can do this easily with JavaScript using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/fetch):

```javascript
fetch("https://target.example.com/", {
  credentials: "include", // "omit"
});
```

_Note: for optimal results on a case by case basis, one may further tune the parameters of the requests & the behavior of the browser according to the [Fetch Standard](https://fetch.spec.whatwg.org/), by using the `mode` & `redirect` options._

Now these requests are made from a given origin, let's say `https://evil.example.com`, to another origin, say `https://target.example.com`.
Of course, in real life those would be totally different domains, but in any case, they're different origins, and, for this example to be correct, we're assuming that session cookies weren't set with the `Domain` directive.

Due to [Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) (and the target origin presumably not allowing our evil origin with the `Access-Control-Allow-Origin: https://evil.example.com` response header), the responses from these requests are **opaque** and **cannot be read**, so we can't poke around in the response HTML for the actual content.
But we're fine with that, as mentioned earlier, we're interested in the timing of things, and that, we do have access to!
All we have to do is to measure the time it took between the firing of the request, and the subsequent settlement (rejection) of the Fetch promise.
Again, this is only possible because `https://target.example.com` is using `SameSite=None` session cookies.

```javascript
const start = Date.now();
const total = await fetch("https://target.example.com/", {
  credentials: "include", // "omit"
})
  // swallow the likely NetworkError exception occuring from the CORS error.
  .catch(() => {})
  .then(() => {
    const end = Date.now();
    return end - start;
  });
```

With the following code, we are able to put together an attacker webpage, get an hypothetical victim to visit it, and begin poking around to find out if, for a given URL, fetches take significantly longer to respond with credentials (cookies) included, or not.
If they do, chances are you are **able to detect whether or not the user is logged-in to this particular website**.

However, in order to have more confidence in the results, we should update our exploit code to be aware of what constitutes _"significantly longer"_, for instance, we could draw the line at a 40% tolerance.
We also might want our code to repeat the experiment a few times and ensure that the results are the same 80% of the time, so we'd be able to tell the results are _consistent_.

[View Gist with complete example code of the malicious page](https://gist.github.com/ziir/98b79638b4cb889c1b718e56eab0f68a)

### How could this be used?

Such Web Timing Attacks could be used by bad actors in order to further **identify potential targets**, for example as the ~first of a multi-stage attack, starting from a spam campaign linking to a malicious webpage controlled by the attacker and containing the exploit code, allowing them to identify whether or not _victims are users of specific sites or services_, before engaging in a more targeted phishing campaign.
Depending on the website or service being targeted and after careful analysis by the attackers, one may even be able to gather more precise information about the victim than just their authentication status: potentially their _customer profile_ (ie: a specific page of a given website may take a very long or telling time to load, when in the context of a large business).

### How to prevent such web timing attacks?

1. Do not use `SameSite=None`, use either `SameSite=Strict` or `SameSite=Lax`: this way, cookies will not be forwarded in cross-origin subrequests.
2. Use a modern, up-to-date web browser, in fact, [use Firefox](https://www.mozilla.org/en-US/firefox/browsers/).

### Bad Performance is Bad Security

One slightly disingenuous way of looking at this is to blame the feasability of this attack on the lack of performance focus: if **everything** is as fast as it can be, then a third party cannot interpolate private information based on response times.
Alternatively, one other way to look at this is to aim for making all requests roughly respond in the same time, by artificially slowing them down, but that would be silly, **please don't do that**.

### Don't try this at home

Or rather, **please do try this at home**, but don't use this for _nefarious purposes_, I wrote this article for fun <u>and profit</u> and somewhat educational purposes.

### Share the love

If you enjoyed this article, please feel free to share it and let me know on [Twitter](https://twitter.com/tpillard) or [Mastodon](https://hachyderm.io/@tpillard), don't hesitate to take a look at [my about page](/about/) as well.
Did I mention I'm looking for a job?
