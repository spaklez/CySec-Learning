In Linux, files have different permissions and file modes, one can check this using ls -l. 

```console
hacker@dojo:~$ mkdir pwn_directory
hacker@dojo:~$ touch college_file
hacker@dojo:~$ ls -l
total 4
-rw-r--r-- 1 hacker hacker    0 May 22 13:42 college_file
drwxr-xr-x 2 hacker hacker 4096 May 22 13:42 pwn_directory
hacker@dojo:~$
```

The nine characters are the actual access permissions of the file or directory, split into 3 characters denoting the permissions that the user who owns the file (termed the "owner") has to the file, 3 characters denoting the permissions that the group that owns the file (termed the "group") has to the file, and 3 characters denoting the permissions that all other access (e.g., by other users and other groups) has to the file. 

There are two columns showing the _user_ that owns the file (in this case, user `hacker`) and then the _group_ that owns the file (in this case, also group `hacker`). You'll mess around with that here!

In general, one should also know the difference between Authentication and Authorization

-) Authorization, what can you do on the system? What are you authorized to do on the system? 
-) Authentication, how do we know you are who you say you are? Simply put, Who are you? 

When thinking about a system at a high level, you need to think and manage about 

1) Authorization
2) Trust

And the main point of this is to manage risk, a general pointer to know is that you can never completely eliminate risk, but rather mitigate it, i.e. manage and reduce it as much as possible. 

Simply put, Authorization is the policy whereas the Access Control is the actual mechanism with which we implement the authorization rights. 

Another important thing is that in a system, we should be able to model our Access Control system, i.e.

- A set of subjects S, i.e. things in the system that can act. For example a process
- A set of objects O, i.e. objects/assets that are acted upon in the system by the subjects 
- A set of rights R, i.e. what can the subjects do to the objects 

A way to model this is with an Access Matrix Model. In an Access Matrix Model, the columns are the objects and the rows are the subjects, and their intersections show the rights that the objects have on the subject. 

Basically, the whole point of Access Matrix Models are to see how it chances depending on the actions taken by each object on a subject, i.e. can the system ever get into a particular state, this is why it's important to do such modelling.  

Think of it this way, you can't use a particular command to read a file, but you can use another command that can read the file! 

In UNIX systems, subjects are processes and files are objects, and the rights that the subjects have are 

-) read 
-) write 
-) execute 
-) append 
-) own 

However in reality, managing such Access Control Models is inefficient, as you would have thousands of subjects and objects that need to be tracked, i.e. their respective rights.

Now how do you actually implement an Access Control mechanism into a system, one implementation is 
### Access Control: ACLs, Capabilities, Relations

### The Access Matrix

|                 | `/etc/passwd` | `report.docx` | `backup.sh`   |
| --------------- | ------------- | ------------- | ------------- |
| **Alice**       | read          | read, write   | read, execute |
| **Bob**         | read          | read          | —             |
| **Backup proc** | read          | read          | read, execute |

### Access Control Lists, ACLs

Each column of the matrix is kept with the object itself, i.e. a simple list where for an object f (file f) each process's rights will be in that lists, if a process doesn't exist in that list, that process doesn't have any rights over the file/object. This is what most modern systems use, for example UNIX.

> [!example] ACL for `report.docx` (a column)
> | Subject     | Rights      |
> | ----------- | ----------- |
> | Alice       | read, write |
> | Bob         | read        |
> | Backup proc | read        |

### Capability Lists

Basically each row of the access matrix is stored with the subject, so a process has the rights over a particular file/object. The operating system stores a mapping of this, because each list cannot be stored with it's particular process, as then the process could change it.

> [!example] Capability list for Alice (a row)
> | Object          | Rights        |
> | --------------- | ------------- |
> | `/etc/passwd`   | read          |
> | `report.docx`   | read, write   |
> | `backup.sh`     | read, execute |

> [!example] Capability list for Bob
> | Object          | Rights |
> | --------------- | ------ |
> | `/etc/passwd`   | read   |
> | `report.docx`   | read   |

### Relation

Another option is forming relations, i.e. a 3 tuple relation, where we say, SUBJECT p has ACCESS r over OBJECT f, just forming a simple table.

| Subject     | Access        | Object        |
| ----------- | ------------- | ------------- |
| Alice       | read          | `/etc/passwd` |
| Alice       | read, write   | `report.docx` |
| Alice       | read, execute | `backup.sh`   |
| Bob         | read          | `/etc/passwd` |
| Bob         | read          | `report.docx` |
| Backup proc | read          | `report.docx` |
| Backup proc | read, execute | `backup.sh`   |

### ACL vs Capability

