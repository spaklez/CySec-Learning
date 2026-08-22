 Globbing lets you reference files without typing them all out, or typing out their full paths. 

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

The glob can just as simply match only one as well. When zero files are matched, by default, the shell leaves the glob unchanged. Remember that the shell isn't randomly guessing letters or going on forever. It is literally looking at the actual files and folders that exist on your hard drive.

Another type of Globbing is about  `?`. When it encounters a `?` character in any argument, the shell will treat it as a **single-character** wildcard. This works like `*`, but only matches _one_ character. For example:

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

Bash also supports the expansion of multiple globs in a single word. For example:

```console
hacker@dojo:~$ cat /*fl*
pwn.college{YEAH}
hacker@dojo:~$
```

What happens above is that the shell looks for all files in `/` that start with _anything_ (including nothing), then have an `f` and an `l`, and end in _anything_ (including `ag`, which makes `flag`).

You can also filter out files in a glob, If the first character in the brackets is a `!` or (in newer versions of bash) a `^`, the glob inverts, and that bracket instance matches characters that _aren't_ listed.

