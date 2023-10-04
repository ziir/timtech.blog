---
title: React Server Components, without a framework?
description: Can we use React Server Components today, without a framework like Next.js? As our experiment shows, not without significant challenges - for now at least?
image:
  path: /img/react-server-components-rsc-no-framework-demo-screenshot.png
  width: 800
  alt: "Screenshot of the RSC Notes App Demo / Playground"
date: 2023-11-07
tags:
  - react
  - frameworks
  - performance
  - JavaScript
standalone: true
layout: layouts/post.njk
---

<img src="/img/react-server-components-rsc-no-framework-demo-screenshot.png" alt="Screenshot of the RSC Notes App Demo / Playground" width="800" style="border: 1px solid #282c34;" />

- [Live RSC Notes App Demo / Playground](https://rsc-demo.timtech.blog)
- [Source code on GitHub](https://github.com/ziir/rsc-demo)

## Introduction

**React Server** features, concepts & APIs, in particular **React Server Components** & **React Server Actions** have been in the spotlight for the past week, following recent announcements by the React Team ([**React Server & Client Actions** on October 23rd](https://twitter.com/reactjs/status/1716573234160967762)) & Next.js - **Next.js 14 “Server Actions (stable)”** announced on [October 26th 2023](https://twitter.com/nextjs/status/1717596665690091542) at Next.js Conf - which had a very strong _resonance_ in tech communities & on social media.

Before then, I was somewhat oblivious to the <abbr title="React Server Components">RSC</abbr> model, and upcoming [React Canary](https://react.dev/community/versioning-policy#canary-channel) features.

Sure, over the years, I had heard & thought quite a bit about the possibilities of a React putting a stronger focus on the server, especially when I was working at _Gandi_ for 6 years, where we had been working with React since `~v0.13.0 / 2015`, building our own, in-house, custom **_React / Redux isomorphic / universal SPA / SSR full-stack framework_**, architecture & ecosystem powering ~10 business-critical production applications.
I had also dipped a toe in the [mozilla/addons-frontend](https://github.com/mozilla/addons-frontend) code base of [Mozilla Addons marketplace (AMO)](https://addons.mozilla.org), which leverages a comparable architecture, from the same _era_.

Despite the years, the distance & perspective I now have from my old job at Gandi (I left in 2020 after 6years), the technical challenges related to my experience, striving to build a performant, resilient, capable, productive, tailored, in-house **React framework** with [progressive enhancement](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement) and `no-js` first-class support (essentially what **meta-frameworks** such as [Remix](https://remix.run/), _Next.js_ & maybe others **_can_** do **today** if you're **serious** about it), have been living rent-free in my head ever since (and I have many more like that, such as [when to use React.memo](https://timtech.blog/posts/react-memo-is-good-actually/)).

So of course, when I saw on Twitter the [RSC / React Canary / Server & Client Actions announcements](#introduction) mentioned earlier, I was excited!
Could it be? A _React-native_, official first-party solution to all the _problems of React server-side rendering_: data-fetching, waterfall, routing, latency, partial / selective hydration, in-or-out-of-order streaming, you name it: **maybe that was it**.

And so, I began an unreasonable journey, starting to look into **React Server APIs**, the **RSC model**, **Client / Server actions**, **React Canary**, etc. from scratch, with close to zero prior knowledge, and a lot of interrogations.

## RSC Survival Guide

Before going any further, chances are the average reader is as confused about all this as I was when I first started looking into this a few days ago.
Indeed, _X dot com_ is full of keywords, code snippets, memes & engagement baits while actual information regarding this topic is scattered, incomplete, missing, tied to a specific Meta-framework, focused on high level use cases, or simply slightly harder to find than the typical React content covering mature & stable patterns, such as [React controlled inputs](https://timtech.blog/posts/presentation-dashlane-react-form-handling-giving-up-control-example-controlled-input/) - ok that's the last internal backlink, I [Promise concurrently](https://timtech.blog/posts/limiting-async-operations-promise-concurrency-javascript/).

### Disclaimer

This section aims to serve as a **quick reference** for **<abbr title="React Server Components">RSC</abbr>**, centralizing what I consider to be the most important terms, definitions & concepts about these features, just so a maximum amount of readers who made it this far (thank you! means a lot to me) can follow along with the rest of the blog post.

The goal is not to provide a guide on how to use these features, rather an extract & structuring of my personal understanding of the different pieces of the puzzle based on my personal research, with links to relevant resources.

Please bear in mind this blog post mentions **experimental & under-documented features & APIs** and generally covers a **complex & continously-evolving topic** - it may not be the best source of information for beginners or readers who have never yet been exposed to the concepts of the RSC Model.

> _THE BLOG POST IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED…_

### React Client Components

**React Client Components** are traditional React Components - which **can run on the client**, traditionally the browser.
Chances are, most of the React components you've ever been in contact with and that you're used to, at least prior to [React 18](https://react.dev/blog/2022/03/29/react-v18) & [Next.js 13](https://nextjs.org/blog/next-13) were Client Components, and this, regardless of them being rendered by [React DOM Server APIs](https://react.dev/reference/react-dom/server) to produce or stream <abbr title="Hypertext Markup Language">HTML</abbr> for _server-side rendering (SSR)_ or _build-time static site generation (SSG)_.

> _“The name “Client Component” doesn’t mean anything new, it only serves to distinguish these components from Server Components”_

> _“If you have an existing client-side app, you can think of all of it as a Client Component tree”_

[RFC: React Server Components #188](https://github.com/reactjs/rfcs/pull/188)

### The React Server Components Model

#### React Server Components Announcements Timeline

- [“Introducing Zero-Bundle-Size React Server Components” December 2020](https://react.dev/blog/2020/12/21/data-fetching-with-react-server-components)
- [“Data Fetching with React Server Components” December 2020](https://www.youtube.com/watch?v=TQQPAU21ZUw)
- [“RFC: React Server Components #188” December 2020 - October 2022](https://github.com/reactjs/rfcs/pull/188)
- [“Next.js 13 (stable) #server-components” October 2022](https://nextjs.org/blog/next-13#server-components)
- [“React from Another Dimension” by Dan Abramov at #RemixConf (May) 2023](https://www.youtube.com/watch?v=zMf_xeGPn6s)

#### React Server Components

**React Server Components**, strictly speaking, are **React components that run exclusively on the server**, they never run or are even hydrated on the client, in fact, **aren't even included, bundled with alongside client-side code**, and thus have **no client-side JavaScript bundle size impact**.

However, and even more importantly:

> _“Server Components allows the server and the client (browser) to collaborate in rendering your React application.
> Consider the typical React element tree that is rendered for your page, which is usually composed of different React components rendering more React components.
> RSC makes it possible for some components in this tree to be rendered by the server, and some components to be rendered by the browser.”_

[RFC: React Server Components #188](https://github.com/reactjs/rfcs/pull/188)

<hr />

##### A quick note on “React SSR”

It's important to note that RSC is different than “React SSR”.
Without diving in a **“RSC vs React SSR”** comparison, I'll just mention a few points:

- (React) “SSR” is most commonly used to refer to the rendering on the server of a tree of React components to _inert_, non-interactive HTML to serve a page's **initial load**, the same components tree which is then potentially _hydrated_ in the client to enable interactivity
- RSCs, on the other hand, **are not rendered to HTML**, but to a JSON-like format.

<hr />

In the **RSC paradigm**, all components are Server Components by default.
However, since an instance of a Server Component **runs only once**, Server Components can only use a subset of React Component APIs, and can't use client-only APIs such as `React.useState` / `React.useReducer` (same thing), `React.useEffect`, `Context.Provider` etc.

Furthermore Server Components cannot be _directly_ imported nor rendered by Client Components.
But their serialized render output can be fetched at the server from the client & inserted in within a client-side components tree.

Server Components also cannot render Client Components, but they can pass **serializable props** to them - by “serializable” props, we mean JavaScript data types such as simple objects, arrays, strings, numbers, as well as Promises wrapping such data types, but not functions & classes for example.

#### React Shared Components

**React Shared Components** are “neutral” components: they can be both Server Components & Client Components - not using either of client or server specific features & APIs.
As such, a given Shared Component in a React components subtree ultimately ends being either a Server Component or a Client Component, depending on the parent context in which they are rendered:

> a shared component becomes a server component when it is rendered with React Server APIs or from a server component

and conversely:

> a shared component becomes a client component when it is rendered with React DOM APIs, or from a client component

_**Plot twist**_: components can be marked **client-only** using the **`'use client'` directives**.

#### `'use server'` & `'use client'` directives

Although they look similar to [the standard, runtime `'use strict'` directive](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode), `'use server'` & `'use client'` are specific to React and don't do anything by themselves or at runtime: today they're **build-time hints for the bundler** (or equivalent tooling) to consider the marked modules as **a boundary between server & client in the module dependency tree**.

_Note_: I'm not going into more details about these directives here, because the official documentation is already really good at the time of writing, and paraphrasing doesn't bring more value.
There are a couple of caveats / subtleties that aren't necessarily covered in the official documentation, but mentioning them at this point of the blog post I think would only risk additional confusion.
But we'll get back to these directives further in the blog post.

##### `'use client'`

`'use client'` is primarily used to mark a specific component as being client-only, including all of its components subtree.
See ['use client' official React documentation](https://react.dev/reference/react/use-client).

##### `'use server'`

See ['use server' official React documentation](https://react.dev/reference/react/use-server).

#### Suspense & Error Boundaries

The RSC Model leverages existing Stable React APIs such as [Suspense](https://react.dev/reference/react/Suspense) & [Error Boundaries](https://react.dev/reference/react/Component#static-getderivedstatefromerror), these two APIs in fact play an central role in the **RSC Paradigm**, alongside **the `use()` hook, a new React Experimental API**.

#### `use()`

I'll just mention that the `use()` hook enables dealing with _Promises_ inside React Components.
This is over-simplified, and we'll see in more concrete terms how to use them further in the blog post.

See [`use()` hook React Experimental API official documentation](https://react.dev/reference/react/use).

## Demo: migrating a React Notes App to RSC without a framework

### A take-home technical assignment backstory

**In 2020**, while looking for my next job, I was contacted by a well known tech company regarding a remote-friendly open position, which really resonated with my interests and I was thrilled at the opportunity.
The hiring process required to complete a take-home technical exercise - although I'm really not a fan of this hiring practice, on either side of the table - I decided that I would try.

### A React Notes App (2020)

The assignment was essentially: **build a React Notes App**, and then some functional UI requirements.

And so I started working on building the following [React Notes App](https://react-notes-app.timtech.blog/) with **Create React App**, [documenting my process on Twitter](https://twitter.com/tpillard/status/1305889708430553088) & publishing [the code on GitHub](https://github.com/ziir/react-notes-app).

_Notes:_

- while this app still holds up fairly well in my opinion, it is far from perfect, especially by today's standards.
- you may notice that the hosted version while functionally identical to the source code in the repository, has been optimized for bundle size, most notably by fine-tuning the Babel configuration & aliasing React to Preact: from 161KB (according to my notes, I haven't been bothered to run the project again locally but the details are somewhere in the ~live-tweeting) to 38.5 kB (uncompressed production build).

To wrap this up, let's just say the hiring team was pleased with my work, everything was going well after that, until the recruiter asked me if I was willing to relocate to another country (despite being clear about working remotely from France - at absolute peak COVID-19 times - and having spent many hours working on the assignment).
Ironically, it was only after I had already accepted an offer at **Dashlane** that the first company reached back to me & suggested starting out remotely as a contractor.

> _Ok, ok, but what does it all have to do with Server Components?_

Well I've lately been using it this notes app as a playground for experiments & side projects.
So naturally, when I started looking into RSC last week, I thought to myself:

> _why not try to see to which extent I can naively migrate this simple notes app to RSC without a framework?_

<h3>Actually migrating a <abbr title="Create React App">CRA</abbr> React Notes App to RSC with no framework (2023)</h3>

No framework? Really?
Yes! Really! It's an experiment! It's fun!
And it's a great way to understand how things work, as well as illustrating the value of a "real" framework.
That being said, I am a huge supporter of building your own production framework for "real world" applications if you have the opportunity, the resources and can justify it to address your use cases, it'll just need to go much further than what is described in this blog post, more on this later.

### Disclaimer

Please note, there is absolutely nothing optimal about the following implementation, it is purposely naive, incomplete, and is intended for learning & demo purposes.

### Inspiration

Our demo is inspired by [React Team react server components demo from December 2020](https://github.com/reactjs/server-components-demo).
The idea was not to re-create the demo above, or to create as good a showcase of RSC, rather to **try** & use RSC features & APIs in an existing app.
We will be borrowing some code patterns, with varying degrees of correctness & success.
I also invite you to take a look at the official demo linked above & related materials.

### RSC Notes App Demo / Playground

- [Live RSC Notes App Demo / Playground](https://rsc-demo.timtech.blog)
- [Source code on GitHub](https://github.com/ziir/rsc-demo)
- Open your browser devtools!

### Objectives

- a somewhat functional React Notes App (listing, creating, editing & deleting notes with a text title & Markdown content)
- React Server Components used alongside Client Components
- some sort of server-side rendering
- some kind of data-fetching using Server Components
- routing that somewhat works both in the client & on the server
- refreshing server components from the client
- a usable RSC playground
- above all else: learn

### Dependencies

- `node` v18.8.1 & `npm` v9.8.1
- [Fastify](https://fastify.dev/) for the HTTP web server
- [Webpack](https://webpack.js.org) bundler
- [Babel](https://babeljs.io) JavaScript compiler, for transpiling JSX syntax
- [a-route](https://github.com/WebReflection/a-route) extremely minimal JavaScript / DOM routing library leveraging Custom Elements (also used in our original [React Notes App](#a-react-notes-app))

#### React Canary

At the time of writing, the latest React Canary release is `18.3.0-canary-8039e6d0b-20231026`.
We need to use the same version for the `react` & `react-dom` packages.

```json
// package.json
"dependencies": {
  ...
  "react": "^18.3.0-canary-8039e6d0b-20231026",
  "react-dom": "^18.3.0-canary-8039e6d0b-20231026",
  ...
}
```

#### React Server & React Flight APIs

The traditional React dependencies are no longer sufficient when it comes to using Server Components, and for `'use client'` directives to play [their role](#'use-server'-%26-'use-client'-directives). For this, specialized tooling is required, and we're only interested in using what is provided by the React Team.

Like the [official RSC demo](#inspiration) we've mentioned earlier, we will be relying on the [react-server-dom-webpack](https://github.com/facebook/react/tree/main/packages/react-server-dom-webpack) package to enable RSC features.

> _"Experimental React Flight bindings for DOM using Webpack - Use it at your own risk"._

_Note:_ These APIs are **completely undocumented** and there might be better alternatives, provided by the React Team or otherwise.

In particular, we will be using the following APIs, which I've done my best to succintly describe:

##### `react-server-dom-webpack/plugin`

[`react-server-dom-webpack/plugin`](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackPlugin.js) is a Webpack Plugin to:

- generate the client chunks based on the presence of the `'use client'` directive within modules.
- reference the client chunks & the other chunks they depend on in a `react-client-manifest.json` file.
- leverage Webpack's runtime to load the client chunks on demand: **code splitting**.

###### Example Usage

```js
// webpack.config.js
const ReactServerWebpackPlugin = require("react-server-dom-webpack/plugin");

return {
    ...
  plugins: [
    ...
    new ReactServerWebpackPlugin({ isServer: false }),
  ],
}
```

```json
{
  // react-client.manifest.json
  ...
  "file:///rsc-no-framework-demo/src/app/tabs.jsx": {
    "id": "./src/app/tabs.jsx",
    "chunks": [
      "client5",
      "client5.chunk.js"
    ],
    "name": "*"
  }
}
```

##### `react-server-dom-webpack/node-register`

[`react-server-dom-webpack/node-register`](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeRegister.js), to create client & server references at runtime on the Node.js server, and creating proxies for client modules. This what effectively enables React Server Components to pass props to Client Components.

###### Example Usage

```js
// server entry-point
const reactServerRegister = require("react-server-dom-webpack/node-register");
reactServerRegister();
```

##### `renderToPipeableStream` from `react-server-dom-webpack/server`

[`react-server-dom-webpack/server#renderToPipeableStream`](https://github.com/facebook/react/blob/ce2bc58a9f6f3b0bfc8c738a0d8e2a5f3a332ff5/packages/react-server-dom-webpack/src/ReactFlightDOMServerNode.js#L75), renders a provided React Element with a Server Components subtree to an object matching a `PipeableStream` interface & returns it, allowing the consumer to `pipe` it to a [`WritableStream`](https://nodejs.org/api/stream.html#writable-streams). This is what effectively renders React Server Components to the **React Flight format** & allows the output to be streamed to the client over HTTP.

###### Example Usage

```js
// server.js

const MANIFEST = readFileSync(
  path.resolve(__dirname, "../dist/react-client-manifest.json"),
  "utf8"
);
const MODULE_MAP = JSON.parse(MANIFEST);

function renderReactTree(writable, component, props) {
  const { pipe } = renderToPipeableStream(
    React.createElement(component, props),
    MODULE_MAP
  );
  pipe(writable);
}

...

// streaming the rendering output of the App element & its RSC subtree
// to the React Flight format in a Fastify route handler
reply.header("Content-Type", "application/octet-stream");
renderReactTree(reply.raw, App, props);

...

// accumulating to a string the rendering output of the App element & its RSC subtree
let flightResponse = "";
const flightStream = new stream.Writable({
  write: (chunk, encoding, next) => {
    flightResponse += chunk;
    next();
  },
});
renderReactTree(flightStream, App, props);

await finished(flightStream);
```

##### `createFromReadableStream` from `react-server-dom-webpack/client`

[`react-server-dom-webpack/client#createFromReadableStream`](https://github.com/facebook/react/blob/ce2bc58a9f6f3b0bfc8c738a0d8e2a5f3a332ff5/packages/react-server-dom-webpack/src/ReactFlightDOMClientBrowser.js#L71) is essentially a wrapper around [**React Flight Client API**](https://github.com/facebook/react/tree/main/packages/react-client).

> _This is an experimental package for consuming custom React streaming models.
> Its API is not as stable as that of React, React Native, or React DOM, and does not follow the common versioning scheme.
> Use it at your own risk._

Provided with a [ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) of **React Flight serialized Server Components** data, it outputs a **Thenable**, essentially reading & wrapping the data chunks in a **Promise-like data structure**, which can be consumed by APIs dealing with **Promises**, essentially the [**`use()` experimental hook**](#use).
This is the main **RSC API** that allows for **React Server Components tree output** computed on the server and **serialized to the React Flight format** to be rendered **on the client**.

###### Example Usage

```js
import { createFromReadableStream } from "react-server-dom-webpack/client";

...

const flightTreePromise = createFromReadableStream(
  // fake a ReadableStream with the Response constructor
  new Response(flightResponseText).body
);
```

##### `createFromFetch` from `react-server-dom-webpack/client`

[`react-server-dom-webpack/client#createFromReadableStream`](https://github.com/facebook/react/blob/ce2bc58a9f6f3b0bfc8c738a0d8e2a5f3a332ff5/packages/react-server-dom-webpack/src/ReactFlightDOMClientBrowser.js#L80) is essentially a wrapper around [`createFromReadableStream` from `react-server-dom-webpack/client`](#createfromreadablestream-from-react-server-dom-webpackclient), unwrapping the `Response` promise to retrieve the underlying `ReadableStream`.
It serves the same purpose, but allows for the most common React Flight Client use case: rendering streamed React Server Components serialized output elements on the client, from a single [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) call.

###### Example Usage

```js
const [flightTreePromise, setFlightTreePromise] = useState(
  initialFlightTreePromise
);

...

const refreshFlightTreePromise = createFromFetch(
  fetch(
    `/react-flight?props=${encodeURIComponent(JSON.stringify(props))}`
  )
);

setFlightTreePromise(refreshFlightTreePromise);

...

return use(flightTreePromise);
```

### Our first React Server Component

In this hypothetical example, `<App />` and all its subtree are Shared Components.
They do not use any client-specific APIs, they can be both Server Components or Client Components, they can be both rendered on the server or on the client.

```jsx
export function App() {
  const items = [];

  return (
    <Layout header={<Header />}>
      <List items={items} />
    </Layout>
  );
}
```

Because we will be rendering the `<App />` component and its subtree with React Server APIs, all these components are automatically Server Components.
The rendering of the `<App />` component, even though it is streamed, can be considered synchronous: a single **React Flight chunk** is emitted.

### Server-Side Rendering of Server Components

[React Server & React Flight APIs](#react-server--react-flight-apis) being completely undocumented as mentioned earlier, properly rendering output of React Server Components to HTML, and streaming it efficiently to the browser, without a framework, is quite challenging.
Indeed, while it is relatively straightforward to obtain the rendering output of a server components tree in the React Flight Format (also known as the **RSC payload**), as a `PipeableStream` or a string using these APIs - [renderToPipeableStream](#rendertopipeablestream-from-react-server-dom-webpackserver) in particular - at the time of writing, I don't know how to efficiently translate it to HTML - and I was too lazy to deep dive in Next.js' implementation.
But for the purposes of learning & demonstration, we really want to do _something, anything_ that ressembles Server Components SSR.

#### An Unorthodox SSR Implementation

##### Traditional SSR Store Serialization

Traditionally, one way of doing React SSR is to fetch necessary data prior to or while rendering the application, using React DOM Server APIs, such as `ReactDOM.renderToString()`.
As the data is fetched, it populates a store such as a Redux store.

Once the rendering of the application is completed, the Redux store is serialized to JSON, so it can be used as the client-side application's store initial state.
In order to be picked up when the client application loads, the serialized store output is generally included at the end of the HTML document, as a JSON string inside a `<script>` tag, or as a `string` or assigned directly to a variable with `const state = JSON.parse('{serialized_store}')`.
This is necessary to correctly hydrate the sever-side rendered React tree, allowing client-side React to re-run the components tree with the same data used for the server-side rendering.

##### Server Components React Flight Output Serialization

As Server Components are rendered to the **serialized React Flight format**, we can apply the same technique to the **RSC payload**.
Of course, this is incredibly suboptimal, and eliminates most of the benefits of Server-side rendering.

We actually shouldn't even call it Server-side rendering, because the Server components tree is not even rendered as HTML in the served initial HTML document, and JavaScript is necessary to render the RSC tree into the DOM.

But regardless, this still has some benefits: we don't have to make one extra Fetch HTTP request when the client-side app loads to retrieve the RSC tree: the data is already there on first load, serialized in the DOM or in a JavaScript variable, at the cost of a higher <abbr title="Time To First Byte">TTFB</abbr>.
Good enough for a first try, and a good first exposure to the relevant React Server / Client APIs, but what would that look like?

_Note:_ actually, this technique may be a good fit for a specific variant of SSG, where the Server Components are rendered during the build process, and only static files are shipped to production.

##### Inlining Server Components Output into the HTML Shell

First, we need to adapt our `index.html` HTML shell template, and define a slot to inject our React Flight data.

```html
<body>
  <div id="root"></div>
  <script id="react-flight" type="react/flight">
    <!--FLIGHT-->
  </script>
</body>
```

Then, in our web server (here, written with Fastify), create a route handler that will render our React Server Components tree into a _PipeableStream_, which we `pipe` to our `flightStream` _WritableStream_, letting us accumulate the React Flight chunks into a string.
We then `await finished(flightStream)` to wait for all the chunks to be written, and finally inject the complete React Flight Response in our template.

```js
const { readFileSync } = require("node:fs");
const stream = require("node:stream");
const { finished } = require("node:stream/promises");

const React = require("react");
const { renderToPipeableStream } = require("react-server-dom-webpack/server");

const { App } = require("../src/app/app");

const MANIFEST = readFileSync(
  path.resolve(__dirname, "../dist/react-client-manifest.json"),
  "utf8"
);
const MODULE_MAP = JSON.parse(MANIFEST);

const HTML_SHELL = readFileSync(
  path.resolve(__dirname, "../dist/index.html"),
  "utf8"
);

function renderReactTree(writable, component, props) {
  const { pipe } = renderToPipeableStream(
    React.createElement(component, props),
    MODULE_MAP
  );
  pipe(writable);
}

const fastify = Fastify();

fastify.get(async function routeHandler(request, reply) {
  let flightResponse = "";
  const flightStream = new stream.Writable({
    write: (chunk, encoding, next) => {
      flightResponse += chunk;
      next();
    },
  });
  renderReactTree(flightStream, App, {});

  await finished(flightStream);

  reply.header("Content-Type", "text/html");
  return HTML_SHELL.replace("<!--FLIGHT-->", JSON.stringify(flightResponse));
});
```

Now we're able to render React Server Components on the server, and inline the output data inside our HTML template.
Nice, but how do we then pick that up to be rendered on the client?

##### Rendering Server Components Output on the Client

We first need to retrieve the inlined React Flight data from the DOM `<script>` tag, transform it into a _ReadableStream_ (here using the `Response().body` trick for convenience), before we feed it to React Flight Client API `createFromReadableStream` & obtain a Promise-like object, which we can finally pass to the `use()` hook from our `<Root />` application root React Component.

```js
import { createElement, use } from "react";

import { createRoot } from "react-dom/client";
import { createFromReadableStream } from "react-server-dom-webpack/client";

function Root({ flightTreePromise }) {
  return use(flightTreePromise);
}

const flightEl = document.getElementById("react-flight");
const flightResponseText = JSON.parse(flightEl.textContent);

const flightTreePromise = createFromReadableStream(
  new Response(flightResponseText).body
);

const root = createRoot(document.getElementById("root"));

root.render(<Root flightTreePromise={flightTreePromise} />);
```

### Data-Fetching in Server Components

#### Adding Data-Fetching with `async` / `await`

```jsx
export async function App() {
  const items = await new Promise((resolve) =>
    setTimeout(() => resolve([]), 1000)
  );

  return (
    <Layout header={<Header />}>
      <List items={items} />
    </Layout>
  );
}
```

The rendering of the `<App />` component becomes truly asynchronous, it is _streamed_ in multiple chunks, starting with a first chunk, immediately before the `await`, then followed by the remaining second chunk after the `Promise` resolves, including the rendering output of the `<Layout>`, `<Header />` & `<List>` components.

#### Data-Fetching in child components: waterfall?

Let's now make the `<List />` component `async` with a similar data-fetching constraint.

```jsx
export async function List({ items }) {
  await new Promise((resolve) => setTimeout(resolve, 2000));
  ...
}
```

The rendering of the `<List />` component only starts after the `<App />`'s awaited _Promise_ resolves, and the awaited _Promise_ of `<List />` in turn, blocks the rendering of further child components. The second chunk now only includes the output of the `<App />` component, the output of the `<List />` component is only emitted in a third chunk, after it's awaited `Promise` has resolved.

In other words,the Server Components tree is rendered & streamed asynchronously, naive use of `async`/`await` for **data-fetching in server components** induces a **waterfall**.

Are there other options available to us?

#### Moving the `async`/`await` down

Let's simplify our `<App />` for clarity, and adapt it to trigger data-fetching at render-time as previously, but we remove the `async` / `await` from it, and move the `async`/`await` down to the `<List />` component:

```jsx
export function App() {
  const itemsPromise = new Promise((resolve) =>
    setTimeout(() => resolve([]), 1000)
  );

  return <List itemsPromise={itemsPromise} />;
}
```

```jsx
export async function List({ itemsPromise }) {
  const items = await itemsPromise;
  ...
}
```

A fourth chunk makes its appearance, containing the output of the `<List />` component, after the `await` of the `itemsPromise` Promise has resolved.
Although this is progress, in that instead of passing a resolved Promise value prop to children, we're passing a Promise props directly, and the component actually making use of the Promise is the one `await`ing it, without blocking the rendering of (or data-fetching inside) other children of the `<App />` component: still a waterfall, still idiomatic JavaScript & Promise handling, but slightly better

We mentioned earlier that React Server Components integrate with **Suspense**, but how do we leverage it?

#### Suspense-enabled streaming

##### With async/await

Using `async`/`await` **in Server Components** is compatible with `<Suspense>`, so all we need to do is to wrap it around our `<List />` component, and give it a `fallback` _React Node_, to be used while the _Promise_ is _pending_.

```jsx
export function App() {
  console.log("rendering <App /> component");
  const itemsPromise = new Promise((resolve) =>
    setTimeout(() => resolve(["Suspense-enabled streaming"]), 1000)
  );

  return (
    <Suspense fallback="Loading ...">
      <List itemsPromise={itemsPromise} />
    </Suspense>
  );
}
```

```jsx
export async function List({ itemsPromise }) {
  console.log("before the await");
  const items = await itemsPromise;
  console.log("after the await");

  return (
    <ul className="List">
      {items.map((item) => (
        <li>{item}</li>
      ))}
    </ul>
  );
}
```

We can see in our server logs the following sequence:

```
rendering <App /> component
before the await
```

Then as you'd expect, one second later:

```
after the await
```

Which translates to the following _React Flight chunks (RSC Payload)_:

```
1:"$Sreact.suspense"
0:["$","$1",null,{"fallback":"Loading ...","children":"$L2"}]
```

Then one second later:

```
2:["$","ul",null,{"className":"List","children":[["$","li",null,{"children":"Suspense-enabled streaming"}]]}]
```

The usage of `async / await` in a **Server Component wrapped in Suspense** causes said component's render to be **postponed**.
This is the idiomatic, recommended pattern for **data-fetching in Server Components with Suspense**.

In the browser - with either a specific SSR implementation with RSC streaming support, or a client app able to render streamed React Flight chunks - this would result in the following sequence displayed on the screen:

- render the `Loading...` suspense fallback
- wait 1second for the next chunk to come in
- replace the `Loading...` suspense fallback with the following HTML / DOM:

```html
<ul>
  <li>Suspense-enabled streaming</li>
</ul>
```

But what about [`use()` experimental hook](#use)?
Didn't we say that `use()` allowed us to handle Promises in components, in conjunction with _Suspense_?

##### use() vs async/await

The [`use()` experimental hook](#use) indeed lets us consume Promises inside React Components.
If we take the code of our `<List />` **server component** from the previous example, remove the **async / await** and replace it with `use(itemsPromise)` instead, our server components behave **almost** the same:

```js
import { use } from 'react';

export function List({ itemsPromise }) {
  console.log("before the use()");
  const items = use(itemsPromise);
  console.log("after the use()");
  ...
}
```

The practical **difference between use() and async/await** in server components is that in the `use()` case, the **use** hook tracks the state of the provided _Promise_, by checking its `Thenable.status`.

- if the `Thenable.status` is `pending` or `rejected`, it **suspends** (interrupts) the rendering of the component
- if the `Thenable.status` is `fulfilled`, it returns the resolved value, and _resumes_ rendering the component

The rendering of a component **suspended** with _use()_ - contrarily to a **postponed** server component with _async/await_ - will be _resumed_ once the _Thenable_ (Promise) ultimately resolves.
Effectively, the render of the component is **replayed**.
The details of the _use() hook_ are complex, but we can observe the general behavior in our server logs:

```
rendering <App /> component
before the use()
```

Then one second later:

```
before the use()
after the use()
```

Notice how the `before the use()` message is logged twice.
The `use()` hook has caused our `<List />` component to **suspend**, and once `itemsPromise` has resolved, its rendering is _resumed_ by being **replayed**.

This translates to both an identical **RSC Payload**, and an identical output displayed in the browser, as in the previous example with `async/await`.

#### Passing the Server's pending Promise to the Client

Honestly, this is all already pretty good.
But we can go even further than that: what if we could **start fetching data on the server** and directly _consume the Promise on the client_?
Let's consider _the perfect use() case_: that our `<List />` component, is in fact a _Client Component_.

First, we need to add the [`'use client'` directive](#'use-client') to our `list.jsx` module.

```jsx
"use client";

// List component ...
```

As previously, we still want to consume the `itemsPromise` in the `<List />` component.
But our `<List />` component is **now a Client Component**, and as such, cannot be `async`, thus cannot use `async`/`await`.

The solution, you've guessed it, is with the _use()_ hook.

##### `use()` the force, Luke

```jsx
"use client";

export function List({ itemsPromise }) {
  // `itemsPromise` is passed from a Server Component to a Client Component!
  const items = use(itemsPromise);
  ...
}
```

This results in the following _RSC Flight chunks_:

```
1:"$Sreact.suspense"
2:I["./src/app/list.jsx",["client3","client3.chunk.js"],"List"]
0:["$","$1",null,{"fallback":"Loading ...","children":["$","$L2",null,{"itemsPromise":"$@3"}]}]
```

And then one-second later, the contents of our `itemsPromise`:

```
3:["Suspense-enabled streaming"]
```

This is where things get really interesting - to be continued...

#### Waterfall waterfall waterfall

We succeeded in performing React Server Components Suspense-enabled data-fetching & streamed rendering!
This is great! It's better than our previous solutions of `await`ing Promises in any component, because our solution now correctly leverages `<Suspense />`!
But what about the _waterfall_? We only have a single (`<App />`) component that is doing data-fetching, and only a single component (`<List />`) consuming an **async data source** here...

If we add another async component with `await` (or `use()`) anywhere in the `<List />` subtree, we will still be introducing an **async waterfall**.
**Fetching data in components is inherently subject to waterfall**, even when leveraging best practices with _React Server Components, Suspense, async rendering_, simply because the data fetching within a component, cannot start until this component has started to render, which is potentially blocked by the data-fetching of the parent component.

And _that's fine actually™_: even though waterfall results in slightly suboptimal UX, it doesn't mean it's bad UX either!
Most of the time, waterfalls - even without all the combined benefits of RSC & Suspense - can yield a perfectly satisfactory experience.
And when it doesn't, there are multiple ways to mitigate the waterfall itself or its impact: **composition**, moving data-fetching up or down the tree, backend for frontend, optimizing, adapting or re-designing the API, etc...

Alternatively, if one really wants to avoid the waterfall problem inherent to fetching data from components, then one must move the data fetching **outside** of the component.
This problem has been known for a long time now and frameworks, libraries & patterns exist to solve it, such as Remix / React Router Loaders or Relay / GraphQL, in which case, the data requirements are defined, not in the component, but generally in the module, alongside the component.

Further alternatively, components could be responsible for defining their data requirements "what to fetch", and delegating the actual data-fetching to another system.

Finally, there might be compilers which will attempt to _optimize data-fetching in RSC_, and eliminate or mitigate **some** waterfalls automatically.

### Routing with Server Components?

Our original React Notes App uses a custom Router, leveraging the React Context API & the [a-route](https://github.com/WebReflection/a-route) library.
As such, it is client-side only & does not work with React Server Components.
In fact, the majority of the React Routing tooling out there, won't work with React Server Components.

I had to make a few tweaks to the original router implementation in our RSC Notes App, here is the (suboptimal) outcome:

- the (not great) initial implementation of the router was completely broken when I migrated to RSC, partly due to now unnecessary use of `useEffect` & render-functions.
- routes and their matching component are defined in a `routes.js` module, which is imported both on the server & on the client
- routes are matched both on the server for the initial load, and on the client for client-side navigation
- routes can be matched similarly by chance, because `a-route`'s route matching is similar to Express' with [path-to-regexp](https://github.com/pillarjs/path-to-regexp), and Fastify uses a similar route format
- after matching the route on the server, we deduce a `location` object which is similar to the few properties of `window.location` we use on the client
- the routing props are passed when rendering the `App` server component, so server components can use that
- the client-side router is autonomous, doesn't rely on values computed on the client
- all routes are Client Components, only the root component including the layout shared by all routes, as well as some of its children, are server components in our implementation

But all in all, our solution kinda works and **the app is functional**, objective completed!

A couple more things I would like to mention here in regard to routing with Server Components is that:

- Route components should probably either be all client components or server components, supporting both risks being formidably complex & confusing for the developer.
- The route matching logic should live outside of React, so it can be used on the server as well.
- Moving route paths & components definition outside of React, ie, in a boring module which can be imported, makes things easier.

I invite you to take a look at our RSC React Notes App routing implementation & experiment for yourself.
Maybe try to plug-in some of the most popular React routing libraries out there, see what happens.
Maybe I'll try some things myself and update this section at a later time.

### Form handling with Server Components

I didn't have the opportunity to experiment a lot with form handling in server components for two reasons:

- Server Actions for example are not supported by our setup
- our forms (note creation, edition, deletion) are all client-side:
  - due to limitations of our router implementation
  - to prevent abuse (since the demo is hosted & available to the public)

But I took the opportunity to play with **React Client Actions**.

#### React Client Actions

**React Client Actions** (like React Server Actions) are **async functions** that can be passed to the `<form action={...}>` & `<button formAction={...}>`, `<input type="submit" formAction={...} />` **React host / native elements props**.
In the case of Client Actions only, they allow developers to write form handling logic, slightly more efficiently, and handle form submission render updates to be handled asynchronously by React.

Until now, these props were expected to follow the
[`<form>` Form HTML Element's `action`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#action) [`<input>`, `<button>` Form HTML Element's `formaction`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button#formaction) attributes **standard behavior**, but not anymore: with Server / Client actions: they accept async functions.

Server / Client Actions encourage [Locality of Behaviour](https://htmx.org/essays/locality-of-behaviour/).

However, in our original React Notes App, we use the same `<Form />` component in the `Create` & `Edit` route components.
Therefore the note creation & edition logic is actually split between two components.
So our `Form.jsx` component looks a little bit like this:

```jsx
import { useState } from "react";

export function Form({
  formId,
  handleAction,

  children: renderAdjacent = null,

  item = null,
}) {
  const { id = null, title = "", content = "" } = item ?? {};
  const [submitting, setSubmitting] = useState(false);

  return (
    <>
      <form
        id={formId}
        role="form"
        className="Form"
        action={async function handleFormAction(formData) {
          setSubmitting(true);
          await handleAction(Object.fromEntries(formData));
          setSubmitting(false);
        }}
      >
        ...
      </form>
      <div className="Form-actions">
        {typeof renderAdjacent === "function" && renderAdjacent({ submitting })}
      </div>
    </>
  );
}
```

And our `Create.jsx` component, like this:

```jsx
"use client";
import { useRouter } from "./router";
import { useItemMutation } from "./items";

import { Form } from "./form";
import { Button } from "./button";

import { generateId } from "../utils/id";

export function Create() {
  const router = useRouter();
  const { addItem } = useItemMutation();

  return (
    <div className="Create">
      <Form
        formId="create-form"
        handleAction={async function handleCreateAction(values) {
          const item = {
            ...values,
            id: generateId(),
          };

          await new Promise((resolve) => setTimeout(resolve, 500));
          addItem(item);
          router.navigate(`/view/${item.id}`);
        }}
      >
        {({ submitting }) => (
          <Button type="submit" form="create-form" disabled={submitting}>
            Create
          </Button>
        )}
      </Form>
    </div>
  );
}
```

As you may notice, our implementation relies on a `submitting` React state variable.
I wanted to use the new [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus) experimental hook in the `Create` component to enable the `disabled` prop on our `<Button />` component when the form is `submitting`.

Unfortunately, this is not possible because `useFormStatus` only works for components rendered within a `<form>` element, and our submit button actually lives **outside of the `<form>` element**.
This can be easily fixed by creating a `<SubmitButton />` component and rendering it as part of the `<form>` subtree.

However, rendering form submit buttons outside of a `<form>` element, leveraging the `form` property with a form `id` is very common nowadays.
I wish we could have used the `useFormStatus` hook outside of a `<form>` element, possibly supplying it a `formId`.

## Should you be writing your own RSC framework?

There is no doubt, the RSC model is very powerful.

As we've seen together, implementing it _from scratch_ is not going to be an easy feat, due to the inherent complexity of React Server Components of course, and in my opinion, due to the lack of shared building blocks provided by the React Team, and the fact that the ones provided are largely under-documented today.
Which of course leads to frameworks (Next.js) owning a very large part of the model - and necessarily leading to slightly different implementations, probably incompatible with one another.

Therefore I would not recommend product or platform teams to implement their own RSC framework in the current state of things.
However, I do recommend individuals, Open Source contributors to experiment with this technology and challenge the status-quo.
I know some exceptionally smart & capable folks (much more than me) in the community are actively reverse-engineering Next.js & driving innovation, I just wish there **more**.

## Should you be migrating your Full-Stack React Framework to RSC?

This is a trick question.

On one hand, I absolutely want to recommend teams with an existing in-house Full-Stack React Framework to **progressively migrate to RSC**.
On the other hand, this is going to be a tremendous amount of research, reverse-engineering, migration & documentation work that will span over months if not years.

But at the same time, chances are, a move to RSC is going to allow you to modernize your architecture, leverage ~~experimental~~ modern React features, improve the architecture of your applications, write more efficient code more efficiently while also being a formidable catalyst for removing legacy debt.

Conceptually, RSC features allow to simplify or completely remove a lot of things that your framework was doing:

- **code-splitting** becomes infinitely easier if not automatic with `'use client'` whereas it was probably manual previously (if it was even in place)
- **server-side data-fetching in components** is baked-in whereas it was previously entirely custom & full of caveats (ie: confined to route components, relying on a feature of a specific router library)
- **SSR streaming** although currently incomplete (to my knowledge) as provided by the React Team today, is incredibly more efficient with React Flight than anything we've seen or done before
- **progressive enhancement** with Server Actions, despite their downsides, is likely to be a better solution than whatever we used to do to handle React form submissions without JavaScript
- **expensive components rendering** can in some cases be moved to the server
- **bundle size reduction** with Server Components & Server Actions, huge JavaScript libraries can be moved to the server while only their final output is sent to the client
- and more!

In any case, I would honestly **wait a bit longer**.
There are plenty of debates about RSC & other modern / experimental React features being stable or production-ready (and more) going on these days.
It's pretty obvious from the content of this blog post, despite my potential mistakes and it not being exhaustive, that this debate is legitimate.
I also think there needs to be more _divesity_ & **a wider adoption** in the ecosystem.

## Going further with RSC

### server-only & client-only packages

The [`server-only`](https://www.npmjs.com/package/server-only?activeTab=code) & [`client-only`](https://www.npmjs.com/package/client-only?activeTab=code) packages (or equivalent) are strongly recommended when working with RSC / Server Actions, for security reasons first, and potentially bundle size or load/runtime performance second.
These packages help preventing that code, supposed to run exclusively on the server, to be inadvertently imported from client code, or vice versa.

It's important to understand that these are nothing more than packages with an explicit name, placed at a convenient place - the top of a file with other imports - which immediately throw an error with a descriptive error message, at runtime.

> _`'server-only' cannot be imported from a Client Component module. It should only be used from a Server Component.`_

```js
// server-only index.js
throw new Error(
  "This module cannot be imported from a Client Component module. " +
    "It should only be used from a Server Component."
);
```

```json
// server-only package.json
"exports": {
  ".": {
    "react-server": "./empty.js",
    "default": "./index.js"
  }
```

As we can see, these packages leverage **package.json conditional exports** with the `react-server` custom condition - supported by [Node.js](https://nodejs.org/api/packages.html#exports) (`node --conditions react-server server.js`) & bundlers (such as [Webpack](https://webpack.js.org/guides/package-exports/) - with [`resolve.conditionNames`](https://webpack.js.org/configuration/resolve/#resolveconditionnames)) to inject either the throwing module, or an empty "no-op" module, depending on the consumer environment.

_Note:_ a potential alternative, would have been to use the `default: null` target, which would have rendered the package impossible to import from an environment that does not satisfy the `react-server` condition, causing an error at build time, or at runtime if there's no build step, and even possibly surfacing right in the developer's code editor thanks to static analysis with TypeScript for example.
Although this alternative seems to be superior in every aspect to me, I haven't tested it so it's only theoritical.
I am not aware of whether or not this alternative has been considered by the React Team, or if it did, why it wasn't chosen.

If custom resolution conditions such as `react-server` don't really work for you, or if you want to go further, rather than just throwing an error at runtime - which can be overlooked in some scenarios, and, from a productivity standpoint, potentially occurs too late in the developer's workflow, incurring an avoidable loss of productivity - it's a good idea to, like [Next.js does](https://github.com/vercel/next.js/blob/canary/packages/next/src/build/webpack-config.ts), error at build time at the bundler level, which is guaranteed to be more reliable (and still relying on the error being thrown at runtime as a safeguard), while also happenning much earlier in the process.
This however assumes bundling for the target environment & requires additional processing & configuration, which of course, come at a cost, however relatively small.

### React taint APIs

We've mentioned earlier that [Server Components can pass props to Client Components (React docs)](https://react.dev/reference/react/use-client#passing-props-from-server-to-client-components).
Simply put, React Taint APIs allow you to “taint” or mark JavaScript values & objects that **must not be passed** from the server to client, either by being passed from a Server Component to a Client component - in other words: which may not be serialized by React Server serialization APIs.
I haven't had the occasion to personally try them out & research them further, but these APIs are without a doubt primordial to handle secrets with React Server features, and should be used in conjunction with previously mentioned `server-only`.

Official React docs:

- [experimental_taintUniqueValue](https://react.dev/reference/react/experimental_taintUniqueValue)
- [experimental_taintObjectReference](https://react.dev/reference/react/experimental_taintObjectReference)

Facebook/react package source code for the APIs mentioned above, on GitHub:

- [React Taint APIs](https://github.com/facebook/react/blob/main/packages/react/src/ReactTaint.js)

### React Server & Client Actions

#### Announcements

**React Server & Client Actions** were [announced on October 23rd 2023](https://twitter.com/reactjs/status/1716573234160967762) to be available in [React Canary releases](https://react.dev/community/versioning-policy#canary-channel), starting from [October 5, 2023 (18.3.0-canary-546178f91-20231005)](https://github.com/facebook/react/blob/main/CHANGELOG-canary.md#october-5-2023-1830-canary-546178f91-20231005).

Three days later, [Next.js announced on October 26th 2023 in Next.js 14](https://twitter.com/nextjs/status/1717596665690091542) [“Server Actions (stable)”](https://nextjs.org/blog/next-14#server-actions-stable).

#### Ecosystem

This means the React Team considers these features frameworks, libraries (and the rest of the relevant ecosystem) can now adopt these features.
At the time of writing and to my knowledge, these features are for the most part, only adopted by **Next.js**.
The **Remix** React framework, has had a custom, somewhat similar feature to _Server Actions_ for some time with **Remix Actions**.

#### Philosophy

React Server & Client actions are inscribed in a dynamic from the React Team to provide efficient APIs for form handling, mutations, data fetching & **reducing client-side JavaScript bundle size**, promoting a [Progressive Enhancement](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement) design philosophy with <abbr title="Server-side Rendering">SSR</abbr>.

## Thanks for reading!

If you enjoyed this blog post, please feel free to share it and let me know on [Twitter](https://twitter.com/tpillard) or [Mastodon](https://hachyderm.io/@tpillard), don't hesitate to take a look at [my about page](/about/) as well.

Further content may be added in the future, clarifications & corrections as well if needed.
Follow-up blog posts covering experimental or modern React features should be added in the future.
Clarifications & corrections may be added as well if needed - please let me know, feedback & discussions are welcome!
