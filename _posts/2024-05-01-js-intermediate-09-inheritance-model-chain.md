---
title: Javascript Intermediate 9 - Inheritance model with a prototype chain, a demostration with pseudoclassical inheritance
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

## Runtime Code vs Library Code

First of lets clarify a distinction between the following concepts in a programming language.
 - Core or runtime code of a Programming language: Defines the interpreter or compiler
 - Library Code : Code written on the same programming language (same level of abstraction) that defines basic functionalities that every programmer would use. Sometimes known as standard-library for other programming languages.

On the Core of a Programming language it is established how the keywords interact together, so the compiler/interpreter translates them to machine code.
Example: Why const , defines a new bind to a label (aka variable) , and this label cannot be re-defined but with let you can instead

On the Library Code: Although Javascript doesnt have such as thing as a standard-library, it depends on your environment of execution of this code, been this a certain Browser, Node.js or other engine, we can establish that there is a set of Objects been defined, for most of the Javascript implementations, these are available in the Mozilla MDN Web Docs Javascript part.
Example: Why there is a Global Object called Array, what Properties it has and what functionalities it brings.
You could potentially alter some of these functionalities on the code that will be executed


Now we can jump to the inheritance model in Javascript by checking on Objects defined on the Standard API.

## Into Inhertinace model in Javascript

To understand inheritance in Javascript we will be using the following instance method of Object.


```
Object.prototype.hasOwnProperty(prop)

The  hasOwnProperty() method of  Object instances returns a boolean indicating whether this object has the specified property as its own property (as opposed to inheriting it).
```

For our example we are defining two Objects that. 
 - `myOwnObj` : Defined by a object literal
 - `myObj` : Defined by using the new keyword, using thus the constructor function


```
const myInstObj = {a: 1};
const instObj = new Object();


console.log(myInstObj.hasOwnProperty('a'));               // True
console.log(myInstObj.hasOwnProperty('hasOwnProperty'));  // False
console.log(instObj.hasOwnProperty('hasOwnProperty'));     // False
```


None of them has, `hasOwnProperty` as its own property , yet they can access it. 
We need to clarify two things in here.
 - Object , is an Object , (like Array , RegExp etc..) meaning that:
There has been defined a constructor function somewhere in the Source Code (within the Library Code)
 - Whereas `myInstObj` , and `instObj` they are instances, they dont have a constructor function to defines new Instances of them.

Some properties of objects are accesible through their instances, yet they are not their ownProperties.
These are defined in a special property called `.prototype` this property is special in the sense that is a link to other Objects (ancestor objects), Meaning that when you access a property of an Object , the JS engine will (say for the case of):
`console.log(myInstObj.hasOwnProperty('whateVer');`

 - 1st) Check for myInstObj , if it exists. If it does then:
 - 2nd) Check for myInstObj properties for a property called 'hasOwnProperty' :
   - If it exists execute it with 'whatEver' as an argument
   - If it doesnt exists , check on myInstObj.prototype property to see if there is any property on there named 'hasOwnProperty' 
     - If it exists execute it with 'whatEver' as an argument
     - If it doesnt exists return 'undefined'


For this tutorial we will avoid the use of classical inheritance (with classes), and rather use pseudoclassical inheritance (with constructors prototype property)

We need to first understand what is pseudoclassical inheritance, this is is oposition to classical inheritance which is made via Classes , keywords that we wont be using it in here as we are going to demostrate that we can do the same via lower level concepts.

First example will be a simple one to demonstrate how the JS engine works in its most basic form.
We will be creating the most of the simplest Object in Javascript , defined by an empty Constructor function we will be calling this Object Canvas.

> In JS there are 6 primitives datatypes, these can be checked using the typeof operator onto their literals or variables holding them. 
> String, Number, Boolean, Undefined, Null, and Symbol.

```
console.log(typeof true) // boolean
var myNum = 58;
console.log(typeof myNum)
```

Plus object datatype ,
Creating an empty bodied function will inherit its Properties from Function. object
In JS every defined function is deemed to be used as a constructor of an Object, say we create. We can see that the function has some properties which are inherited by Function object which is its __proto__

```
function Canvas () {
}

console.log(typeof Canvas); // function

var myCanvas = new Canvas();

console.log(typeof myCanvas); // object

console.log(Object.keys(Object.getOwnPropertyDescriptors(Canvas)));
// [ 'length', 'name', 'arguments', 'caller', 'prototype' ]
console.log(Object.keys(Object.getOwnPropertyDescriptors(myCanvas)));
// []
```

Distinctions between prototype of the Constructor function of the object and the prototype ofthe object itself


On the next example we are going to perform a basic functionality.
Taking an Economic Sciences aproach we will be creating  two Classes:
 - Resource: as in natural resources 
 - Food: as in anything that is subject to be human edible
Food is inherits the properties of Resource.
 - Every Resource instance will have a property called hasValue always set to true
 - Every Food instance will have a property called isEdible always set to true
 - Every Food instance will be in turn a Resource , with all its properties such as hasValue

