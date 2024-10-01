---
title: Type checking in bash and correct argument handling
tags: bash, linux, programming, scripting
article_header:
  type: cover
  image:
    src: images/bash-type-checking-01.jpg
---

# Type checking in bash and correct argument handling

I just want to write a quick tutorial, after stumbled with yet another stone in the path, so you don't have to (although chances are that if you have landed in this article you may have probably looked for an answer already).

Bash is an sripting language, but its full interoperability outside the box with Linux, makes it an incredibly powerfull tool as it can be runned in almost every server in this planet.

This language has been envisaged for scripting purposes, but when writting a tool, you may cross the thin line of scripting to programming as your script grows in complexity and at that point and for not wanting to change all the infrastructure of it you will stick to it regardless of certain ineficiencies.

In order to write good code we will end up with the necesity of implementing certain code blocks repeteadly , a.k.a **boilerplate** . We will see below some useful snippets.

## Type checking

When repeating patterns in your program you may want to encapsulate its functionalities for easy reuse in functions.

Functions in bash work like executables themselves (meaning executables), they take arguments as a concatenation of strings, there is no control on how many strings will they take (even if any), and what their form is gonna be.
Calling a function in bash it is the same as invoking a program from the command line.

A well writen executable will be implemented in a way that you pass the arguments the following way.
`./executable.sh -a value_a -b value_b `
`-a` and `-b` are flags and `value_a` , `value_b` their respective values.

Excutables can be written in many different ways, though for the clarity of the user this pattern is desirable, and can even be refined way more, we wont get into that in this article.

The point is that functions in a `bash` program work the same way as executables

What for an executable is
```
# vim ./executable.sh
#!/bin/bash

echo ${1}  
echo ${3}  

$ ./executable.sh -a value_a -b value_b 
-a
-b
```

For a function is

```
$ vim ./myscript.sh
#!/bin/bash
my_function -a value_a -b value_b
my_function() {

    echo ${1}  
    echo ${3}  

}

$ ./myscript.sh 
-a
-b
```


## The issue

Scripts above are really simple code, they won't withstand many execution environments though. 
In regards to functions if you write them expecting the parameters to be there you may run into big problems (I did) , for instance executing `rsync` with shifted arguments can lead you to a big trouble.


```
use_rsync "etc"
# use_rsync "home" "etc"

use_rsync () {

    SUBDIR_ONE=${1}
    SUBDIR_TWO=${2}

    rsync --remove-source-files "/${SUBDIR_ONE}/${SUBDIR_TWO}" 192.168.43.42:/home/otheruser/

}
```

In the silly example above (in which you have a user called `etc`) `use_rsync` if you miss the first argument as in the first invocation `use_rsync "myuser"` you are no longer checking on `/home/etc/` but in `/etc/` and will be deleting that directory after the transfer (not to say that you modify the same paths on the receiver too).
You can destroy hosts in a fraction of a second, and althogh this scenario is unlikely, sometimes functions don't parse arguments at all for little issues in your code, or a missuse by the user. Then we can be at risking directories like the root directory.


## Doing some boiler plate

It is good then to apply this snippet even in your funtions


```
functionname() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: functionname -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: functionname -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: functionname requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------


    # CODE GOES IN here

}

```

Even for the simplest of the functions, if you save this snippet, this will implement a basic type checking , we can re-write the above as


```

use_rsync "etc"
# use_rsync -a "home" -b "etc"

user_rsync() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    # Initialize variables
    SUBDIR_ONE=""
    SUBDIR_TWO=""

    while getopts "a:b:" opt; do
        case "${opt}" in
            a) SUBDIR_ONE="${OPTARG}" ;;
            b) SUBDIR_TWO="${OPTARG}" ;;
            *)
                echo "Usage: user_rsync -a SUBDIR_ONE -b SUBDIR_TWO" >&2
                echo "Example: user_rsync -a SUBDIR_ONE -b SUBDIR_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${SUBDIR_ONE}" || -z "${SUBDIR_TWO}" ]]; then
        echo "Error: user_rsync requires -a SUBDIR_ONE -b SUBDIR_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------

    rsync --remove-source-files "/${SUBDIR_ONE}/${SUBDIR_TWO}" 192.168.43.42:/home/otheruser/


}
```

