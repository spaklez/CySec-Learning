Every process in Linux has three initial, standard channels of communication:

- Standard _Input_ is the channel through which the process takes input. For example, your shell uses Standard Input to read the commands that you input.
- Standard _Output_ is the channel through which processes output normal data, such as the flag when it is printed to you in previous challenges or the output of utilities such as `ls`.
- Standard _Error_ is the channel through which processes output error details. For example, if you mistype a command, the shell will output, over standard error, that this command does not exist.

Because these three channels are used so frequently in Linux, they are known by shorter names: `stdin`, `stdout`, `stderr`.

First, let's look at redirecting stdout to files. You can accomplish this with the `>` character, as so:

```console
hacker@dojo:~$ echo hi > asdf
```

This will redirect the output of `echo hi` (which will be `hi`) to the file `asdf`. You can then use a program such as `cat` to output this file:

```console
hacker@dojo:~$ cat asdf
hi
```

One can also think of it in this way, the fd1 points to the asdf file, 

You can redirect input in _append_ mode using `>>` instead of `>`, as so:

```console
hacker@dojo:~$ echo pwn > outfile
hacker@dojo:~$ echo college >> outfile
hacker@dojo:~$ cat outfile
pwn
college
hacker@dojo:$
```

 A File Descriptor (FD) is a number that _describes_ a communication channel in Linux. You've already been using them, even though you didn't realize it. We're already familiar with three:

- FD 0: Standard Input
- FD 1: Standard Output
- FD 2: Standard Error

When you redirect process communication, you do it by FD number, though some FD numbers are implicit. For example, a `>` without a number implies `1>`, which redirects FD 1 (Standard Output). Thus, the following two commands are equivalent:

```console
hacker@dojo:~$ echo hi > asdf
hacker@dojo:~$ echo hi 1> asdf
```

Redirecting errors is pretty easy from this point. If you have a command that might produce data via standard error, you can do:

```console
hacker@dojo:~$ /challenge/run 2> errors.log
```

That will redirect standard error (FD 2) to the `errors.log` file. Furthermore, you can redirect multiple file descriptors at the same time! For example:

```console
hacker@dojo:~$ some_command > output.log 2> errors.log
```

That command will redirect output to `output.log` and errors to `errors.log`.

Just like you can redirect _output_ from programs, you can redirect input _to_ programs! This is done using `<`, as so:

```
hacker@dojo:~$ echo yo > message
hacker@dojo:~$ cat message
yo
hacker@dojo:~$ rev < message
oy
```

Using the `|` (pipe) operator. Standard output from the command to the left of the pipe will be connected to (_piped into_) the standard input of the command to the right of the pipe. For example:

```
hacker@dojo:~$ echo no-no | grep yes
hacker@dojo:~$ echo yes-yes | grep yes
yes-yes
hacker@dojo:~$ echo yes-yes | grep no
hacker@dojo:~$ echo no-no | grep no
no-no
```

 The `|` operator redirects _only standard output_ to another program, and there is no `2|` form of the operator! It can _only_ redirect standard output (file descriptor 1). The shell has a `>&` operator, which redirects a file descriptor _to another file descriptor_. First, we redirect standard error to standard output (`2>& 1`) and then pipe the now-combined stderr and stdout as normal (`|`)!

Some useful commands to know from this module are, 

-) grep -v - i.e. normal grep but the -v inverts
-) sed "s/oldword/newword/g" - basically replacing oldword with newword, the g stands for globally. 
-) tee - duplicates data flowing through pipes, an example is

```console
hacker@dojo:~$ echo hi | tee pwn college
hi
hacker@dojo:~$ cat pwn
hi
hacker@dojo:~$ cat college
hi
hacker@dojo:~$
```

As you can see, by providing two files to `tee`, we ended up with three copies of the piped-in data: one to stdout, one to the `pwn` file, and one to the `college` file.

One can  hook input and output of programs to _arguments_ of commands. This is done using Process Substitution. For reading from a command (input process substitution), use `<(command)`. When you write `<(command)`, bash will run the command and hook up its output to a temporary file that it will create. This isn't a _real_ file, of course, it's what's called a _named pipe_, in that it has a file name.
Bash can also make commands look like files using process substitution! For writing to a command (output process substitution), use `>(command).

You can also create your own _persistent_ named pipes that stick around on the filesystem! These are called **FIFOs**, which stands for First (byte) In, First (byte) Out.

You create a FIFO using the `mkfifo` command:

```console
hacker@dojo:~$ mkfifo my_pipe
hacker@dojo:~$ ls -l my_pipe
prw-r--r-- 1 hacker hacker 0 Jan 1 12:00 my_pipe
hacker@dojo:~$ ls -l some_file
-rw-r--r-- 1 hacker hacker 0 Jan 1 12:00 some_file
hacker@dojo:~$
```

Unlike the automatic named pipes from process substitution:

- You control where FIFOs are created
- They persist until you delete them
- Any process can write to them by path (e.g., `echo hi > my_pipe`)
- You can see them with `ls` and examine them like files

However, FIFOs block operations until the read and write side of the pipe is ready. 