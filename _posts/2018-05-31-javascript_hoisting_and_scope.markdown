---
layout: post
title:      "JavaScript Hoisting and Scope"
date:       2018-05-31 20:36:30 +0000
permalink:  javascript_hoisting_and_scope
---


When first learning JavaScript, the concepts of scope and hoisting were somewhat confusing to me. I had encountered scope in other contexts before, but hoisting was an entirely foreign concept. This post will attempt to explain these two topics in simple terms.

## Differences between const, let and var
Before getting into scope and hoisting, it may be useful to review the different ways we can declare variables in JavaScript.

There are a few ways to declare variables in JavaScript, using the keywords `var`, `let`, and `const`. Briefly, the differences between these methods of declaring variables are as follows:

1) **Re-declaration:** `var` will allow you to declare the same variable twice. With `let` and `const`, you will get an error if you try to re-declare a variable, that is, if you try to create a new variable with the same name as a variable you have already declared.

2) **Block scope:** Variables declared with `var` are not block-scoped. That is, if you declare a variable with `var` inside a block (e.g. an if statement), that variable is still accessible outside that block. Variables declared with `let` and `const` _are_ block scoped, such that you will get an undefined error if you attempt to access a variable declared with `let` or `const` from outside the block it was declared in.

3) **Re-assignment:** Variables declared with `const` cannot be re-declared and **also cannot be reassigned**. The thing to know here is that if you assign a primitive value to a variable declared with `const`, the value of the variable will never change; however, if you assign an _object_ to a variable declared with `const`, the reference to the object will never change **but the object's properties can still be modified**. That means, for example, that the following will work just fine:
```
const newVariable = [];
newVariable.push('adding!');

//=>["adding!"]
```

4) **Declaring without assigning a value:** With `let` and `var`, you can declare a variable without assigning it a value. e.g. `let newVariable;`. This won't work with `const` because once declared, a variable declared with `const` cannot be re-assigned, so it wouldn't make sense to create a `const` variable without assigning it a value.

In general, it is wise to use `let` and `const` for variable declarations in new code, as this avoids scoping issues you may run into with `var` (discussed below). One can also declare and assign a variable without using any of these keywords, but this is generally a bad idea (see below).

## Scope
Now that we have an understanding of the different ways to declare variables in JavaScript, let's examine the concept of scope.

Essentially, scope refers to **where something is available**. In JavaScript, all code is run in what's called an **execution context**, which can be either the global context, or a smaller "scope" created by a function or block. Variables and functions declared in a given context will then be available in that "scope", and _will also be available_ in the scope of any functions and blocks _declared within_ (i.e. nested within) that context.

This brings us to the idea of the scope chain. Essentially, the scope chain is a mechanism whereby variables and functions declared in nested contexts have access to information in the outer scope in which their scope was declared. For example:

```
const firstVar = 'one';

function newFunction() {
  const secondVar = 'two';
  console.log(firstVar, secondVar);
}

newFunction();
//=> one two
```

`newFunction` created its own scope, within which it defined `secondVar`. But it also has access to the variables and functions declared in the scope in which it was declared, in this case, it has access to `firstVar`. Note that a function's outer scope is defined by where the function is _**declared**_ (not by where it is invoked).

When it comes to variable declarations, one thing to know is that if you declare a variable without `var`, `let`, or `const`, it will be available in the global scope!

There is a lot more to scope in JavaScript, but this is the basic idea. The basic thing to remember is as follows:
- _All variables and functions declared in outer scopes are **available in inner scopes** via the scope chain._
  -Learn.co curriculum
- **BUT! It doesn't work the other way. Above, outside of `newFunction`, `secondVar` is not defined!**


## Hoisting
Now let's take a look at hoisting. In order to understand hoisting, we have to understand how the JavaScript engine reads and executes our code. There are 2 phases that occur when JavaScript engine reads a JavaScript file:

1) **Compilation phase:**
During this phase, the engine reads the code from top to bottom and is just looking at _function and variable declarations_. For variables, memory is allocated and a reference to their identifier (the variable name) is created. For functions, the same thing also occurs, and in addition, a new execution context with a new scope is created for the function, along with _a reference to the outer scope in which the function was declared._ This is how the variables and functions declared in the outer scope of a function are made available within that function's scope. In this first phase, function invocations are ignored.

2) **Execution phase:** During the second phase, the code is actually executed. Values are assigned to variables, and functions are invoked.

**Hoisting** basically refers to the idea that since function and variable declarations are set up in the compilation phase, which runs before the execution phase, you can actually call functions in your code "before" (as in, higher up in your file) you declare them. This is because the declarations are figuratively "hoisted" to the top of the file during the compilation phase. Thus, something like the following works just fine:
```
printHello();

function printHello() {
  console.log('Hello');
}

//=> Hello
```
The function declaration is set up during the compilation phase, so when `printHello()` is actually executed in the execution phase, the reference to the code contained in the function is already defined, and everything works.

It is important to remember that hoisting only applies to variable _declarations_, not variable _assignments_. That is, the following won't work so nicely:
```
function printHello() {
  console.log(newVariable);

  var newVariable = 'hello';
}

printHello();

//=> undefined
```

During the compilation phase, a reference is set up to `newVariable`, but it is not assigned a value until the execution phase. That means until we hit the assignment to `'hello'`, `newVariable` will return `undefined`.

The good thing to know is that if you use `let` or `const` in this context, you can avoid this issue because JavaScript will not allow you to reference variables declared with `let` or `const` if they have not been defined. For example:

```
function printHello() {
  console.log(newVariable);

  let newVariable = 'hello'; // **changed var to let**
}

printHello();

//=> ERROR:Uncaught ReferenceError: newVariable is not defined
```

In general, to avoid issues with hosting that can crop up with `var`, make sure to declare functions and variables at the top of their scope. Basically, if you declare your functions and variables above where you are going to use them, and make sure to always use `let` and `const` rather than `var`, you shouldn't run into any hoisting issues.

### Some Examples
 I found the below examples illustrative when thinking about hoisting.

#### var v. let
##### var
```
function hoisting() {
 console.log('My name is', name);
 var name = 'Allyson';
}

hoisting();

//=> My name is undefined
```

- Variable declaration for variable declared with `var` was hoisted
- Assignment took place after `console.log` so the value of the variable was `undefined` when it was logged
- For variables declared with `var`, JavaScript _will_ let us reference the variables before they are defined

##### let
```
function hoisting() {
  console.log('My name is', name);
  let name = 'Allyson';
}

hoisting();

//=> ERROR: ReferenceError: name is not defined
```

- Variable declaration for variable declared with `let` was hoisted
- Assignment took place after `console.log`
- Error at the `console.log` line because we can't reference a variable declared with `let` before it's defined

#### Changing name to be a function

```
function hoisting() {
  console.log('My name is', name());

  function name() {
    return 'Allyson';
  }
}

hoisting();

//=> My name is Allyson
```
- Function declaration for `name()` was hoisted
- `name()` function is executed when we call `console.log`
- Works because the reference to `name()` was set up in the compilation phase
