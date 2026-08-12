---
Course: "[[All About Programming]]"
Date: 2026-03-08
---
This module will teach you the VERY basics of interacting with the command line! The _command_ line lets you execute _commands_. When you launch a terminal, it will execute a command line "shell", but before we begin, what is the Command Line? 

# The Command Line: 

The command line, aka "the shell", it's also called the **shell** because of a simple metaphor: it acts like the outer shell of a nut or a seed, wrapping around the core of the operating system.

This is because we can still use the "computer" via the shell itself, the graphical desktop (with windows, mouse pointers, and icons) is completely optional. It is just another program running on top of the system.

The basic idea behind the Command Line is:
	1)  You type a command 
	2)  The system executes it and outputs the results

Typically a command will contain a program name and arguments/parameters to that program. An example of a Linux program is
-) cat - concatenates and prints the contents of any file that is named after cat, example cat file1.txt
#### Command Parameters/Arguments

With cat it's simple, you just give a file name, but some commands/programs take in multiple arguments

![[hello hacker, multiple arguments.png]]

For an extensive deepdive into more about bash, follow the link --> https://bash.cyberciti.biz/guide/Main_Page

#### The Starting Prompt

```console
hacker@dojo:~$
```

This is called the "prompt", and it's prompting you to enter a command. Let's take a look at what's going on here:

- The `hacker` in the prompt is the _username_ of the current user. In the pwn.college DOJO environment, this is "hacker".
- In the example above, the `dojo` part of the prompt is the _hostname_ of the machine the shell is on (this reminder can be useful if you are a system administrator who deals with many machines on a daily basis, for example). In the example above, the hostname is `dojo`, but in pwn.college, it will be derived from the name of the challenge you're attempting.
- We will cover what `~` means later :-)
- The `$` at the end of the prompt signifies that `hacker` is not an administrative user. In much later modules in pwn.college, when you learn to use exploits to become the administrative user, you will see the prompt signify that by printing `#` instead of `$`, and you'll know that you've won!

Additionally, to access old/history of commands typed during a session, just hit the up and down arrows.

# Linux File Paths

A file system in general is a storage structure used to store files. Files live inside directories, Directories themselves can live inside other Directories. 

In Linux, this goes on until you hit the 'outermost' directory which is the 'root' directory!
The Linux filesystem is a "tree". That is, it has a root (written as `/`). The root of the filesystem is a directory, and every directory can contain other directories and files. You refer to files and directories by their _path_. A path from the root of the filesystem starts with `/` (that is, the root of the filesystem), and describes the set of directories that must be descended into to find the file. Every piece of the path is demarcated with another `/`.

![[linux filesystem.png]]

#### Navigating File Systems 

![[navigating file systems.png]]

#### Specifying Paths

There are two ways to specify paths 
![[paths.png]]

For example
	-) ls /home/yans/flags is an Absolute Path
	-) cat TOPSECRET is a Relative Path

Additionally, "." represents the CWD, and ".." represents the directory that the CWD lives in 

# Comprehending Commands

One of the most critical Linux commands is `cat`. `cat` is most often used for reading out files, like so:

```console
hacker@dojo:~$ cat /path/to/file
Hello Hackers!
```

`cat` will con**cat**enate (hence the name) multiple files if provided multiple arguments. For example:

```console
hacker@dojo:~$ cat myfile
This is my file!
hacker@dojo:~$ cat yourfile
This is your file!
hacker@dojo:~$ cat myfile yourfile
This is my file!
This is your file!
hacker@dojo:~$ cat myfile yourfile myfile
This is my file!
This is your file!
This is my file!
```

Finally, if you give no arguments at all, `cat` will read from the terminal input and output it.

Sometimes, the files that you might `cat` out are too big. Luckily, we have the `grep` command to search for the contents we need! There are many ways to `grep`, and we'll learn one way here:

