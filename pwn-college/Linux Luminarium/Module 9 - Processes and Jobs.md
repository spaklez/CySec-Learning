In modern computing, software is split into two categories, operating system kernels and processes. When Linux starts up it launches an init process that in turn launches other processes until your Shell is launched up. (The init process is the ultimate parent process of all other processes). 

The whole goal about this module is to view and interact with processes. Remember that each process has a numerical identifier (the Process ID).

-) ps - used to list processes, the processes listed are those that are running in the terminal. An example is 

```console 
hacker@dojo:~$ ps
    PID TTY          TIME CMD
    329 pts/0    00:00:00 bash
    349 pts/0    00:00:00 ps
hacker@dojo:~$
```

In the above example, we have the process bash (i.e. the shell) and the ps process itself. These processes are running on this specific terminal. 

In this case, the designation pts/0 refers to the the terminal that is running these processes .
There are two ways to specify these arguments,

**"Standard" Syntax:** in this syntax, you can use `-e` to list "every" process and `-f` for a "full format" output, including arguments. These can be combined into a single argument `-ef`.

**"BSD" Syntax:** in this syntax, you can use `a` to list processes for all users, `x` to list processes that aren't running in a terminal, and `u` for a "user-readable" output. These can be combined into a single argument `aux`.

To kill processes, just use the kill command, you pass the process identifier as an argument with the command. An example is 

```console
hacker@dojo:~$ ps -e | grep sleep
 342 pts/0    00:00:00 sleep
hacker@dojo:~$ kill 342
hacker@dojo:~$ ps -e | grep sleep
hacker@dojo:~$
```

Ctrl + C - Interrupt processes
Ctrl + Z - Suspend processes
To resume a suspended process, make use of the fg command, press enter to quit this resumed process 
You make use of the bg command, when you want to keep a  process running while giving us control of the shell
You can background processes without having to always suspend them first, you do this by appending a & to the command. 

To check the exit code of any process, you use 'echo $?', this gives you the exit code of the of most recently terminated command. 
Typically, a return value of 0 means that the command terminated successfully, whereas any non-zero return value (typically 1) indicates failure.  