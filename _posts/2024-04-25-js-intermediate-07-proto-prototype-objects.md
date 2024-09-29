---
title:  Javascript Intermediate 7 - Differences between __proto__ and .prototype on Objects
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---


In its most basic form __proto__ is a built-in property in all objects in JS , that you set to other object so it inherits its properties.
The firstObject will inherit secondObjects properties , and the properties of the secondObject.__proto__ object and so on.

```
let tangible = {
    isTouchable: true
}
let resource = {
    hasUtility: true,
    __proto__: tangible
}

let food = {
    isEdible: true
}
food.__proto__ = resource;

console.log(food);              // { isEdible: true }
console.log(food.hasUtility);   // true
console.log(food.isTouchable);  // true
```

In JS all the functions can work as Constructors using the new keyword. If instanciated an Object via a constructor function , then the __proto__ of the instance will be identical to its Constructor function .prototype property

```
function Food () {
    this.isEdible = true;
}

var pasta = new Food();

console.log(Food.prototype === pasta.__proto__);  // true
```

Remember that non enumerable properties wont be listed , which happens with are almost all the properties of the built in objects, so if you want to see them use the following method.
`Object.getOwnPropertyDescriptors(<myObject>);`
Even better if you make an array
`Object.keys(Object.getOwnPropertyDescriptors(<myObject>));`

__proto__ has some limitations.

 - The references cant go in circles. JavaScript will throw an error if we try to assign __proto__ in a circle (if for instance food is at the same time __proto__ of tangible)
 - The value of __proto__ can be either an object or null. Other types are ignored.
 - __proto__ can only be one object !!
 - __proto__ is a historical getter/setter of [[Prototype]] , which is the actual property
 
 
 - Prototyped accessed properies are fallback properties in the sense that (On data properties)
   - Attempting to Delete them in the child object has no effect (cause there is in fact no such as property)
   - Attempting to Write them on the child object will create an actual property on the child object leaving untouched the parents property

```
let resource = {
    hasUtility: true,
    hasHumanLabour: true
}

let food = {
    isEdible: true,
    __proto__: resource
}

delete food.hasUtility ;
console.log(resource.hasUtility);  // true
console.log(food.hasUtility);      // true

food.hasHumanLabour = false;
console.log(resource.hasHumanLabour);  // true
console.log(food.hasHumanLabour);      // false
```

(On Accessor properties) such as getters and setters the behaviour is derived of what it happens under the hood.
 - Attempting to Delete them in the child object has no effect
 - Attempting to write or read them will work as normal (because they are calling the function anyways)


```
let resource = {
    hasUtility: true,
    price: 0,
    set setPrice(price) {
        this.price = price;
    },
    get getPrice() {
        return this.price;
    }
}

let food = {
    isEdible: true,
    __proto__: resource
}


delete food.setPrice ;
food.setPrice = 20;
console.log(food.getPrice);    // 20
```

Last clarification: this keyword will correspond to the object calling it

```
let resource = {
    hasUtility: true,
    name: '',
    set setName(name) {
        this.name = name;
    },
    get getName() {
        return this.name;
    }
}

let food = {
    isEdible: true,
    __proto__: resource
}


food.setName = 'Rabioli';
console.log(food.getName);    // Rabioli
```

 You can demostrate that an object is prototype of some other object with 
`Object.prototype.isPrototypeOf(obj)`
For oposition `instanceof` wont work the other way around as it is not comparing object to object but object to Constructor


```
let resource = {
    hasUtility: true,
}

let food = {
    isEdible: true,
    __proto__: resource
}


console.log(resource.isPrototypeOf(food));    // True 
console.log(food instanceof resource);        // Type Error (right hand side not-callable)
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
