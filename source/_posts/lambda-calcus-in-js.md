---
title: Lambda Calculus in Javascript
date: 2017-09-10 00:07:57
tags:
 - Javascript
 - Functional Programming
 - Computer Science
 - Lambda Calculus
categories:
 - Javascript
---

Lambda calculus is a description of computation that is functionally equivalent to Alan Turing's Turing Machines, though it works a bit differently. It only includes functions and while this may seem limiting at first we'll show some basics of how you can encode boolean logic in lambda calculus.

Much of my inspiration for this post come from the book [An Introduction to Functional Programming Through Lambda](https://www.amazon.com/Introduction-Functional-Programming-Calculus-Mathematics-ebook/dp/B00CWR4USM/ref=sr_1_5?ie=UTF8&qid=1505088825&sr=8-5&keywords=introduction+functional). This is a great introductory book to lambda calculus for those that find the subject interesting.  It starts from the very fundamentals and builds all the way up to a typed langauage with many of the functional tools we've come to expect.

## Lambda Calculus Basics

The basic unit of lambda calculus is a function. The simplest of those functions is the identity function and it looks like this: (&lambda;x.x). The x before the dot indicates what we refer to in javascript as a parameter while the x after the dot is like a function body. The equivalent javascript looks like this:

``` javascript
const identity = x => x
```

## Executing Statements

As a tool for our expirements in lambda calculus its useful have a function that simply logs a function name so we can see our results.

``` javascript
const execute = fn => console.log(fn.name)
```

When you call that on the identify function we've declared above it should simply log `identity`

``` javascript
execute(identity) // identity
```

## Boolean Values

To start off we need to define our boolean values, which aren't given to us to start with. You could choose another way to define these but you'll see that these definitions make constructing more advanced functions quite simple. We simply say that true is the function that returns its first argument and false is the function that returns its second argument. In lambda calculus these would be wrtten as (&lambda;x.&lambda;y.x) and (&lambda;x.&lambda;y.y) respectively. The functions are called myTrue and myFalse simply because javascript doesn't let us redfine those terms.

``` javascript
const myTrue = (first, second) => first
const myFalse = (first, second) => second
```

## Conditionals

The first thing we need in a program is the ability to branch based on some value, this is what we usually see as an `if` statement.  To do this we can take advantage of the fact that our boolean values are defined as functions with two parameters, much the how an if-else is defined by its two code blocks. Consider these two blocks of code:

``` javascript
// myTrue 
console.log(myTrue(3,5)) // 3

// if-else 
const ifElse = () => {
  if (true) {
    return 3;
  } else {
    return 5;
  }
}
console.log(ifElse()) // 3
```

These two statements do nearly the same thing. The way we can use this is by creating a function that takes our two branches and allows us to apply the condition later.

``` javascript
const cond = (a, b) => pred => pred(a, b)

// example use
cond(3,5)(myTrue) // 3
cond(3,5)(myFalse) // 5
```

The equivalent lambda calculus expression would look like this: (&lambda;a.&lambda;b.&lambda;bool.(bool a b)).

## Logical Operations

Since we now have conditionals it very easy to start defining our basic logic functions like not, and, and or. I've provided some definitions below and some examples of how they might be called. Think about how you might would write these functions normally and how that maps to our lambda calculus version.

``` javascript
const not = cond(myFalse, myTrue)
const and = (b) => cond(b,myFalse)
const or = (b) => cond(myTrue, b)

const xor = (a,b) => and(or(a,b), not(and(a,b)))

execute(not(myTrue)) // myFalse
execute(and(myTrue)(myFalse)) // myFalse
execute(or(myTrue)(myFalse)) // myTrue
```

We can also use a process called &Beta;-reduction, which is basically a fancy term for substitution to make our expressions simpler.


``` javascript
const not = (input) => input(myFalse, myTrue)
const and = (a, b) => a(b, myFalse)
const or = (a, b) => a(myTrue, b);

const xor = (a,b) => and(or(a,b), not(and(a,b)))
```

## Why does this matter?

Lambda calculus like isn't practical at all in Javascript I'm not advocating anyone try programming like this as it is beyond useless in an envirnoment that is so heavy with state.  I do think there is some value in learning this as it helps open you up to knew ways of thinking about things and gives some context for just how much theory exists behind what we do every day.

If like me you find this interesting I would highly recommend the book linked at the top of this post as it gives a good guided tour of building up a language from basic parts. The highlight of the book is how recursion(self-referencing) is accomplished in lambda calculus and all the doors it opens. It's certainly not an exhaustive book and can be read in about a week but I found it very enjoyable.