To understand the way Objects work under the hood in Javascript, first we will go to some basic axioms and demonstrate them:
 - Every defined function in Javascript has its own Properties that gives it a unique functionality.
 Note: The this keyword address to one instance of that object created with such a constructor

```
function Resource () {
    this.hasValue = true;
}

//console.log(Object.getOwnPropertyDescriptors(Resource));
console.log(Object.keys(Object.getOwnPropertyDescriptors(Resource)));
//  [ 'length', 'name', 'arguments', 'caller', 'prototype' ]
```

Every Object that we see in the object model it is not but a constructor function, modified, the same way we can see the Array properties , which are just the properties of its constructor function.

```
console.log(Object.keys(Object.getOwnPropertyDescriptors(Array)));
//   [ 'length', 'name', 'prototype', 'isArray', 'from', 'of' ]
```

The prototype property of a Constructor, sets the properties/methods that any instance of an Object created by it will get.


```
console.log(Object.keys(Object.getOwnPropertyDescriptors(Array.prototype)));
/*
[
  'length',      'constructor',    'concat',
  'copyWithin',  'fill',           'find',
  'findIndex',   'lastIndexOf',    'pop',
  'push',        'reverse',        'shift',
  'unshift',     'slice',          'sort',
  'splice',      'includes',       'indexOf',
  'join',        'keys',           'entries',
  'values',      'forEach',        'filter',
  'flat',        'flatMap',        'map',
  'every',       'some',           'reduce',
  'reduceRight', 'toLocaleString', 'toString',
  'at',          'findLast',       'findLastIndex'
]
*/
```

The instance.__proto__  property lists is a universal property in Javascript that lists out the Constructor function prototype property (or class)


```
function Resource () {
    this.hasValue = true;
}

var iron = new Resource();
console.log(Object.getOwnPropertyDescriptors(iron.__proto__));         // Is equal to
console.log(Object.getOwnPropertyDescriptors(Resource.prototype));     // this
```

Dont missunderstand the Object global Object with a Javascript Object. Meaning, everything in Javascript is a Javascript Object, but it doesnt have to inherit from Object , global Object



```
var pepe = {
}
console.log(Object.keys(Object.getOwnPropertyDescriptors(Object.prototype)));  // Is equal to
console.log(Object.keys(Object.getOwnPropertyDescriptors(pepe.__proto__)));    // This
// ['constructor', '__defineGetter__', '__defineSetter__', 'hasOwnProperty', '__lookupGetter__', '__lookupSetter__', 'isPrototypeOf', 'propertyIsEnumerable', 'toString', 'valueOf', '__proto__', 'toLocaleString']
console.log(Object.getOwnPropertyDescriptors(Resource.prototype));
// { constructor: { value: [Function: Resource], writable: true, enumerable: false, configurable: true } }
```

You can set the prototype of an instance of an Object by using the following syntax
 - 1st) Within the Objects contructor function
Object.setPrototypeOf(this, Object.prototype)
 - 2nd) Outside the constructor, applied to the instance itself afterwards its creation

```
var iron = new Resource();
Object.setPrototypeOf(iron, Object.prototype)
```

Note its constructor function will not change its Prototype , which is defined upon function definition


```
function Resource () {
    this.hasValue = true;
    Object.setPrototypeOf(this, Object.prototype)
}

var iron = new Resource();
console.log(Object.keys(Object.getOwnPropertyDescriptors(iron.__proto__)));    // Equal to
console.log(Object.keys(Object.getOwnPropertyDescriptors(Object.prototype)));  // this
// ['constructor', '__defineGetter__', '__defineSetter__', 'hasOwnProperty', '__lookupGetter__', '__lookupSetter__', 'isPrototypeOf', 'propertyIsEnumerable', 'toString', 'valueOf', '__proto__', 'toLocaleString']
console.log(Object.getOwnPropertyDescriptors(Resource.prototype));     
// { constructor: { value: [Function: Resource], writable: true, enumerable: false, configurable: true } }
```


- Or what is the same, we can establish that the .__proto__ property of any object is been set by its constructor .prototype property , which is in turn been set by the method.

Object.setPrototypeOf(<objectToChange>, <prototypeToApply>)
    
For what we found that this Method has to be applied in the previous way:
<objectTochange>:
 - Cannot be the Constructor function.
 - If this statement is within the Constructor function use this
 - It it is outside, use the object instanciated
<prototypeToApply>:
 - Either, ConstrutorFn.prototype
 - Either, Object.getPrototypeOf(instanceObj)
 - If it works, instanceObj.__proto__



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

 - [Getting to grips with JavaScript’s prototypal inheritance by Max Powell Medium](https://medium.com/@maxfpowell/getting-to-grips-with-javascripts-prototypal-inheritance-d6cd57ddee59)
 - [The JavaScript Standard Library - JavaScript: The Definitive Guide, 7th Edition](https://www.oreilly.com/library/view/javascript-the-definitive/9781491952016/ch11.html)
