---
title: Vim adding tags on your syntax files
tags:  vim, linux, plugin, programming, ctags, links, syntax
article_header:
  type: cover
  image:
    src: images/vim-tags-on-your-syntax-file01.png
---

## Introduction

[Workshop.txt Vim Reference](https://vimhelp.org/workshop.txt.html)

After all we have seen on previous tutorials about how to create our own syntax, for our own files, that can serve us many different purposes, we are going to go one step further into creating navigable tags on vim so we can create links in our files to be linked in other files achieving this way a HyperLink way of navigating through our files.

Just giving credits to the tutorial linked below, which have made me understand the **ctags** part of the issue. Although for whoever is reading this tutorial the later tutorial doesn't have to be read at all, I will be explaining the things for my use case.

[EdwinWenink Custom Note Taggig system with Ctags and Vim](https://www.edwinwenink.xyz/posts/43-notes_tagging/)

### Vim motions and jumps

Vim features powerful ways to deal with the complexity of huge text projects. From recognising code-blocks, function scopes, function declaration, methods etc... This helps a great deal the end-user managing repositories made of hundreds of thousands of files, and lines, providing a reasonable way of navigating files, better than just going line by line or page by page, but identifying these blocks within the Code Base and providing of ways from jumping from one char,line,file into other char,line,file.

[Jumps Vim Reference](https://vimhelp.org/motion.txt.html#jump-motions)
[Tagsrch.txt Vim Reference](https://vimhelp.org/tagsrch.txt.html)


For simplicity just remember two motions for this tutorial.

```
CTRL-O			Go to [count] Older cursor position in jump list              
CTRL-]			Jump to the definition of the keyword under the cursor.
```

### Vim tags

Tags are a standarised format to show the text-editors or IDE's to quickly jump into a certain char,line,file , they are generated automatically according to some pre-existing syntax rules. 
The program that generates them is called [ctags](https://docs.ctags.io/en/latest/)

Currently there are two variants of this program *exuberant-ctags* and *universal-ctags* been the later a fork of the former with richer features, we will then use thatone , but regardless the tutorial can be followed with the otherone.

The normal way of using it is by simply running it recursively on the root folder of your project.
```
$ cd /path/to/my-project/
$ ctags -R .
```

This will generate a file called ./tags with the information vim needs to have in order to provide that Hyperlink functionality.

The most basic example is to have some simple , for instance, `java` repository with different methods declared scattered in different files and also been called in other files, so when opening the program entry point file, you can navigate through function calls to their definitions by pressing `Ctrl + ]` to jump to the definitions , returning by pressing `Ctrl + O`. `ctags` deal with a whole bunch of well-known languages taking care of this automatically, you just dont forget on running it and re-running it when changing something on the project.

### Our use case , hyperlinks on notes.

For our tutorial we are going to mimic the way vim helpfiles work.
We wont be using this for any code but just link positions of our notes together.
I always liked the way the vim help documentation works.

[Helphelp.txt Vim Reference](https://vimhelp.org/helphelp.txt.html)

If you are not fully familiarised with it , there are links scattered all over the documentation, with concepts that are defined somewhere else in the manuals, this links show up in blue and you can navigate to them by hovering over with the cursor pressing `Ctrl + ]`.

We are unveiling the magic behind this tonight ladies and gents, and is nothing more than the following:
    - 1st) A procedural generated tag using ctags (on the linked file)
    - 2nd) A syntax highlight feature (on the file that shows the link)

There is no full correspondence between the link and the linked file like in *html* documents, but just a `./tags` file on the codebase directory root which points a certain name to that tag.
You will be seeing this shortly.

### A recipe book full of files, characters, lines and flour.

In freddieventura.github.io , we are hardcore cooks, we like to praise life at every meal, been grateful that should we live in a Matrix-like simulated universe, at least is a delicious one.
So we have a cookbook which the recipes are scattered in different files.

```
mkdir my-cookbook
cd my-cookbook
vim mainBook.not
Welcome to freddieventuras cookBook , you wont need anything else:
- macAndCheese
- spanishTortilla


vim italianBook.not

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Duis convallis convallis tellus id interdum velit laoreet id donec. Morbi tincidunt augue interdum velit euismod in pellentesque massa. Viverra accumsan in nisl nisi scelerisque.
@macAndCheese
    - Macaroni: This homemade mac and cheese starts with a box of uncooked macaroni noodles.
    - Butter and flour: You'll need butter and flour to make a roux for the cheese sauce. You'll also need two tablespoons of butter for the topping.

vim spanishBook.not

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
@spanishTortilla
    - 8 eggs
    - 1/2 cup olive oil
    - 5 medium-sized potatoes, diced into 1-inch piece
```

You should have created those files , or something similar on structure, we will see the magic soon.


### Our own syntax, our own rules.

As you can see we have followed our own filetype .not` (filetype not) coming from notes.
This comes from our tutorial [Creating your own syntax highlight for vim](https://freddieventura.github.io/2023/12/22/creating-own-syntax-highlight-vim.html), if you haven't read them just make sure you do `:set ft=not` anytime you open one of them.

This will serve 2 purposes:
    - For ctags: Identify a new kind of syntax for `.not` extension files, parse their tags according to the language-specific rules.
    - For vim: Identify `.not` extension files, so their syntax highlight will be different to others.

### Procedural creation of tags.

We are going to define the parsing rules for **universal-ctags** , we can simply do it by.
    - 1st) Creating our syntax specific parsing configuration file

```
mkdir -p $HOME/.ctags.d/
vim $HOME/.ctags.d/not.ctags
```

    - 2nd) Inside it defining the directives for procedural parsing the tags out of the ocurrences of certain regexes

```
--langdef=nottags
--langmap=nottags:.not
--kinddef-nottags=t,tag,tags
--regex-nottags=/\B@(\w+)\b/\1/t
```
Below I explain each directive:
    - Langdef defines the name of our language
        (This would be used in the next directives such as --langmap=<nameOfOurLanguage> --kinddef-<nameOfOurLanguage> --regex-<nameOfOurLanguage> etc...)
    - Langmap line associates our new language with a file extension `.not`
    - Kinddef identify a new type of syntax feature to be parsed , its short form is the firstone on the commas and will be used on the following parsing rules.
    - Regex-<ourLanguage> : It has got a regex (by default ERE engine), enclosed in // in which it has to have at least one capturing group , then specify the tag associated to it.
    In the form `--regex-<myLanguage>=/<someRegex>/<capturingGroup>/<tagShortform>`

In our case we define the tags as the Strings that are started by \B@, but the tag itself is made of as many letters before coming across the word boundary found, captured by the group `(\w+)`. This would be reflected on `\1` afterwards and identified with the language feature `t`.

If you are interested in more complex forms of parsing rules, you can see more examples in here, [Universal ctags scope tracking in a regex parser examples](https://docs.ctags.io/en/latest/optlib.html#scope-tracking-in-a-regex-parser)


### Parsing the tags.

Now ctags will automatically recognise these files, so you can just go to your projects directory subfolder and pass

```
cd my-cookbook
ctags -R .
```
This will generate the file `tags` which is automaticaly parsed by vim when placed in the parent directorty of your project.

```
cat tags

!_TAG_FILE_FORMAT	2	/extended format; --format=1 will not append ;" to lines/
!_TAG_FILE_SORTED	1	/0=unsorted, 1=sorted, 2=foldcase/
!_TAG_OUTPUT_EXCMD	mixed	/number, pattern, mixed, or combineV2/
!_TAG_OUTPUT_FILESEP	slash	/slash or backslash/
!_TAG_OUTPUT_MODE	u-ctags	/u-ctags or e-ctags/
!_TAG_PATTERN_LENGTH_LIMIT	96	/0 for no limit/
!_TAG_PROC_CWD	/home/fakuve/myrepos/fakuve/messing-not/my-cookbook/	//
!_TAG_PROGRAM_AUTHOR	Universal Ctags Team	//
!_TAG_PROGRAM_NAME	Universal Ctags	/Derived from Exuberant Ctags/
!_TAG_PROGRAM_URL	https://ctags.io/	/official site/
!_TAG_PROGRAM_VERSION	5.9.0	//
macAndCheese	italianBook.not	/^@macAndCheese$/;"	r
spanishTortilla	spanishBook.not	/^@spanishTortilla$/;"	r
```

You dont need to know about this, but you can check more info in `man ctags` **TAG FILE FORMAT**.


### Vims fake hyperlinks

Now you can simply open any file on that subdirectory with vim , and do `:tag macAndCheese` it will take you to that recipe definition.
If you dont want to go to the tag file , you can also see all the tags by doing `:echo taglist('.*')`

You can also locate your cursor on top of the words macAndCheese or spanishTortilla and by pressing `Ctrl + ]` it will access them , `Ctrl + O` to return to the main document.

This is OK'ish as we want it to be clear that those words can really take us somewhere, mimicking the functionality of hyperlinks, so we are just going to do a syntax feature for our `.not` files that will highlight all the words enclosed in `||` hidding these chars, so to instruct the user that they can be used to access somewhere else.

Just create or add to the  `$HOME/.vim/syntax/not.vim` the following

```
" quit when a syntax file was already loaded
if exists("b:current_syntax")
  finish
endif

" Link to tags
if has("conceal")
  setlocal cole=2 cocu=nc
endif

if has("ebcdic")
  syn match helpHyperTextJump	"\\\@<!|[^"*|]\+|" contains=helpBar
else
  syn match helpHyperTextJump	"\\\@<!|[#-)!+-~]\+|" contains=helpBar
endif
if has("conceal")
  syn match helpBar		contained "|" conceal
else
  syn match helpBar		contained "|"
endif

hi def link helpHyperTextJump	Identifier
hi def link helpBar		Ignore
```

Explaining the directives below:
    - First if is needed to load the syntax
    - The if of coneal, check if your vim instance has the conceal feature, and set it to 2 if so.
        Conceal will hide the || to the user.
    - The if of ebcdic will set up the Syntax Regex match, I dare to say that most of modern cases will use the else case, but for backwards compatiblity better use both of them.
        The pattern literally says , match everything that is in between || , unless the first | is escaped by \ .
        - This will match: |myTag|
        - This wont match: \|myTag|
    - The if conceal last rule will effectively *conceal* aka hide the || from the user.
    - The last lines will identify the syntax groups to certain  built-in highlighting groups that already have assign certain display properties.

Now you can just re-do the mainBook.not enclosing the links with ||
`vim mainBook.not`
```
Welcome to freddieventuras cookBook , you wont need anything else:
- |macAndCheese|
- |spanishTortilla|
```

### Bonus track: Highlighting linked lines

As an exercise you could try to follow the same last procedure, just to highlight the linked lines, say those @macAndCheese definitions, so to make them look red and *conceal* their `@`.

Try to think it yourself, if you cannot find the solution find below the answer.

`vim %HOME/.vim/syntax/not.vim

```
" Linked items
syn match helpHyperTextEntry "@\w\+\n"he=e-1 contains=helpAt
syn match helpAt		contained "@" conceal

hi def link helpAt		Ignore
hi def link helpHyperTextEntry	String
```
