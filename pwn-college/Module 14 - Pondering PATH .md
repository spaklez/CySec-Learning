PATH  an environmental variable in Linux and other Unix-like operating systems that tells the shell which directories to search for executable files, i.e. when one writes ls, how does the shell know where it's file path is? Which is needed to execute the ls command. 

PATH (which is written with all upper case letters) should not be confused with the term path (lower case letters). The latter is a file's or directory's address on a filesystem (i.e., the hierarchy of directories and files that is used to organize information stored on a computer). A relative path is an address relative to the current directory (i.e., the directory in which a user is currently working). An absolute path (also called a full path) is an address relative to the root directory (i.e., the directory at the very top of the filesystem and which contains all other directories and files).

To view the value of the PATH variable, one does 

```console
env | grep PATH
```

or by doing 

```console 
echo $PATH
```

A good use case of PATH manipulation is that we can store the directories of frequently used scripts to call them by their bare name rather than always typing out the complete path. 

To find out which file path gets executed for a command, use the which command to find out the path. 


