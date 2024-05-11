---
title: Javascript Intermediate 1 - First-class object, JS functions 
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

# Javascript Intermediate 1 - First-class object, JS functions 


There are really only two distinctions worth making: first-class and not first-class

These terms barely have any technical meaning , but its purpose is merely comparative in between languages.

In a given programming language design, a first-class citizen is an entity which supports all the operations generally available to other entities. These operations typically include being passed as an argument, returned from a function, and assigned to a variable.


Some examples:
    - Function pointers in C are first-class values because they can be passed to functions, returned from functions, and stored in heap-allocated data structures just like any other value.
    - Functions in Pascal and Ada are not first-class values because although they can be passed as arguments, they cannot be returned as results or stored in heap-allocated data structures.

    - Struct types are second-class types in C, because there are no literal expressions of struct type.
    - In Java, the type int is second class because you can't inherit from it. Type Integer is first class.
    - In C, labels are second class, because they don't have values and you can't compute with them.

Other way of seeing it first-class vs non first-class citizens in programming is.

Something is first-class if it is explicitly manipulable in the code. In other words, something is first-class if it can be programmatically manipulated at run-time.

When something is reified at run-time, it becomes explicitly manipulable.
    Reifiy stands for "make (something abstract) more concrete or real."
Some Examples
    - We speak of first-class object, because objects can be manipulated programmatically at run-time (that's the very purpose).
    - In java, you have classes, but they are not first-class, because the code can normally not manipulate a class unless you use reflection
    - In java, you have packages (modules), but they are not first-class, because the code does not manipulate package at run-time.


According to the MDN Web Docs

```
Functions are first-class objects

JavaScript functions are first-class objects. This means that they can
be assigned to variables, passed as arguments to other functions, and
returned from other functions. 


// Functions returning functions
// const add = (x) => (y) => x + y;
function add (x) {
    return function (y) {
        return x + y;
    }
}

console.log(add(4));       // [Function (anonymous)]
console.log(add(4)(2));    // 6


// Functions accepting functions as parameters
function compareAtoB (numA, numB, numACbFn, numBCbFn) {
    if (numA > numB) {
        numACbFn('Is A, from Alpha, corresponding to ' + numA + ' the maximum');
    } else {
        numBCbFn('Is B, from Bravo, corresponding to ' + numB + ' the maximum');
    }
}

compareAtoB (8 , 4, (numACbParam) => {
    console.log(numACbParam)
},  (numBCbParam)  => {
    console.log(numBCbParam)
}); // Is A, from Alpha, corresponding to 8 the maximum
```


I add the following examples

```
// Functions can be assigned to variables
const myFunction = function () {
    console.log('pep');
}

myFunction();   // pep


// Functions can be assigned to properties
var myObject = {
    myFunction: function () {
        console.log('pep');
    }
}

myObject.myFunction(); // pep
```


## Named function vs anonymous functions

We are gonna see the difference between an assigned an anonymous function to a variable vs declaring a named function

Function name vs variable the function is assigned to

 - Anonymous Function Assigned to a Variable:
    When you assign an anonymous function to a variable, the function itself is created and stored in memory, along with the variable reference. If you create multiple instances of this function, each instance will consume memory for the function definition and a reference to the variable.

 - Named Function Declaration:
    When you declare a named function, the function is created and stored in memory once, regardless of how many times it's called or referenced. The function name becomes a reference to the function object in memory.

In terms of memory usage then

 - Anonymous Function Assigned to a Variable: Requires memory for the function definition and a reference to the variable. If you create multiple instances, each instance consumes additional memory for its own function definition and reference to the variable.

 - Named Function Declaration: Requires memory for the function definition only once, regardless of how many times it's called or referenced. The function name serves as a reference to the function object in memory.    Anonymous Function Assigned to a Variable: Requires memory for the function definition and a reference to the variable. If you create multiple instances, each instance consumes additional memory for its own function definition and reference to the variable.


See some examples
```
const myFunctionOne = function () {
    console.log('myFuncionOne');
}

function myFunctionTwo () {
    console.log('myFuncionTwo');
}

myFunctionOne();
myFunctionTwo();
```

Dynamically declaring a function  with the Function constructor, (potential security issues in some cases)

```
//const i = 5
const i = 1

if ( i > 2 ) {
    var saySomething = new Function('console.log("Hello World")');
} else {
    var saySomething = new Function('console.log("Goodbye World")');
}

saySomething();
```


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


## Other references

 - [JavaScript language overview - Mdn Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Language_overview)
 - [First class citizen - Wikipedia](https://en.wikipedia.org/wiki/First-class_citizen)
 - [Stack Overflow - About first second and third class value](https://stackoverflow.com/questions/2578872/about-first-second-and-third-class-value)