```console
hacker@dojo:~$ grep SEARCH_STRING /path/to/file
```

Invoked like this, `grep` will search the file for lines of text containing `SEARCH_STRING` and print them to the console.

When looking for changes between similar files, eyeballing them might not be the most efficient approach! This is where the `diff` command becomes invaluable. The `diff` command is strictly a **read-only** tool.

`diff` compares two files line by line and shows you exactly what's different between them. For example:

```console
hacker@dojo:~$ cat file1
hello
world
hacker@dojo:~$ cat file2
hello
universe
hacker@dojo:~$ diff file1 file2
2c2
< world
---
> universe
```

The output tells us that line 2 changed (`2c2`), with `world` in the first file (`<`) being replaced by `universe` in the second file (`>`).

Sometimes, when new lines are added, you'll see something like:

```console
hacker@dojo:~$ cat old
pwn
hacker@dojo:~$ cat new
pwn
college
hacker@dojo:~$ diff old new
1a2
> college
```

This tells us that after line 1 in the first file, the second file has an additional line (`1a2` means "after line 1 of file1, add line 2 of file2").

`ls` will list files in all the directories provided to it as arguments, and in the current directory if no arguments are provided. Observe:

```console
hacker@dojo:~$ ls /challenge
run
hacker@dojo:~$ ls
Desktop    Downloads  Pictures  Templates
Documents  Music      Public    Videos
hacker@dojo:~$ ls /home/hacker
Desktop    Downloads  Pictures  Templates
Documents  Music      Public    Videos
hacker@dojo:~$
```

Interestingly, `ls` doesn't list _all_ the files by default. Linux has a convention where files that start with a `.` don't show up by default in `ls` and in a few other contexts. To view them with `ls`, you need to invoke `ls` with the `-a` flag, as so:

```console
hacker@dojo:~$ touch pwn
hacker@dojo:~$ touch .college
hacker@dojo:~$ ls
pwn
hacker@dojo:~$ ls -a
.college	pwn
hacker@dojo:~$
```

You can also _create_ files! There are several ways to do this, but we'll look at a simple command here. You can create a new, blank file by _touching_ it with the `touch` command:

```
hacker@dojo:~$ cd /tmp
hacker@dojo:/tmp$ ls
hacker@dojo:/tmp$ touch pwnfile
hacker@dojo:/tmp$ ls
pwnfile
hacker@dojo:/tmp$
```

In Linux, you **r**e**m**ove files with the `rm` command, as so:

```console
hacker@dojo:~$ touch PWN
hacker@dojo:~$ touch COLLEGE
hacker@dojo:~$ ls
COLLEGE     PWN
hacker@dojo:~$ rm PWN
hacker@dojo:~$ ls
COLLEGE
hacker@dojo:~$
```

You can also _move_ files around with the `mv` command. The usage is simple:

```console
hacker@dojo:~$ ls
my-file
hacker@dojo:~$ cat my-file
PWN!
hacker@dojo:~$ mv my-file your-file
hacker@dojo:~$ ls
your-file
hacker@dojo:~$ cat your-file
PWN!
hacker@dojo:~$
```

But what if you want to keep the original file? You can do so with the `cp` command. The usage is the same as with `mv`, but it will keep the source file. The command is `cp SOURCE DESTINATION`: the first argument is the existing file, and the second is where the copy should be created.

**NOTE:** When a `cp` destination is a directory, `cp` places the copy inside it using the source file's name.

You **m**a**k**e **dir**ectories using the `mkdir` command. Then you can stick files in there!

```console
hacker@dojo:~$ cd /tmp
hacker@dojo:/tmp$ ls
hacker@dojo:/tmp$ ls
hacker@dojo:/tmp$ mkdir my_directory
hacker@dojo:/tmp$ ls
my_directory
hacker@dojo:/tmp$ cd my_directory
hacker@dojo:/tmp/my_directory$ touch my_file
hacker@dojo:/tmp/my_directory$ ls
my_file
hacker@dojo:/tmp/my_directory$ ls /tmp/my_directory/my_file
/tmp/my_directory/my_file
hacker@dojo:/tmp/my_directory$
```

