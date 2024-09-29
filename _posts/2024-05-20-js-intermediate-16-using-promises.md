---
title: Javascript Intermediate 16 - Using promises
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

Javascript allows concurrency in the sense that some functions can be executed and putted on the background , while they wait for some external process to finnish (send a message) such as I/O operations , network operations etc... . 

Using promises will help and clarify the way we deal with asynchronous operations in a syncrhonous way.
Asnychronous running vs syncrhonous running comes from the way of dealing with functions that "branch out" of the normal sequencial execution , via the Callstack , waiting for a process to finnish in the background (waiting for an Event to happen)
When that operation send this event to the JS engine it gets recorded in the Callback queue. So it will queue that Callback Function related to that event.

As explained before, as soon as the Callstack is cleared out, the Event Loop will start executing the Callback queue in order.


There are two ways of dealing with this. You can asyncrhonously or syncrhonously run this associated callback Functions

 - Asynchronous execution means that each of this functions will be triggered in order according to the order of Receiving this Event Messages
        
 - Synchronous execution is a way to sequence the execution of this Associated code, ensuring that only after the completion of one async process the nextone will take place and so on... , this will ensure the sequencial order of execution of processes regardless of the completion of them.


Promises are a Global Object that define a simple syntax to deal with these operations


```
console.log(Reflect.ownKeys(Promise));
// ['length', 'name', 'prototype', 'all', 'allSettled', 'any', 'race', 'resolve', 'reject', Symbol(Symbol.species)]
console.log(Reflect.ownKeys(Promise.prototype));
// ['constructor', 'then', 'catch', 'finally', Symbol(Symbol.toStringTag)]
```

For our example we will be Creating a process that gets delayed, waiting for that timer to return an event which upon completion will register a callbackFunction in the Callback Queue

We wont be using callback functions anymore. Lets use a promise straightaway , for this the function needs to return a Promise , with a resolveFunc and rejectFunc (former not strictly neededfor the example although real processes may get rejected)


 - The function needs to return the promise Object
 - The resolveFunc , needs to be executed and will be the exit point of the function.
Note!!
   - The resolveFunc() , needs to be executed at the end of async Function.
   - If executed outside the async Function , eventhought it is inside the Promise, it will be recorded in the call

## Wrong way of calling it


```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            console.log(message);
        }, delay)
            resolveFunc(); // This is called immediately after setTimeout is called
    })
}


delayedMessage(300, 'First Message')
    .then( () =>  delayedMessage(200, 'Second Message'))
    .then( () =>  delayedMessage(100, 'Third Message'))
// Third Message
// Second Message
// First Message
```

