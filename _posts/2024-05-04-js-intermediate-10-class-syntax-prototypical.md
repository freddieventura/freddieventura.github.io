---
title: Javascript Intermediate 10 - Class syntax in Javasript as an abstraction from prototypical inheritance
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

Distinction between fields and properties


Within a class body, there are a range of features available.


The structure of a class definition via the class keyword is

```
    class MyClass {
      // Constructor
      constructor() {
        // Constructor body
      }
      // Instance field
      myField = "foo";
      // Instance method
      myMethod() {
        // myMethod body
      }
      // Static field
      static myStaticField = "bar";
      // Static method
      static myStaticMethod() {
        // myStaticMethod body
      }
      // Static block
      static {
        // Static initialization code
      }
      // Fields, methods, static fields, and static methods all have
      // "private" forms
      #myPrivateField = "bar";
    }
```

Which has almost the same effects than its pseudoclassical syntax counterpart


```
    function MyClass() {
      this.myField = "foo";
      // Constructor body
    }
    MyClass.myStaticField = "bar";
    MyClass.myStaticMethod = function () {
      // myStaticMethod body
    };
    MyClass.prototype.myMethod = function () {
      // myMethod body
    };

    (function () {
      // Static initialization code
    })();
```

One limitation of pseudoclassical syntax is that it doesnt allow for private fields.

One exception from classes syntax to pseudoclassical syntax is that the class declarations are not hoisted. Meaning you cannot instantiate a class before declaring it.


```
new MyClass(); // ReferenceError: Cannot access 'MyClass' before initialization
class MyClass {}
```

So we can see that the pseudoclassical expression becomes


```
function Food (name) {
    this.isEdible = true;
    this.name = name;
    this.eating = function () {
        console.log('I am eating ' + this.name);
    }
}
Food.sayHi = function () {
    console.log('Saying Hi !!');
}
```

This with classes


```
class Food {
    constructor (name) {
        return this.name = name;
    };
    isEdible = true;
    name = '';
    eating () {
        console.log('I am eating ' + this.name);
    };
    static sayHi () {
        console.log('Saying Hi !!');
    }
}

const popcorn = new Food('Popcorn');
popcorn.eating();
Food.sayHi();
```

On constructors type checking:
 - In Javascript there is no constructor overloading like in Java, this feature lets you assign more than 1 constructor per Class , so depending on the datatypes passed on the arguments and number of arguments function will choose either a different constructor funtion.
 - Although this feature is not supported in the syntax, you can create a conditional statement to achieve it by using the arguments Array or the constructor 


```
class Food {
    constructor() {
        if (arguments.length == 1) {
            if (typeof arguments[0] == 'string' ) {
                this.name = arguments[0];
            } else {
                throw new Error('Incorrect datatypes');
            }
        } else if(arguments.length == 2) {
            if (typeof arguments[0] == 'string' || typeof arguments[1] == 'number' ) {
                this.name = arguments[0];
                this.kcal = arguments[1];
            } else {
                throw new Error('Incorrect datatypes');
            }
        } else {
                throw new Error('Expected 1 < arguments < 3');
        }
    }
}

const popcorn = new Food('Popcorn', 20);
const leaves = new Food('leaves');
console.log(popcorn);
console.log(leaves);
```

Note is preferable to use the rest parameter using args as a variable, also the previous syntax can be simplified to


```
class Food {
    constructor(...args) {
        if (args.length === 1 && typeof args[0] === 'string') {
            this.name = args[0];
        } else if (args.length === 2 && typeof args[0] === 'string' && typeof args[1] === 'number') {
            this.name = args[0];
            this.kcal = args[1];
        } else {
            throw new Error('Incorrect datatypes or number of arguments');
        }
    }
}
```

## Derived Classes

We are going to introduce two keywords , two features. extends and super
Two classes can be related together in a relationship of derivation
Say we have got : firstClass , secondClass 
If secondClass extends firstClass , then we can say that:
 - firstClass is the "base class" or "superclass"
 - secondClass is the "derived class" or "subclass"

In its most basic form : by just adding the `extends` keyword on the subclass declaration, the subclass instances will declare the properties/methods of the superclass


```
class Resource { 
    hasValue = true;
}
class Food extends Resource {
    isEdible = true;
}

var popcorn = new Food ();
console.log(popcorn); // Food { hasValue: true, isEdible: true }
```

## Super keyword
Additionally, you can use now the `super` keyword in the constructor function of the subclass
 - super has to be called before any this. statement is used 
 - super calls the constructor of the superclass


```
class Resource { 
    hasValue = true;
    constructor () {
        console.log('Creating a resource');
    }
}
class Food extends Resource {
    isEdible = true;
    constructor () {
        super();
        console.log('Creating food');
    }
}

var popcorn = new Food ();   // Creating a resource
                             // Creating food
```

`super` can be used from static methods/properties to access static methods/properties


```
class Resource { 
    hasValue = true;
    static viento = 'Aire';
}
class Food extends Resource {
    isEdible = true;
    static sayWhat () { 
        console.log(super.viento);
    }       
}

var popcorn = new Food ();   
Food.sayWhat();     // Aire
```

`super` cannot be used to access the instance field of a superclass , as instance fields are set on the instance instead of on the constructor


```
class Base {
 baseField = 10;
}

class Extended extends Base {
 extendedField = super.baseField; // undefined
}
```

Final note *A class can only extend from one class.*

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
 - [Difference between field and property by Kamlesh Singh Medium](https://medium.com/@kamaleshs48/difference-between-field-and-property-34c122aa429c)
