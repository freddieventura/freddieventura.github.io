---
title: Javascript Intermediate 6 - What is a prototype
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

First off
> All object literals in javascript inherit from Object.prototype global object


```
var myObject = { };
console.log(myObject instanceof Object);     // True
console.log(myObject instanceof Function);   // False
```

Also, out of the inheritance model we can say that 
> Javascript objects have a property called prototype accessed as __proto__ , which behaves as a link to the parentObject prototype


```
var myObject = { }
console.log(myObject.length);  // Undefined
```

But mutating the object

```
var myObject = {
    __proto__ : Object.getPrototypeOf(Array)
}

console.log(myObject.length);        // 0
```

Or doing afterwards its creation


```
var myObject = { }
Object.setPrototypeOf(myObject, Object.getPrototypeOf(Array));

console.log(myObject.length);        // 0
```

Providing also that the new operator does
```
    -     new constructor() 
    -     new constructor(arg1, arg2, /* ╬ô├ç┬¬, */ argN) 
```

In which Parameters are
```
 constructor 
    A class or function that specifies the type of the object instance. 
```

It will check in its environment namespace for a function or a class of such a name. And return an instance of that function Constructor


```
function DimArray (width,height) {
    this.width = width;
    this.height = height;
}

var myDimArray = new DimArray(20,20);
console.log(myDimArray);     //DimArray { width: 20, height: 20 }
```

Mind that this construct is a constructor function no the instance of the Object itself.
__proto__ , property thus will not work the same way.


```
function DimArray (width,height) {
    this.width = width;
    this.height = height;
    __proto__: Object.getPrototypeOf(Array);
//    prototype: Object.getPrototypeOf(Array); // this wont work neither

}

var myDimArray = new DimArray(20,20);
console.log(myDimArray.length);  // Undefined
```

You have to instead


```
function DimArray (width,height) {
    this.width = width;
    this.height = height;
}
DimArray.prototype = Object.create(Array.prototype);

var myDimArray = new DimArray(20,20);
```

## Distinction between constructor Function prototype and Object Instance Prototype

Instead the prototype property of the Constructor function , it will be similar to the prototype of an instance.


```
function DimArray (width,height) {
    this.width = width;
    this.height = height;
}

var myDimArray = new DimArray(20,20);
console.log(Object.getPrototypeOf(myDimArray) === DimArray.prototype);  //  True
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