The `find` command takes optional arguments describing the search criteria and the search location. If you don't specify a search criteria, `find` matches every file. If you don't specify a search location, `find` uses the current working directory (`.`). For example:

```console
hacker@dojo:~$ mkdir my_directory
hacker@dojo:~$ mkdir my_directory/my_subdirectory
hacker@dojo:~$ touch my_directory/my_file
hacker@dojo:~$ touch my_directory/my_subdirectory/my_subfile
hacker@dojo:~$ find
.
./my_directory
./my_directory/my_subdirectory
./my_directory/my_subdirectory/my_subfile
./my_directory/my_file
hacker@dojo:~$
```

And when specifying the search location:

```console
hacker@dojo:~$ find my_directory/my_subdirectory
my_directory/my_subdirectory
my_directory/my_subdirectory/my_subfile
hacker@dojo:~$
```

And, of course, we can specify the criteria! For example, here, we filter by name:

```console
hacker@dojo:~$ find -name my_subfile
./my_directory/my_subdirectory/my_subfile
hacker@dojo:~$ find -name my_subdirectory
./my_directory/my_subdirectory
hacker@dojo:~$
```

You can search the whole filesystem if you want!

```console
hacker@dojo:~$ find / -name hacker
/home/hacker
hacker@dojo:~$
```

# File types in Linux

There are many different types of files in Linux, using ls -l or ls -ld shows us the types of these files 

![[linux file types.png]]

A symbolic link basically is a link that a file uses which references another file, 

![[symbolic links.png]]

![[symbolic link gotchas.png]]

![[Hard links.png]]

Links come in two flavors: _hard_ and _soft_ (also known as _symbolic_) links. We'll differentiate the two with an analogy:

- A **hard** link is when you address your apartment using multiple addresses that all lead directly to the same place (e.g., `Apt 2` vs `Unit 2`).
- A **soft** link is when you move apartments and have the postal service automatically forward your mail from your old place to your new place.

If you use Linux (or computers) for any reasonable length of time to do any real work, you will eventually run into some variant of the following situation: you want two programs to access the same data, but the programs expect that data to be in two different locations. Luckily, Linux provides a solution to this quandary: _links_.

Links come in two flavors: _hard_ and _soft_ (also known as _symbolic_) links. We'll differentiate the two with an analogy:

- A **hard** link is when you address your apartment using multiple addresses that all lead directly to the same place (e.g., `Apt 2` vs `Unit 2`).
- A **soft** link is when you move apartments and have the postal service automatically forward your mail from your old place to your new place.

In a filesystem, a file is, conceptually, an address at which the contents of that file live. A hard link is an alternate address that indexes that data accesses to the hard link and accesses to the original file are completely identical, in that they immediately yield the necessary data. A soft/symbolic link, instead, contains the original file name. When you access the symbolic link, Linux will realize that it is a symbolic link, read the original file name, and then (typically) automatically access that file. In most cases, both situations result in accessing the original data, but the mechanisms are different.

Hard links sound simpler to most people (case in point, I explained it in one sentence above, versus two for soft links), but they have various downsides and implementation gotchas that make soft/symbolic links, by far, the more popular alternative.

Symbolic links are created with the `ln` command using the syntax `ln -s TARGET LINK_NAME`. `TARGET` is the path that the link will point to, and `LINK_NAME` is the new path you are creating. For example:

```console
hacker@dojo:~$ cat /tmp/myfile
This is my file!
hacker@dojo:~$ ln -s /tmp/myfile /home/hacker/ourfile
hacker@dojo:~$ cat ~/ourfile
This is my file!
hacker@dojo:~$
```

