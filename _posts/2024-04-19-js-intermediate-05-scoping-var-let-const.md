---
title: Javascript Intermediate 5 - About scoping var, let, const 
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

## Global-Scope vs Local-Scope

In terms of the call-stack Global-Scope vs Local-Scope


Global Scope:
 - The global scope represents the outermost scope in JavaScript.
 - Variables and functions declared in the global scope are accessible from anywhere in the script. 
 - When a JavaScript program starts executing, the global scope is created and remains active until the program finishes executing. 
 - Functions and variables defined in the global scope are stored in memory for the entire duration of the program's execution.
 - The global scope is the first scope pushed onto the call stack when the program starts executing, and it remains at the bottom of the call stack throughout the program's execution.

Local Scope:
 - Local scopes are created when functions or blocks (like those in if statements, for loops, etc.) are executed.
 - Variables and functions declared within a local scope are only accessible within that scope.
 - When a function is invoked, a new local scope is created for that function's execution context.
 - Each time a function is called, a new instance of its local scope is created, which is added on top of the call stack.
 - When a function finishes executing, its local scope is removed from the call stack, and any variables or functions declared within that scope are destroyed, freeing up memory.
 - Nested function calls create nested local scopes, with each scope having access to variables declared in its outer scopes (lexical scoping).


## Javascript Variable Declaration keywords


```
 var

The  var statement declares function-scoped or globally-scoped
variables, optionally initializing each to a value.



 let

The  let declaration declares re-assignable, block-scoped local 
variables, optionally initializing each to a value. 



 const

The  const declaration declares block-scoped local variables. The value 
of a constant can't be changed through reassignment using the assignment 
operator , but if a constant is an object , its properties can be added, 
updated, or removed. 
```

A main axiom is that:
Every variable declared in the global scope will be readable in the Local Scope


```
var myVar = 5;
const myConst = 10;
let myLet = 15;

console.log(myVar);
console.log(myConst);
console.log(myLet);

myFunction();

function myFunction () {
    console.log('Executing within myFunction');
    console.log(myVar);   // 5
    console.log(myConst); // 10
    console.log(myLet);   // 15
}
```

```
-   Global scope: The default scope for all code running in script mode. 
-   Module scope: The scope for code running in module mode. 
-   Function scope: The scope created with a function . 

In addition, variables declared with  let or  const can belong to an 
additional scope: 

-   Block scope: The scope created with a pair of curly braces (a block 
    ).
```


Block scope only available for let and const
 - let and  const declarations can also be scoped to the block statement that they are declared in.


```
if (true) {
    var myVar = 5;
    let myLet = 10;
    const myConst = 15;
}

console.log(myVar);
// console.log(myConst); // myConst is not defined
//console.log(myLet); // myLet is not defined
```

## Variable Hoisting



```
 var -declared variables are hoisted , meaning you can refer to the 
variable anywhere in its scope, even if its declaration isn't reached 
yet. You can see  var declarations as being "lifted" to the top of its 
function or global scope. However, if you access a variable before it's
declared, the value is always  undefined , because only its declaration
is hoisted, but not its initialization .
```



```
console.log(myVar); // Undefined
//console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
var myVar = 5;
let myLet = 10;
console.log(myVar);
console.log(myLet);
```

The space in which a variable is not able to be accessed due to not hoisting is called
**temporal dead zone**

In the previous code regarding to myLet , the first 3 lines of code are a "temporal dead zone" for that variable as it cannot be accessed on there



## Javascript Intermediate tutorials

This tutorial is part of **freddieventura** Javascript Intermediate tutorials.
By writting this I aim to give my little contribution to all of those who are in the process of learning Javascript taking it a step further.

May I have been lucky a Search Engine has taken you here, I did pass you the link, or you are my follower for any other reason, I encourage you to take your time and read through it.

None of the text has been written with the help of LLM (AI), only the head image.

The tutorial are examples I have been executing myself in order to proove the concepts.
I encourage you to do your own ones. 

I have used **nodejs** as Execution environment.


Apologies if the formatting is not perfectly coherent, but they are just a transcription of my notes to this markdown processor. 

As main source of information I list the following.
 - [Mdn Web Docs Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
 - [Mdn Web Docs Web API](https://developer.mozilla.org/en-US/docs/Web/API)

Which I comfortably read from my terminal via [vim-dan](https://github.com/freddieventura/vim-dan)

## Reference sources

 - [var - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)
 - [let - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
 - [const - JavaScript | MDN] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
