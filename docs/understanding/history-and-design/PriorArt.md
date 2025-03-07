---
id: prior-art
title: Prior Art
description: 'Introduction > Prior Art: Influences on the design of Redux'
---

# Prior Art

Redux has a mixed heritage. It is similar to some patterns and technologies, but is also different from them in important ways. We'll explore some of the similarities and the differences below.

## Developer Experience

Dan Abramov (author of Redux) wrote Redux while working on his React Europe talk called [“Hot Reloading with Time Travel”](https://www.youtube.com/watch?v=xsSnOQynTHs). His goal was to create a state management library with a minimal API but completely predictable behavior. Redux makes it possible to implement logging, hot reloading, time travel, universal apps, record and replay, without any buy-in from the developer.

Dan talked about some of his intent and approach in [Changelog episode 187](https://changelog.com/187).

## Influences

Redux evolves the ideas of [Flux](https://facebook.github.io/flux/), but avoids its complexity by taking cues from [Elm](https://github.com/evancz/elm-architecture-tutorial/).
Even if you haven't used Flux or Elm, Redux only takes a few minutes to get started with.

### Flux

Redux was inspired by several important qualities of [Flux](https://facebook.github.io/flux/). Like Flux, Redux prescribes that you concentrate your model update logic in a certain layer of your application (“stores” in Flux, “reducers” in Redux). Instead of letting the application code directly mutate the data, both tell you to describe every mutation as a plain object called an “action”.

Unlike Flux, **Redux does not have the concept of a Dispatcher**. This is because it relies on pure functions instead of event emitters, and pure functions are easy to compose and don't need an additional entity managing them. Depending on how you view Flux, you may see this as either a deviation or an implementation detail. Flux has often been [described as `(state, action) => state`](https://speakerdeck.com/jmorrell/jsconf-uy-flux-those-who-forget-the-past-dot-dot-dot-1). In this sense, Redux is true to the Flux architecture, but makes it simpler thanks to pure functions.

Another important difference from Flux is that **Redux assumes you never mutate your data**. You can use plain objects and arrays for your state just fine, but mutating them inside the reducers is strongly discouraged. You should always return a new object, which can be done using the object spread operator or [the Immer immutable update library](https://immerjs.github.io/immer/).

While it is technically _possible_ to [write impure reducers](https://github.com/reduxjs/redux/issues/328#issuecomment-125035516) that mutate the data for performance corner cases, we actively discourage you from doing this. Development features like time travel, record/replay, or hot reloading will break. Moreover it doesn't seem like immutability poses performance problems in most real apps, because, as [Om](https://github.com/omcljs/om) demonstrates, even if you lose out on object allocation, you still win by avoiding expensive re-renders and re-calculations, as you know exactly what changed thanks to reducer purity.

For what it's worth, Flux's creators [approve](https://twitter.com/jingc/status/616608251463909376) of [Redux](https://twitter.com/fisherwebdev/status/616286955693682688).

### Elm

[Elm](https://elm-lang.org/) is a functional programming language inspired by Haskell and created by [Evan Czaplicki](https://twitter.com/evancz). It enforces [a “model view update” architecture](https://github.com/evancz/elm-architecture-tutorial/), where the update has the following signature: `(action, state) => state`. Elm “updaters” serve the same purpose as reducers in Redux.

Unlike Redux, Elm is a language, so it is able to benefit from many things like enforced purity, static typing, out of the box immutability, and pattern matching (using the `case` expression). Even if you don't plan to use Elm, you should read about the Elm architecture, and play with it. There is an interesting [JavaScript library playground implementing similar ideas](https://github.com/paldepind/noname-functional-frontend-framework). We should look there for inspiration on Redux! One way that we can get closer to the static typing of Elm is by [using a gradual typing solution like Flow](https://github.com/reduxjs/redux/issues/290).

### Immutable

[Immutable](https://facebook.github.io/immutable-js) is a JavaScript library implementing persistent data structures. It is performant and has an idiomatic JavaScript API.

(Note that while Immutable.js helped inspire Redux, today we recommend [using Immer for immutable updates instead](../../style-guide/style-guide.md#use-immer-for-writing-immutable-updates).)

**Redux doesn't care _how_ you store the state—it can be a plain object, an Immutable object, or anything else.** You'll probably want a (de)serialization mechanism for writing universal apps and hydrating their state from the server, but other than that, you can use any data storage library _as long as it supports immutability_. For example, it doesn't make sense to use Backbone for Redux state, because Backbone models are mutable.

Note that, even if your immutable library supports cursors, you shouldn't use them in a Redux app. The whole state tree should be considered read-only, and you should use Redux for updating the state, and subscribing to the updates. Therefore writing via cursor doesn't make sense for Redux. **If your only use case for cursors is decoupling the state tree from the UI tree and gradually refining the cursors, you should look at selectors instead.** Selectors are composable getter functions. See [reselect](https://github.com/faassen/reselect) for a really great and concise implementation of composable selectors.

### Baobab

[Baobab](https://github.com/Yomguithereal/baobab) is another popular library implementing immutable API for updating plain JavaScript objects. While you can use it with Redux, there is little benefit in using them together.

Most of the functionality Baobab provides is related to updating the data with cursors, but Redux enforces that the only way to update the data is to dispatch an action. Therefore they solve the same problem differently, and don't complement each other.

Unlike Immutable, Baobab doesn't yet implement any special efficient data structures under the hood, so you don't really win anything from using it together with Redux. It's easier to just use plain objects in this case.

### RxJS

[RxJS](https://github.com/ReactiveX/RxJS) is a superb way to manage the complexity of asynchronous apps. In fact [there is an effort to create a library that models human-computer interaction as interdependent observables](https://cycle.js.org).

Does it make sense to use Redux together with RxJS? Sure! They work great together. For example, it is easy to expose a Redux store as an observable:

```js
function toObservable(store) {
  return {
    subscribe({ next }) {
      const unsubscribe = store.subscribe(() => next(store.getState()))
      next(store.getState())
      return { unsubscribe }
    }
  }
}
```

Similarly, you can compose different asynchronous streams to turn them into actions before feeding them to `store.dispatch()`.

The question is: do you really need Redux if you already use Rx? Maybe not. It's not hard to [re-implement Redux in Rx](https://github.com/jas-chen/rx-redux). Some say it's a two-liner using Rx `.scan()` method. It may very well be!

If you're in doubt, check out the Redux source code (there isn't much going on there), as well as its ecosystem (for example, [the developer tools](https://github.com/reduxjs/redux-devtools)). If you don't care too much about it and want to go with the reactive data flow all the way, you might want to explore something like [Cycle](https://cycle.js.org) instead, or even combine it with Redux. Let us know how it goes!

## Testimonials

> [“Love what you're doing with Redux”](https://twitter.com/jingc/status/616608251463909376)
> Jing Chen, creator of Flux

> [“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)
> Bill Fisher, author of Flux documentation

> [“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)
> André Staltz, creator of Cycle

## Thanks

- [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) for a great intro to modeling state updates with reducers;
- [Turning the database inside-out](https://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) for blowing my mind;
- [Developing ClojureScript with Figwheel](https://www.youtube.com/watch?v=j-kj2qwJa_E) for convincing me that re-evaluation should “just work”;
- [Webpack](https://webpack.js.org/concepts/hot-module-replacement/) for Hot Module Replacement;
- [Flummox](https://github.com/acdlite/flummox) for teaching me to approach Flux without boilerplate or singletons;
- [disto](https://github.com/threepointone/disto) for a proof of concept of hot reloadable Stores;
- [NuclearJS](https://github.com/optimizely/nuclear-js) for proving this architecture can be performant;
- [Om](https://github.com/omcljs/om) for popularizing the idea of a single state atom;
- [Cycle](https://github.com/cyclejs/cycle-core) for showing how often a function is the best tool;
- [React](https://github.com/facebook/react) for the pragmatic innovation.

Special thanks to [Jamie Paton](https://jdpaton.github.io) for handing over the `redux` NPM package name.

## Patrons

The original work on Redux was [funded by the community](https://www.patreon.com/reactdx).
Meet some of the outstanding companies that made it possible:

- [Webflow](https://github.com/webflow)
- [Ximedes](https://www.ximedes.com/)

[See the full list of original Redux patrons](https://github.com/reduxjs/redux/blob/master/PATRONS.md), as well as the always-growing list of [people and companies that use Redux](https://github.com/reduxjs/redux/issues/310).
