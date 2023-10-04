---
title: React memo is good actually
description: React memo, useMemo & useCallback can be controversial. Let us review the most common misconceptions about them, and see how they're good, actually.
image:
  path: /img/always-has-been-meme.webp
  width: 480
  alt: "Wait React meme is good actually? Always has been"
date: 2023-10-06
tags:
  - react
  - memo
  - performance
  - JavaScript
standalone: true
layout: layouts/post.njk
---

<img src="/img/always-has-been-meme.webp" alt="Wait React meme is good actually? Always has been" width="480" />

## Disclaimer

This blog post expresses my **personal views** on a **controversial, complex & ever evolving topic**, based on my personal experience.
Your mileage may vary.

## memo, useMemo, useCallback for performance (& logic correctness)

[React.memo](https://react.dev/reference/react/memo), [React.useMemo](https://react.dev/reference/react/useMemo) & [React.useCallback](https://react.dev/reference/react/useCallback), respectively a <abbr title="Higher Order Component">HOC</abbr> & performance hooks, are official APIs provided by the React library ~since React@16 (2018), commonly yet hesitantly used in React applications.
They are notably used for improving application runtime & **rendering performance**, and regarding **React.useMemo & React.useCallback hooks**, also to **ensure stability of non-primitive JavaScript values**, when such values are used as effect hooks, ie [React.useEffect](https://react.dev/reference/react/useEffect) dependencies.
In which case, they also contribute to application logic correctness.

_Note: if you're not familiar with these APIs, I can't recommend enough to read the excellent [React API Reference](https://react.dev/reference/react)._

## Great APIs, bad reputation, strong positions

Since these they were introduced (and even before with **React.PureComponent** & **React.Component.shouldComponentUpdate**), **React performance APIs** have, in my opinion, suffered time and again from bad press: countless incorrect, oriented, biased, dishonest articles, medium posts, tweets, code review comments, opinions & rants, describing the APIs as “_counter-productive, unnecessary, bad, evil_” or even “_dangerous_”.
All with varying degrees of over-simplification, absolutes, subscribe buttons, links to YouTube channels, paid courses & workshops.

Thankfully, most of this content is old now and it's not brought up nearly as often as it used to a few years ago.
Unfortunately, they still rank very well on search engines and regularly make their way into colorful engagement-bait info-graphics on X dot com.

Of course, some actually great, nuanced & technically correct content exists covering in-depth **React Rendering Behavior** or alternative _ways to improve React rendering performance_ - it's not necessarily the easiest to find, but not too difficult either.

But the problem is, bad press & demonization of APIs: it sticks.
Over the years (since what, 2017, 2018?) I have seen open source maintainers, tech leads, engineers, Senior, Staff+ alike, take very strong positions pushing back against the use of React performance APIs at work or in public, all in the name of “_readability, anti-fragility, performance, simplicity, convenience, consistency, dogma_”, and more.

All this with an outstanding immediate effect of leading to heated debates in code reviews, stalling pull requests, gatekeeping, drama, frustration, and too often as a result: a halt to efforts aiming to **improve rendering performance** for the users.

## Most Common Misconceptions

Let us review some of the most common misconceptions I've come across over the years, which, I believe, are both a consequence & a catalyst of React performance APIs bad reputation & related difficulties mentioned previously.

### 1. “React is so fast anyway, React.memo is unnecessary”

**INCORRECT**

Although yes, React is fast enough in most cases, there's a reason these React performance APIs exist in the first place: they fill in a gap in React's model and a gap in the ability of teams to maintain a code base in par with state of the art practices over time.

Furthermore, the actual rendering performance of a client-side web application or <abbr title="Single Page Application">SPA</abbr> is dependent on many factors beyond the underlying view rendering library being “fast”.
These factors include the state of the code base, dependencies, use cases & functionalities, as well as external factors like the user's device or web browser.

Accordingly, rendering performance issues in React projects are extremely common, and can most of the time be mitigated or fully resolved with a combination of best practices, patterns, refactoring, re-architecture and tools, including **React.memo**.

### 2. “React.memo is counter-productive”

_Also known as **“React.memo's props shallow-equality check's overhead is detrimental to performance**”._

**INCORRECT**

React memoization in general & React.memo specifically can be counter-productive if it's not used correctly, ie: when used on a component that more often than not receives different props between renders, which will render memoization ineffective, while still incurring the cost of comparing previous memoized props with new props, (by default using a shallow-equal comparison).

In other cases where React.memo is used correctly, and some re-renders of the memoized components are legitimate because their props are legitimately different across renders, then yes, `React.memo` incurs an unnecessary overhead cost, mostly through its `compare` function - [React's shallow equal comparison](https://github.com/facebook/react/blob/85c2b519b54269811002d26f4f711809ef68f123/packages/shared/shallowEqual.js#L18).

In any case, in real-life applications, **React's shallow comparison's cost** is arguably negligible & highly unlikely to be more expensive than a prevented re-render of any non-trivial component.

### 3. “React.memo doesn't work with React element / children props”

**INCORRECT**

Well, not _entirely_ incorrect, since React elements are commonly created using JSX, which is transformed to `React.createElement(...)` calls (or equivalent) , always returning new object, then yes, naive passing of React elements as props to a memoized component will render its memoization ineffective.

However, it won't be the case if you pass **stable / constant React elements** - ie defined outside of the parent function component's scope or memoized with `React.useMemo`.

The former can be done automatically where applicable using [babel-plugin-transform-react-constant-element](https://babeljs.io/docs/babel-plugin-transform-react-constant-elements.html) or equivalent compiler transforms - although when dealing with stable props to be passed to a `React.memo`'d component, it's best to be explicit than to rely on an implicit compiler transform.

**Bonus**: the re-render of a **stable** / **constant** React element will be skipped by React, regardless of it being memoized with React.memo or not.
This React behavior is also known as **"same reference element optimization"**.

For instance, the following code renders the memoization of `<MemoizedLayoutComponent>` inneffective:

```jsx
function App() {
  return (
    <MemoizedLayoutComponent>
      <SomeUIComponent />
    </MemoizedLayoutComponent>
  );
}
```

But the following - whether done manually as in this example, or automatically via `babel-plugin-transform-react-constant-element` - allows `<MemoizedLayoutComponent>`'s memoization to work as expected, and ensures our `<SomeUIComponent />` child component to never be re-rendered (other than through an update of its own state).

```jsx/0
const someUIElement = <SomeUIComponent />;
function App() {
  return <MemoizedComponent>{someUIElement}</MemoizedComponent>;
}
```

_Note:_ The following variant, using `React.useMemo` on `<SomeUIComponent />` when dealing with some dynamic props, although less recommendable, also works - both `<SomeUIComponent>` & `<MemoizedComponent>` will be re-rendered if `dynamicProp` changes (assuming a state update), but neither will ever be re-rendered otherwise:

```jsx/2-5
function App() {
  const dynamicProp = useDynamicProp();
  const someUIElement = React.useMemo(
    <SomeUIComponent staticProp="static" dynamicProp={dynamicProp} />,
    [dynamicProp]
  );
  return <MemoizedComponent>{someUIElement}</MemoizedComponent>;
}
```

### 4. “React.memo is too fragile”

**DISAGREE**

React.memo does have a weakness: in order to be effective, it requires the consumer of a memoized React component, to be aware that said React component has been wrapped by the `React.memo` <abbr title="Higher Order Component">HOC</abbr> and thus, that its props are expected to be stable for component memoization to be effective.

Some argue that consumers of a React component shouldn't be aware of how said component is defined or exported, that it's an implementation detail.
**I disagree.**

I believe, like any function, any API you didn't write yourself, you should refer to the documentation, JSDoc definition, better yet, also refer to the source of truth: the source code, tests, and even in more cases that I'd like, compiler configuration & output!
What if the underlying component uses some of the provided props as `React.useEffect()` dependencies?

Regardless, this weakness can be addressed in multiple ways, such as **adopting & documenting a naming convention** within your team, **indicating the memoized nature** of the exported component, ie:

```jsx
export const MemoizedList = React.memo(List);
```

Another naming convention could be to convey the necessity for specific non-primitive value props to be referentially stable as much as possible:

```jsx
const { data = [] } = useData(queryKey);
const stableData = React.useMemo(data, []);
return <List stableData={stableData} />;
```

_Note: this example relies on React.useMemo with an empty dependency array just for demonstration purposes._

Alternatively, you can to **shift the responsability** of wrapping the component to memoize with `React.memo` by doing it, not in the file where the component is exported, but it in the file where the component is used:

```jsx
import React from "react";
import { List as RawList } from "./List";

const MemoizedList = React.memo(List);

function App() {
  const { data } = useStableData();
  return <MemoizedList data={data} />;
}
```

Additionally, [`React.memo()` being a HOC](https://github.com/facebook/react/blob/85c2b519b54269811002d26f4f711809ef68f123/packages/react/src/ReactMemo.js#L14), you _could_ create your own wrapper around it, so it can be leveraged in a **declarative way**.

Something along those lines for example:

```jsx
const { items } = useItems();
return <MyMemo component={List} items={items} />;
```

_Note:_ I wouldn't recommend this, but it's possible.

Finally, **alternatives to React.memo** can be just as much fragile, in slightly different & subtle ways.
Arguably, React.memo & React.useMemo can be helpful if you want to render your code more robust.

Custom React Context Provider components for instance, are often used to provide both a getter & a setter to consumer components, encapsulating specific domain specific logic, and used in the application's entry-point component.
We can carefuly write our Custom React `Context.Provider` component in a way that doesn't make it necessary to use `React.memo` on the `<Main />` component, assuming it renders an expensive subtree, and we can also avoid wrapping the provided value in `React.useMemo()`.

```jsx
const ThemeContext = React.createContext();

function ThemeContextProvider({ children }) {
  const [theme, setTheme] = useState("dark");
  const value = { theme, setTheme };
  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
}

export function App() {
  return (
    <ThemeContextProvider>
      <Main />
    </ThemeContextProvider>
  );
}
```

_Note:_ I would probably do this too in most cases.

However, additional wrappers around or replacing an application's entry-point component over time are a reality, and any future changes in the code base introducing additional state updates in or above `<App />` will render these careful efforts in avoiding `React.memo` & `React.useMemo` counter-productive.

Indeed, both `<Main />`, with its expensive subtree & `<ThemeContextProvider>` will re-render, a new `value` object created and then all `React.useContext(ThemeContext)` consumers will be re-rendered, even if the `theme` value hasn't changed.
Although we could split our `<ThemeContextProvider>` into a `<ThemeGetterContextProvider>` & `<ThemeSetterContextProvider>` to address this issue, we would lose the consolidation of our theme handling domain logic in a single component, introduce more friction in the code base & have to go through a refactor of all context consumers, potentially introducing risks.

Whereas we _could_ have ensured the _robustness_ of both our `<App />` component, our `ThemeContextProvider` component & consumers from the start with `React.memo` & `React.useMemo`.
For the sake of demonstration:

```jsx
const ThemeContext = React.createContext();

function ThemeContextProvider({ children }) {
  const [theme, setTheme] = useState("dark");
  const value = React.useMemo({ theme, setTheme }, [theme]);
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

const MemoizedMain = React.memo(Main);
export function App() {
  return (
    <ThemeContextProvider>
      <MemoizedMain>
    </ThemeContextProvider>
  )
}
```

_Note:_ I would do this in some cases.

Or more explicitly:

```jsx
const ThemeContext = React.createContext();

function ThemeContextProvider({ children }) {
  const [theme, setTheme] = useState("dark");

  // Although currently this component is rendered exactly once,
  // we memoize the context value preemptively to ensure it will never cause
  // unnecessary re-renders of its consumers, ie if this component becomes
  // subject to updates from its parent component, such as if it's moved closer to its consumers
  const value = React.useMemo({ theme, setTheme }, [theme]);
  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
}

export const MemoizedApp = React.memo(
  function App() {
    return (
      <ThemeContextProvider>
        <Main />
      </ThemeContextProvider>
    );
  },
  /**
   * The App component is our application entry-point.
   * and should only be rendered exactly once.
   **/
  (/* prevProps, newProps */) => {
    if (__DEV__) throw new Error("Unexpected re-render of <App /> entry-point");
    return true;
  }
);
```

_Note:_: I would only do this in specific cases.

### 5. “React.memo / React.useMemo / React.useCallback are signs of poor architecture”

**INCORRECT**

Occasionally, maybe, it depends.
See [6. “There are better alternatives to React.memo / React.useMemo / React.useCallback”](#6.-“there-are-better-alternatives-to-react.memo-%2F-react.usememo-%2F-react.usecallback”).
But my point is, even when the use of these APIs make possible improvements apparent - that may or may not allow not to use these APIs in the first place, **the use of React Performance APIs in itself, is perfectly valid**, at least pending the refactoring of the relevant code, which may or may not happen in the future, depending on a multitude of factors: complexity, resources, product pressure, etc.

### 6. “There are better alternatives to React.memo / React.useMemo / React.useCallback”

**SOMETIMES**

Known best practices, patterns such as **keeping component state as local as possible** & **lifting content up**, tools such as state containers, etc. can often have an outstanding effect on rendering performance and render some React.memo uses unnecessary.

Effects with `React.useEffect` can sometimes be written in a way that doesn't rely on non-primitive value dependencies, thus removing the need for `React.useMemo` or `React.useCallback`.

But in a lot of real world situations, these aren't enough, nor even the best choice depending on the project's context: for example, more often than it seems, using these APIs and opting into “inferior” patterns is a better trade-off than the re-architecture of an application, rewriting a lot existing of code.

### 7. “React.memo / React.useMemo / React.useCallback are detrimental to readability & maintainability”

**DISAGREE**

If they're misused? Sure - and I'm all for enforcing either using them correctly or not using them at all.

But as soon as these APIs are used correctly, I find they actually improve one's understanding of the code upon reading, and practically convey or highlight important information.

When I see `React.memo` used in a code base that otherwise doesn't use it much, I don't read:

> _“This component is optimized.”_

I read:

> _“The components subtree in which this component is rendered, is maybe in the critical path, probably subject to unnecessary & potentially expensive re-renders relative to the other components in the code base.”_

When I see `React.memo` used in a lot of places, I read:

> _“It's our philosophy that every component should be as efficient as possible.”_

Similarly, when I see `React.[useCallback/memo]`, I read:

> _“It's important for this value to adhere to immutability principles because it's computationally heavy or because we need to keep a stable reference, according to the following dependencies.
> It's likely to be used by effects or other performance APIs.”_

### 8. “But \<authority figure\> said not to use these APIs! (or similar)”

**INCORRECT**

First of all they probably didn't say or mean that litterally.
Or maybe they did, but regardless - and however appreciative we can be for their contributions, recommendations, etc:

- they don't write the code: we do.
- they're not the ones who pay the price of bugs & inaction: our users do, and ultimately, we do too.
- it's up to us, not them.

### 9. “Premature optimization is the root of all evil”

**DISAGREE**

What is a _premature optimization_?
When does an _optimization_ cease to be _premature_?
What _evil_ are we talking about here?

More seriously, [this quote](https://www.cs.sjsu.edu/~mak/archive/CS185C/KnuthStructuredProgrammingGoTo.pdf), albeit super catchy & effective, is always dropped outside of its original context, which ironically, relates a **very strong pro-optimization position**.

> _”The conventional wisdom shared by many of today's software engineers calls for ignoring efficiency in the small; but I believe this is simply an overreaction to the abuses they see being practiced by penny-wise-and-pound-foolish programmers.”_ > [Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)

I find it's often used as a lazy, vague, catch-all way to undermine someone's efforts or dismiss potential improvements, without much more justification - which, I'm sure you'll agree, isn't very nice or respectful of said someone's work.
In any case, I find it subjective, and would rather discuss the impact, the trade-offs or questions left to be answered, regarding the idea being challenged, in a more specific way.

Moreover, looking back at the previous points, one could absolutely use this quote in a similarily disinguenous way to argue in favor of using these APIs over the potentially more complex, consequential, time consuming & risky alternatives, when any.

_Note_: This quote is also used in a more seasoned fashion, as an invitation to perform measurements, use profiling, to identify hot code paths in order to guide & validate performance improvement efforts, which in itself is perfectly recommendable.

## React memo, useMemo & useCallback are good, actually

React performance APIs have some weaknesses, yes, most of which inherent to their nature, filling in the gaps in React's rendering model & allowing teams to mitigate issues of their own making.
However, incorrect uses of React performance APIs, in my experience, are mostly due to minor mistakes and inconsequential.
On the other hand, strategic & proper use of these APIs can have an outstanding beneficial impact, on rendering performance of course, but also more than that: in expressing intent, surfacing key components, highlighting opportunities, enabling use cases that would otherwise require introducing a tremendous amount complexity or additional work, shipping now what would otherwise require days.
All of which, in my book, makes **React performance APIs exceptionally good, actually**.

### Demo: Let's build a notes app highlighting the value React performance APIs

_To be continued...:_ the idea would be to build, from scratch & step by step, a simple (fake) notes app, and during the process, highlighting the value of React performance APIs versus other possible alternatives, when any.

## I'm not trying to convince you

I consider that the uphill battle [~~of memoization~~](https://tkdodo.eu/blog/the-uphill-battle-of-memoization) for the just & pragmatic recognition of the React performance APIs has long been lost.
I'm primarily writing this blog post for myself, and to serve as an educational reference and clarification of my views in case of future, alas inevitable, push-back & debates regarding the use of these APIs.
I wish the React community generally wasn't so dogmatic, but regardless, for as long as I will even remotely be in contact with React code & React developers, I know I will continue to actively push for writing React code that's as efficient as reasonably possible, not just because I'm a performance enthusiast and it's fun, but because it matters.

## To be continued...

There's obviously much more to say on React Performance APIs and other gravitating topics.
Further content (including the upcoming [demo](#demo%3A-let's-build-a-notes-app-highlighting-the-value-react-performance-apis)) may be added in the future, clarifications & corrections as well if needed.

## Prior art: [parisweb.app (2018)](https://parisweb.app)

Demo apps for the React/Redux performance workshop [“Petits trucs pour rendre vos applications React plus performantes”](https://www.paris-web.fr/2018/ateliers/petits-trucs-pour-rendre-vos-applications-react-plus-performantes.php) performed with [Julien Wajsberg](https://twitter.com/jwajsberg) at a meetup hosted by [Gandi](https://www.gandi.net) and then at [Paris Web](https://www.paris-web.fr) in 2018.
The materials produced at this occasion (slides, code, examples) are mostly outdated, yet still somewhat relevant resources to understand and improve performance of React apps, with or without Redux, in 2023.

## Thanks for reading!

If you enjoyed this blog post, please feel free to share it and let me know on [Twitter](https://twitter.com/tpillard) or [Mastodon](https://hachyderm.io/@tpillard), don't hesitate to take a look at [my about page](/about/) as well.

## Recommended further reading

- [A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior), by [Redux maintainer Mark "acemarke" Erikson](https://twitter.com/acemarke).
