---
title: Limiting Asynchronous Operations Concurrency In JavaScript.
description: "Say one has a list of things, and for every of these things, one needs to perform an asynchronous operation that returns a Promise.
How does one limit the number of asynchronous operations being performed concurrently?
Without 3rd party dependencies and just a few lines of code?"
date: 2020-08-07
tags:
  - JavaScript
  - Node.js
  - concurrency
  - async
standalone: true
layout: layouts/post.njk
---

Today's post is about a little trick I have learned about 2years ago, and that I have since used numerous times in short Node.js scripts I had to write.
It really isn't much, but I guess it may be helpful to others, and probably is an alright first JavaScript post on this blog.

> _Say one has a list of things, and for every of these things, one needs to perform an asynchronous operation that returns a Promise, and be notified when all operations have been performed._ > _How does one limit the number of asynchronous operations being performed conccurently? Without 3rd party dependencies and just a few lines of code?_

Are you seeing the problem yet? Let's build some context:

### We have a list of 99 things, say resource identifiers

```javascript
const ids = Array.from(Array(99), (_, i) => i + 1);
> [1, 2, 3, ... 99]
```

### For each of these things, we need to perform an asynchronous operation

First, let's define the function that will perform said asynchronous operation:

```javascript
// We can pretend this performs an HTTP request
// an API call, or a database query.
function performAsyncOperation(id) {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(`Performed operation for resource ${id}.`);
      resolve();
    }, 300);
  });
}
```

Now, we know that we need to call this function of every single thing out of our list of 99 things, every resource id we have in `ids`.
Our first approach, would be to naively use [Array.prototype.map](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map) and [Promise.all](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) and call it a day. But let's have a closer look to this innocent line of code and its implications:

```javascript
await Promise.all(ids.map(performAsyncOperation));
console.log("All operations performed, moving on to something else now.");
```

_Note: we might arguably use [Promise.allSettled](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled) in a real world scenario, but let's assume our asynchronous operations cannot fail._

Which we would give us the following complete script:

```javascript
function performAsyncOperation(id) {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(`Performed operation for resource ${id}.`);
      resolve();
    }, 300);
  });
}

async function run() {
  const start = Date.now();
  const ids = Array.from(Array(99), (_, i) => i + 1);

  await Promise.all(ids.map(performAsyncOperation));
  const end = Date.now();
  console.log(
    `All operations performed in ${
      end - start
    }ms, moving on to something else now.`
  );
}

run();
```

Now, let's run it:

```
$ node async-operations-promise-all.js
```

```
[tim@praxis ~]$ node limit-async.js
Performed operation for resource 1.
Performed operation for resource 2.
Performed operation for resource 3.
...
Performed operation for resource 97.
Performed operation for resource 98.
Performed operation for resource 99.
All operations performed in 338ms, moving on to something else now.
```

As we can see, ~all the asynchronous operations were performed **concurrently**, and sometimes, that's perfectly fine and exactly what we want.

_Note: `Promise.all` only waits for all the **Promises** to be resolved. The actual asynchronous operation is triggered by the function call from `Array.prototype.map`, creating a new **Promise** on each call._

### The problem of performing operations concurrently

Now, let's assume that our `performAsyncOperation` really performs an HTTP request, an API Call, a database query, or really anything else that would either suffer or punish us for the load we push on to it. It might be a rate-limited API, or a fragile host.

_Note: in the actual case of an HTTP request, or database query, the Web/Node.js/etc... API being used will already probably limit concurrency, of which value may or may not configurable, but let's assume that we cannot or do not want to change such settings._

In such cases, we simply cannot allow our script to perform all these operations **concurrently**, as it risks getting our API key revoked, our IP blocked, or the target host/database responding unreliably, etc... Thus we will be searching for a way to limit **JavaScript concurrency** to avoid such scenario.

_Note: depending on the actual work performed by the asynchronous operation, performing many of these concurrently may also make the process running it consuming significant amount of CPU and/or memory resources on the machine it runs._

### A solution: performing asynchronous operations sequentially

The first solution that may come to mind, would be to perform each of these operations **sequentially**, on task at a time:

```javascript/5-7/4
async function run() {
  const start = Date.now();
  const ids = Array.from(Array(99), (_, i) => i + 1);

  - await Promise.all(ids.map(performAsyncOperation));
  + for (const id of ids) {
  +   await performAsyncOperation(id);
  + }
  const end = Date.now();
  console.log(`All operations performed in ${end - start}ms, moving on to something else now.`);
}
```

Which, if we run this script, gives the following output:

```
[tim@praxis ~]$ node limit-async.js
Performed operation for resource 1.
Performed operation for resource 2.
Performed operation for resource 3.
...
Performed operation for resource 97.
Performed operation for resource 98.
Performed operation for resource 99.
All operations performed in 29825ms, moving on to something else now.
```

