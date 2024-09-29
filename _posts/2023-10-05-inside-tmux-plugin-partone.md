---
title: Inside a Tmux Plugin Repository (Part One), Variable declaration
tags:  tmux, linux, plugin, programming, keybindings
article_header:
  type: cover
  image:
    src: images/inside-tmux-plugin-partone-01.png
---

## Inside a Tmux Plugin Repository (Part One), Variable declaration

In this blog article we are going to take a further look at the code inside a **tmux plugin** , checking the basic boiler-plate that it needs to be done in order to start developing a new functionality packed in the tmux plugin system.

### Brief introduction of tmux

For those who doesn't know what [Tmux](https://github.com/tmux/tmux) is, it is a Console terminal multiplexer that work mainly in Linux and macOS consoles. It allows you from a simple plain prompt, to spawn multiple Windows and Splits (panes), also been able to select text, copy pasting it with few keyboard presses.
It is a huge productivity booster, and if you haven't used it you should add it to your workflow.
This is one of *the tools* that were/are/ and will be used, just as many **GNU tools** , **git** and **vim** they are supperior than other choices.

### Tmux environment

**tmux** is a highly customizable program, from keybindings to layouts to almost whatever you can imagine can be customised by the user, you can do so altering the configuration file `$HOME/.tmux.conf` , you can check for snippets on how achieve certain functionality, but when it comes to adding something more complex you can rely also on the [plugin ecosystem](https://github.com/tmux-plugins/tpm).
[Here](https://github.com/tmux-plugins/list) you can find a directory with some plugins

### Basic requirements

As pre-requisites for these you should have a fair knowledge on:
1. bash scripting `man bash` : Assuming you know How to declare functions, how to return values from them "the *bash* way" 
2. tmux environment `man tmux` : You have been tweaking with your `$HOME/.tmux.conf` for a while.
3. Know how to create a basic plugin, [How to create a tmux plugin]([PropertiesService](https://developers.google.com/apps-script/reference/properties)

For this tutorial we will be taking the following plugin [tmux-yank](https://github.com/tmux-plugins/tmux-yank) , which allows you to share the clipboard of tmux with Linux, macOS , WSL2 etc...
We will be taking that plugin code, analizing the basic Boiler-Plate needed in order to declare Variables that are going to be used by the plugin, that set a default behaviour of the plugin but could be User Changed so to change the functionality accordingly. See what that means in terms of code, how to separately nicely in different files that serve for different purpose.

### Code Overview

Below I am pasting the output of the overview of the repository for this plugin, milked down just to the point of adding *Plugin Variables* with their default VarName and Value.

```
./
├── plugin-name.tmux
└── scripts
    ├── helpers.sh
    └── variables.sh

2 directories, 3 files

.


./plugin-name.tmux

#!/usr/bin/env bash

CURRENT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

source "$CURRENT_DIR/scripts/helpers.sh"
source "$CURRENT_DIR/scripts/variables.sh"

set_copy_mode_bindings() {
    tmux bind-key -T copy-mode-vi "$(yank_key)" send-keys -X "$(yank_action)"
}


main() {
    set_copy_mode_bindings "someCopyCommand"
}
main

./scripts


./scripts/variables.sh

yank_default="y"
yank_option="@copy_mode_yank"

yank_action_default="copy-pipe-and-cancel"
yank_action_option="@yank_action"


./scripts/helpers.sh

#!/usr/bin/env bash

get_tmux_option() {
  local option="$1"
  local default_value="$2"
  local option_value=$(tmux show-option -gqv "$option")
  if [ -z "$option_value" ]; then
    echo "$default_value"
  else
    echo "$option_value"
  fi
}

yank_key() {
    get_tmux_option "$yank_option" "$yank_default"
}

yank_action() {
    get_tmux_option "$yank_action_option" "$yank_action_default"
}
```


### Static Analysis

Knowing that the entry point of the plugin is the file `plugin-name.tmux` which sources the other accesory files found on the `./script/` subdirectory called ]
- `helpers.sh` : 
    - Boiler-plate funcions: such as `set_tmux_option()` or `get_tmux_option()` they are available in pretty much all the plugins.
    - Script specific functions: such as `yank_key()`

- `variables.sh`: Just a separated file for **Plugin-variable** declarations

For the sake of clarity, mentioning and separating two concepts is important in here, you can see that we are dealing with two environments, two architectures:
1. Bash Scripting Syntax : In which the plugin will process its features.
2. Tmux Syntax : The plugin will have to ultimately perform tmux commands, the same way you would do system calls when operating with an operative system.

They both have different *namespaces* so I refer to:
- **built-in-tmux-variables** , the ones that come setted from the environment execution of executing tmux, with a certain `$HOME/.tmux.conf` , and that are viewable at any time within tmux by doing `:show-options -g my-option`
- **plugin-variables** , the ones written in bash-scripting for the internal functioning of the plugin defined alongside the code-base like `yank_default="y"`
- **inserted-tmux-variables** , the ones that will be inserted in tmux , by doing the call to tmux 
`tmux set-otion -g '@my-var' 'my-value'`

As you can see variables become options and options variables, I have named them here all variables, although they become an option in tmux, a user made option (which come with @ prepended to its name)


### Dynamic Analysis

Suggesting you to have the **Code Overview** in front of you to read the following lines.

You can see the **plugin-variables** , are been defined in the `helpers.sh` file, they come in two types

```
varname_default="the default value of the variable"
varname_option="@inserted-tmux-variable-name"
```

In this case, this plugin doesn't necesarily parses all the **plugin-variables** into **inserted-tmux-variables** , it just check that those **inserted-tmux-variables** exists in the `$HOME/.tmux.conf` file, if it doesn't it gives it a default value.

This is been done thanks to the generic helper function `get_tmux_option()` 
That allows the user to change the default functionality of the plugin, for instance this plugin will yank text by pressing the key `y` when in **copy-mode**. That is the default keybind, but you can choose it by adding the following line to your `tmux.conf`

```
set -g @copy_mode_yank 'z'      ## Change yank bind to z
```

Variables are checked by the **specific helper function** created for each variable to check such as `yank_key()`
Those **specific helper functions** They will be called on the plugin entry point file `plugin-name.tmux` which some **specific tmux calling functions** such as `set_copy_mode_bindings()` which in turn will be called by the **plugin entry point function** `main()`


### Final notes

Note 1) Tmux calls are perfomed in `./plugin-name.tmux` file.

Note 2), for this plugin , there would'nt be any **inserted-tmux-variable** should this key-bind not been customised by the user. The plugin would work the same way, but you wouldnt be able to

```
:show-options -g '@copy_mode_yank'
```

In some other plugins we will see that they will set all their **plugin-variables** into **inserted-tmux-variables** , for that there is another generic helper function that will serve the purpose

```
set_tmux_option() {
  local option="$1"
  local value="$2"
  tmux set-option -gq "$option" "$value"
}
```

So far in this article you have seen twice the concept of **generic helper function** which I have made up but I use it to define the boiler-plate functions that show up in many tmux plugins codebase ( mainly within the `helpers.sh` ) that serve a generic function that may be re-used by other plugins.

This is in opossition to **specific helper functions** like `yank_key()` which are plugin-specific.

Identifying those **generic helper functions** will be key to code reusability in this developing environment, and I will probably be posting more articles about some others of these functions.


Hope you have enjoyed the article!!.