In this previous example the resolveFunc() is placed outside the setTimeout callbackFunc(), which means it is invoked inmediatly after the setTimeout() function is called (outside it) .
 - This results in the resolvFunc() being called before the delayed specified by setTimeout and as a result promises are resolved almost inmediately.
 - Upon execution what it happens is that the program will run all the functions in its callstack , once it finnish , it will go to the callbackstack

 - The callbackstack will be formed in the order the functions have finnished their execution, and as the resolvFunc() is outside the setTimeout Callback , it will get executed inmediately.
 - So the order of the callback stack will be the order in which the Async operations had finnished running, effectively the delay will determine the messages to output (console.log(message)


Doing it the good way is basically nesting its execution. If you call the resolveFunc() within the setTimeout, you will be calling it recursively.
 - It will print the message then call the next callback passed with .then , as so forth.

```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            console.log(message);
            resolveFunc(); // Calling resolveFunc(); recursively
        }, delay)
    })
}


delayedMessage(300, 'First Message')
    .then( () =>  delayedMessage(200, 'Second Message'))
    .then( () =>  delayedMessage(100, 'Third Message'))


// First Message
// Second Message
// Third Message
```

Mind also that adding some messages outside the Async function , will get executed always first as they are part of the main function. They will live in the callstack queue before the other code that starts its execution in the callback queue

```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            console.log(message);
            resolveFunc(); 
        }, delay)
    })
}

console.log('Callstack one');   // --------> Directly to the callstack queue
console.log('Callstack two');   //--------> Directly to the callstack queue

delayedMessage(300, 'First Message')
    .then( () =>  delayedMessage(200, 'Second Message'))
    .then( () =>  delayedMessage(100, 'Third Message'))
```

If our function is returning some data, this will be sent as the argument on resolveFunc(returnVar)
Now handling it it is a bit different , as remember if you are going to use more than one line in the arrow function you need to be using the return statement.



```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            resolveFunc(message); 
        }, delay)
    })
}


delayedMessage(300, 'First Message')
    .then( (message) => { 
        console.log(message);
        return delayedMessage(200, 'Second Message');
    })
    .then( (message) => { 
        console.log(message)
        return delayedMessage(100, 'Third Message');
    })
    .then ( (message) => {
        console.log(message)
    });
```


We havent used rejectFunc , but it should be called upon failure of completion of the procedure Also recursively (inside setTimeout()) 
 - See in this example if we pass on a message that is too long it will be rejected
 - See for the chained execution , .catch() it will be the last method to call in the chain.
 - If any of the executions is rejected , at that time this will return the Error message , and not carry on with the following .then()
 - Note in the example you can change the commented out lines to appreciate that


```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            if ( message.length < 20 ) {
                resolveFunc(message); 
            } else {
                rejectFunc('Message too long');
            }
        }, delay)
    })
}


delayedMessage(300, 'First Message')
    .then( (message) => { 
        console.log(message);
 //       return delayedMessage(200, 'Second Message');
        return delayedMessage(200, 'Second Message which is gonna be rejected');
    })
    .then( (message) => { 
        console.log(message)
        return delayedMessage(100, 'Third Message');
//        return delayedMessage(100, 'Third Message which is gonna be rejected');
    })
    .then ( (message) => {
        console.log(message)
    })
    .catch ( (error) => {
        console.log(error);
    });

// First Message
// Message too long
```

Note , API's written for asyncrhonous purposes need to follow some guidelines. In this case the calls of either callbacks (resolveFunc) or (rejectFunc) they are executed in the context of setTimeout() , if not it would be called syncrhonously , added to the  normal callstack (not the callback queue) , and will be executed always first.
This would lead to a "state of Zalgo" which is undesirable


```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        if ( message.length < 20 ) {
            setTimeout ( () => {
                resolveFunc(message); 
            }, delay)
        } else {
                rejectFunc('Message too long'); // This would go straightaway to the callstack
        }
    })
}
```


Promise.prototype.finally() method , will be executed upon completion of the execution of the promise block, (we can say that the promise has been settled) , this code will be execute regardless of if any .catch() has been found on the promise block or not



```
function delayedMessage(delay, message) {
    return new Promise ( (resolveFunc, rejectFunc) => {
        setTimeout ( () => {
            if ( message.length < 20 ) {
                resolveFunc(message); 
            } else {
                rejectFunc('Message too long');
            }
        }, delay)
    })
}


delayedMessage(300, 'First Message')
    .then( (message) => { 
        console.log(message);
 //       return delayedMessage(200, 'Second Message');
        return delayedMessage(200, 'Second Message which is gonna be rejected');
    })
    .then( (message) => { 
        console.log(message)
        return delayedMessage(100, 'Third Message');
//        return delayedMessage(100, 'Third Message which is gonna be rejected');
    })
    .then ( (message) => {
        console.log('Promise Fulfilled');
        console.log(message);
    })
    .catch ( (error) => {
        console.log(error);
    })
    .finally ( () => {
        console.log('Promise Settled');
    });

// First Message
// Message too long
// Promise Settled
```

 We can determine 2 states of a promise then
 - Pending: The promise is waiting for an asyncrhonous operation to complete
 - Settled: When the promise has finnished all its operations
   - Fulfilled (Resolved): All the operations had exited by the resolveFunc();
   - Rejected: At least one of the operations had exited by the rejectFunc();


Unless we create our own library, the most common scenario is that we are going to use API's with Object methods that return promises, so lets use one.

Lets use  fetch api()
Fetching a website , note from mdn.webapi


```
# API fetch() global function # 
    fetch(resource, options) 
 resource 
    -   A string or any other object with a stringifier including a 
 options  Optional  
Return value 
A  Promise that resolves to a  Response object. 

# API Response # 

Instance methods

 Response.text()

    Returns a promise that resolves with a text representation of the
    response body.

Response: text() method
Return value 

A Promise that resolves with a  String . 

```

1st thing will get the response object.


```
fetch('https://lukesmith.xyz')
    .then( (response) => { 
        console.log(response.ok);
    })
    .catch( (reject) => { 
        console.log(reject);
    })
    .finally( () => {
        console.log('Finnished executing fetch()');
    });

// True
// Finnished executing fetch()
```

2nd we will be transforming it to a string with its .text() instance method


```
fetch('https://lukesmith.xyz')
    .then( (response) => { 
        response.text() 
            .then( (text) => {
                console.log(text);
            });
    })
    .catch( (reject) => { 
        console.log(reject);
    })
    .finally( () => {
        console.log('Finnished executing fetch()');
    });


// Finnished executing fetch()
// ALL HTML OF THE WEBSITE TO STDOUT
```

Say that we will be using the wrong url such as lukasmith.xyz


```
fetch('https://lukasmith.xyz')
    .then( (response) => { 
        response.text() 
            .then( (text) => {
                console.log(text);
            });
    })
    .catch( (reject) => { 
        console.log(reject);
    })
    .finally( () => {
        console.log('Finnished executing fetch()');
    });

// TypeError: fetch failed
// ....
//   cause: Error: getaddrinfo ENOTFOUND lukasmith.xyz
// ...
// Finnished executing fetch()
```

We dont want floating promises, this are the ones that dont have a return statement, in our previous case we had a promises that itself called console.log(text); but it was a floating promise in the sense that it didnt return anything.

The best way is to have a .then() , case at the end handling the result
Quoting the mdnjs docs , 

```
Therefore, as a rule of thumb, whenever your operation encounters a
promise, return it and defer its handling to the next  then handler.
```


```
fetch('https://lukesmith.xyz')
    .then( (response) => { 
        return response.text()  // -----> Now it returns a value
            .then();
    })
    .then( (result) => {        // We perform whatever action
        console.log(result);    // we want with the result
    })                          // 
    .catch( (reject) => { 
        console.log(reject);
    })
    .finally( () => {
        console.log('Finnished executing fetch()');
    });


// ALL HTML OF THE WEBSITE TO STDOUT
// Finnished executing fetch()
```

Chaining loads of promises together will end up us having severe nested .then for that we have got the following syntax
async function (we will check them latter)


Running async operations concurrently.
These methods all run promises concurrently - a sequence of promises are
started simultaneously and do not wait for each other. 

There are four composition tools for running asynchronous operations
concurrently:  Promise.all() ,  Promise.allSettled() ,  Promise.any() ,
and  Promise.race() .

The  Promise.all() static method takes an iterable of promises as input 
and returns a single  Promise . This returned promise fulfills when all
of the input's promises fulfill (including when an empty iterable is 
passed), with an array of the fulfillment values. It rejects when any of
the input's promises rejects, with this first rejection reason.


