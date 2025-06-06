---
title: Gnu parallel basic usage
tags: parallel, gnu, bash, linux
article_header:
  type: cover
  image:
    src: images/gnu-parallel-basics-01.png
---

# Introduction to GNU Parallel

`parallel` is a GNU utility that allows you to run processes simultaneously, sending them to different CPU cores, threads or even other hosts to be runned.

There are millions of use cases for this, and it is incredible useful when you need to process tasks that are CPU intensive or network intensive , or you are just lack of resources and need to find a way to `load balance` avoiding those bottle-necks

Thinking twice before starting one if these batch processing tasks, can save you and your organization incredible amounts of time.


In this article I will be explaining my usecase for this utility, and how to understand everything around the tools I am going to use.
Basically I needed to run a batch processing task. Was in need of Indexing entire websites, which are sometimes made of Millions of files.
Breaking down the tasks into getting a `url-list.txt` (spidering) , keeping track of what it has been downloaded so far and what not, been able to then start the download process from different hosts to multiply the Download Speed from the server is crucial to avoid bringing the Download time from Months to Weeks to days even hours. 

Knowing how to parallel process this task is vital in this use case.

As you can see the main point of this is to avoid **bottle necks** originated by the nature of the task, and the reources available to process that task, may they be:

    - Processing power
    - Storage availabilty
    - Network resources

In my case I am hitting with *Network Resources* and *Storage Availability* , I will be using `GNU parallel` to allocate and run the Download task across different hosts that can multiply my *Network Resources* capabilities , running also with *Storage Availability* issues as we will be seeing how I tackle this.

Thus I will be more interested in the feature of `parallel` of running the task in different hosts, rather than running the task in different threads. I will give few examples of both.

## Introduction: When can you load balance?

Understanding always the nature of computing `load balancing techniques` , they can be achieved as long as the task can be broken down into different system calls to the Operating System, thus the processor.

Understanding that CPU's can run just one process at a time, but contemporany CPU's are made of more than a single Core, thread etc... Operative systems work on the basis of allocating system calls to different threads according to their internals. Each ssystem call cannot be divided and sent to different threads for its execution to speed up its execution. Statements of programming languages often are made to run on a Single thread. 

It is important then, nowadays that Programmers undersand the nature of **System Calls** knowing that their Code is made of a bunch of them, and if not specified otherwise , the program will run sequencially on a Single Thread, meaning that each System Call will have to wait for the previous one to be completed in order to start its execution.

But some parts of a program or script, may be made of tasks that can run asynchronously in different threads. That will spead up the overall execution time of the program. Identifying them and programming to **load balance** these tasks into additional threads in the CPU, or even (if they are networking tasks) you can spread its execution across different Hosts. This is the great usefullness of `GNU parallel`



## Installing parallel, and sources of information


You may want to install the package available on your distro repositories. My opinion is that I have had mixed, results with different versions. Being an old-stable one buggy with some issues, being the latest one buggy too so I recommend this version.

```
wget https://ftp.gnu.org/gnu/parallel/parallel-20240222.tar.bz2
sudo tar xjf parallel-20240222.tar.bz2
sudo ./configure && make
sudo make install
```

If you want to get familiarised with `parallel` I recommend you to follow their tutorials found on their manuals, just gonna do a quick view in here with my understanding of it which may be not 100% accurate. `man parallel` `man 

```
man parallel
man parallel_tutorial
man parallel_examples
man env_parallel 
```


## Breakdown on how to build the command-line

By the nature of the utility which deals with running other commands within a command, running this directly in the *shell* or inside a *shell-script* the syntax can get quite overwhelming , loosing the ability to understand what is doing each part, how to avoid escaping issues with the Shell or with the commands used.

Taking up some simple commands

```
# Running a command with one argument, multiple times (varying the arguments)
## Arguments been read from the CLI directly
parallel echo ::: 'pep' 'costa' 'tom'
# Output
pep
costa
tom

## Arguments been read from stdin
# newlines are interpretted as arguments to be executed parallely on the same command
echo -e "pep\ncosta\ntom" | parallel echo
# Same Output


