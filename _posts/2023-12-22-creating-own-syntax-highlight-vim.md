---
title: Creating your own syntax highlight for vim
tags:  vim, linux, plugin, programming, syntaxhighlight, notes
article_header:
  type: cover
  image:
    src: images/creating-own-syntax-highligh-vim01.png
---

## Introduction

Just doing in here a quick tutorial which will allow you create your own type of files in your computer to be displayed differently in `vim`.

We just going to perform something quickly, as a proof of concept, and I will give you the references to the documentation in which is thoroughly explained.

## Purpose


Say you have some files you want them to be displayed differently on the terminal (not plain white).
In my case my notes, I use them as reference in plain-text files , but seeing that they are starting to accumulate some similarities in syntax, I am seeing the possibility of making them display a bit better in my terminal.

You can name them with your own extension (in my case <filename>.not)

## My notes case

So for instance this is the begining one note file from me `vim l3-learning.not` 

```
 _ _____       _                       _              
| |___ /      | | ___  __ _ _ __ _ __ (_)_ __   __ _  
| | |_ \ _____| |/ _ \/ _` | '__| '_ \| | '_ \ / _` | 
| |___) |_____| |  __/ (_| | |  | | | | | | | | (_| | 
|_|____/      |_|\___|\__,_|_|  |_| |_|_|_| |_|\__, | 
                                               |___/  

# l3-learning
notes from l3 layer networking learning, mainly focused on CCNA , but also guides to set up any kinds of services 

# PACKET TRACER LABS
 ____            _        _     _____                        
|  _ \ __ _  ___| | _____| |_  |_   _| __ __ _  ___ ___ _ __ 
| |_) / _` |/ __| |/ / _ \ __|   | || '__/ _` |/ __/ _ \ '__|
|  __/ (_| | (__|   <  __/ |_    | || | | (_| | (_|  __/ |   
|_|   \__,_|\___|_|\_\___|\__|   |_||_|  \__,_|\___\___|_|   
                                                             
  ____ ____ _   _    _     
 / ___/ ___| \ | |  / \    
| |  | |   |  \| | / _ \   
| |__| |___| |\  |/ ___ \  
 \____\____|_| \_/_/   \_\ 

```

I use [FIGlet](http://www.figlet.org/) ASCII art text generator to create titles , I want them to be displayed differently.


## Actual vim process

- 1st) Creating an automatic filetype detection

[ftdetect](https://vimhelp.org/filetype.txt.html#ftdetect)
You need to create a subdirectory named `ftdetect` somewhere on your `runtimepath` 
Normaly you would do this on your User namespace , so `mkdir -p $HOME/.vim/ftdetect`

- 2nd) Create the extension filetype detector file

```
~/.vim/ftdetect$ vim not.vim
# with the directive
 au BufRead,BufNewFile *.not set filetype=not
```
- 3rd) Create your custom syntax folder
[mysyntaxfile](https://vimhelp.org/syntax.txt.html#mysyntaxfile)

Similarily to step 1 , vim will parse from your `runtimepath` all the files from the subdirectory `syntax` so you want to `mkdir -p $HOME/.vim/syntax`

- 4rd) Creating your own syntax file
```
~/.vim/syntax$ vim not.vim
# with the following content
" Vim syntax file
" Language:	not (for .not , freddieventura notes)
" Current Maintainer: freddieventura (https://github.com/freddieventura)
" Last Change:	2023 Dec 22

" quit when a syntax file was already loaded
if exists("b:current_syntax")
  finish
endif


" Read the sh syntax to start with
" runtime! syntax/sh.vim
" unlet b:current_syntax

syntax match pmenu '[<>\/\-_|()'.,`]\+[<>\/\-_|()'.,`[:space:]]\{5,}\n'
```

- 5th) Intro to defining the vim syntax file directive

[sy-define](https://vimhelp.org/syntax.txt.html#%3Asyn-define)

In regards to syntax , vim understands 3 types of items:

1. Keyword: Single individual words
2. Match: A match against a regex
3. Region: A block in between 2 or more regex.

What I am looking for in here is basicaly to make vim understand where it comes across pieces of text like thisone.

```
 _ _____       _                       _              
| |___ /      | | ___  __ _ _ __ _ __ (_)_ __   __ _  
| | |_ \ _____| |/ _ \/ _` | '__| '_ \| | '_ \ / _` | 
| |___) |_____| |  __/ (_| | |  | | | | | | | | (_| | 
|_|____/      |_|\___|\__,_|_|  |_| |_|_|_| |_|\__, | 
                                               |___/  
```

I guess it could be done by Using a **Region** but provided that we are not completely new on *Regex* , I am gonna try to **Match** every character.

- 6th) Time to regex

So the logic for this regex would be, start looking for characters of the ASCII art such as `|\/-_` etc... , once you get one ocurrence, then check for more of them or whitespaces , a bunch of them say 5 till a line break. No more characters nothing. If you get 5 of them at least I am pretty sure it is going to be this. 

(May the regex not be perfect but it will work)

I recommend you practicing your regex on [Regex101](https://regex101.com/) , you should have known them by now but If you dont and are in a hurry , maybe the community may help you. I have found [IRC Libera](https://web.libera.chat) and pop to `#regex` people are often helpful there.


After you find your regex expression in my case. 

```
[<>\/\-_|()'.,`]\+[<>\/\-_|()'.,`[:space:]]\{5,}\n'
```

Remember vim falvoured Regex are different to otherones. If you are also in doubt and the expression has been correctly formulated, you can translate it using **ChatGPT** most definitely.

- 7th) The Highlight groups

[highlight](https://vimhelp.org/syntax.txt.html#%3Ahighlight)

Prior to writting the directive we need to understand a little concept in regards to vim syntax functioning.
The syntax rules don't go referred directly to styles, so I wont be giving matching this Regex and just tell vim to style it in pink background and so forth.
Vim refer the syntax rules to the **highlight-groups**

They are basicaly Variables, which take an arbitrary name, which the syntax rules are refered to. They in turn have a way of styling that will be marked by the colour scheme (we wont be touching this). 
What you need to see are the current defined **highlight-groups** for the filetype you are operating in. So say I am inside `vim l3-learning.not`  again.

I use the following command

```
:so $VIMRUNTIME/syntax/hitest.vim
```

You can see now an split with highlight-group names, and the way they are been displayed under their current colorscheme. Just pick one of them.
I have chosen `pmenu` (note their are case insensitive) 


- 8th) Crafting your own vim syntax directive

[syn-match](https://vimhelp.org/syntax.txt.html#%3Asyn-match)

Finaly we can go and create our directive. If we can see in the documentation , the signature for the directive to define a syntatical match is.

```
DEFINING MATCHES					*:syn-match*                          
:sy[ntax] match {group-name} [{options}] [excludenl] [keepend] {pattern} [{options}] 
	{group-name}		A syntax group name such as "Comment".       
	[{options}]		See |:syn-arguments| below.                      
	[excludenl]		Don't make a pattern with the end-of-line "$"    
	keepend			Don't allow contained matches to go past a match with the end pattern. 
	{pattern}		The search pattern that defines the match.      
```

And filling in we can finally do the rule

```
syntax match pmenu '[<>\/\-_|()'.,`]\+[<>\/\-_|()'.,`[:space:]]\{5,}\n'
```

Which we will append to our syntax file `~/.vim/syntax/not.vim`
As it has been done before I dont paste again the code.


As you can see in the image of the post you can appreciate how beautiful our notes look like now.


This is call ricing it , innit? :)
