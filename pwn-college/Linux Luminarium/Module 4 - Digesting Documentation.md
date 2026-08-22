Nothing much to know in this module apart from the usage of the 'man' command which is short for manual. Doing man "any command in Linux" will pull up the Manual page for that command

For example, let's say we wanted to learn about the `yes` command (_yes_, this is a real command):

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

You can scroll around the manpage with your arrow keys and PgUp/PgDn. When you're done reading the manpage, you can hit `q` to quit. MAN pages are stored in a centralized database which lives in the /usr/share/man directory

You can search man pages with `/`. After searching, you can hit `n` to go to the next result and `N` to go to the previous result. Instead of `/`, you can use `?` to search backwards!

Some programs don't have a man page, but might tell you how to run them if invoked with a special argument. Usually, this argument is `--help`, but it can often be `-h` or, in rare cases, `-?`, `help`, or other esoteric values like `/?` (though that latter is more frequently encountered on Windows).

Some commands, rather than being programs with man pages and help options, are built into the shell itself. These are called _builtins_. Builtins are invoked just like commands, but the shell handles them internally instead of launching other programs. You can get a list of shell builtins by running the _builtin_ `help`, as so:

```console
hacker@dojo:~$ help
```