Now the program will exit with error, specifying it this way with `getopts` function can save you from a lot of hassle.

More info on `getopts` on `man bash`

```
       getopts optstring name [arg ...] is used by shell procedures to parse positional parameters. 
        (...)
```

But hold on dont copy that snippet, we will add something else as it may still malfunction (although not messing things up anymore).

## The problem with getopts 


Following this previous programming practice I have carried on programming, and when it came to debugging it was easier, yet I was having issues, with functions that were not parsing its arguments (despite them been passed)


This situation was detected when a function was calling another function within it, following two function calls (ergo two argument passes), `getopts` will malfunction if written this way as for its functioning is using and setting a `bash` built in variable, `OPTIND` . So the second invocation will read this already altered variable and malfunction.


Follow my example below

```
./myscript.sh

#!/bin/bash

main() {

    wrapper -a "Luis" -b "Pepe"

}

wrapper() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: wrapper -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: wrapper -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: wrapper requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------

###   echo say_hi -a ${ARG_ONE} -b ${ARG_TWO}  ## FOR DEBUGGING
    say_hi -a ${ARG_ONE} -b ${ARG_TWO}
}



say_hi() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:z:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: say_hi -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: say_hi -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: say_hi requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------

    echo "Mr ${ARG_ONE} is saying hi to ${ARG_TWO}"

}

main
```

When executed it throws you out with `Error: say_hi requires -a ARG_ONE -b ARG_TWO`

If we uncomment the line for debugging echoing the function invocation we get
`say_hi -a Luis -b Pepe` , so the arguments are been passed correctly, you may get stuck.

The solution is to reset and assign the variable `local OPTIND=1` making it locally for each function definition.

Rewritting the code as

```
#!/bin/bash

main() {

    wrapper -a "Luis" -b "Pepe"

}

wrapper() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    local OPTIND=1
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: wrapper -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: wrapper -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: wrapper requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------

    say_hi -a ${ARG_ONE} -b ${ARG_TWO}
}



say_hi() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    local OPTIND=1
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:z:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: say_hi -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: say_hi -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: say_hi requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------

    echo "Mr ${ARG_ONE} is saying hi to ${ARG_TWO}"

}

main
```

Now we get 

`Mr Luis is saying hi to Pepe`

## Final snippet

We have got the resulting snippet, which you may want to copy.

```
functionname() {
    ## PARSING ARGUMENTS ---------------------------------
    ## ---------------------------------------------------
    local OPTIND=1
    # Initialize variables
    ARG_ONE=""
    ARG_TWO=""

    while getopts "a:b:" opt; do
        case "${opt}" in
            a) ARG_ONE="${OPTARG}" ;;
            b) ARG_TWO="${OPTARG}" ;;
            *)
                echo "Usage: functionname -a ARG_ONE -b ARG_TWO" >&2
                echo "Example: functionname -a ARG_ONE -b ARG_TWO" >&2
                return 1
                ;;
        esac
    done
    # Variable check
    if [[ -z "${ARG_ONE}" || -z "${ARG_TWO}" ]]; then
        echo "Error: functionname requires -a ARG_ONE -b ARG_TWO" >&2
        return 1
    fi
    ## PARSING ARGUMENTS ---------------------------------
    ## EOF EOF EOF EOF------------------------------------


    # CODE GOES IN here

}
```


## References


[1](https://unix.stackexchange.com/questions/233728/bash-function-with-getopts-only-works-the-first-time-its-run)
[2](https://stackoverflow.com/questions/49439994/getopts-in-sourced-bash-function-works-interactively-but-not-in-test-script)
[3](https://stackoverflow.com/questions/42336595/in-bash-why-does-using-getopts-in-a-function-only-work-once)
[4](https://stackoverflow.com/questions/79041382/getopts-used-to-parse-function-parameters-failing-to-do-so)