## Arguments been read from a file
# input.txt
pep
costa
tom
#
parallel -a "input.txt" echo
# Same Output

## Running in a script
# myscript.sh
#!/bin/bash
printing_stuff() {
    echo $1
}
export -f printing_stuff
parallel printing_stuff ::: 'pep' 'costa' 'tom'

#
./myscript.sh
# Same Output
```

You can achieve the same of this examples with different syntaxes, refining the combinations to achieve different things, refer to the documentation for that, we will be keeping it simple in here to achieve what we wanted.

Clarifying the last example. Running it inside a script with a function, you need to export that function in bash with `export -f` statement , then you can recall that function `parallel my_func`


### Allocating jobs in different threads

In the examples before all the echo commands were sent to different threads to be runned parallely as many as possible (that is the default behavior). As soon as each return it will be logged on the stdout (asyncrhonous execution) 
So if one of the jobs will take longer, even if executed first it will finnish the execution later

Note `{}` is a replacement string for all the arguments been passed to that job

```
echo -e "5\n2\n1" | parallel 'sleep {}; echo {}'
# Output
1
2
5
```

We can pick now different parameters to be used in different parts of our command
`{n}` will be replace by parameter no `n`

Though we need to specify how to separate the parameters in our input, we will tell that for each `whitespace` it will be a different parameter.
`--colsep " "`

```
echo -e "5 pep\n2 costa\n1 tom" | parallel --colsep " " 'sleep {1}; echo {2}'
# Output
tom
costa
pep
```

We have demonstrated that tom returns earlier than pep despite it's execution have been started earlier.

That is because `parallel` runs those jobs in as many threads as possible by default.
You can change this by doing `-j <num-jobs>` to run maximum those number of jobs parallely.

```
echo -e "5 pep\n2 costa\n1 tom" | parallel -j 1 --colsep " " 'sleep {1}; echo {2}'
# Output
pep
costa
tom
```

If 1 job is running parallely it will be a sequenced execution (synchronous)


### No arguments

You can truncate the number of arguments been sent to the command with `-n <numer>`
In the special case that you have run commands that need no arguments as input, `parallel` wont let you , pass arguments and use `-n0`

```
echo -e "Whatever\nNothing" | parallel -n0 "ip addr"
## Will run twice asyncrhonously the command ip addr
```

### Remote execution, running jobs in other hosts


We are going to run a bash function in different hosts:
First we need to pass each Host IP we want to use for the execution after `-S`
We keep adding more to add more the more hosts we want to add up for the processing.
Note that to execute the function we need to pass it's name as an environment variable with `--env <function_name>`
Note also that if you want to your localhost to be added to the pool of hosts that execute the processes, you need to add it manually `-S 127.0.0.1` (is not added by default)

```
# input.txt
pep
costa
tom

# my_script.sh
check_hostname(){
    echo $HOSTNAME
}
export -f check_hostname

parallel --env check_hostname -S <WAN_SERVER> -S <LOCAL_HOST> -n0 -a "input.txt" check_hostname

# Output
<LOCAL_HOST>
<LOCAL_HOST>
<WAN_SERVER> 
```

Would it have been `parallel -S <WAN_SERVER> -S <LOCAL_HOST> -n0 -a "input.txt" check_hostname` it wouldnt have worked


### Sending environment variables to the remote host

Sometimes you want to work with different data across the hosts
If you need the otherhosts, to adopt certain environment variables of the executing host you need to use `env_parallel` and then add that environment variable with `--env <MY_VAR>`


```
# my_script.sh
source `which env_parallel.bash`
check_hostname(){
    echo $HOSTNAME
}
export -f check_hostname

env_parallel --env check_hostname --env HOSTNAME -S <WAN_SERVER> -S <LOCAL_HOST> -n0 -a "input.txt" check_hostname

