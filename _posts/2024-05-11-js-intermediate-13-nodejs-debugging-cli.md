---
title: Javascript Intermediate 13 - Nodejs debugging on the CLI 
tags: javascript, programming, coding, nodejs, syntax
article_header:
  type: cover
  image:
    src: images/js-intermediate-dog-02.png
---


There may be thousands of libraries and utilities that allow you debug **nodejs** projects, but I am going to write few lines on how to do it using the node built-in tool.
Is not nearly as powerful as **gdb** , but using the right strategy to approach the process can clarifying as well as timesaving, as you wont be needing to install some other additional tools.


You can start debugging your proyect via the entry file by
```
node inspect <myfile.js>
help
```

You get the commands over there but mainly we will be focusing on the state of the variables as the code statements are been executed.


First set a breakpoint, someline on your code so you can iterate from there

```
# set breakpoint
sb(<lineNum>)
```

Then you want to basically use one of these three commands

```
next, n               Continue to next line in current file
step, s               Step into, potentially entering a function
cont, c               Resume execution
```

Continue to iterate to the next time you get to the following break point
Next and step if you want to execute one line at a time.
Note that step implies check further into function calls. 
If using Step within buil-in functions and libraries, it will take you to lines of code outside the codebase of the project. 
Just bear that in mind as it may confuse you.

At any prompt point on the debugger (when it is paused) , you can issue commands such as.

```
#exec('<codeToExecute>'
exec('console.log(i)')
```

But what you really want to do, is to watch variables. They will get printed out at each Step.

```
watch('<varName>')
```

Normaly you want to watch the state many variables at the same time, for that you would need like various watch commands. 
What I personally use (because I am going to be repeating the same steps backs and forths) is to write a one-liner to activate such as watch command. 
So at the end of my .js file  I write something like

```
/*
sb(20)
watch('varA');watch('varB');watch('varC');watch('varD');watch('varE');
*/
```

So I remember, the commands I have to do and just focus on reading the output of the debugger instead.

Personally it is a shame that you cannot set a breakpoint somewhere on your code in a deterministic way.
Say you want to target a function entry point (a function call) you check the lineNumber where that happens and do `sb(23)` , but say you modify some lines previous that line , then that `sb(23)` is no longer pointing to the same statement.

If you can and want modify the codeIn node.js you have the following statement which will tell the debugger to create a breakpoint there
```
debugger;                       // DEBUGGER BREAKPOINT
```

But if not , note that breakpoint lineNo
 

Finally to exit the debugger

```
.exit
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

1. []()
2. []()
