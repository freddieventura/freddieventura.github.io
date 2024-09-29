---
title: Vim Quickfix Lists Advanced Usage
tags:  vim, linux, plugin, programming, quickfix, list, advanced usage
article_header:
  type: cover
  image:
    src: images/creating-own-syntax-highligh-vim01.png
---

## Introduction

[Quickfix Reference Manual](https://vimhelp.org/quickfix.txt.html)

Quickfix Lists and Location Lists are a built in functionality in vim, that can be really useful in many use cases.
It can help you out setting sort of bookmarks within a file for certain search terms.

This ocurrences will be parsed automatically and been shown on a separate buffer (the Quickfix List) in which you can navigate by j down and k up and pressing Enter to access each ocurrence, this will then set your initial buffer into the line of the ocurrence.

Even the file to be parsed can be an entire subdirectory, for instance a project, and this Quick Fix list will help you access each file of the ocurrence in a new buffer.

I can't help but wonder the amount of use cases I have for my workflow, so we will get into oneuse case for me and see how I adapt it to make the perfect usage of it, learning the insights of the tool on the way of customizing our experience. 


## My use case , learning notes

I have simplyied the way I study almost everything in this life by using text files.
From programing languages , to cooking notes, maths, networking technologies, economies etc...

I have got an entire subdirectory of notes which looks similar to

```
$ ls $HOME/notes/
(...)
learning.java.not
learning.js.not
learning.ps1.not
learning.tmux.not
learning.vim.not
(...)
cheatsheet.java.not
cheatsheet.js.not
(...)
```

- The **learning** files are actual notes with code snippets and notes written by myself after having learned something in particular.
- The **cheathseet** files are made of pure documentation dumps for each technology


For instance `cheatsheet.vim.not` will be all the documents in plain-text of the documentation of vim , been parsed.

Starting from A-Z
[Arabic.txt Vim Reference](https://vimhelp.org/arabic.txt.html)
[Workshop.txt Vim Reference](https://vimhelp.org/workshop.txt.html)


This ends up me having a document with 123.684 Lines!!
The reason of storing this offline , is to make my own notes, tracking the progression of functionalities I have already seen, mostly a way to quickly search for  method Signatures that I have already used for a certain API.

Mainly I write labels to separate parts of the documents such as

```
goto:index
arabic.txt   help.txt.vim-tiny os_dos.txt       quotes.txt    usr_02.txt usr_40.txt(X)
autocmd.txt  howto.txt         os_haiku.txt     README.Debian usr_02.txt usr_41.txt(X)
(...)
goto:events
Name            triggered by ~

    Reading
    |BufNewFile|        starting to edit a file that doesn't exist              (X)
    |BufReadPre|        starting to edit a new buffer, before reading the file  (X)
    |BufRead|       starting to edit a new buffer, after reading the file   (X)
(...)
goto:builtinfunctions
(...)
USAGE               RESULT  DESCRIPTION ~

abs({expr})         Float or Number  absolute value of {expr}
acos({expr})            Float   arc cosine of {expr}                             (X)
```

As you can see as well I append a `(X)` to each line that I considered imporant (such as Method signatures of already used methods) so I can grep the document by `grep "(X)" cheatsheet.vim.not`

As my lines highlighted have been growing this method of parsing by grep  has prooved insuficient , having to start using a pager, then using vim.

Is not the case of study I wanted for this article, I want yout o focus on my labels goto:

What I want is to create a New Tab that parses all the goto: lines and that I can access them quickly.

For that we can use Quickfix Lists


### Greping those labels

I am going to explain broadly all the concepts , refering to the documentation for the functionalities that we are going to use.

Please find the  [Quickfix Reference Manual](https://vimhelp.org/quickfix.txt.html) , and have it alongside you if you want to grasp each of the Vim Commands and functions.

By using `:vimgrep` we can create such a Quickfix list and open it in a tab

```
:set switchbuf+=usetab,newtab
:vimgrep /goto:/ %
:tab copen
```
And voila !! you just have an interactive new tab
By using j,k you can scroll up , down and by using <Enter> you will be switching the focused line of the document buffer.

```
1   cheatsheet.vim.not|63 col 1-6| goto:index
  1 cheatsheet.vim.not|635 col 1-6| goto:events(X)
  2 cheatsheet.vim.not|13261 col 1-6| goto:predefinedvimvariables(X)-------------------------------    --------------------
  3 cheatsheet.vim.not|13924 col 1-6| goto:builtinfunctions(X)
```


That was the main functionality that I wanted to add. But I am going to go an extra mile.
I want to have 2 different Tabs:
	- One for the entries with the ocurrence `goto:`
	- Other for the ocurrence `(X)`


### Having two simultaneous lists opened at the same time

One would say , it will be easy just do something like

```
:vimgrep /goto:/ %
:tab copen
:vimgrep /(X)/ %
:tab copen
```

This wont work , as the Window used for the QuickFix list is the sameone , you can only have one Quickfix List opened at once (though you can have multiple ones in different buffers , but you need to be switching the focus of the Quickfix Window by doing `:cnext :cprev`

So use the other Quickfix lists format , they can be used alternatively at the same time, called **Location Lists** , this are intended to use for only within one file ocurrence mainly (see documentation for more)

The Location List are used by almost the same syntax commands but use an l instead of a c on them

```
:lvimgrep
:lopen
:lnext
(...)
```

I have finally this on my .vimrc so to identify the file extension I am using and execute this


```
augroup OpenFileAnd
      autocmd!
      autocmd BufReadPost *.not call MyFunction()
augroup END

def MyFunction()
    lvimgrep /goto:/ %
    tab lopen
    vimgrep /(X)/ %
    tab copen
enddef
set switchbuf+=usetab,newtab
```

It works perfectly now. Only issue is that my lines now look like

```
   2 cheatsheet.vim.not|670 col 71-74| |BufNewFile|      starting to edit a file that doesn't exist                   (X)
   3 cheatsheet.vim.not|671 col 71-74| |BufReadPre|      starting to edit a new buffer, before read     ing the file  (X)

```

They look too wide to be displayed on a narrow screen , plus all the info relating to fileName , and ocurrence line and column is a bit useless , if only I could trim it?



### Adanced usage of Quickfix Lists Functions overview

You can fine create and modify Quickfix Lists in a finer way by using the functions `setqflist()` `getqflist()` see reference pages.

[setqflist Function Reference](https://vimhelp.org/builtin.txt.html#setqflist%28%29)
[getqflist Function Reference](https://vimhelp.org/builtin.txt.html#getqflist%28%29)

In a nutshell I will paste it as plain-text the signatures and the arguments that we are going to use.


```
Note all of the parameters are either Dictionaries (Objects in vim script) or Lists (Arrays in vim script)

# setqflist({list} [, {action} [, {what}]])    
		Create or replace or add to the quickfix list.                       (X)

		{action} values:		*setqflist-action* *E927*                  (X)
		- 'a'	The items from {list} are added to the existing     (X)
		- 'r'	The items from the current quickfix list are replaced  (X)
(...)


{what} is optional , but if used , it will deem {list} useless
	 	{what}:  (X)
		  -  idx		index of the current entry in the quickfix list specified by 'id' or 'nr'. If set to '$', then the last entry in the list is set as the current entry. (X)
		  -  quickfixtextfunc function to get the text to display in the quickfix window.  (X)

		  -  items	list of quickfix entries. Same as the {list} argument. (X)
					 -  bufnr	buffer number; must be the number of a valid buffer (X)
					 -  lnum	line number in the file   (X)
		 			 -  text	description of the error  (X)


# getqflist([{what}])                            
		Returns a |List| with all the current quickfix errors.  Each       (X)
		{what} dictionary (X)
		 -	id	get information for the quickfix list with             (X)
		 -	items	quickfix list entries                              (X)
```


### Creating a quickfix list

For creating a quickfix list with just one entry pointing to line number 42 of the current opened buffer do

```
call setqflist([{'bufnr': bufnr(''), 'lnum': 42, 'text': 'entry'}], 'a')
```

You can put the {list} as a property of the property `items` in `{what}` argument

```
call setqflist([] , 'a' , {'title' : 'pepe', 'items': [{'bufnr': bufnr(''), 'lnum': 42, 'text': 'entry'}]} )
```

### Modifying an existing quickfix list

Seeing that the commands can get messy with all of those nested dictionaries and Lists we are going to write a Quickfix List , creating it first , then modifying its content adding two entries afterwards.

```
" Establishing the current buffer to be referenced
let bufferNoRef = bufnr("")

" Creating a new qflist
call setqflist(
\     [] , 'r',
\     {
\     'title': 'pepe'
\    }
\)

" Getting the qfId of the newly created qflist
let qfIdRef = getqflist({'id' : 0}).id

" Adding two entries to that mentioned list
call setqflist(
\     [] , 'r',
\     {
\     'id': qfIdRef,
\     'items': [ {
\           'bufnr': bufferNoRef,
\           'lnum': 42,
\           'text': 'entry'
\       },{
\           'bufnr': bufferNoRef,
\           'lnum': 5,
\           'text': 'entry'
\       }]
\    }
\)
```

### Breaking down an item for each list

Following the previous code we are going to retrieve information of each item of the list by

```
let myItemList = getqflist( {'id' : qfIdRef , 'items': 1}).items
echo myItemList[0]
# {'lnum': 42, 'bufnr': 1, 'end_lnum': 0, 'pattern': '', 'valid': 1, 'vcol': 0, 'nr': 0, 'module': '' , 'type': '', 'end_col': 0, 'col': 0, 'text': 'entry'}
echo myItemList[0].lnum
# 42
```

As you can see you first grab an item , so item:1 and perform the query with `getqflist()` this will return you an iterable object (a List) with all the entries for that Quickfix List
Each of this entries have all the properties accesible such as `lnum` `bufnr` or `text`


### Customizing your quickfix list

Following again the previous code, we can grab an existing List, and modify its property `quickfixtextfunc` which value points to a Function that will set its format.

First we define the function. then we modify the list to make an effect on that List.

```
"Customizing the style of the quickfix text displayed
func QfFormating(info)
	let items = getqflist({'id' : a:info.id, 'items' : 1}).items
	let l = []
	for idx in range(a:info.start_idx - 1, a:info.end_idx - 1)
	  call add(l, items[idx].text)
	endfor
	return l
endfunc


" Applying the formating function to the current Quickfix List 
call setqflist(
\     [] , 'r',
\     {
\     'id': qfIdRef,
\     'quickfixtextfunc': 'QfFormating'}
\)

```

Applying this to any list will only leave the text of the ocurrence, no more filename or linenumber and other clutter.

Objective accomplished.


