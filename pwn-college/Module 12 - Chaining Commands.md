Piping is one way to chain commands, another way to do the same is using semicolons to chain commands. In most contexts, `;` separates commands in a similar way to how Enter separates lines.

One can also use logic operators on commands, for example && - AND means a second command only runs if the first command succeeds, similarly there exists the || operator means that the second command only runs if the first command fails, or if the first command doesn't fail, the second command won't run, similar to OR. 

However in reality, chaining multiple commands on a single line in a terminal is extremely inefficient, in such cases one writes shell scripts which contains these commands, thus when one executes the script, those commands are executed. By convention shell scripts are named with a .sh suffix, we execute it by passing it as an argument to an instance of our shell, i.e. using the bash command. 

Similarly one can output the contents of a shell script as if it were a singular command, so all of the various redirection methods work, i.e. `>` for stdout, `2>` for stderr, `<` for stdin, `>>` and `2>>` for append-mode redirection, `>&` for redirecting to other file descriptors, and `|` for piping to another command.

One can also directly execute shell scripts by naming the Absolute or Relative paths to the shell script, for this we have to give the execute permission to the shell script. 

A property of Linux is that it doesn't rely on file extensions to know how a file should be executed, instead it looks at the first few bytes to determine the type f file, so we can write our shell scrips with any other extension apart from sh, and it will still run with the bash command.  If a program starts with #! (often termed as a shebang). Linux treats the file as an _interpreted program_, and the contents of the rest of the line as the path to the _interpreter_. It then invokes the interpreter with the path to the program file as its only argument.

Basically the beginning of the file must have this shebang, as then Linux extracts the interpreter and then executes the shell script appropriately. Some common shebangs are 

- `#!/bin/bash` for bash scripts
- `#!/usr/bin/python3` for Python scripts
- `#!/bin/sh` for POSIX shell scripts --- this is a more primitive predecessor to `bash` with fewer features, but more compatibility to non-Linux systems!

You can also make shell scripts take in arguments, the script can access these arguments using $1 for the first, $2 for the second and so on

You can also use conditionals in your script, these conditionals are done using if statements:

```bash
if [ "$1" == "ping" ]
then
    echo "pong"
fi
```

SO if the first argument is ping, echo pong. You always need to have spaces in front of if, and it should always use square brackets and also if, then and fi must always all be on different lines.  Similar to normal if conditionals, we can also include an else block. 

```bash
if [ "$1" == "hello" ]
then
    echo "Hi there!"
else
    echo "I don't understand"
fi
```

Additionally in our conditionals we can also include the elif, i.e. else if block aswell:

```bash
if [ "$1" == "one" ]
then
    echo "1"
elif [ "$1" == "two" ]
then
    echo "2"
elif [ "$1" == "three" ]
then
    echo "3"
else
    echo "unknown"
fi
```

Unlike many other languages, bash requires the `[` and the `]` to be separated from other characters by spaces, otherwise it cannot parse the condition.