Because in ACL, we store the access with the object, we need very strong authentication of subjects, for Capability lists on the other hand, it doesn't require authentication of subjects, but it means that capabilities have to be unforgeable and propagation must be controlled. Another way to think about this is through the notion of least privilige, i.e. you should only have the capabilities/access, that you only need to do your job and no more.

CAP provides this concept of least privilige control with respects to subjects, Another point of thinking is about access review, i.e. being able to see who has access to what. ACL can provide for better access review than Capability lists. So in the case of ACL, if we want to know if a process p can access a specific file, we just look at the ACL of that file and see if process p has that particular access. In the case of Capability, you would have to go through all the Capabilites of all the subjects on the system, and seeing if anyone has that particular right.

> [!summary] Comparison
> |                       | ACL                              | Capability                                       |
> | --------------------- | -------------------------------- | ------------------------------------------------ |
> | Stored with           | object                           | subject (OS-held)                                |
> | Needs                 | strong authentication of subjects| unforgeable capabilities, controlled propagation  |
> | Least privilege       | weaker                           | stronger                                         |
> | Access review         | better                           | worse (scan all subjects)                        |
> | Revocation by object  | better                           | worse                                            |
> | Revocation by subject | worse                            | better                                           |

### Revocation

Another important thing is Revocation, i.e. being able to remove access, ACL is better for revocation on object basis whereas CAP is better for revocation on subject basis

In a POSIX system, it makes use of ACL, POSIX comes from UNIX-Like systems, each file (object) has 12 permission bits logically grouped into 4 sets of 3 bits each. 

-) First 3 bits represent the 
- SUID bit - Set UID bit
- SGID bit - Set Group ID bit
- Sticky-bit - Set Sticky bit 

-) Next 3 bit sets apply to file's owner, user in file's group and all users respectively, i.e. whether they can read, write or execute. 

Thus every file in Linux is owned by a user on the system, 

To change the owner of files, we can use the chown command, which stands for change owner, 

```
chown [username] [file]
```

Typically chown can only be invoked by the root user. Files in Linux have both an owning user and group, a group can accordingly have multiple users and a user can be part of multiple groups. 

Using the id command, users can check which groups they're part of, the most common use-case for groups is to control access to different system resources. By making a user be part of a particular group, the user gains access to run commands that couldn't be run otherwise, this is done using the chgrp command.

As we saw earlier, the access permissions of a file or directory are split into 3 groups of 3, highlighting the read, write and execute of the Owner, Group and All users. Each character of the three represent permission for a different type:

```
r - user/group/other can read the file (or list the directory)
w - user/group/other can modify the files (or create/delete files in the directory)
x - user/group/other can execute the file as a program (or can enter the directory, e.g., using `cd`)
- - nothing 
```

To alter these permissions, one uses the chmod (change mode) command, The basic usage for chmod is:

```
chmod [OPTIONS] MODE FILE
```

You can specify the `MODE` in two ways: as a modification of the existing permissions mode, or as a completely new mode to overwrite the old one.

### Modifying an existing mode 

It involves the usage of the command in the form of of `WHO`+/-`WHAT`, where `WHO` is user/group/other and `WHAT` is read/write/execute. For example, to add _read_ access for the owning _user_, you would specify a mode of `u+r`. `w`rite and e`x`ecute access for the `g`roup and the `o`ther (or `a`ll the modes) are specified the same way. More examples:

- `u+r`, as above, adds read access to the user's permissions
- `g+wx` adds write and execute access to the group's permissions
- `o-w` _removes_ write access for other users
- `a-rwx` removes all permissions for the user, group, and world

### Overwriting old permissions 

One can set permissions altogether using the = command instead of the +/-. For example:

- `u=rw` sets read and write permissions for the user, and wipes the execute permission
- `o=x` sets only executable permissions for the world, wiping read and write
- `a=rwx` sets read, write, and executable permissions for the user, group, and world!

You can chain multiple modes to chmod using `,` : 

- `chmod u=rw,g=r /challenge/pwn` will set the user permissions to read and write, and the group permissions to read-only
- `chmod a=r,u=rw /challenge/pwn` will set the user permissions to read and write, and the group and world permissions to read-only

Additionally, you can zero out permissions with `-`:

- `chmod u=rw,g=r,o=- /challenge/pwn` will set the user permissions to read and write, the group permissions to read-only, and the world permissions to nothing at all

The permissions of a file with SUID look like this:

```console
hacker@dojo:~$ ls -l /usr/bin/sudo
-rwsr-xr-x 1 root root 232416 Dec 1 11:45 /usr/bin/sudo
hacker@dojo:~$
```

The `s` part in place of the executable bit means that the program is executable _with SUID_. It means that, regardless of what user runs the program (as long as they have executable permissions), the program will execute as the owner user (in this case, the `root` user).

As the owner of a file, you can set a file's SUID bit by using chmod:

```
chmod u+s [program]
```



