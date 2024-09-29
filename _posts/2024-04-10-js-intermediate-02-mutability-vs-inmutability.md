---
title: Javascript Intermediate 2 - Mutability vs Inmutability of datatypes
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

# Javascript Intermediate 2 - Mutability vs Inmutability of datatypes 

It has been said that  Javascript has 6 primitive datatypes

All types except Object define immutable values represented directly at
the lowest level of the language. We refer to values of these types as 
primitive values .

String, Number, Boolean, Undefined, Null, and Symbol.

They can be checked with the typeof operator , except null that you need to use === null

In other sense of the words , Data types are categorized into Primitive and Reference types in JavaScript.

Primitive datatypes are stored on the Stack, while in the Reference datatypes a pointer to the address on the heap where the datatype is stored , that is what it is actually stored. To say is stored by reference.


```
let a = 5;
a = 10;
```


That would mean that you are changing in the stack the value a, now is 10.
Whereas with non-primitives
If you assign a non-primitive to two variables , this will be references to the same object
Thus changing any of the properties in one of the objects will change it in all the objects.



```
const band = {
    name: "Rage Against",
    creation: 1992
}

const band2 = band
console.log(band);    // { name: 'Rage Against', creation: 1992 }
console.log(band2);   // { name: 'Rage Against', creation: 1992 }


band2.name = 'Perales';

console.log(band);    //  { name: 'Perales', creation: 1992 }
console.log(band2);   //  { name: 'Perales', creation: 1992 }
```

For that we can do


```
Object.assign(target)
Object.assign(target, source1, source2, /* Γò¼├┤Γö£├ºΓö¼┬¼, */ sourceN)

const band = {
    name: "Rage Against",
    creation: 1992
}
 
const band2 = Object.assign({}, band)
console.log(band);    // { name: 'Rage Against', creation: 1992 }
console.log(band2);   // { name: 'Rage Against', creation: 1992 }


band2.name = 'Perales';

console.log(band);    //  { name: 'Rage Against', creation: 1992 }
console.log(band2);   //  { name: 'Perales', creation: 1992 }

```

Likewise it can be done by using the spread operator

```
const band2 = {...band};
```

So in contradiction to the Mutability of Reference datatypes that are created by reference , so they are able to be changed.

Primitive datatypes are deemed to be inmutable , in the sense that 

```
const num = 46;
const newNum = num;
```

newNum , would be 46 too , but changing num will not change newNum , they are independent records on the memory stack


## Limiting Mutability of Objects

With Object.preventExtensions(obj) no more properties can be added to the object


```
const band = {
    name: "Rage Against",
    creation: 1992
}
 
Object.preventExtensions(band);

band.firstAlbum = 'Evil Empire';
console.log(band.firstAlbum);   // undefined
```

If using dot notation no error message will be found, but if used Object.defineProperty() we will get error message

```
const band = {
    name: "Rage Against",
    creation: 1992
}
 
Object.preventExtensions(band);

Object.defineProperty(band, 'firstAlbum',{
    value: 'Evil Empire'
}) // TypeError: Cannot define property firstAlbum, object is not extensible
```

In a similar fashion the following methods limit the Mutability of an object


```
    Object.seal(obj)
    Object.freeze(obj)
```

The delete operator deletes a property from an object using the dot notation.

```
    delete object.property
```

Note that freezing or sealing an object wont make it inmutable to its nested properties that are objects itself. You will need to use a construct for that

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

 - [Mutability vs Immutability in JavaScript – Explained with Code Examples](https://www.freecodecamp.org/news/mutability-vs-immutability-in-javascript/)