```
function processMessage (delay, message) {
    return new Promise ( (resolveFn, rejectFn) => {
        setTimeout(() =>{ 
            resolveFn(message);
        }, delay);
    });
}

var promiseArray = [processMessage(1000, "firstMessage"), processMessage(2000, "secondMessage")];
Promise.all(promiseArray).then( (resolve) => {
    console.log(resolve);
});
// [ 'firstMessage', 'secondMessage' ]
```

Now the same example but with one of the promises rejecting. 
If any of the promises does reject, it will only exit that message.


```
function processMessage (delay, message) {
    return new Promise ( (resolveFn, rejectFn) => {
        setTimeout(() =>{ 
            if ( message.length < 20 ) {
                resolveFn(message);
            } else {
                rejectFn('Message too long');
            }
        }, delay);
    });
}

var promiseArray = [processMessage(1000, "firstMessage"), processMessage(1000, "this is gonna be failing"), processMessage(2000, "thirdMessage")];
Promise.all(promiseArray).then( (resolveMsg) => {
    console.log(resolveMsg);
})
    .catch( (rejectMsg) => {
        console.log(rejectMsg);
    });

// Message too long
```

Promise.allSettled()

Promise.allSettled(iterable) 
It is similar to Promise.all() , it also waits for the promise to Settle, executing their resolveFn when they fullfill or the rejectFn when they get rejected. It will return an array with object with the value of each promise and its status as fullfilled or rejected



