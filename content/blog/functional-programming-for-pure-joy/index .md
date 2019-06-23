---
title: Functional Programming for Pure Joy
date: 2017-08-26 10:41:37
tags:
 - Javascript
 - Functional Programming
categories:
 - Javascript
---

I recently gave a talk by this name at the [Charlotte Node User Group](https://www.meetup.com/Charlotte-NodeJS-User-Group/). My goal was to present portions of the Function Programming(FP) paradigm that are easy, high value adds.  Since every new technique has a cost in developer time to learn the technique you need to include that in your consideration whether techniques are right for you project/team. 

## A Brief History of Computation

In the 1930's the ground work of computability theory was being done, two of the people doing this were [Alan Turing](https://en.wikipedia.org/wiki/Alan_Turing) and [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church). Turing created amount many thing the idea of a Turing Machine, a machine that has a tape that it can move back and forth and record information on.  While Turing was creating the Turing Machine, Church was creating the λ-Calculus, the two were eventually shown to be equivalent though they take different approaches.  The λ-Calculus is a stateless way of examining computation based solely on evaluating function. These two concepts show two approaches to computing Imperative(Turing Machine) and Declarative(λ-calculus).

## Theory

What exactly is Functional programming? 

Functional programming focuses on building the output from a series of function call while avoiding state and mutable data. Lets break that statement down:

### Pure Functions (avoiding state)

*Pure functions* are a constraint in FP that helps avoid state.  To talk about pure functions its helpful to define another term, *side effect*.  A side effect is simply when a function does something that affects the state of the program that it doesn't "own". In addition to the programming state you cannot modify state outside of the system this includes things like database calls and disk access.  A pure function is simply a function with no side effects. Lets look at some examples:

``` javascript Pure Function
function count(current) {
  return current + 1;
}

console.log(count(0)); // 1
console.log(count(0)); // 1
```

This is a pure function because it will return the same value every time it is called with the same argument.

``` javascript Impure Function
let count = 0;

function counter(){
    count ++;
    return count
}

console.log(counter()); // 1
console.log(counter()); // 2
```

This is an example of an impure function because it modifies state that it doesn't "own" (`count`).

Pure functions have several advantages over impure functions:
- Predictable - you know that you will always get the same value for a set of arguments
- Testable - you don't have to do any setup for your test
- Easier to debug - you don't have to recreate the application state once you know the problem function

### Immutable Data (avoiding mutable data)

*Immutable data* is simply data that doesn't change.  This is a concept closely tied to pure functions as a pure function will never mutate data that it is passed as that would be a side effect. Enforcing immutable data in your application will make it easier to reason about how the application works as a function call in one place will not affect any function that works on the same piece of data.

``` javascript Mutable Data
const nums = [1, 2, 3, 4];

function getSquares(array) {
  for (var i = 0; i < array.length; i += 1) {
    array[i] = array[i] * array[i];
  }

  return array;
}

console.log(getSquares(nums)); // [1, 4, 9, 16]
console.log(nums); // [1, 4, 9, 16]
```
Here our function replaces each item in the array with its square, now any code further on which expects the original nums array will get the new squared data.

``` javascript Immutable Data
const nums = [1, 2, 3, 4];

function getSquares(array) {
  const result = [];

  for (var i = 0; i < array.length; i += 1) {
    result.push(array[i] * array[i]);
  }

  return result;
}

console.log(getSquares(nums)); // [1, 4, 9, 16]
console.log(nums); // [1, 2, 3, 4]
```
By constructing a new array here we have prevented modifying the original array and any other code using the nums array will get the data it expects.

Immutable data is not built into javascript however the tools you need to work on data in a non-mutating way are available.  `Object.assign` allows us to create shallow copies of objects which we can then change safely. `Array.prototype.slice` allows us to create copies of array which we can then safely mutate. To get truly immutable data types you can use a library like [Immutable.js](https://facebook.github.io/immutable-js/).

### Requirements for FP

In order to be able to take advantage of FP concepts the language must have one critical feature, first-class functions. *First-class functions* simply means that the language must allow the programmer to use a function anywhere that an expression can be used. The simplest example in javascript is a function expression  i.e. `var fn = function(){}`.  First class functions are important because they allow us to create higher-order functions.  *Higher-order functions* are simply functions that either take a function as a parameter or return one as a result, we'll look at a few of these in the next section.

One language feature that is not present in most Javascript environments but is useful is *tail call optimization*. This allows recursive functions to be more efficient but is not required.  It is part of the ES2015 spec but is currently only implemented in Webkit browsers.

## Synchronous Collections

JavaScript contains built in functions for handling arrays inspired by FP called `map`,`filter`,`reduce`, and `forEach`. We'll construct these functions so that we know how they work and then we'll look at the built in versions. Once you've mastered these functions there is no need to manually iterate over an array using a for loop. Further more since these functions all return arrays we can conviently chain them together to get more complex behavior.

### map

The map function takes an array and transforms it to a new array by repeated application of a function to its elements.

``` javascript Map
function map(fn, array) {
  const result = [];

  for (var i = 0; i < array.length; i += 1) {
    result.push(fn(array[i]));
  }

  return result;
}

map(x => x * x, [1, 2, 3, 4]) // [1, 4, 9, 16]
```

This ordering of parameters is typical of functional programming as it allows partial application to create reusable map functions.

``` javascript Partial Application
const getSquares = map.bind(null, x => x * x)
getSquares([1, 2, 3, 4]) // [1, 4, 9, 16]
```

The builtin for all of these functions exists on the Array prototype so the call looks a little different

``` javascript BuiltIn
[1, 2, 3, 4].map(x => x * x) // [1, 4, 9, 16]
```

### filter

The filter function returns a new array where the members satisfy whatever condition is encoded in the function it is passed.

``` javascript Filter
function filter(fn, array) {
  const result = [];

  for (var i = 0; i < array.length; i += 1) {
    if (fn(array[i])) {
      result.push(array[i]);
    }
  }

  return result;
}

filter(x => x % 2 === 1, [1, 2, 3, 4, 5]) // [1, 3, 5]
```

``` javascript BuiltIn
[1, 2, 3, 4, 5].filter(x => x % 2 === 1) // [1, 3, 5]
```

### reduce

The reduce function builds up a single value by repeatedly applying the function it is passed to an array.

``` javascript Reduce
function reduce(fn, initial, array) {
  let acc = initial;

  for (var i = 0; i < array.length; i += 1) {
    acc = fn(acc, array[i]);
  }

  return acc;
}

reduce((x,y) => x + y, 0, [1, 2, 3, 4]) // 10
```

``` javascript BuiltIn 
[1, 2, 3, 4].reduce((x,y) => x + y, 0) // 10
```

### forEach

The forEach function is a bit different as it doesn't have much of a use in FP but allows us to nicely iterate over an array in order to cause side-effects with similar syntax to the other array methods.

``` javascript forEach
function forEach(fn, array) {
  for (var i = 0; i < array.length; i += 1) {
    fn(array[i]);
  }
}

forEach(console.log, [1, 2, 3, 4]) // Logs 1/n 2/n 3/n 4/n
```

``` javascript BuiltIn
[1, 2, 3, 4].forEach(console.log) // Logs 1/n 2/n 3/n 4/n
```

## Asynchronous Collections

The functions discussed above are very helpful when working in a totally synchronous environment but JavaScript is rarely ever a completely synchronous environment.  For using the above functions with promises we can use the library [bluebird.js](http://bluebirdjs.com/docs/getting-started.html).  This allows us to return promises from our iteration functions.  

``` javascript
const Promise = require("bluebird");

const values = ["1", "2", "3", "4", "5"];

function getValue(value) {
  return new Promise(resolve => setTimeout(resolve(value), 1000));
}

const sum = (a, b) => a + b;
const isEven = num => !(num % 2);

const fileData = Promise.map(values, getValue)
  .map(data => parseInt(data))
  .filter(isEven)
  .reduce(sum)
  .then(console.log); // Logs 6 to the console
```

## Further Resources

The git repo for my talk can be found [here](https://github.com/mStegall/functional-programming-for-pure-joy).

For a FP centric library take a look at [Ramda](http://ramdajs.com/).