You can see that accessing the symlink results in getting the original file contents! In this example, `/tmp/myfile` is the target and `/home/hacker/ourfile` is the link name.

A symlink can be identified as such with a few methods. For example, the `file` command, which takes a filename and tells you what type of file it is, will recognize symlinks:

```console
hacker@dojo:~$ file /tmp/myfile
/tmp/myfile: ASCII text
hacker@dojo:~$ file ~/ourfile
/home/hacker/ourfile: symbolic link to /tmp/myfile
hacker@dojo:~$
```

The `man` command. `man` is short for `manual`, and will display (if available) the manual of the command you pass as an argument. For example, let's say we wanted to learn about the `yes` command (_yes_, this is a real command):

```console
hacker@dojo:~$ man yes
```

This will display the manual page for `yes`, which will look something like this:

```text
YES(1)                           User Commands                          YES(1)

NAME
       yes - output a string repeatedly until killed

SYNOPSIS
       yes [STRING]...
       yes OPTION

DESCRIPTION
       Repeatedly output a line with all specified STRING(s), or 'y'.

       --help display this help and exit

       --version
              output version information and exit

AUTHOR
       Written by David MacKenzie.

REPORTING BUGS
       GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
       Report any translation bugs to <https://translationproject.org/team/>

COPYRIGHT
       Copyright  ©  2020  Free Software Foundation, Inc.  License GPLv3+: GNU
       GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
       This is free software: you are free  to  change  and  redistribute  it.
       There is NO WARRANTY, to the extent permitted by law.

SEE ALSO
       Full documentation <https://www.gnu.org/software/coreutils/yes>
       or available locally via: info '(coreutils) yes invocation'

GNU coreutils 8.32               February 2022                          YES(1)
```

The important sections are:

```text
NAME(1)                           CATEGORY                          NAME(1)

NAME
	This gives the name (and short description) of the command or
	concept discussed by the page.

SYNOPSIS
	This gives a short usage synopsis. These synopses have a standard
	format. Typically, each line is a different valid invocation of the
	command, and the lines can be read as:

	COMMAND [OPTIONAL_ARGUMENT] SINGLE_MANDATORY_ARGUMENT
	COMMAND [OPTIONAL_ARGUMENT] MULTIPLE_ARGUMENTS...

DESCRIPTION
	Details of the command or concept, with detailed descriptions
	of the various options.

SEE ALSO
	Other man pages or online resources that might be useful.

COLLECTION                        DATE                          NAME(1)
```

You can scroll around the manpage with your arrow keys and PgUp/PgDn. When you're done reading the manpage, you can hit `q` to quit.

Manpages are stored in a centralized database. If you're curious, this database lives in the `/usr/share/man` directory, but you never need to interact with it directly: you just query it using the `man` command. The arguments to the `man` command aren't file paths, but just the names of the entries themselves (e.g., you run `man yes` to look up the `yes` manpage, rather than `man /usr/bin/yes`, which would be the actual path to the `yes` program but would result in `man` displaying garbage).

You can scroll man pages with the arrow keys (and PgUp/PgDn) and search with `/`. After searching, you can hit `n` to go to the next result and `N` to go to the previous result. Instead of `/`, you can use `?` to search backwards!

Some programs don't have a man page, but might tell you how to run them if invoked with a special argument. Usually, this argument is `--help`, but it can often be `-h` or, in rare cases, `-?`, `help`, or other esoteric values like `/?` (though that latter is more frequently encountered on Windows).

Some commands, rather than being programs with man pages and help options, are built into the shell itself. These are called _builtins_. Builtins are invoked just like commands, but the shell handles them internally instead of launching other programs. You can get a list of shell builtins by running the _builtin_ `help`, as so:

```console
hacker@dojo:~$ help
```

You can get help on a specific one by passing it to the `help` builtin. Let's look at a builtin that we've already used earlier, `cd`!

