The full list of users on a Linux system are specified in the /etc/passwd file. 

 Each line contains, separated by `:`s, the username, an `x` as a placeholder for where the password used to be, the numerical user ID, the numerical default group ID, long-form user details, the home directory, and the default shell.
 
 There exists two utilities to gain root access, su and suid, 
-) su - i.e. substitute user, it's a suid binary, i.e. it has the suid bit set, thus it runs as root. Because it can run as the root user, it can start a root shell. But it checks to make sure that the user knows the `root` password.

```console
hacker@dojo:~$ su
Password: 
su: Authentication failure
hacker@dojo:~$
```

This check against the `root` password is what obsoletes `su`. Modern systems very rarely have `root` passwords.

With no arguments, su runs a root shell, however giving a username as an argument causes su to switch to that user as well. 

Passwords in linux are stored in /etc/shadow, separated by `:`s, the first field of each line is the username and the second is the password. A value of `*` or `!` functionally means that password login for the account is disabled, a blank field means that there is no password (a not-uncommon misconfiguration that allows password-less `su` in some configurations).

When you input a password into `su`, it one-way-encrypts (hashes) it and compares the result against the stored value. If the result matches, `su` grants you access to the user!

But what if you don't know the password? If you have the hashed value of the password, you can _crack_ it!

-) sudo - substitute user, do. Unlike `su`, which defaults to launching a shell as a specified user, `sudo` defaults to running a command as `root`. Unlike `su`, which relies on password authentication, `sudo` checks policies to determine whether the user is authorized to run commands as `root`. These policies are defined in `/etc/sudoers`. 



