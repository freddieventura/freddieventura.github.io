---
title: Javascript Intermediate 11 - Traversing the Javascript standard-library Object Model
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

First of all there is not a clear definition of **Standard-Library** for Javascript as it depends on the context of execution that varies a lot , depending if you are running it on a Browser, or on a container such as `node.js` we will be using the later one.

The global object is binded to globaThis
So you can see all of the Global Objects derived from it by


```
console.log(Object.getOwnPropertyDescriptors(globalThis));
```

Or just listing them

```
console.log(Object.keys(Object.getOwnPropertyDescriptors(globalThis)));
```

Or simplyfied with

```
Reflect.ownKeys(target) 
console.log(Reflect.ownKeys(globalThis));
```

We can in turn check each of the Objects from this namespace by

```
console.log(Reflect.ownKeys(globalThis.Object));
console.log(Reflect.ownKeys(globalThis.Function));
```

globalThis is linked to the root of the namespace , so you can alsoc

```
console.log(Reflect.ownKeys(Object));
console.log(Reflect.ownKeys(Function));
```

Note that all of these Objects correspond to Constructor Functions (aka Classes in JS)
In which you can check its:

```
// Static Methods/Properties
console.log(Reflect.ownKeys(Array));

// Instance Methods/Properties
console.log(Reflect.ownKeys(Array.prototype));
```

On checking the prototype chain on built-in objects
Provided that we can check if an object is prototype of another object ,by accesing to its instance propery `.isPrototypeof()`

```
const resource = {
}
const food = {
}

console.log(resource.isPrototypeOf(food));    // False
Object.setPrototypeOf(food , resource);
console.log(resource.isPrototypeOf(food));    // True
```

Be careful in here , we are going to check Constructors prototype and see if they are prototype of other object

```
// From the expression Object.prototype.isPrototypeOf(object) ;

console.log(ParentConst.prototype.isPrototypeOf(ChildConst));
```

We can see that all the Constructors have Object constructor as its prototype. Not the other way around


```
console.log(Object.prototype.isPrototypeOf(Array));    // True
console.log(Array.prototype.isPrototypeOf(Object));    // False


console.log(Object.prototype.isPrototypeOf(Function)); //  True
console.log(Object.prototype.isPrototypeOf(Number));   //  True

// ...


console.log(Object.prototype.isPrototypeOf(Math));   //  True
console.log(Number.prototype.isPrototypeOf(Math));   //  False
```

Notice that we can see if we create a Constructor , it is inheriting from Function and also from Object


```
function Food () {
}
console.log(Object.prototype.isPrototypeOf(Food));    // True
console.log(Function.prototype.isPrototypeOf(Food));    // True
```

Once instantiated , it will inherit from Object , but not from Function anymore as it is not a constructor


```
function Food () {
}

var popcorn = new Food(); 

console.log(Object.prototype.isPrototypeOf(popcorn));    // true
console.log(Function.prototype.isPrototypeOf(popcorn));  // false
```

Both the instance of an empty constructor and an emtpy object literal have no ownKeys

```
const popcorn = new Food(); 
const objectLiteral = {
}
console.log(Reflect.ownKeys(popcorn)); // []
console.log(Reflect.ownKeys(objectLiteral)); // []
```

Still by inheritance they have access to at least the Object.prototype properties/methods

```
console.log(Reflect.ownKeys(Object.prototype)); 
['constructor', '__defineGetter__', '__defineSetter__', 'hasOwnProperty', '__lookupGetter__', '__lookupSetter__', 'isPrototypeOf', 'propertyIsEnumerable', 'toString', 'valueOf', '__proto__', 'toLocaleString']

console.log(popcorn.toLocaleString);   // [Function: toLocaleString]
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
