---
title: TypeScript Legacy Experimental Decorators with Type Metadata in 2023
description: I've been researching a way to modernize & speed-up a production setup making heavy use of TypeScript Legacy Experimental Decorators & Decorators Type Metadata.
image:
  path: /img/benchmark-typescript-decorators-compilation-tsc-vs-vite-swc.webp
  width: 640
  alt: "Screenshot of a benchmark where Vite + SWC is 2.77 times faster than the TypeScript compiler"
date: 2023-10-14
tags:
  - vite
  - swc
  - compilers
  - tooling
  - decorators
  - performance
  - TypeScript
  - JavaScript
standalone: true
layout: layouts/post.njk
---

<img src="/img/benchmark-typescript-decorators-compilation-tsc-vs-vite-swc.webp" width="640" alt="Screenshot of a benchmark where Vite + SWC is 2.77 times faster than the TypeScript compiler" />

## TL;DR

> _“Just use Vite” (+ SWC)_

- https://vitejs.dev
- https://github.com/ziir/vite-plugin-swc-transform

```
npm i --save-dev vite vite-plugin-swc-transform
```

```js
import { defineConfig } from "vite";
import swc from "vite-plugin-swc-transform";

export default defineConfig({
  plugins: [
    swc({
      swcOptions: {
        jsc: {
          target: "ES2021",
          transform: {
            legacyDecorator: true,
            decoratorMetadata: true,
          },
          // externalHelpers: true,
        },
      },
    }),
  ],
});
```

> _“kthxbye”_

## Problem

I _need_ to compile TypeScript code leveraging **TypeScript Legacy / Experimental Decorators with Type Metadata**.
_I also need_ this compilation to be reasonably fast.
And then, _I really want_ this compilation to be identical, as much as possible, between automated (unit & integration) & production.

## The Old Ways

So far, my current setup involves:

- `jest` framework for automated tests
- `babel-jest`, the default Jest transformer, with a custom [Babel](https://babeljs.io) preset including [@babel/plugin-proposal-decorators](https://babeljs.io/docs/babel-plugin-proposal-decorators) & [babel-plugin-transform-typescript-metadata](https://github.com/leonardfactory/babel-plugin-transform-typescript-metadata)
- `tsc`, the TypeScript compiler to transpile the code for production for an ES2021 browser target, using `--experimentalDecorators` & `--emitDecoratorMetadata` flags

**Bonus:** leveraging the `importHelpers` `tsconfig.json` flag & [tslib](https://github.com/Microsoft/tslib) to optimize TypeScript compilation output with `tsc` & final JavaScript bundle size by reducing duplication of TypeScript runtime helpers - which in the case of **legacy decorators & type metadata**, quickly add up to hundreds of kilobytes of pure, unnecessary bloat.

The above works reasonably well, both in the context of automated testing (tens of developers) & production (hundreds of thousands of end-users).

But as you can see, there are two problems with this current setup:

1. The compilation is **slow enough** to:
   - be frustrating in many developer scenarios
   - incur significant infrastructure costs for building packages & applications at this scale
2. Slightly different transforms are used by different compilers in automated testing & production build:
   - potentially leading to subtle bugs
   - overcomplicating the overall setup

## Why not compile for production using Babel?

Yes. Using _Babel_ instead of _tsc_ for the production code compilation would improve the current situation in regard to the two problems mentioned above:

- Babel is slightly faster than tsc
- Babel & popular Babel plugins are often considered _the source of truth_ when it comes to compile transforms correctness

**But**, Babel's rich plugin ecosystem & fine-grained transforms being its most important strengths, they're also its biggest weakness in that, they make, by nature, Babel more difficult to correctly configure & maintain over time.

That is the main reason why I couldn't convince my peers to use Babel over tsc for production code compilation of a certain kind of library packages in early 2022.
As a result, production code compilation with Babel did not make the cut to [ts-package-base](https://github.com/ziir/ts-package-base), the first iteration of my TypeScript / Decorators / Metadata / React library package setup design with dual CJS/ESM compilation.

_Note:_ To this day, Babel remains my personal favourite JavaScript compiler.

## Times have changed, expectations too

As sad as it may be, _Babel_ & _tsc_ are not longer the JavaScript compilers of choice in modern projects.

The lines between a runtime, compiler, bundler & minimizer have been blurred by a new generation of tools and their combination, such as [esbuild](https://esbuild.github.io/), [SWC](https://swc.rs/), [tsup](https://tsup.egoist.dev/), [Vite](https://vite.dev) or even [Bun](https://bun.sh/) and more..

These next-gen tools are all powerered or super-charged, in one way or another, by the arcane power of compiled languages such as Go, Rust or Zig, offering build performance unparalleled in the JavaScript world.

Although they can, occasionally & to some degree, lack the maturity, correctness, ecosystem or functionality offered by their classic counterparts such as Webpack & Babel, there's also been a perception shift in the JavaScript community.
Whereas a few years ago, Webpack & Babel were _the tools we loved to hate_, next-gen tooling is **hype**, thanks to, in some cases, funding, marketing, dedicated conferences, polished communication & integration in popular / hyped web frameworks.

In conclusion: the classic tooling simply isn't good enough anymore, not necessarily in absolute, objective terms such as maturity, correctness, ecosystem or functionality, but, according to my own experience, in raw performance & developers perception.

_Note:_ The next-gen tools I'm mentioning here generally also have a remarkable focus on Developer Experience & sensible feature priorization according to popular use cases, which of course also contributes to their success.
There is also a multitude of other factors explaining the fall or rise in popularity of these tools, whether they be considered as classic or next-gen.

## Anyway

Let's get back to our [initial problem statement](#problem), after a quick re-introduction to **TypeScript Decorators & Metadata**.

### TypeScript Legacy / Experimental / Stage 2 Decorators (experimentalDecorators)

[TypeScript Legacy / Experimental / Stage 2 Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) are a natural evolution of TypeScript Classes.
Composed & applied to classes, class accessors, method & method parameters, they are commonly used for performing validation, transformation, augmentation, proxying or dependency injection, for which they're notoriously used with [Angular](https://angular.io/guide/dependency-injection), [Nest](https://docs.nestjs.com/fundamentals/custom-providers) & [TypeORM](https://typeorm.io/).

Since their introduction in TypeScript, **Legacy / Experimental / Stage 2 Decorators** support can be enabled through the `--experimentalDecorators` flag (`experimentalDecorators` in `tsconfig.json`).

_Note:_ see also:

- [current Stage 3 TC39 Decorators Proposal](https://github.com/tc39/proposal-decorators)
- [TypeScript 5 Decorators](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators)

### Decorators Type Metadata (emitDecoratorMetadata / Reflect.metadata / reflect-metadata)

The [Metadata API Proposal](https://rbuckton.github.io/reflect-metadata/) specifies how to inject metadata to classes & objects, primarily through decorators ("annotations"), as well as reading this metadata through [reflection](<https://en.wikipedia.org/wiki/Reflection_(computer_programming)>).

[TypeScript's Decorators Type Metadata](https://www.typescriptlang.org/docs/handbook/decorators.html#metadata) in particular, allows to inject **design-time** types:

- `design:type`
- `design:paramtypes`
- `design:returntype`

It relies on the [experimental metadata API](https://github.com/rbuckton/reflect-metadata), also know as `Reflect.metadata` & for which the `reflect-metadata` npm package provides a _shim_.

In TypeScript, experimental decorators type metadata support & transforms are available through the [`--emitDecoratorMetadata` flag, (`emitDecoratorMetadata` in `tsconfig.json`)](https://www.typescriptlang.org/tsconfig/emitDecoratorMetadata.html).

_Note:_ see also [detailed ECMAScript Metadata Proposal](https://rbuckton.github.io/reflect-metadata/).

> _TypeScript, putting the Java back in JavaScript_

## Vite?

Yes.

## Vitest?

Also yes.

## TypeScript Legacy Experimental Decorators with Type Metadata with Vite?

**No.**
Or at least, not by default.
**Vite** uses the [esbuild](https://esbuild.github.io/) transformer / compiler / bundler / minimizer by default, but unfortunately **esbuild does not support emitting decorators metadata**.
Therefore, we need to use a different JavaScript transformer / compiler.

## The best compiler for TypeScript Legacy Experimental Decorators with Type Metadata?

We've previously mentioned that **TypeScript / tsc** & Babel classic compilers support transforming both features, either behind flags, or through plugins, both of which are used in my current production setup.

Surely there must be a next-gen compiler which supports those features as well, right? right?

According to my research, [SWC](https://swc.rs/) is the only next-gen compiler supporting `legacyDecorator` & `emitDecoratorMetadata` at the moment.
Furthermore, SWC also has a `externalHelpers` option, similar to TypeScript's `importHelpers` / `tslib`, it allows re-using injected helper code by relying on helpers code from a `swc/helpers` package instead of inlining helpers, which is invaluable to limit bundle size impact.
But wait... it gets better!
Indeed, [@swc/helpers](https://www.npmjs.com/package/@swc/helpers) actually leverage's `tslib`'s own helpers for decorator & metadata helpers!

- [swc/packages/helpers/esm/\_ts_decorate.js](https://github.com/swc-project/swc/blob/main/packages/helpers/esm/_ts_decorate.js)
- [swc/packages/helpers/esm/\_ts_metadata.js](https://github.com/swc-project/swc/blob/main/packages/helpers/esm/_ts_metadata.js)

And that's absolutely splendid for us!
Since our production code is currently compiled with `typescript` / `tsc` + `tslib`, using **SWC** to compile our **TypeScript Decorators & Metadata** code is **guaranteed** to produce identical output!

However, it's important to note that the rest of the transforms incurred by TypeScript may be implemented differently by SWC and can still be subject to subtle bugs.

### Vite + SWC?

Multiple plugins exist for handing over transform responsability to **SWC in Vite**, to name a few:

- [@vitejs/plugin-react-swc](https://www.npmjs.com/package/@vitejs/plugin-react-swc)
- [unplugin-swc](https://www.npmjs.com/package/unplugin-swc)

They all seemed to work, as far as I could tell from my limited testing.
But diving at the implementation & documentation for each of them, it appeared they all covered much larger use cases (React Refresh, Rollup, minification ...) than my own, sometimes with significant overhead, complexity or opinionated API.

### vite-plugin-swc-transform

I'm probably already paying an overhead cost by using Vite instead of running SWC directly for reasons explained above and then some, and I wanted to use a **SWC plugin for Vite** with the most minimal & transparent API surface.

Therefore, I decided to implement my own simple **Vite plugin for transforming TypeScript Decorators + Metadata source files with SWC**.

See [vite-plugin-swc-transform on GitHub](https://github.com/ziir/vite-plugin-swc-transform).

Please note this plugin is not necessarily meant as a replacement of alternative plugins.
It is rather intended as a way to address my own use cases efficiently.

However, please feel free to open an issue on GitHub if you have ideas on how it could be improved, or additional use cases you'd like it to support.

## tsc vs Vite + SWC

For comparing **tsc** to **Vite + SWC**, we'll be focusing on a single case study matching my main current use case:
Building a typical library package for production as ES2021 target <abbr title="EcmaScript Modules">ESM</abbr>, composed of 124 TypeScript modules, a fair amount of which leverage TypeScript Legacy Decorators & Metadata, intended to run primarily in a browser after being bundled in a web project.

### Speed: build performance benchmark

The benchmark is running on an ~old Intel Macbook Pro.
Both builds output **ESM for an ES2021 target**.
The `tsconfig.json` file used for the `tsc` build is reasonably optimized for build performance with minimal type-checking.
Vite is configured to not attempt bundling nor minifying, and preserve the source directory structure in the output directory.

```
Benchmark 1: pnpm run build:esm:tsc
  Time (mean ± σ):      3.733 s ±  0.028 s    [User: 6.339 s, System: 0.429 s]
  Range (min … max):    3.696 s …  3.799 s    10 runs

Benchmark 2: pnpm run build:esm:vite-swc
  Time (mean ± σ):      1.347 s ±  0.031 s    [User: 1.650 s, System: 0.309 s]
  Range (min … max):    1.324 s …  1.432 s    10 runs

Summary
  pnpm run build:esm:vite-swc ran 2.77 ± 0.07 times faster than pnpm run build:esm:tsc
```

**Vite + SWC is ~2.77 times faster than `tsc` in this configuration.**

### Correctness: compiled JavaScript output review

When evaluating or switching JavaScript compilers, simply relying on an existing automated tests suite or manual testing is, in my experience, generally not enough.
It's important to also review the compiled outputs, look for differences & identify potential issues, opportunities for improvement.

```
$ wc -l esm-es2021-tsc-vs-vite-swc.pretty.patch
> 6826
```

#### SWC adds `.js` file extension to relative module imports

The first & omnipresent difference in compilation output between `tsc` & **Vite + SWC** is SWC appending the `.js` file extension to relative module imports.

Our example package is advertised as **ESM** with `type: 'module'` in its `package.json` file & the input source code is composed of `.ts` files which use **relative, extension-less imports**.

This library package uses `typescript@5.2`, is configured with `module: 'esnext'` + `moduleResolution: 'bundler'` compiler options in `tsconfig.json` and is primarily intended to be **consumed by a bundler**, for building a web app **running in the browser**.

Extract from the actual diff between **tsc & Vite + SWC outputs**:

```javascript/1/0
-import { isModuleImplementationDefinition } from '../dependency-injection/module.types';
+import { isModuleImplementationDefinition } from '../dependency-injection/module.types.js';
```

In our case, it's a safe change, and arguably a good one.

Indeed, bundlers & runtimes use **different module resolution algorithms** to be able to resolve actual modules from an import statement.
These algorithms can behave slightly differently from one to another, and by appending `.js` file extension to relative module imports, `SWC` clears the ambiguity of the possible files that should match a given compiled import.

This also results in a **faster & more efficient module resolution** process, because the algorithm doesn't have to check for files matching the import specifier, with multiple possible file extensions.

<hr />

##### About TypeScript module resolution, module formats & compilation targets

With `tsc`, **module specifiers are not transformed**.

If your project targets Node.js & compiles to ESM though, `module: 'node16'` (or `'nodenext`) & `moduleResolution: 'node16'` (or `nodenext`) is the only correct `tsconfig.json` combination matching **Node.js ES modules resolution algorithm**.
In which case, you will be required by the TypeScript compiler to **author relative module imports with their runtime specifier**, in other words, with their runtime extension, ie `.js` - but that doesn't apply if you use `moduleResolution: 'bundler'` (which is our case here) or compile to <abbr title="CommonJS">CJS</abbr>.

[Learn more about TypeScript Module Theory & Module Resolution](https://www.typescriptlang.org/docs/handbook/modules/theory.html#module-resolution).

<hr />

#### SWC & tsc set class properties differently: useDefineForClassFields

_Note: to be continued..._

## To be continued...

This blog post only covers one, although crucial, aspect of my current _quest_, quest which I hope will result in a succesful & impactful migration.
Follow-up blog posts should be added in the future.
Clarifications & corrections may be added as well if needed.

## Thanks for reading!

If you enjoyed this blog post, please feel free to share it and let me know on [Twitter](https://twitter.com/tpillard) or [Mastodon](https://hachyderm.io/@tpillard), don't hesitate to take a look at [my about page](/about/) as well.