# Output
<LOCAL_HOST>
<LOCAL_HOST>
<LOCAL_HOST> 
```


### Reading environment variables from the remote host

In complex script interactions, you may want to define host-specific values, to perform certain things differently in each one.

This is not strictly of `GNU parallel` but I have found that trying to remotely echo a variable that has been defined in `$HOME/.bashrc` is not working. And it is not caused specificaly by no `GNU parallel` configuration but it is an actual `ssh` security feature
The variables we want to be accessing through ssh need to be contained in a file called `$HOME/.ssh/environment`

```
MY_VAR=some value
```

Then change `/etc/ssh/sshd_config` directive

```
PermitUserEnvironment yes
```

Restart your `sshd server` and try echoing from another host

```
ssh hostIp 'echo ${MY_VAR}'
# Output
some_value
```


### Putting this together

See the example below

`vim parallel_echo_variables.sh`

```
#!/bin/bash

## Previously I have modified ./.bashrc and $HOME/.ssh/environment
## In each host (they hold different values)
# DOWN_PATH=/home/fakuve/downloads

source `which env_parallel.bash`

## SETTING GLOBAL SCRIPT VARIABLES
# this will be shared regardless to which hosts is executing
MASTER_HOSTNAME=$(hostname)
MASTER_DOWNPATH=${DOWN_PATH}


main() {

# each newline echoed will fire the command 1 more time (just for testing)
seq 2 | env_parallel \
        `## PARALLEL ARGUMENTS` \
        -n0 \
        --jobs 1 \
        `## GLOBAL SCRIPT VARIABLES` \
        --env MASTER_HOSTNAME \
        --env MASTER_DOWNPATH \
        `## FUNCTIONS TO BE EXECUTED` \
        --env echo_variables \
        `## WORKER_HOSTS` \
        -S 192.168.43.241 \
        -S 127.0.0.1 \
        `## FUNCTION TO CALL` \
        echo_variables \

}

echo_variables() {

    echo "WORKER_HOST      : ${HOSTNAME}"
    echo "WORKER_DOWNPATH  : ${DOWN_PATH}"
    echo "MASTER_HOST      : ${MASTER_HOSTNAME}"
    echo "MASTER_DOWNPATH  : ${MASTER_DOWNPATH}"
    echo "-------------------------------------"

}
export -f echo_variables

main


### Output
#WORKER_HOST      : verneynas
#WORKER_DOWNPATH  : /home/fakuve/files/downloads
#MASTER_HOST      : elitebook-x360
#MASTER_DOWNPATH  : /home/fakuve/downloads
#-------------------------------------
#WORKER_HOST      : elitebook-x360
#WORKER_DOWNPATH  : /home/fakuve/downloads
#MASTER_HOST      : elitebook-x360
#MASTER_DOWNPATH  : /home/fakuve/downloads
#-------------------------------------
```

You can see how we can proceed with clarity from now on.
At the begining of the script we assing the GLOBAL SCRIPT VARIABLES, which regardless of which host is the caller they are not going to be modified. In this case I like to call `MASTER_<VARNAME>` to caller variables, and to host specific variables will call them `WORKER_<VARNAME>`

As you can see the MASTER variables need to be also passed via `env_parallel` , dont forget that.
The rest they get assigned for each call of parallel in each host. For clarity you can reassign them within the function

```
echo_variables() {
WORKER_HOST=${HOSTNAME}
WORKER_DOWNPATH=${DOWN_PATH}
}
```

Note that `DOWN_PATH` has been assigned for each host in each `.bashrc` and `$HOME/.ssh/environment`



## Extra chapter, parallel environment variables

In for execution parallel sets some environments variables that you can access.
See `man parallel` `Environment Variables`

Example

```
#!/bin/bash

main(){
    seq 2 | parallel -n0 \
            --jobs 1 \
            -S 192.168.43.241 \
            -S 127.0.0.1 \
            --env echo_info \
            echo_info
}


echo_info(){
    echo ${PARALLEL_SSHLOGIN}
}
export -f echo_info

main
```

Will output

```
127.0.0.1
192.168.43.241
```


## Conclusion

After all of this we will have unravelled this `gnu parallel` tool, and will be able to work with it for our use case in the next chapter.
**Building up a parallel webcrawler**
