---
title: Javascript Intermediate 15 - Recursive calling for asynchronous effects part 2
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

A callback function associated with an asynchronous code waits for its execution inside either
 - A callback queue
 - A microtask queue
Most of callback functions associated with an asynchronous code have to wait for their execution in the callback queue.
But some of callback functions with asynchronous code , like with the promises wait for their execution in the microtask queue

 - The microtask queue callbacks have priority onto the the callback queue

The job of the event loop is to push callback functions from the microtasks or callback queue into the stack


The nature of Javascript been able to asynchronously run code is due to various capabilities depending on the running environment.
If using Vanilla Javascript , it can be using webworkers that allow run some code in separate background threads 
If using node.js
 Event Loop: 


## The components

There is basically 4 components:
 - Call stack:
 - Event loop:
 - Callback queue:
 - Separate threads operations (such as Node API's, workers)

We know the functioning of the call stack, adding functions to the top Last In first Out, executing each function of our code.

There are some special functions in our enviornment , such as in node.js setTimeout() does run in a separate thread.
 - 1st) It records an event on the Call stack , (event) to wait for
 When that code is executed is not added directly to the call stack , but goes to the Separate thread. When this functions return from that thread they will be added to the Callback Queue

The event loop its constantly checking if the Call Stack is empty , and if it is, it will check for the Callback Queue. 
Functions in the callback queue will be pushed to the Call Stack as soon as the event loop picks them
If both queues are empty , and it doesnt have any pending event recorded to wait for it will finnish the program 

Note , none of the asynchronous functions will run before the main function has finnished runned

In Node.js and browser environments, the event loop manages asynchronous operations and callbacks. It allows JavaScript code to handle I/O operations and timers without blocking the main execution thread.


## Our code example

For instance the following code


```
function printCharmap(charMap,successCallback, failureCallback) {
    let qweCharmap = ['q', 'w', 'e', 'r', 't', 'y', 'u', 'i', 'o', 'p'];
//    let asdCharmap = ['a', 's', 'd', 'f', 'g', 'h', 'j', 'k', 'l', ';'];
    let asdCharmap = ['a', 's', 'd', 'f'];
    let zxcCharmap = ['z', 'x', 'c', 'v', 'b', 'n', 'm', '<', '>', '/'];
    let charmapChosen ;
    let i = 0;
    let asyncBuffer = '';

    switch (charMap) {
        case 'qweCharmap':
            charmapChosen = qweCharmap;
        break
        case 'asdCharmap':
            charmapChosen = asdCharmap;
        break
        case 'zxcCharmap':
            charmapChosen = zxcCharmap;
        break
        default:
            failureCallback('charMap doesnt exists'); // Call the failureCallback

    }

    function asyncProcess() {
        debugger;                       // DEBUGGER BREAKPOINT
        if (i < charmapChosen.length) {
            setTimeout( () => {
                if (charmapChosen[i]) asyncBuffer += charmapChosen[i];
                i++;
                asyncProcess();
            },100);
        } else {
            successCallback(asyncBuffer); // Call the successCallback with the accumulated result
        }
    }

    asyncProcess(); // Start the first iteration
}

/*
// Synchronous execution
printCharmap('qweCharmap',
    (res) => {
        console.log('Success:', res);
        printCharmap('asdCharmap', (res) => console.log('Success:', res), (err) => console.log('Failure:', err) );
    },
    (err) => console.log('Failure:', err)
);
*/

// Asynchronous execution
printCharmap('qweCharmap',
    (res) => {
        console.log('SUCCESS!!:', res);
    },
    (err) => console.log('FAILURE:', err)
);

printCharmap('asdCharmap',
    (res) => {
        console.log('SUCCESS!!:', res);
    },
    (err) => console.log('FAILURE:', err)
);
```

Comment and uncomment accordingly Synchronous execution (sequencial execution) vs asynchronous one (all at the same time)

Asynchronous execution will see both functions executed at the same time, been the shortest to finniish the firstone to return

## Callback hell

See the cumbersome syntax of synchronous execution , if you need processes to be runned sequentially (in a synchronous way or chaining), then you needed to use that syntax. If you need to use 3 or more functions they all have to be nested as callbacks of eachother, leading to a really messy code


## Promises

Taking advantages of the promises syntax we can re-write the code like this


```
function printCharmap(charMap) {
    return new Promise((resolve, reject) => {
        let qweCharmap = ['q', 'w', 'e', 'r', 't', 'y', 'u', 'i', 'o', 'p'];
        let asdCharmap = ['a', 's', 'd', 'f'];
        let zxcCharmap = ['z', 'x', 'c', 'v', 'b', 'n', 'm', '<', '>', '/'];
        let charmapChosen;
        let i = 0;
        let asyncBuffer = '';

        switch (charMap) {
            case 'qweCharmap':
                charmapChosen = qweCharmap;
                break;
            case 'asdCharmap':
                charmapChosen = asdCharmap;
                break;
            case 'zxcCharmap':
                charmapChosen = zxcCharmap;
                break;
            default:
                reject('charMap doesn\'t exist');
                return;
        }

        function asyncProcess() {
            if (i < charmapChosen.length) {
                setTimeout(() => {
                    if (charmapChosen[i]) asyncBuffer += charmapChosen[i];
                    i++;
                    asyncProcess();
                }, 100);
            } else {
                resolve(asyncBuffer);
            }
        }

        asyncProcess();
    });
}

// Using the function with Promises
printCharmap('qweCharmap')

    .then(res => {
        console.log('Success:', res);
        return printCharmap('asdCharmap');
    })
    .then(res => {
        console.log('Success:', res);
    })
    .catch(err => {
        console.log('Failure:', err);
    });
```

Note there is no longer successCallback or failureCallback

We return a new object Promise which gets a callback function with (resolve, reject) as its parameters
```
    return new Promise((resolve, reject) => {
```
The line of sucessCallback is instead that resolve(); parameter executed

The failureCallback is now reject();


Now upon calling this function you can use the following methods
.then
Which takes a callback function with the response parameter
.catch
Which takes a callback function with the error parameter


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

 - [Callback Queue VS Microtask Queue - YouTube](https://www.youtube.com/watch?v=ZeTa2ID2W9E)
 - [6.3 Call Stack, Callback Queue, and Event Loop - YouTube](https://www.youtube.com/watch?v=FVZ-A_Akros)
