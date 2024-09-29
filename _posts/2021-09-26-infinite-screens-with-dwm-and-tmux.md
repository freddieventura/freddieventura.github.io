---
title: Infinite Extended Screens with dwm and tmux
tags:  tutorials, linux, suckless, dwm, tmux, debian, screens
article_header:
  type: cover
  image:
    src: /images/infinite-extended-screens.jpg
---

## Infinite Extended Screens with dwm and tmux

> Getting an extended desktop with as many sceens as you want
> Wireless connection in between screens
> Introducing suckless dwm ghostscratchpads patch

This is just a quick tutorial on how to achieve the functionality I show in the video , [video link](https://youtu.be/GHwqsyNeiRU) .
After completing it you will be setting a multi-monitor workflow via terminal.

Requirements:
    - Linux 
    - Suckless dwm
    - [dwm - ghostscratchpads Patch](https://github.com/bakkeby/patches/blob/master/dwm/dwm-ghostscratchpads-20210920-a786211.diff)
    - Tmux
    - 1 or More Additional host with Screen SSH capable

After patching your suckless installation your `config.h` file should look something like this
```
(...)
    { NULL,       NULL,   "scratchpad1",  0,            1,           -1,       's' },
    { NULL,       NULL,   "scratchpad2",  0,            1,           -1,       'd' },
(...)

static const char *scratchpadcmd1[] = {"s", "st", "-g" , "238x85" , "-t", "scratchpad1", NULL};
static const char *scratchpadcmd2[] = {"d", "st", "-g" , "238x85" , "-t", "scratchpad2", NULL};
(...)

    	{ MODKEY,                       XK_grave,  focusscratch,  {.v = scratchpadcmd1 } },
    	{ MODKEY,                       XK_5,  focusscratch,  {.v = scratchpadcmd2 } },
(...)
```
Note that in my case I have just done two scratchpads binded to Modkey,5 and Modkey+` , but you can follow the same logic by adding additional ones by


```
(...)
    { NULL,       NULL,   "scratchpad1",  0,            1,           -1,       's' },
    { NULL,       NULL,   "scratchpad2",  0,            1,           -1,       'd' },
    { NULL,       NULL,   "scratchpad3",  0,            1,           -1,       'a' },
(...)

static const char *scratchpadcmd1[] = {"s", "st", "-g" , "238x85" , "-t", "scratchpad1", NULL};
static const char *scratchpadcmd2[] = {"d", "st", "-g" , "238x85" , "-t", "scratchpad2", NULL};
static const char *scratchpadcmd3[] = {"d", "st", "-g" , "238x85" , "-t", "scratchpad3", NULL};
(...)

    	{ MODKEY,                       XK_grave,  focusscratch,  {.v = scratchpadcmd1 } },
    	{ MODKEY,                       XK_5,  focusscratch,  {.v = scratchpadcmd2 } },
    	{ MODKEY,                       XK_6,  focusscratch,  {.v = scratchpadcmd3 } },
(...)
```

Those 's' , 'd', 'a' , are pretty arbitrary , I'm not sure exactly what they mean , but additional ones do have to differ
    
After making this patch you should be able to spawn and toggle the scratchpads with those key combinations , you can move those terminals to a non-used tag, and just focus it whenever you want to bring it to the front in your main monitor.

Right after invoking one scratchpad just spawn a tmux session such as

```
tmux new-session -s t1
```
Note: Second monitors can be any host capable of SSH with a screen (for obvious reasons) and connected to the same LAN

Then you want to go to your Second Monitor Host and ssh into your main host

```
ssh user@myHost
```

Once with the shell in your main host just attach to that same newly created session

```
tmux attach -t t1
```

Repeat the process as many times as you want, with Care :) .

### Edit1 26/09/2021 Issue with Clipboard

I have noticed that the Clipboard was not been shared properly among these `ssh`ed shared tmux session's , and it was due to a strange behaviour on `tmux` with theseones , the `$DISPLAY` environment variable of the system was not setted for succesive Pane's and Window splits (whereas for the first shared Pane if you `echo $DISPLAY` it will output `:0`). 

After following this [article](https://goosebearingbashshell.github.io/2017/12/07/reset-display-variable-in-tmux.html) the solution it seems to work for me.

Just add this line to `vi ~/.bashrc`

```
## tmux issue with ssh shared sessions and DISPLAY variable
export DISPLAY="`tmux show-env | sed -n 's/^DISPLAY=//p'`"
```

You may have to also add this line to `vi ~/.tmux.conf`
```
# Solving issue with $DISPLAY and ssh shared tmux sessions
set-option -g update-environment " DISPLAY"
```

If with the two modifications above you still have some issues, update tmux to the last version. `tmux -V` I use to have `2.8` for `Debian 10 Buster Stable` but you can build your own from source with the following commands

```
sudo apt install libevent-dev
sudo apt install byacc
git clone https://github.com/tmux/tmux
cd tmux
./configure && make
sudo make install
```

All credit goes to [Mr Bakkeby](https://github.com/bakkeby/) , he is the author of this patch.
