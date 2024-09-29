---
title: Javascript Intermediate 4 - Object Data Properties vs Accesor Properties
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

In the same way you can create Accesor properties which are a bit different.

Associates a key with one of two accessor functions (  get and  set ) to retrieve or store a value.

An accessor property has the following attributes:

```
 get

    A function called with an empty argument list to retrieve the
    property value whenever a get access to the value is performed. See
    also getters . May be  undefined .

 set

    A function called with an argument that contains the assigned value.
    Executed whenever a specified property is attempted to be changed.
    See also setters . May be  undefined .

 enumerable
    (like on Data properties)
 configurable
    (like on Data properties)
```

This accessor properties can have both defined a set attribute and a get attribute under a single property
Though this is not common


```
var myMan = {
}

Object.defineProperty(myMan , '_name' , {
    value: 'Zack',
    writable: true,
    enumerable: false,
    configurable: false
});
Object.defineProperty(myMan , 'getsetName' , {
    get() {
        console.log('Getting myMan');
        return this._name;
    },
    set(name) {
        console.log('Setting myMan');
        return this._name = name;
    }

});

console.log(myMan._name); // Zack
myMan.getsetName = 'Rishi'; // Setting myMan
console.log(myMan.getsetName); // getting myMan Rishi
```

The default behaviour by using Object literals with the set or get syntax would be to place them in a separate property


```
var myMan = {
    _name: 'Zack',
    get getName() {
        console.log('Getting myMan');
        return this._name;
    },
    set setName(name) {
        console.log('Setting myMan');
        return this._name = name;
    }
}

console.log(myMan._name); // Zack
myMan.setName = 'Rishi';  // Setting myMan
console.log(myMan.getName);  // Getting myMan Rishi
```

A clarification, setters wont let you override a Data Property if this is writable: false
They are not made for that.

```
var myMan = {
}

Object.defineProperty(myMan , '_name' , {
    value: 'Zack',
    writable: false,
    enumerable: false,
    configurable: false
});
Object.defineProperty(myMan , 'setName' , {
    set(name) {
        console.log('Setting myMan');
        return this._name = name;
    }
});
Object.defineProperty(myMan , 'getName' , {
    get() {
        console.log('Getting myMan');
        return this._name;
    }
});

console.log(Object.getOwnPropertyDescriptors(myMan));

console.log(myMan._name); // Zack
myMan.setName = 'Rishi';
console.log(myMan.getName); // Zack
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