As we'd expect, every operation awaits for the previous one to be completed, and, in a way, we found a succesful solution to our problem.
However, one would probably argue this being far from optimal, and that the system against which we run these asynchronous operations can safely withstand up to 3 concurrent operations, thus, our script could theoretically be ~3 times faster.

### The solution: limiting concurrency using workers

Now, let's take a look at how we can update our script to perform at most 3 operations concurrently.
For that, let's introduce a `worker` function, to which we will delegate the job of performing one asynchronous operation at a time.

```javascript
const worker = (next_) => async () => {
  let next;
  while ((next = next_())) {
    await performAsyncOperation(next);
  }
};
```

_Note: such a `worker` function can be implemented in various ways, this implementation is just an example of what I've used in the past._

However big the number of asynchronous operations we wish to perform, we will ever need at most, 3 concurrent `workers`.

```javascript
const CONCURRENT_WORKERS = 3;
```

We will then want to give a way to our `workers` to retrieve the next resource identifier, or thing, to perform the asynchronous operation from.

```javascript
const workers = [];
for (let i = 0; i < CONCURRENT_WORKERS; i++) {
  const w = worker(ids.pop.bind(ids));
  workers.push(w(ids));
}
```

Let us update our script:

```javascript/9-28/
function performAsyncOperation(id) {
  return new Promise((resolve) => {
    setTimeout(() => {
    console.log(`Performed operation for resource ${id}.`);
    resolve();
    }, 300);
  });
}

const worker = next_ => async () => {
  let next;
  while ((next = next_())) {
    await performAsyncOperation(next);
  }
};

const CONCURRENT_WORKERS = 3;

async function run() {
  const start = Date.now();
  const ids = Array.from(Array(99), (_, i) => i + 1);

  const workers = [];
  for (let i = 0; i < CONCURRENT_WORKERS; i++) {
    const w = worker(ids.pop.bind(ids));
    workers.push(w(ids));
  }

  await Promise.all(workers);

  const end = Date.now();
  console.log(`All operations performed in ${end - start}ms, moving on to something else now.`);
}

run();
```

And run it:

```
[tim@praxis ~]$ node limit-async.js
Performed operation for resource 1.
Performed operation for resource 2.
Performed operation for resource 3.
...
Performed operation for resource 97.
Performed operation for resource 98.
Performed operation for resource 99.
All operations performed in 9948ms, moving on to something else now.
```

### Bonus solution: batching concurrent operations

There's one more way to look at the problem, and reach another solution: splitting the asynchronous operations to be performed in **batches**.
In our case, if we have 99 operations to perform, we can split these into 33 batches of 3 operations.
With batching, every operation within one batch will be performed conccurently, and the next batch will only be started once the previous one is finished.
How does one implement concurrent tasks batching with JavaScript?

```javascript/5-10/4
async function run() {
  const start = Date.now();
  const ids = Array.from(Array(99), (_, i) => i + 1);

  - await Promise.all(ids.map(performAsyncOperation));
  + let i = 1;
  + while (ids.length) {
  +   await Promise.all(ids.splice(0, 3).map(performAsyncOperation);
  +   console.log('Performed async operactions batch number', i);
  +   i++;
  + }
  const end = Date.now();
  console.log(`All operations performed in ${end - start}ms, moving on to something else now.`);
}
```

Which gives us the following output:

```
[tim@praxis ~]$ node limit-async.js
Performed operation for resource 1.
Performed operation for resource 2.
Performed operation for resource 3.
Performed async operactions batch number 1
...
Performed operation for resource 97.
Performed operation for resource 98.
Performed operation for resource 99.
Performed async operactions batch number 33
All operations performed in 9981ms, moving on to something else now.
```

### Conclusion

Asynchronous JavaScript can be tricky, and though one may argue the language lacks high-level APIs to deal more efficiently with some of these tricky cases like the one we've just covered, it's always worth taking our chance at solving the problems leveraging the features that we're provided by the language.

In this post we've built and lived through a common scenario of managing asynchronous operations concurrency in JavaScript, along with a common pitfall associated with a naive approach, as well as an intermediary solution that is also very useful on some other cases, and finally the optimal solution for dealing with this problem, all without using third party modules. I hope this may be useful to some of you, and that I did not take too many shortcuts in the process.

If you liked this post, feel free to encourage me by saying so on [my Twitter](https://twitter.com/tpillard) and/or by liking/retweeting the associated tweet or sharing the article around you.
Finally, feel free to share your feedback if you have ideas or know how I can improve this post or this blog, thank you :)

_Note: special thanks to [baloo](https://twitter.com/baloose) for teaching me this trick while I was hacking together a script purposed to pre-fill HTTP cache server entries for a given website from its sitemap file._
