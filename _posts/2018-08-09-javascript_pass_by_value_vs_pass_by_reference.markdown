---
layout: post
title:      "JavaScript Pass by Value vs. Pass by Reference"
date:       2018-08-09 11:18:00 -0400
permalink:  javascript_pass_by_value_vs_pass_by_reference
---

One topic I struggled with when implementing [Word Scramble](https://unscramble-it.herokuapp.com/) was scrambling the array of letters that made up an individual word without modifying the original "solution" array that actually contained the letters in the correct order. Essentially I was passing the solution array into a scramble function, scrambling the array and updating the game state with it.

```
function scramble(lettersArray) {
  const scrambledLetters = scrambleWords(lettersArray);

  this.setState({
    scrambledLetters: scrambledLetters
  })
}
```

The problem with this method was that when it finished, my original solution array was also scrambled! This made it impossible to detect whether the user ever entered the right solution since my app no longer knew what the correct order of the letters was.

The simple solution was to make a copy of the solution array, scramble it, and use that array to update the state, but the problem I ran into forced me to figure out how JavaScript deals with passing arguments to functions. Clearly, in this case, a _reference_ to my original solution array, rather a copy of the value of the array itself, was being passed into my scramble function. Thus, when I altered the array using that reference, the original array was altered.

What I learned was that pass by reference vs. pass by value is fairly straightforward in JavaScript. Namely:
1. **Primitives are passed by value**
2. **Objects are passed by reference** (technically "copy of a reference")

Basically what this means is that if you're passing primitive values into a function (string, number) a copy of the argument passed is made and when you manipulate the string or number, you're not changing the value of the original argument you passed in. It's helpful to know that in JavaScript, numbers and strings are _immutable_, meaning their values actually _can't_ be changed after they're assigned (a copy is always made when you change an immutable variable), so it makes sense that they are passed "by value" into functions.

```
function changeValues(a, b) {
  a = 3;
  b = 4;
}

let a = 1;
let b = 2;

console.log(`Before changeValues: a = ${a}, b = ${b}`)
changeValues(a, b)
console.log(`After changeValues: a = ${a}, b = ${b}`)

//=> Before changeValues: a = 1, b = 2
//=> After changeValues: a = 1, b = 2
```

For objects (including arrays, functions, classes, etc.) when you pass one into a function, you are actually passing a _copy_ of a reference to that object. You can then use this reference to modify the original object; however, since that reference is actually a primitive itself, it's passed by value, so modifying the reference won't affect the original reference to that object that the calling function has.

```
function changeValues(a, b) {
  a.foo = 3;
  b.bar = 4;
  a = {baz : 5};
}

let a = {foo: 1};
let b = {bar: 2};

console.log(`Before changeValues: a.foo = ${a.foo}, b.bar = ${b.bar}`)
changeValues(a, b)
console.log(`After changeValues: a.foo = ${a.foo}, b.bar = ${b.bar}, a.baz = ${a.baz}`)

//=> Before changeValues: a.foo = 1, b.bar = 2
//=> After changeValues: a.foo = 3, b.bar = 4, a.baz = undefined
```

My scramble function problem helped me learn all this, and now I know I needed to avoid modifying the original array passed by simply creating a copy and scrambling that array.

```
function scramble(lettersArray) {

  // slice() with no arguments will return a shallow copy of the array
  const scrambledLetters = scrambleWords(lettersArray.slice());

  this.setState({
    scrambledLetters: scrambledLetters
  })
}
```
