---
title: Compiling a "Hello World" project locally written on Google Apps Script with Clasp
tags:  Google Apps Script, gas, javascript, clasp, google
article_header:
  type: cover
  image:
    src: images/gas-banner001.png
---

## Compiling a "Hello World" Project locally written on Google Apps Script with Clasp


Rather a long tittle, for a long complex endeavour that gets tangled Programming Languages, Cloud Services, Text Editors and Command Line Applications.

The objective is simple, Coding locally for **Google Apps Script** (from now on **GAS**) executing that code in the terminal.

If you are reading this you should be looking at developing something in [GAS](https://developers.google.com/apps-script/overview), basically a programming environment based in **Javascript** which can be runned with a **Google Account** and that is able to call and automate processes on services such as:
	- Google Sheets
	- Google Docs
	- Google Calendar
	- Google Mail (Gmail)
	- Google Forms etc..

It has it's own **Object Model** , with tons of Objects and methods in order to perform actions on this services such as writting in a certain cell of a spreadsheet upon filling in a web form while also creating a new appointment on the User Calendar and sending and email to a client.

Nobody wants to marry with a payment service, after all **Google** is a private company and everything you will run on their servers is subject to be charged if surpassed cetain [quotas](https://developers.google.com/apps-script/guides/services/quotas). But been able to do all these actions in such a unified way, in a platform that you can access from almost every browser in this World, securely, worth getting in the Enterprise, plus "Who knows?" maybe you end up developing something that you can monetize **$$$**.

### The Official toolset

To start developing on this you just need to get into [script.google.com](https://script.google.com) , while reading the [documentation](https://developers.google.com/apps-script/overview). If you click on **New Project** you will start a script, and their IDE will help you with features such as **Autocomplete** and so on...

If you like this, then go for it, but the next chapters will be explaining a comprehensive way of efficiently using your keystrokes while holding your focus in what you need to do.

### Our toolset

We will be using **Linux** , with `vim` , and `clasp` which is a tool that allow us to have a little vesion control capabilities, while pushing the projects to your **Google Account** and run them on your terminal (we won't say locally as remember you are using a Cloud Service Provider, we wont be downloading the whole **Google Engine** to create our own instance).


```
                                       -----------------------------------------------
                                      | ------------                          _       |
 -----------    ------------------    ||Google Script  __ _  ___   ___   __ _| | ___  |
| Code      |->|Version Manager   |-> ||script.    |  / _` |/ _ \ / _ \ / _` | |/ _ \ |
|           |  |clasp             |   || google.com| | (_| | (_) | (_) | (_| | |  __/ |
| ./Code.gs |  |./appscript,json  |   ||----------    \__, |\___/ \___/ \__, |_|\___| |
| (...)     |  |./.claspignore    |   |               |___/             |___/         |
|           |  |~/.clasprc.json   |   |                                               |
|           |  |./clasp.json      |   |                                               |
|           |  |                  |   |                                               |
|           |  |                  |   |                             ----------------  |
 ----------     ------------------    |  ---------------------     |Google Drive    | |
                                      | |Google Cloud Platform|    |drive.google.com| |
                                      | |GCP                  |     ----------------  |
                                      | |console.google.com   |     ----------------  |
                                      |  --------------------      |Google Calendar | |
                                      |                            |calendar.google.com
                                      |                             ----------------  |
                                      |                             ----------------  |
                                      |                            |more services...| |
                                      |                             ----------------  |
                                       -----------------------------------------------
```

### Process overview
	1. Setting up vim.
	2. Preparing our cheatsheet.
	3. Installing Clasp
	4. Creating our first project.
	5. Running functions from Clasp.

If you don't use `vim` you can skip 1 and 2.

#### Step 1. Setting up vim.

`GAS` uses `.gs` file extension, but the syntax used is a subset of `javascript` , in order to make `vim` recognise its syntax, append the following snippet on your `vim ~/.vimrc` 

```
"read .gs as .js
augroup MyAutoCmds
  autocmd!
  autocmd BufRead,BufNewFile *.gs set filetype=javascript
augroup END
```

#### Step 2. Preparing our cheatsheet.

If you want to be able to handle such a amounts of Objects, properties and methods, you rather use the terminal to traverse through the documentation, for that I would highly recommend installing `w3m` to render the documentation in plaintext , and also the following snippet will make things easier for our cheatsheet.

```
sudo apt install w3m
```
`vim ~/.vimrc`
```
"Open a whole-line link in a new tab by pressing <F6>
function! Scrape()
  let url = getline('.')
  tabnew +set\ buftype=nofile Scraped
  call append(0, systemlist('w3m -dump ' . url))
  normal! gg
endfunction
nnoremap <f6> :call Scrape()<CR>
```

Our documentation can now look something like

`vim gs-cheatsheet.gs`
```
https://developers.google.com/apps-script/reference/drive
https://developers.google.com/apps-script/reference/drive/drive-app
https://developers.google.com/apps-script/reference/drive/file
https://developers.google.com/apps-script/reference/drive/file-iterator
https://developers.google.com/apps-script/reference/drive/folder
https://developers.google.com/apps-script/reference/drive/folder-iterator
https://developers.google.com/apps-script/reference/drive/user
```

Now by locating your cursor on any of the lines that have a link , just press `<F6>` and a new tab will be loaded with all the text from that determined page.

This is in my opinion a fairly nice technique, as not expecting the object model to change drasticaly, minor changes will be updated on the pages that by doing `<F6>` will be re-rendered.

How I personaly like it is to Copy and paste all of this objects in the same document , then I will append a line such as `-------(X)` on the signature of the objects that I have already used and understood ending up with something like.

```
SpreadsheetApp.getActiveRange()              Range                        sheet, or null if       -------(X)
                                                           there is no active
                                                           range.
                                                           Returns the list of
                                                           active ranges in the
SpreadsheetApp.getActiveRangeList()          RangeList                    active sheet or null
                                                           if there are no
                                                           ranges selected.
                                                           Gets the active
SpreadsheetApp.getActiveSheet()              Sheet                        sheet in a               -------(X)
```

This way we can scroll down the document and recall easily the Object Model.
The issue is that we end up with a huge document of more than 100.000 lines, that will take us scrolling down forever to get a big picture of what we know,
We could do something like
```
grep "(X)" gs-cheatsheet.gs
```
But we will have to update this along with the changes we introduce so the best way is to use tmux open a `new-window` then.

```
watch 'grep "(X)" gs-cheatsheet.gs'
```
The problem with this is that once highlight more lines than the number you have available on the height of your screen, then you will be missing the lastones, you could tail this command, but you will be missing the firstones.
For that and with the help of `uwharrie` from `irc.libera.com` we have developed the following script.

```
$ vim ~/bin/cat-alternate.sh

#! /usr/bin/bash

# created by uwharrie from #linux irc.libera
# this script displays a refreshed  multipage output of grep
#   so you can see the output without having to enter user input
# example of use:
#       cat-alternate.sh '(X)' /home/fakuve/baul-documents/gs-learning.js
# it takes two arguments
#       - first argument is the string to grep
#       - second argument is the path of the input file


pat="$1"
file="$2"

while true;do
    res="$(grep "$pat" "$file")"
    res_lines="$(echo "$res"|wc -l)"
    tty_lines="$(($(stty size|cut -d' ' -f1)-2))"
    iters="$((res_lines/tty_lines+1))"

    for n in $(seq "$iters");do
        beg="$(((n-1)*tty_lines+1))"
        end="$((beg+tty_lines))"
        clear
        echo "$res"|sed -n "$beg,${end}p"
        sleep 5
    done
done
```
Now you can do `cat-alternate.sh gs-cheatsheet.gs` and will have as many pages alternating on your terminal without you having to do any keystroke.
If you want to stop on a page use `tmux` `copy-mode` to freeze the `STDOUT` of your terminal (default keybinding `Ctrl + b and [`)

#### Step 3. Installing Clasp

`clasp` is a CLI to interface with `script.google.com`.
It will let you `create` a project, `push` it , `pull` , `clone` and `run` functions from the terminal.

`clasp` is an `npm` dependency so install it globally 

```
sudo npm install -g @google/clasp
```

You can add the following lines to your cheatsheet `gs-cheatsheet.gs` you will probably need to perform some actions that may be documented on their repo.
```
https://github.com/google/clasp
https://github.com/google/clasp/blob/master/docs/run.md
```
Also you can check the help page
```
clasp help
clasp <command> help
```

#### Step 4. Creating our first project.

You just need to log in your CLI to your **Google Account** with

```
sudo clasp login
```
It will open up an instance of your browser so you can authenticate to it.
In my case as I code from my VPS which has no X-server running I use the following command and paste the link in my localHost **Browser**

```
sudo clasp login --no-localhost
```
Then we want to create a project
```
mkdir -p gas-scripts/myproject001
cd gas-scripts/myproject001 
clasp create --title "project001"
## Select Standalone
```

Now you can see new files have been generated locally with information for `clasp` to do its thing.
```
├── appsscript.json
├── .clasp.json
```
But also on the server side `Google` a script file has been generated (you can see that in `drive.google.com`) , which is also a project in `script.google.com`
You can list all your projects from the CLI with

```
clasp list
```
Ok so lets write some Code. Create a `Code.gs` file and put there a function to be runned. Note that everything has to be encapsulated by a function otherwise it wont be runned. Also you dont need to call the function within the script as their compiler will call a function directly.

```
$ vim ./Code.gs
function main () {
	Logger.log("Hello World");
}
```
Note that `GAS` I said is based in `Javascript` but is not exactly it, so you need to see how to do certain things the `GAS` way , in this case `Logger.log("String");` is the equivalent to `console.log("String");`.

Now you just need to push this changes

```
clasp push
```

Now you can go to the browser and put on the link of your script, something like
` https://script.google.com/d/longAlphaNumStringAkaScriptId/edit`

Press `Run` in the tool bar. It will show you the `Execution Log` popping up , Congratulatione amici !!.
But hold on, this is quite not the 100% terminal based experience you sold us before. Is it? Ok we will learn how to run functions from `clasp` without having to use the browser.

#### Step 5. Running functions from Clasp.

This is quite a tricky step that can consume an unecessary amount of time of your life for just a little tweak. The process is a bit tricky and involves setting up a project on `Google Cloud Platform` **GCP** `console.cloud.googe.com`. The step by step tutorial is writen in the following [link1](https://github.com/google/clasp/blob/master/docs/run.md).
This is the abstract of what I have done, as I got stuck on some parts.

```
## Create a GCP Project
https://console.cloud.google.com/cloud-resource-manager
>> Create Project >> Project name "gas-viewer"
					 Location "No organisation"
							>> Create
>> "gas-viewer" hamburger menu >> Settings >>
	(Write down Project ID "gas-viewer"
				Project number "422101l28129"
```
Now go back to your terminal 
```
$ cd ~/gas-scripts/project001
$ clasp setting projectId gas-viewer
```
Now in your browser

```
https://console.developers.google.com/apis/credentials/consent?project=gas-viewer
>> Applicaton name  "clasp project"
			>> Save
```
Now go to your `script.google.com` project ` https://script.google.com/d/longAlphaNumStringAkaScriptId/edit` on the top right corner `Use Classic Editor` 
```
Resources >> Cloud Platform project >>
						Enter Project Number Here ""
							>> Set Project "422101l28129"
```
Now again in `console.cloud.google.com` Go to your settings project ("gas-viewer") and You need to set up the App `Publishing Status` to `Testing` (it should come like this) . And `add User` on `test Users` and input your email@gmail.com

Then authenticate
```
$ clasp open --creds
>> Create Credentials > OAuth cient ID
>> Application type : Desktop App
	Create > OK
```
Download the .json file and place it somewhere as for instance `~/creds.json`

```
clasp login --creds ~/creds.json
```
You need to enable Apps Script API for that project such as in this [page](https://developers.google.com/apps-script/api/how-tos/enable) shows.
Follow this [link](https://console.developers.google.com/start/api?id=script) clicking `>> Next >> Next`

Also you need to modify your project `vim ~/gas.scripts/projet001/appscript.json`
Enter the following object in the JSON body
```
"executionApi": {
  "access": "ANYONE"
}
```

And finally!! we can run our script. But note that we can get the output return of a certain function runned on the terminal. For that you need to re-write your `vim ./Code.gs` to.
```
function main () {
	return "Hello World";
}
```

And now you can do

```
clasp push
clasp run main
```

Sweating tears and blood we have tamed Google's beast.
