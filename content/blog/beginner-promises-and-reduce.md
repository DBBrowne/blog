---
title: "A beginners look at Promises and Reduce"
description: "Async functions return a promise.  Reduce's first parameter is ***previous***, not *accumulator*. Promise._forEach is interesting."
date: "2022-02-28"
timezone: "Europe/London"
utterances_term: "A beginners look at Promises and Reduce"
categories: [JavaScript, Promise, Reduce]
---
# A beginners look at Promises and Reduce

> Async functions return a promise.  Reduce's first parameter is ***previous***, not *accumulator*.

With thanks to [AJ O'Neil](https://github.com/coolaj86), for his infinite patience with junior engineers.

<hr>

## TLDR
### The async keyword causes a function to return a Promise.  No further returns needed.
### The first argument passed to the reducer function in to Array.Reduce(reducerFn, initial) is *previous*, not *accumulator*.

<hr>


## Parallel Promises
We've all looked at JS's `await`s and thought; "well, those are off waiting outside the main stack, so why not fire off several at once?"

`Promise.all()` is there for us, great for the right situations, but is probably a good route to network, disk, and/or CPU thrashing if you don't have infinite throughput and processing power.
And it assumes that our awaits are all independent.

## Promise Chains

What if we want to take a set of async actions, one after another?

Enter a neat implementation of a method to run async tasks in sequence, care of [AjScript](https://github.com/coolaj86/AJScript/issues/10)

```js
Promise._forEach = async function (arr, fn) {
    await arr.reduce(async function (promise, el, i) {
        await promise;
        await fn(el, i, arr);
    }, Promise.resolve());
};
```

My Junior engineer's mind got rather stuck here though:

 - What does the Promise.resolve() in the initial do?
 - How on earth does the promise from each iteration get fed into Promise.resolve()?
 - Nothing is returned from the Reduce function.  How does this do anything other than cause another one of those bugs where I've forgotten to return inside an array method, and nothing happens.

After a lot of digging, and a lot of advice from [AJ himself](https://github.com/coolaj86) and his [youtube channel](https://www.youtube.com/channel/UC2KJHARTj6KRpKzLU1sVxBA), I finally worked it out.  Hopefully this is useful for other juniors!  

My attempts to explore this knowledge gap are all up here:
https://github.com/DBBrowne/code-challenges-public/blob/main/other/promise.foreach.js

<br><br>
The key lessons come from:

#### Question: 
##### How does this pass back to the `acc`? Why does this do anything other than resolve the initial `Promise.protoype.resolve()`, and then sit quietly?
#### Answer:
The function used as a reducer is async. By definition this returns a promise, which becomes the accumulator (better thought of as `previous` here) in the reducer.  
In fact, ***THERE IS NO ACCUMULATOR IN REDUCE***.  

The definition is `Array.prototype.reduce(function reducer (prev, current, index, array){}, initialValueForPrev)`.

Sometimes we `return prev + current` from the reducer to make an accumulator, but that's an implementation.  
I couldn't get over the lack of a `return` inside `reducer`, and could not get how the promise from  `await fn(el)` was getting into `prev`.  
It doesn't.  A different promise is returned from the reducer function into the `prev` (the one from the reducer function), that promise is waiting for the `fn(el)` promise to resolve.

<br>  

#### Question:
##### How does each iteration get passed into the initial Promise.resolve()?
#### Answer:
It doesn't.  

Reduce's default behaviour, with no second argument, is to use the first element of the array as the initial value.  
In this case, that would mean that the first `await promise` would `await arr[0]`, rather than `await fn(arr[0])`.  `fn(arr[0])` would never run.  
The `initial` argument is only there to prevent Reduce using the first value in the array as the initial, and thereby failing to evaluate the passed `fn` for that value.  The initial value could be anything other than `not defined`, including null or undefined.  

Promise.resolve() is chosen as the initial to indicate to the author that 'prev' is a promise, and to ensure that the type of `prev` does not change between the first and subsequent iterations.
