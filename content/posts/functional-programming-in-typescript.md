---
title: "Functional Programming in Typescript"
date: 2020-08-24T21:37:53+01:00
draft: false
---

## The benefits of functional programming

The industry as a whole from what I've seen has been shifting in the last few years, with developers looking to take some ideas from functional programming to aid them in their day to day job of writing software. Writing software isn't an easy task, a lot of applications have to deal with ever-changing internal state, complex business logic and you're often under constant pressure to deliver new features at a blistering pace; and these are all areas where functional programming can help (shocker - I wouldn't be writing this post if I didn't believe that).

## What is functional programming?

Functional programming is a way of programming in which we try to keep some core ideas in mind whilst we write code.

### Purity

Functional programmers talk about purity a lot, and its the core idea behind functional programming. We usually talk about writing "pure functions" which are functions that when given some input X always return some value Y. That is to say, the output of the function is deterministic - if you give in one value, you'll always get the same value out for that value. Also, to be pure, a function can not modify or rely upon state outside of the function - this would break purity, you never know what will happen to state outside of the function, anyone can change it. If we follow these ideas, our code a lot easier to test! Here's an example:

```typescript
const addFive = (input: number): number => {
    return input + 5
}
```

If we use addFive then we'll always get a number that is 5 greater than the input value out. The classic example of an impure function is something like this:

```typescript

const randomInRange = (minValue: number, maxValue: number): number => {
  const min = Math.ceil(min)
  const max = Math.floor(max)
  return Math.floor(Math.random() * (max - min) + min)
}

const addRandom = (input: number): number => {
    return input + randomInRange(1, 10)
}
```

If we run addRandom, we now no longer have deterministic output for the function, its not pure. We have no idea what number we'll get out if we put any number in, it's random (to a point).

Can you see how much easier addFive is to test? We could easily write some unit tests for it, e.g `addFive(2) == 7`, `addFive(10) == 15`. We'll always know what the return value of the function will be which makes the function inherently testable. Our addRandom function is impossible to test like this, we can only test if the number we get out is bigger than the number we put in - that's not a very good test case if you ask me.

### Side-effect free

Side-effects are external factors to our functions, they're things that affect state outside of our function. For example, making a HTTP call to another application is a side effect, it performs some operation outside of your function, and it breaks purity - you now cannot test your function reliably as you rely on state returned from an external source and the very presence of that external state itself! Think about this, one day your call to the "addFive" service may return a number that is 5 greater than the number you pass in; however, what if the person that wrote the service suddenly decided they now wanted it to be the addSeven service as 7 is a luckier number? You have no control over that, and now because of the reliance on the addFive service, all of your code is broken!

### Referential transparency

Building on the idea of purity and being side-effect free, we can talk about referential transparency. A function can be referentially transparent if we can replace a call to the function with the value of the function without changing the behaviour of our application in any way. Because pure functions are just a simple transformation of X -> Y, this should make intuitive sense. And because our pure functions are side-effect free it means that subsituting in the value has no other affect on the application, we won't get any strange errors for calling out to another service, or having a database insert crashing our application, for example. As a reference, we're saying the following two snippets show that addFive is referentially transparent:

```typescript
const addFive = (input: number): number => {
    return input + 5
}

if (addFive(2) === 7) {
    return true;
}
```

```typescript
if (7 === 7) {
    return true;
}
```

We get the same result if we replace the function call to `addFive(2)` with its result of 7 every time.

That's all for now, that was just a quick primer on some ideas of functional programming that you'll need to keep in your head as you read the next posts. And it was a good way to get back into writing after a long time of course!