```console
hacker@dojo:~$ help cd
cd: cd [-L|[-P [-e]] [-@]] [dir]
    Change the shell working directory.
    
    Change the current directory to DIR.  The default DIR is the value of the
    HOME shell variable.
...
```

# Globbing

Before executing commands that you enter, the shell first performs _expansions_ on them, and one of these expansions is globbing. Globbing lets you reference files without typing them all out, or typing out their full paths. 

The first glob we'll learn is `*`. When it encounters a `*` character in any argument, the shell will treat it as a "wildcard" and try to replace that argument with any files that match the pattern. It's easier to show you than explain:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_*
Look: file_a file_b file_c
```

Of course, though in this case, the glob resulted in multiple arguments, it can just as simply match only one. For example:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ ls
file_a
hacker@dojo:~$ echo Look: file_*
Look: file_a
```

When zero files are matched, by default, the shell leaves the glob unchanged:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ ls
file_a
hacker@dojo:~$ echo Look: nope_*
Look: nope_*
```

The `*` matches any part of the filename except for `/` or a leading `.` character. For example:

```console
hacker@dojo:~$ echo ONE: /ho*/*ck*
ONE: /home/hacker
hacker@dojo:~$ echo TWO: /*/hacker
TWO: /home/hacker
hacker@dojo:~$ echo THREE: ../*
THREE: ../hacker
```

Remember that the shell isn't randomly guessing letters or going on forever. It is literally looking at the actual files and folders that exist on your hard drive.

When you type `/ho*`, the shell stops for a second and asks the computer: _"Do we have any folders in the main directory that start with 'ho'?"_ Because the only folder on the system that starts with "ho" is `/home`, the shell instantly replaces `/ho*` with `/home` before it even runs the command. It stops right at the end of the folder name.

If you had folders named `/home`, `/house`, and `/hound`, typing `ls /ho*` would list all three of them.

Next, let's learn about `?`. When it encounters a `?` character in any argument, the shell will treat it as a **single-character** wildcard. This works like `*`, but only matches _one_ character. For example:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_cc
hacker@dojo:~$ ls
file_a	file_b	file_cc
hacker@dojo:~$ echo Look: file_?
Look: file_a file_b
hacker@dojo:~$ echo Look: file_??
Look: file_cc
```

Next, we will cover `[]`. The square brackets are, essentially, a limited form of `?`, in that instead of matching any character, `[]` is a wildcard for some subset of potential characters, specified within the brackets. For example, `[pwn]` will match the character `p`, `w`, or `n`. For example:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_[ab]
Look: file_a file_b
```

Globbing happens on a _path_ basis, so you can expand entire paths with your globbed arguments. For example:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: /home/hacker/file_[ab]
Look: /home/hacker/file_a /home/hacker/file_b
```

So far, you've specified one glob at a time, but you can do more! Bash supports the expansion of multiple globs in a single word. For example:

```console
hacker@dojo:~$ cat /*fl*
pwn.college{YEAH}
hacker@dojo:~$
```

What happens above is that the shell looks for all files in `/` that start with _anything_ (including nothing), then have an `f` and an `l`, and end in _anything_ (including `ag`, which makes `flag`).

Sometimes, you want to filter out files in a glob! Luckily, `[]` helps you do just this. If the first character in the brackets is a `!` or (in newer versions of bash) a `^`, the glob inverts, and that bracket instance matches characters that _aren't_ listed. For example:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_[!ab]
Look: file_c
hacker@dojo:~$ echo Look: file_[^ab]
Look: file_c
hacker@dojo:~$ echo Look: file_[ab]
Look: file_a file_b
```

As tempting as it might be, using `*` to shorten what must be typed on the commandline can lead to mistakes. Your glob might expand to unintended files, and you might not spot it until the `rm` command is already running! No one is safe from this style of error.

A safer alternative when you are trying to specify a specific target is _tab completion_. If you hit tab in the shell, it'll try to figure out what you're going to type and automatically complete it. Auto-completion is super useful

Consider the following situation:

```console
hacker@dojo:~$ ls
flag  flamingo  flowers
hacker@dojo:~$ cat f<TAB>
```

There are multiple options! What happens?

What happens varies based on the specific shell and its options. By default `bash` will auto-expand until the first point when there are multiple options (in this case, `fl`). When you hit tab a _second_ time, it'll print out those options. Other shells and configurations, instead, will cycle through the options. Tab completion is for more than files! You can also tab-complete commands

# Piping

Every process in Linux has three initial, standard channels of communication:

- Standard _Input_ is the channel through which the process takes input. For example, your shell uses Standard Input to read the commands that you input.
- Standard _Output_ is the channel through which processes output normal data, such as the flag when it is printed to you in previous challenges or the output of utilities such as `ls`.
- Standard _Error_ is the channel through which processes output error details. For example, if you mistype a command, the shell will output, over standard error, that this command does not exist.

Because these three channels are used so frequently in Linux, they are known by shorter names: `stdin`, `stdout`, `stderr`. Here is a great [visual guide to I/O redirection](http://www.rozmichelle.com/pipes-forks-dups/) in Linux.

First, let's look at redirecting stdout to files. You can accomplish this with the `>` character, as so:

```console
hacker@dojo:~$ echo hi > asdf
```

This will redirect the output of `echo hi` (which will be `hi`) to the file `asdf`. You can then use a program such as `cat` to output this file:

```console
hacker@dojo:~$ cat asdf
hi
```

Aside from redirecting the output of `echo`, you can, of course, redirect the output of any command.

A common use-case of output redirection is to save off some command results for later analysis. Often times, you want to do this in _aggregate_: run a bunch of commands, save their output, and `grep` through it later. In this case, you might want all that output to keep appending to the same file, but `>` will create a new output file every time, deleting the old contents.

You can redirect input in _append_ mode using `>>` instead of `>`, as so:

```console
hacker@dojo:~$ echo pwn > outfile
hacker@dojo:~$ echo college >> outfile
hacker@dojo:~$ cat outfile
pwn
college
hacker@dojo:$
```

Just like standard output, you can also redirect the error channel of commands. Here, we'll learn about _File Descriptor numbers_. A File Descriptor (FD) is a number that _describes_ a communication channel in Linux. You've already been using them, even though you didn't realize it. We're already familiar with three:

- FD 0: Standard Input
- FD 1: Standard Output
- FD 2: Standard Error

When you redirect process communication, you do it by FD number, though some FD numbers are implicit. For example, a `>` without a number implies `1>`, which redirects FD 1 (Standard Output). Thus, the following two commands are equivalent:

```console
hacker@dojo:~$ echo hi > asdf
hacker@dojo:~$ echo hi 1> asdf
```

Redirecting errors is pretty easy from this point. If you have a command that might produce data via standard error (such as `/challenge/run`), you can do:

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

 The `|` operator redirects _only standard output_ to another program, and there is no `2|` form of the operator! It can _only_ redirect standard output (file descriptor 1). The shell has a `>&` operator, which redirects a file descriptor _to another file descriptor_. This means that we can have a two-step process to `grep` through errors: first, we redirect standard error to standard output (`2>& 1`) and then pipe the now-combined stderr and stdout as normal (`|`)!

The `grep` command has a very useful option: `-v` (invert match). While normal `grep` shows lines that MATCH a pattern, `grep -v` shows lines that do NOT match a pattern:

```console
hacker@dojo:~$ cat data.txt
hello hackers!
hello world!
hacker@dojo:~$ cat data.txt | grep -v world
hello hackers!
hacker@dojo:~$
```

Sometimes, the only way to filter to just the data you want is to filter _out_ the data you _don't_ want.

`grep` is not the only way to match patterns. Sometimes the real data and the garbage data are mixed in the same line, and we want to filter out the garbage. For that, we have `sed`. `sed` provides an easy way to substitute patterns in text with a different word. The syntax for matching and replacing is simple:

```
sed "s/oldword/newword/g"
```

`s/` - substitute  
`oldword` - the word to replace  
`newword` - the replacement for `oldword`  
`/g` - global (search for all occurrences of the pattern)

When you pipe data from one command to another, you of course no longer see it on your screen. This is not always desired: for example, you might want to see the data as it flows through between your commands to debug unintended outcomes (e.g., "why did that second command not work???").

Luckily, there is a solution! The `tee` command, named after a "T-splitter" from _plumbing_ pipes, duplicates data flowing through your pipes to any number of files provided on the command line. For example:

```console
hacker@dojo:~$ echo hi | tee pwn college
hi
hacker@dojo:~$ cat pwn
hi
hacker@dojo:~$ cat college
hi
hacker@dojo:~$
```

As you can see, by providing two files to `tee`, we ended up with three copies of the piped-in data: one to stdout, one to the `pwn` file, and one to the `college` file. You can imagine how you might use this to debug things going haywire:

```console
hacker@dojo:~$ command_1 | command_2
Command 2 failed!
hacker@dojo:~$ command_1 | tee cmd1_output | command_2
Command 2 failed!
hacker@dojo:~$ cat cmd1_output
Command 1 failed: must pass --succeed!
hacker@dojo:~$ command_1 --succeed | command_2
Commands succeeded!
```

Sometimes you need to compare the output of two commands rather than two files. You might think to save each output to a file first:

```console
hacker@dojo:~$ command1 > file1
hacker@dojo:~$ command2 > file2
hacker@dojo:~$ diff file1 file2
```

But there's a more elegant way! Linux follows the philosophy that ["everything is a file"](https://en.wikipedia.org/wiki/Everything_is_a_file). That is, the system strives to provide file-like access to most resources, including the input and output of running programs! The shell follows this philosophy, allowing you to, for example, use any utility that takes file arguments on the command line and hook it up to the output of programs. 

Interestingly, we can go further, and hook input and output of programs to _arguments_ of commands. This is done using [Process Substitution](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html). For reading from a command (input process substitution), use `<(command)`. When you write `<(command)`, bash will run the command and hook up its output to a temporary file that it will create. This isn't a _real_ file, of course, it's what's called a _named pipe_, in that it has a file name:

```console
hacker@dojo:~$ echo <(echo hi)
/dev/fd/63
hacker@dojo:~$
```

Where did `/dev/fd/63` come from? `bash` replaced `<(echo hi)` with the path of the named pipe file that's hooked up to the command's output! While the command is running, reading from this file will read data from the standard output of the command. Typically, this is done using commands that take input files as arguments:

```console
hacker@dojo:~$ cat <(echo hi)
hi
hacker@dojo:~$
```

Of course, you can specify this multiple times:

```console
hacker@dojo:~$ echo <(echo pwn) <(echo college)
/dev/fd/63 /dev/fd/64
hacker@dojo:~$ cat <(echo pwn) <(echo college)
pwn
college
hacker@dojo:~$
```

You can duplicate data to two files with `tee`:

```console
hacker@dojo:~$ echo HACK | tee THE > PLANET
hacker@dojo:~$ cat THE
HACK
hacker@dojo:~$ cat PLANET
HACK
hacker@dojo:~$
```

And you've used `tee` to duplicate data to a file and a command:

```console
hacker@dojo:~$ echo HACK | tee THE | cat
HACK
hacker@dojo:~$ cat THE
HACK
hacker@dojo:~$
```

But what about duplicating to two commands? As `tee` says in its manpage, it's designed to write to files and to standard output:

```text
TEE(1)                           User Commands                          TEE(1)

NAME
       tee - read from standard input and write to standard output and files
```

But wait! You just learned that bash can make commands look like files using process substitution! For writing to a command (output process substitution), use `>(command)`. If you write an argument of `>(rev)`, bash will run the `rev` command (this command reads data from standard input, reverses its order, and writes it to standard output!), but hook up its input to a temporary named pipe file. When commands write to this file, the data goes to the standard input of the command:

```console
hacker@dojo:~$ echo HACK | rev
KCAH
hacker@dojo:~$ echo HACK | tee >(rev)
HACK
KCAH
```

Above, the following sequence of events took place:

1. `bash` started up the `rev` command, hooking a named pipe (presumably `/dev/fd/63`) to `rev`'s standard input
2. `bash` started up the `tee` command, hooking a pipe to its standard input, and replacing the first argument to `tee` with `/dev/fd/63`. `tee` never even saw the argument `>(rev)`; the shell _substituted_ it before launching `tee`
3. `bash` used the `echo` builtin to print `HACK` into `tee`'s standard input
4. `tee` read `HACK`, wrote it to standard output, and then wrote it to `/dev/fd/63` (which is connected to `rev`'s stdin)
5. `rev` read `HACK` from its standard input, reversed it, and wrote `KCAH` to standard output

You've learned about pipes using `|`, and you've seen that process substitution creates temporary named pipes (like `/dev/fd/63`). You can also create your own _persistent_ named pipes that stick around on the filesystem! These are called **FIFOs**, which stands for First (byte) In, First (byte) Out.

You create a FIFO using the `mkfifo` command:

```console
hacker@dojo:~$ mkfifo my_pipe
hacker@dojo:~$ ls -l my_pipe
prw-r--r-- 1 hacker hacker 0 Jan 1 12:00 my_pipe
hacker@dojo:~$ ls -l some_file
-rw-r--r-- 1 hacker hacker 0 Jan 1 12:00 some_file
hacker@dojo:~$
```

Notice the `p` at the beginning of the permissions - that indicates it's a pipe! That's markedly different than the `-` that's at the beginning of normal files, such as `some_file` in the above example.

Unlike the automatic named pipes from process substitution:

- You control where FIFOs are created
- They persist until you delete them
- Any process can write to them by path (e.g., `echo hi > my_pipe`)
- You can see them with `ls` and examine them like files

One problem with FIFOs is that they'll "block" any operations on them until both the read side of the pipe and the write side of the pipe are ready. For example, consider this:

```console
hacker@dojo:~$ mkfifo myfifo
hacker@dojo:~$ echo pwn > myfifo
```

To service `echo pwn > myfifo`, bash will open the `myfifo` file in write mode. However, this operation will hang until something _also_ opens the file in read mode (thus completing the pipe). That can be in a different console:

```console
hacker@dojo:~$ cat myfifo
pwn
hacker@dojo:~$
```

What happened here? When we ran `cat myfifo`, the pipe had both sides of the connection all set, and _unblocked_, allowing `echo pwn > myfifo` to run, which sent `pwn` into the pipe, where it was read by `cat`.

Of course, this can somewhat be done by normal files: you've learned how to `echo` stuff into them and `cat` them out. Why use a FIFO instead? Here are key differences:

1. **No disk storage:** FIFOs pass data directly between processes in memory - nothing is saved to disk
2. **Ephemeral data:** Once data is read from a FIFO, it's gone (unlike files where data persists)
3. **Automatic synchronization:** Writers block until the readers are ready, and vice-versa. This is actually useful! It provides automatic synchronization. Consider the example above: with a FIFO, it doesn't matter if `cat myfifo` or `echo pwn > myfifo` is executed first; each would just wait for the other. With files, you need to make sure to execute the writer before the reader.
4. **Complex data flows:** FIFOs are useful for facilitating complex data flows, merging and splitting data in flexible ways, and so on. For example, FIFOs support multiple readers and writers.
# Shell Variables

