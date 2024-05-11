---
title: Javascript Intermediate 8 - Differences between __proto__ and .prototype on Constructor Functions
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

Now that we have established the relationship between `instanceObject.__proto__` (we are going to use `Object.getPrototypeOf())` , and  Constructor.prototype we see that.


```
function Food () {
}

var popcorn = new Food();
console.log(Object.getPrototypeOf(popcorn) === Food.prototype); // True
```

We can set its prototype properties as


```
function Food () {
   this.isEdible = true
}


Object.setPrototypeOf(Food.prototype , {
    hasValue: true
});

var popcorn = new Food();
console.log(popcorn);          // Food {}
console.log(popcorn.hasValue); // true
console.log(Object.getOwnPropertyDescriptors(popcorn));
//   isEdible: { value: true, writable: true, enumerable: true, configurable: true }
```

Note: Food instances such as popcorn , has ownProperties such as .isEdible and .hasValue is inherit from the previous defined Object literal.


Note that for establishing inheritance between two Object constructors , this is a bit of a particular case and it is not as straight to understand.
 - First you need to call the parent constructor function, so the instance of the children will have instantiated all the properties/methods of the parent Constructor


```
function Resource () {
    this.hasValue = true;
}

function Food () {
    Resource.call(this);      // 1st Step 
    this.isEdible = true;
}


var iron = new Resource();
var popcorn = new Food();

console.log(Object.keys(Object.getOwnPropertyDescriptors(popcorn)));// [ 'hasValue', 'isEdible' ]
console.log(popcorn.hasValue); // true
```

In the 2nd step we will add the Object.setPrototypeOf statement on Food constructor , after it we can add new methods to it.


```
function Resource () {
    this.hasValue = true;
}

function Food () {
    Resource.call(this);
    this.isEdible = true;
    Object.setPrototypeOf(Food.prototype, Resource.prototype); // 2nd step added
}

Resource.prototype.whatIam = function () {   // 2nd step added
    return 'Economic Resource';
}

var iron = new Resource();
var popcorn = new Food();

console.log(Object.keys(Object.getOwnPropertyDescriptors(popcorn)));
console.log(popcorn.hasValue);

console.log(popcorn.whatIam());  // Economic resource    2nd Step added
```

## Defining a Static property/method:

Since static property/methods are called from the constructor you will just assign a function to a respective property on the constructor such as

```
function Food () {
}

Food.isEdible = true;
Food.sayHi = function () {
    console.log("Saying Hi!!");
}

console.log(Food.isEdible);  // true
Food.sayHi();                // Saying Hi!!
```

In a nutshell two clarifications to have after this tutorial.
 - Do not confuse between objects prototype and Constructor prototype (class)
 - Objects set its prototype to other object, then they inherit their properties that way (you can view/execute them by inheritance)


```
function Resource () {
    this.hasValue = true;
}
function Food () {
    this.isEdible = true;
}

const iron = new Resource ();
iron.tradeInternationally = true;

const popcorn = new Food ();
console.log(popcorn.tradeInternationally);       // undefined
// popcorn.__proto__ = iron;    // or 
Object.setPrototypeOf(popcorn , iron);
console.log(popcorn.tradeInternationally);       // True
```

When dealing with Constructor prototypes situation is a bit different
Because all the functions in JS (thus Constructors too) , have the Static property .prototype , which sets the __proto__  property of each instance.
Thus

```
console.log(Food.prototype === Object.getPrototypeOf(popcorn)); // True
```


 - The Constructor function prototype is a bit different than the __proto__ property of an object . 
 - It represents an object that all the instances of that Constructor will set its __proto__ to
 - Meaning that, by prototypical inheritance it will inherit all the properties of such a object,
 - In the following case we will be assigning a single function  as the .prototype property of the constructor of the parent Constructor . We could be assigning an object literal too
 - Yet note that the Resource Instance properties such as .hasValue are not inherited


```
function Resource () {
    this.hasValue = true;
}
function Food () {
    this.isEdible = true;
    Object.setPrototypeOf(Food.prototype, Resource.prototype);
}


Resource.prototype.sayHi = function () {         // Assigning
    console.log("Saying Hi !!");
}


const popcorn = new Food ();

console.log(Object.keys(Object.getOwnPropertyDescriptors(popcorn)));  // [ 'isEdible' ] 
popcorn.sayHi();                  // Saying Hi !!
console.log(popcorn.hasValue);    // undefined  ------> NOTE!!
```

If you want the child Constructor to inherit the instance properties of the parent Constructor , you need to call the parent Constructor within the child constructor. This model of inheritance is called , "Recursive constructor Invocation"

```
function Resource () {
    this.hasValue = true;
}
function Food () {
    Resource.call(this);        // -------> Added!! Calling the parent constructor
    this.isEdible = true;
//    Object.setPrototypeOf(Food.prototype, Resource.prototype); // Not needed for this example
}

const popcorn = new Food ();

console.log(Object.keys(Object.getOwnPropertyDescriptors(popcorn))); // [ 'hasValue', 'isEdible' ] 
console.log(popcorn.hasValue);    // true
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
