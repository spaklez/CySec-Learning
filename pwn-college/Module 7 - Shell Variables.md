You can write complete programs in the Linux command line interface, these are known as "shell scripts". The shell supports variables.  

An example of a variable is PWD, this variable always stores the current working directory of the current shell. You can thus print out such variables using echo, an example is,

```console
hacker@dojo:~$ echo $PWD
/home/hacker
```

You can also write values to variables, this is done so by using the = similar to many programming languages. An example is 

```console
hacker@dojo:~$ VAR=1337
```

Putting spaces confuses the shell, i.e. writing VAR = 1337 causes the shell to interpret running VAR command which doesn't exist. Here also we use VAR as is, and not `$VAR`: the `$` is only prepended to _access_ variables. In shell terms, this prepending of `$` triggers what is called _variable expansion_.

Also remember that Spaces have a special meaning in the shell, whenever the Shell sees a space, it ends it's variable assignment and interprets the word after the space as a command. An example is, 

```console
hacker@dojo:~$ VAR=1337 SAUCE
```

is wrong, but the correct way to do this would be

```console
hacker@dojo:~$ VAR="1337 SAUCE"
```

By default, variables that you set in a shell session are local to that shell process, if you invoke another shell process it doesn't receive those variables. An example is 
```console
hacker@dojo:~$ VAR=1337
hacker@dojo:~$ echo "VAR is: $VAR"
VAR is: 1337
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 
```

In the output above, the `$` prompt is the prompt of `sh`, a minimal shell implementation that is invoked as a _child_ of the main shell process.

To use variables in other instances, you export those variables, these are passed into the _environment variables_ of child processes. An example of exporting such variables, 

```console
hacker@dojo:~$ VAR=1337
hacker@dojo:~$ export VAR
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 1337
```

You can also combine those first two lines, 

```console
hacker@dojo:~$ export VAR=1337
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 1337
```

The env command is also used to print out every exported variable set in your shell. You can also store the output of commands into a variable. We make use of Command Substitution. An example is 

```console
hacker@dojo:~$ FLAG=$(cat /flag)
hacker@dojo:~$ echo "$FLAG"
pwn.college{blahblahblah}
hacker@dojo:~$
```

To read from the user, use the simple 'read' command, this command reads input into a variable. Using the -p flag let's you specify a prompt. 'read' reads from your Standard Input. Additionally when reading a file into an environment variable, one can use the 'read' command as it takes the standard output as it's standard input. An example is

```console
hacker@dojo:~$ echo "test" > some_file
hacker@dojo:~$ read VAR < some_file
hacker@dojo:~$ echo $VAR
test
```

