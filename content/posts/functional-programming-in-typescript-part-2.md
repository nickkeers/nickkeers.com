---
title: Functional Programming in Typescript Part 2
date: 2020-08-25T21:25:04.000+01:00
math: true

---
Last time I wrote about pure functions and the benefits they can bring to your code. However, it may not be immediately clear how to use pure functions and how to handle side-effects in your software - after all, without side-effects, your software won't be able to do anything meaningful.

Side-effects bring us the ability to persist data to files or a database for example or allow us to split our software into microservices that can communicate over HTTP or a similar protocol. I try to keep in mind the idea of keeping a functional core for my logic and having side-effect producing functions separate. This has the advantage of keeping your main business/app logic separate from your functions that persist your data, meaning that they are far easier to test as you no longer have to mock out the database to test core logic. However, this does leave us with more functions lying around in our code, to fix we can look to function composition as a way to combine these functions back together again so we can persist our data.

## Function composition

Function composition is a simple, but powerful idea; all it means is that if we have two separate functions, we can combine them to get a single result. In mathematical terms, if we have the functions $$f(x) \\rarr Y$$ and $$g(y) \\rarr Z$$ then our new function h is:

$$h(x) = (g \\cdot f)(x)$$

If that looks scary, think about it like the below. We have a function `f` that takes a number and adds 2 and a function `g` that multiplies by 2 and we want to combine them into a single function that adds 2 and multiplies 2 to the given number

```typescript
const f = (x: number): number => x + 2
const g = (y: number): number => y * 2

const h = (n: number): number => g(f(n))
```

Instead of calling `g` with the result of calling `f` with n as the parameter, we compose them into a single function `h`. So, function composition is just calling the functions together, chaining the result of one into the next function.

## Function composition applied

So now we know how to compose functions we can start to apply that to our code. Imagine we want to take the details of a person and then store their details into a database, we can have one function that structures our data correctly and that can be a pure function, and we can have another function that stores that data into a database which has a side effect. However, we don't want to expose both functions to the end-user of our new API, we want it to be easy to use, so we can compose the functions together to make it easier to use.

```typescript

interface UserRecord {
    forename: string;
    surname: string;
}

const nameToRecord = (forename: string, surname: string): UserRecord  => {
    return {
        forename,
        surname
    }
}

const saveUserRecord = (record: UserRecord): boolean => {
    return db.save(record)
}

export const saveUser = (forename: string, surname: string): boolean => {
    return saveUserRecord(nameToRecord(forename, surname))
}
```

Can you see how `saveUser` takes in the parameters for `nameToRecord` and returns the value of calling `saveUserRecord`? That's function composition in action, we've exposed a friendly function for the end-user (I've deliberately simplified `saveUser` and not included exceptions for example - we can deal with those in a later post!).

At the minute you're probably thinking, big deal, I write code like this all the time. And yes, you probably do write a lot of code like this, calling functions in a nested way like this - and you've never realised its function composition! However, function composition gets more useful if you have a generic way to compose arbitrary functions together in a declarative way. So instead of having `saveUserRecord(nameToRecord(forename, surname))` we could have something that looks like this:

```typescript
export const saveUser = (forename: string, surname: string): boolean => {
    return compose(saveUserRecord, nameToRecord)(forename, surname)
}
```

This new `compose` function returns us a new function which is the composition of `nameToRecord` and `saveUserRecord` and then we can simply call that new function with the arguments to the first function in `compose` which in this case is the `nameToRecord` function. Note that we have to pass in the functions in reverse order to what we may think due to how compose works under the hood, we can think of it as calling `saveUserRecord` and then looking to the next function for its arguments which is the result of applying that next function `nameToRecord`.

### Writing a compose function

So, our compose function needs to take in an arbitrary list of functions and return a new function from them, so how might that look? Well, we know that we need to accept a variable number of functions which could take any number of parameters, so our declaration would be:

```typescript
type ComposeFunctionArg = (...args: any[]) => any

const compose = (...args: ComposeFunctionArg[]) => {}
```

I found it helpful to make a new type for the functions that could be passed in as there were a lot of nested brackets. So now we have our declaration, we need to call each function in turn, passing in the return value of the previous function as the arguments to the next function:

```typescript
type ComposeFunctionArg = (...funcArgs: any[]) => any

const compose = <T extends ComposeFunctionArg[], U extends any[]>(...rest: T): ComposeFunctionArg => {
    return rest.reduce((previousFunc: ComposeFunctionArg, currentFunc: ComposeFunctionArg) => {
        return (...args: U): any => previousFunc(currentFunc(...args))
    })
}
```

I used a new Typescript 4.0 feature here - variadic tuples, which saved a lot of typing of generic parameters. Previously you'd have to define many generic parameters equal to the number of functions you want to compose, but with variadic tuples, that's all hidden away. You can read more [here](https://devblogs.microsoft.com/typescript/announcing-typescript-4-0/#variadic-tuple-types) - check out that concat function for what I mean.

I use reduce to loop through all our functions and apply them one by one in the correct order, passing through the arguments each time - using reduce here is also a nice demonstration of functional programming, we're avoiding intermediate variables that we would need if we were to do a complex for loop to do the same task. Once you're familiar with some of the core functions like reduce, map, etc you can use them to save on the amount of you code you're writing.

Reduce takes a function as a parameter and because of that, in the world of functional programming, we would call this reduce function a "higher-order function" (more on that next post).

We can test our code out by running something like the below:

```typescript
let f = (x: number): number => {
    console.log(`f(${x})`)
    return x + 2
}

let g = (x: number): number => {
    console.log(`g(${x})`)
    return x * 2
}

let h = (x: number): number => compose(g, f)(x)

console.log(h(2))
```

We see in the console:

    [LOG]: f(2)
    [LOG]: g(4)
    [LOG]: 8

Hooray, our compose function works! So now we could write `saveUser` like this, just like we wanted to in the first place - very concise!

```typescript
export const saveUser = (forename: string, surname: string): boolean => {
    return compose(saveUserRecord, nameToRecord)(forename, surname)
}
```