```
function processMessage (delay, message) {
    return new Promise ( (resolveFn, rejectFn) => {
        setTimeout(() =>{ 
            if ( message.length < 20 ) {
                resolveFn(message);
            } else {
                rejectFn('Message too long');
            }
        }, delay);
    });
}

var promiseArray = [processMessage(1000, "firstMessage"), processMessage(1000, "this is gonna be failing"), processMessage(2000, "thirdMessage")];
Promise.allSettled(promiseArray).then( (message) => {
    console.log(message);
})

/*
[
  { status: 'fulfilled', value: 'firstMessage' },
  { status: 'rejected', reason: 'Message too long' },
  { status: 'fulfilled', value: 'thirdMessage' }
]
*/
```

Promise.any()
Basically this method will return the firstone of the promises to be fulfilled (so just a single promise not an array) , omiting the rejections.



```
var promiseArray = [processMessage(2000, "firstMessage"), processMessage(500, "this is gonna be failing"), processMessage(1000, "thirdMessage")];
Promise.any(promiseArray).then( (message) => {
    console.log(message);
})

// thirdMessage
```

Promise.race()
Theseone similar to Promise.any() but it will take the result of the firstone Settled ,taking too rejections. 
In this case we need to use .catch() to catch the errors

```
var promiseArray = [processMessage(2000, "firstMessage"), processMessage(500, "this is gonna be failing"), processMessage(1000, "thirdMessage")];
Promise.race(promiseArray).then( (resolveMsg) => {
    console.log(resolveMsg);
})
    .catch(  (rejectMsg) => {
        console.log(rejectMsg);
    })

// Message too long
```

Using static methods such as Promise.resolve() Promise.reject()
Both of these methods return a promise either resolved or rejected (already fulfilled)
with a certain value.

```
Promise.resolve(value)
Promise.reject(value)
```

This may serve some purposes
 - Converting non-promises into promises.
If you need an object to be used as a Promise in a method that expect a Promise , you can use this

Using the previous example, lets use the Promise.all() method passing an array in which one of the members is a number. We will make it  promise with either one of the Methods .resolve() or .reject()



```
const promiseOne = 25;
var promiseArray = [promiseOne, processMessage(2000, "firstMessage"), processMessage(1000, "thirdMessage")];

Promise.all(promiseArray)
    .then ( (resolveMsg) => { 
       console.log(resolveMsg); 
    })

// [ 25, 'firstMessage', 'thirdMessage' ]
```

- Use it for error handling. We can re-write our async function in


```
function processMessage (delay, message) {
    if (message.length < 4) {
        return Promise.reject('Message is too short');
    }
    return new Promise ( (resolveFn, rejectFn) => {
        setTimeout(() =>{ 
            if ( message.length < 20 ) {
                resolveFn(message);
            } else {
                rejectFn('Message too long');
            }
        }, delay);
    });
}

processMessage(100 , 'lo')
    .then( (resolveMsg) => { 
        console.log(resolveMsg);
    })
    .catch( (rejectMsg) => {
        console.log(rejectMsg);
    })
```

Or you can just start a promise chain with it


```
Promise.resolve()
    .then( (message) => {
       return processMessage(200, 'First Message');
    })
    .then( (message) => {
        console.log(message)
        return processMessage(100, 'Second Message');
    })
    .then ( (message) => {
        console.log('Promise Fulfilled');
        console.log(message);
    })
    .catch ( (error) => {
        console.log(error);
    })
    .finally ( () => {
        console.log('Promise Settled');
    });
```

Note for error handling it is adecuate to throw an error inside the catch , as if not the Promise chain will carry on executing regardless of the .catch()



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

1. []()
2. []()
