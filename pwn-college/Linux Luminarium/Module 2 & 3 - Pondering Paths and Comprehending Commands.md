Basically in Linux, / is the root directory, one can think of it as the root of a tree, and all the other directories are child of their respective parents. 

You can invoke a program by providing its path on the command line, if this path starts with / , it's an absolute path, else it's a relative path. Examples are

-) **Absolute Path:** /usr/home/yans/flags/TOPSECRET
-) **Relative Path:** don't start with / , and are relative to the current working directory

Remember that relative paths are always interpreted from your CWD, i.e. your current working directory. So if you are in /home/hackers , typing, 

```console
cat flag.txt
```

will "cat" /home/hackers/flag to the terminal. One could also type out the Absolute Path too! Here are some more good examples from the pwn.college website

Imagine we want to access some file located at `/tmp/a/b/my_file`.

- If my `cwd` is `/`, then a relative path to the file is `tmp/a/b/my_file`.
- If my `cwd` is `/tmp`, then a relative path to the file is `a/b/my_file`.
- If my `cwd` is `/tmp/a/b/c`, then a relative path to the file is `../my_file`. The `..` refers to the parent directory.

Some important commands to know from this module are 

-) cd - change directory, pretty obvious. Also, cd .. -> go to parent directory, . -> refers to parent directory.
-) ls - list files in directories provided as arguments, and in current directory if no arguments provided, use the -a flag to view hidden files (i.e. files starting with .)
-) grep SEARCH_STRING /path/to/file - searches the file for lines of text containing `SEARCH_STRING` and print them to the console.
-) diff - compares two files line by line and shows you exactly what's different between them.
-) touch - creates a new blank file 
-) rm - removes files
-) mv - moving files
-) cp - mv removes the original file, cp thus copies the original file and keeping it, it's usage is `cp SOURCE DESTINATION`: the first argument is the existing file, and the second is where the copy should be created.
-) mkdir - making directories
-) find - used to find files, The `find` command takes optional arguments describing the search criteria and the search location. If you don't specify a search criteria, `find` matches every file. If you don't specify a search location, `find` uses the current working directory (`.`). We can specify the criteria by also filtering by name, i.e. using the -name flag 

There are many different types of files in Linux, using ls -l or ls -ld shows us the types of these files 
1) "-" is a regular file
2) d is a directory files
3) l is a symbolic link (a file that transparently points to another file or directory)
4) p is a named pipe (also known as a FIFO. You will get very familiar with these this module!)
5) c is a character device file (i.e. backed by a hardware device that produces or receives data streams, such as a microphone)
6) b is a block device file (i.e. backed by a hardware device that stores and loads blocks of data, such as a hard drive)
7) s is a unix socket (essentially a local network connection encapsulated in a file)

Links come in two flavors: _hard_ and _soft_ (also known as _symbolic_) links. We'll differentiate the two with an analogy:

- A **hard** link is when you address your apartment using multiple addresses that all lead directly to the same place (e.g., `Apt 2` vs `Unit 2`).
- A **soft** link is when you move apartments and have the postal service automatically forward your mail from your old place to your new place.

Symbolic links are created with the `ln` command using the syntax `ln -s TARGET LINK_NAME`. `TARGET` is the path that the link will point to, and `LINK_NAME` is the new path you are creating. If you want to make Hard links, you write down the same command but without the -s flag. 









