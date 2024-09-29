---
title: Javascript Intermediate 3 - Object data properties and its descriptors 
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

# Javascript Intermediate 3 - Object data properties and its descriptors 

Internaly each object data property has a set of attributes that are called descriptors,
Dont confuse the attributes of a property with properties of a property,
The attributes of a property are the further most division of an Object
These are the following

```
    value: 
    writable: 
    enumerable: 
    configurable: 
```

By creating an object with the object literal, we can see its attributes

```
var myMan = {
    name: 'Zack',
    age: 48
}

console.log(Object.getOwnPropertyDescriptor(myMan, 'name'));
// { value: 'Zack', writable: true, enumerable: true, configurable: true }
```

For each property key, there is associated a value , then 3 boolean states
These cannot be setted while creating an object by using the object literal.

```
var myMan = {
    name: {
        value: 'Zack',
        writable: false,
        enumerable: true,
        configurable: false
    }
}

console.log(myMan.name);  // It will return the object , value: 'Zack, writable:false ....
myMan.name = 'Rishi';
console.log(myMan.name); // Rishi
```

Nor By assigning a property to an existing object

```
myMan.name = {
        value: 'Zack',
        writable: false,
        enumerable: true,
        configurable: false
}
```

By declaring an object with an object literal, each property will be deemed to be , writable, enumerable, and configurable

If you wish to fine-tune the creation of your object properties use the
    `Object.defineProperty(obj, prop, descriptor)`


```
var myMan = {
}

Object.defineProperty(myMan, 'name', {
    value: 'Zack',
    writable: false,
    enumerable: true,
    configurable: false
});

console.log(myMan.name); // Zack
myMan.name = 'Rishi';
console.log(myMan.name); // Zack
console.log(Object.getOwnPropertyDescriptor(myMan, 'name')); 
```

Note property descriptors can be modified, if when first defined the property it has been defined with the descript configurable: true


```
var myMan = {
}

Object.defineProperty(myMan, 'name', {
    value: 'Zack',
    writable: false,
    enumerable: true,
    configurable: true
});
Object.defineProperty(myMan, 'name', {
    value: 'Zack',
    writable: true,
    enumerable: true,
    configurable: true
});

console.log(myMan.name); // Zack
myMan.name = 'Rishi';
console.log(myMan.name); // Rishi
```

##  Enumerable properties

Check on the mdnjs tutorial for the table on effects of enumerability of properties but
[Enumerability and ownership of properties - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)
See for these examples that some Methods are able to see those properties some not , depending on the method used to access them and depending on wether this property is enumerable and if it is own or has been inherited

```
var aMan = {
}
Object.defineProperty(aMan, 'fourLegged', {
    value: false,
    writable: false,
    enumerable: false,
    configurable: true
});

Object.defineProperty(aMan, 'hasName', {
    value: true,
    writable: false,
    enumerable: true,
    configurable: true
});


var myMan = {
}

Object.setPrototypeOf(myMan, aMan);

Object.defineProperty(myMan, 'name', {
    value: 'Rishi',
    writable: false,
    enumerable: true,
    configurable: true
});

Object.defineProperty(myMan, 'isRich', {
    value: true,
    writable: false,
    enumerable: false,
    configurable: true
});

console.log(Object.hasOwn(myMan, 'fourLegged'));  // false
console.log(Object.hasOwn(myMan, 'hasName'));     // false
console.log(Object.hasOwn(myMan, 'name'));        // true
console.log(Object.hasOwn(myMan, 'isRich'));      // true
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
