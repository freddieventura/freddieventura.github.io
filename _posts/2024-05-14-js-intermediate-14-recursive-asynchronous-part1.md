---
title: Javascript Intermediate 14 - Recursive calling for asynchronous effects part 1
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---

I am trying to do a clock handle spinning
It will tick in one of the 8 positions according to the clockhandle been 0 degrees 45 Degrees etc.. | / - \

The clock will spin a number of times (noSpins) in those 8 positions , taking from position to position a time in ms (delay)

Initially my function would have been


```
function clockSpin (noSpins , delay) {
    const clockTicks = ['|', '/', '-', '\\', '|', '/', '-', '\\'];
    let tickNo = 0;

    for (i = 0;i < noSpins; i++) {
        tickNo = setTimeout( () => {
            return tickNo++
        }, delay);

        let tickReg = tickNo % 7;
        console.log(clockTicks[tickReg]);
    }
}

clockSpin(20 , 10000);
```

But this executes all the setTimeout() sequentially as soon as i=0 jumps from i=1 , jumps from i=2
each setTimeout() execution is not for the previous setTimeout() to finnish

For this you need to recursively call this function

```
function clockSpin(noSpins, delay) {
    const clockTicks = ['|', '/', '-', '\\', '|', '/', '-', '\\'];
    let tickNo = 0;

    function spin() {
        if (tickNo < noSpins) {
            setTimeout(() => {
                let tickReg = tickNo % 8; // corrected index range
                console.clear();
                console.log(clockTicks[tickReg]);
                tickNo++;
                spin(); // Recursive call to start next iteration after delay
            }, delay);
        }
    }

    spin(); // Start the first iteration
}

clockSpin(20, 1000); // Adjusted delay for better visualization
```

Basically sustitute the for loop for the async wrapper function lets call it asyncProcess()
It will be called recursively within the higher-order function setTimeout


```
function spin() {
    if (tickNo < noSpins) {
    }
}
```

Which synthetically can be

```
function myAsyncFunction(delay) {
    let maxIterations = 20;
    let i = 0;
    function asyncProcess() {
        if (i < maxIterations ) {
            setTimeout( () => {
                console.log('Doing some asyncProcess step: ' + i);
                asyncProcess();     // Recursive call
            }, delay);
        }
    }

    asyncProcess();     // Start the first Iteration
}

myAsyncFunction(1000);
```

Then inside the call of the async function , you need to nest the recursive call to that


## Creating an asynchronous function

Now we are going to complicate a bit more this
Lets do an async function that process some data, and stores some data too
So say this following function will get one letter of the alphabet, and keeps it in a buffer, when the whole function finnishes will output that buffer


Now we need to create an exit condition of the recursion which will process the asyncBuffer


```
function myAsyncFunction(delay, callback) {
    let abcArray = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o'];
    let maxIterations = 15;
    let i = 0;
    let asyncBuffer = '';         // ---> CREATING THE ASYNCBUFFER

    function asyncProcess() {
        if (i < maxIterations) {
            setTimeout(() => {
                asyncBuffer += abcArray[i];
                i++;
                asyncProcess(); 
            }, delay);
        } else {                   // ---> EXIT CONDITON
            callback(asyncBuffer); // ---> USING THE CALLBACK TO PROCESS THE BUFFER
        }
    }

    asyncProcess(); // Start the first iteration
}

myAsyncFunction(100, function(result) {
    console.log(result); // Output the accumulated result when all iterations are completed
});

Seeing its signature would be something like

function myAsyncFunction(delay: number, callback: (result: string) => void): void
```

Seeing its signature would be something like

```
function myAsyncFunction(delay: number, callback: (result: string) => void): void
```

Now we can see the signatures of the higher-order functions , the ones that have callback functions as arguments

In this case this function accept two parameters 
 - delay: number    , One var name called delay , which is of number datatype
 - callback: (result: string) => void    , Second parameter is a callback function which in turn accepts
 - result: string , it returns nothing 
        
This means that the callback function can be something like


```
function(result) {
    console.log(result);
}
```

Which will be executed upon exit of recursion
Or simplified


```
myAsyncFunction(100, (result) => console.log(result)); // abcdefghijklmno
```
Note that the name of the parameter is not strict, you can change it an it will work nicely

```
myAsyncFunction(100, (res) => console.log(res));       // abcdefghijklmno
```


## Adding a failureCallback


Lets complicate this function so:
 - Upon finnishing reading the alphabetAray (abcArray), it will check for the length of the buffer , asyncBuffer
 - If asyncBuffer length is below 15 , then the process has failed somehow.


We will be doing two things, first introducing a fail check when appending the data of the array to the buffer


```
if (abcArray[i]) asyncBuffer += abcArray[i];
```

Then on the Exit condition of the Recursive function, we will do a check, which will call the success callback


```
function myAsyncFunction(delay,successCallback, failureCallback) {
    let abcArray = ['a', 'b', 'c', 'd', 'e'];
//    let abcArray = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o'];
    let maxIterations = 15;
    let i = 0;
    let asyncBuffer = '';

    function asyncProcess() {
        if (i < maxIterations) {
            setTimeout(() => {
                if (abcArray[i]) asyncBuffer += abcArray[i];
                i++;
                asyncProcess();
            }, delay);
        } else {
            if (asyncBuffer.length >= 15) {
               successCallback(asyncBuffer); // Call the successCallback with the accumulated result
            } else {
                failureCallback('AsyncBuffer length is below 15.'); // Call the failureCallback
            }
        }
    }

    asyncProcess(); // Start the first iteration
}

myAsyncFunction(
    100, 
    (res) => console.log('Success:', res),
    (err) => console.log('Failure:', err)
);
```

Now you can comment out abcArray and uncomment its different forms and check

Now its signature is


```
function myAsyncFunction(
    delay: number, 
    successCallback: (result: string) => void, 
    failureCallback: (error: string) => void
): void
```

And will take us to the next chapter

Note that for the previous examples we have used the global object setTimeout(), so we could simulate processes that take a bit of computing time. 
Though in the last part of the tutorial we have created a fully pleged asyncProcess().
This setTimeout() function is not needed now , it helped at the bengining to identify the issue, but as long as you keep the basic boiler-plate we can establish it like this


```
function myAsyncFunction(someParam, successCallback, failureCallback) {
    let maxIterations = 15;
    let i = 0;
    let asyncBuffer = '';

    function asyncProcess() {
        if (i < maxIterations) {
            // SOME CODE BLOCK HERE USING someParam
            i++;
            asyncProcess();
        } else {
            if (asyncBuffer.length >= 15) {
               successCallback(asyncBuffer); // Call the successCallback
            } else {
                failureCallback('Error message'); // Call the failureCallback
            }
        }
    }

    asyncProcess(); // Start the first iteration
}

myAsyncFunction(100, (res) => console.log('Success:', res), (err) => console.log('Failure:', err));
```

Note that the exit conditions on the else, `(i >= maxIterations)` it will call one of the callback functions


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
