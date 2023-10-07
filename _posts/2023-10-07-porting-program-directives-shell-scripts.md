---
title: Porting program directives into shell scripts
tags:  bash, dash, shell, linux, interpreter, commandline, syntax
article_header:
  type: cover
  image:
    src: images/porting-program-directives-shell-scripts-01.jpg
---

## Introduction

Some programs operations may include commands that can allow you to script them from your shell, such as **bash**.
Moreover , some programs allow for a whole set of scripting capabilities within itself, but been able to ultimately chain them with some interpreter logic in order to perform some more complex actions.

### Tmux case of study

For instance in `tmux` terminal multiplexer for Linux, and other shell systems, the program has such a wide array of commands that customize the user experience, which can be checked on its manpage [man tmux](https://man7.org/linux/man-pages/man1/tmux.1.html)

Say that I want to display a message, this command can be runned in three different places:
    - Inside the program
    - In the configuration file
    - From the command line interpreter (the shell, probably **bash**)

Say for the command 
```
display "Hello World"
```

Will show that message in the status bar of the program.

Running it Inside the program would be

```
:display "Hello World"
```

Running it from its config file `$HOME/.tmux.conf` just append on it the following line.

```
display "Hello World"
```

In order to run it from the shell you need to just append the `tmux` to it and it will be runned.
Ommitting some program specific particularities the former command can be runned as

```
$ tmux display "Hello World"
```

This opens up a whole new dimension of scripting capabilities by hooking up shell scripts performing program calls in and out, creating some more complex form of logic and functionalities.


### A syntax colission

Been `tmux` a complex program itself, the commands can be taking quite sophisticated forms that can collide with the syntax of shell.
Let's take the example of the `tmux` command `bind-key` which can bind to certain key-presses some actions.
Simplified and in a nutshell the command can be explained as
```
bind-key <key> <command01> /; <command02> /; ... /; <commandn>
```
So say I do

```
bind-key a display -p "Hello" \; display -p "World"
```

That would be working from Inside the program , and in its configuration file, but as soon as you try to execute it in the shell

```
$ tmux bind-key a display -p "Hello" \; display -p "World"
World
```

And by pressing a , I will only get the "Hello" message
The reason being is that the shell is "swallowing" the `\;` characters , saying , \ for escape the next character, ; for sequencing the execution of the commands.
Thats why it prints then "World" to stdout

One way of solving this situation is to escape these succession of characters by quoting them , according to `man bash` *Quoting  is  used  to remove the special meaning of certain characters or words to the shell.*. So that is our ally

```
tmux bind-key a display -p "Hello" '\;' display -p "World"
```

Will be the command corrected now

### List of conflicting characters

As you can see, you need to be careful with what the shell can interpret, in terms of characters telling the shell to perform a certain expanwould sion, or to escape some characters etc...

Doing a 'man bash' checking for different expansions and etc... having an in-depth understanding of those rules would be the ultimate objective giving you a clear understanding on what conflicts may arise on such a "translation", but in a nutshell It is important to pay attention to the following characters.

- Quotes : Single (') and double (")
- Escape characters: Such as the backslash ('\')
- Semicolon (;)
- Ampersand (&)
- Pipe (|)
- Dollar Sign ($)
- Parentheses (()) and Curly Braces ({})
- Brackets ([]) and Wildcards (*, ?, [])
- Whitespaces
- Quotes in Quotes:
- Hashtags for Comments #.

