Below is a non-exhaustive collection of seemingly innocent actions that, when executed on a real system, can lead to catastrophic data-loss, denial of service, or even unrecoverable system corruption.

### Fork Bombs

Whenever you start a program the Linux operating system creates a new process. If you create processes faster than the kernel can handle, the process table fills up and _everything_ grinds to a halt. This new process (e.g., of an `ls` invocation) is "forked" off of a parent process (e.g., a shell instance). Thus, the induced explosion of processes is called a "Fork Bomb".

### Disk Space Doomsday

Basically filling up the disk with useless files so that any more files can't be created, one of the simplest ways to do this is the `yes` command!

```
hacker@dojo:~$ yes | head
y
y
y
y
y
y
y
y
y
y
hacker@dojo:~$
```

The `yes` outputs `y` over and over forever. We pipe the output of this command into a file 

### rm -rf / 

The `-r` (recursive) flag removes directories and all files containing them. The `-f` (force) flag ignores any errors the `rm` command runs into or compulsions that it may have. Combined and aimed at `/`, the results are catastrophic: a full wipe of your system. 




