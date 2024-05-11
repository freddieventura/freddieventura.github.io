---
title: Javascript Intermediate 12 - Method chaining in Javascript 
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

**Method chaining** is a programming technique to allow better readiblity and writeability of objects, that allow you to call any instance method of the same object on the return object of another instance method.

May be cryptic the definition, but lets see with an example.

You declare a Class , with some instance methods (we are going to use pseudoclassical inheritance but can be doing classes).
Remember the instance methods are defined on the prototype  property of the constructor function.

You want to be able to apply any instance method to the return of that instance method such as.

```
popcorn.yesEdible().notEdible().yesEdible();
```

Indefinitely ...
You just need to create that instance method returning the this keyword, it will return the instance in turn.

```
function Food (name) {
    this.isEdible = true;
}

Food.prototype.yesEdible = function () {
    this.isEdible = true;
    return this;
}
Food.prototype.notEdible = function () {
    this.isEdible = false;
    return this;
}
const popcorn = new Food('Popcorn');
popcorn.notEdible();
console.log(popcorn); // Food { isEdible: false }
popcorn.yesEdible().notEdible();
console.log(popcorn); // Food { isEdible: false }
popcorn.yesEdible().notEdible().yesEdible();
console.log(popcorn); // Food { isEdible: true }
```

Which is the same syntax as

```
popcorn.yesEdible()
    .notEdible()
    .yesEdible()
    .notEdible()

console.log(popcorn); // Food { isEdible: false }
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

## Reference sources
 - [Method Chaining in JavaScript - GeeksforGeeks](https://www.geeksforgeeks.org/method-chaining-in-javascript/)
