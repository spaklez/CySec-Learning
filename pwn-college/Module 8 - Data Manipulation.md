Some additional important commands to know for Data Manipulation and in general are

# 1) awk

In general, for awk, it's structure is

```awk
BEGIN { ... }        # runs once, before any input is read
pattern { action }   # runs once per input line, if pattern matches
pattern { action }   # ...as many of these as you like, tested in order
END { ... }          # runs once, after all input is exhausted
```

Two rules for this structure are:
- **Pattern with no action** → defaults to `{ print }`, i.e. print the whole line. So `awk '/failed/'` behaves like grep.
- **Action with no pattern** → runs on every line. So `awk '{print $1}'` prints field 1 of everything.

`END` is where you print totals, counts, and summaries are after seeing all the data.

awk reads input one **record** at a time (a line, by default) and splits it into **fields**.

| Command       | Meaning                                              |
| ------------- | ---------------------------------------------------- |
| `$0`          | the entire record                                    |
| `$1`, `$2`, … | field 1, field 2, …                                  |
| `NF`          | Number of Fields on this line                        |
| `$NF`         | the **last** field                                   |
| `$(NF-1)`     | second-to-last                                       |
| `NR`          | Number of Records seen so far (running line counter) |
| `FS`          | input Field Separator (default: whitespace)          |
| `OFS`         | Output Field Separator (default: space)              |

The main three flags of awk are: 

```bash
awk -F: '{print $1}' /etc/passwd     # -F sets the field separator
awk -v threshold=100 '$5 > threshold' file   # -v passes a shell value in
awk -f script.awk data.log           # -f reads the program from a file
```

awk also makes use of regex, a quick refresher on regex commands are 

```awk
/regex/                    # line matches regex
$3 == "root"               # relational test on a field
$5 > 1000                  # numeric comparison
$1 ~ /^10\./               # field matches regex  (!~ for "doesn't match")
NR > 1                     # skip a header line
/start/,/stop/             # range: from first match until next match, inclusive
$3 == "root" && $5 > 100   # combine with && || !
```

Remember that `~` is match, `!~` is negated match. 

the awk command in itself can be considered as a complete programming language, i.e. it has it's own functions, some of the important functions are:


```awk
length($0)              # string length (or array size, in gawk)
substr($0, 5, 10)       # 10 chars starting at position 5
index(s, t)             # position of t within s, 0 if absent
split($0, arr, ":")     # split into array, returns field count
sub(/re/, "new")        # replace first match in $0
gsub(/re/, "new")       # replace all matches, returns count
tolower($0)             # case normalisation 
match(s, /re/)          # sets RSTART and RLENGTH
```

# 2) sed

sed processed input one line at a time, each line goes through the same loop of, i.e., 

- Read the next line into the **pattern space** (a buffer holding the current line).
- Run _every_ command in your script against that pattern space, in order.
- **Auto-print** the pattern space.
- Clear it, repeat

Most sed usage follows the template of s/oldworld/newword/flags, or more "templaty wise", it follows the template s/regexp/replacement/flags. The flags for sed are 

| Flag     | Effect                                                     |
| -------- | ---------------------------------------------------------- |
| _(none)_ | replace first match **on each line**                       |
| `g`      | replace all matches on each line                           |
| `3`      | replace only the 3rd match                                 |
| `3g`     | replace the 3rd match onward                               |
| `I`      | case-insensitive match                                     |
| `p`      | print the line if a substitution happened (pair with `-n`) |
| `w file` | write changed lines to a file                              |
As we know, to regex something literally, such as a slash, one does \ / , but doing this repeatedly for longer file paths can be unreadable. One can use any character after s to act a delimiter, for example 

```bash
sed 's/\/usr\/local\/bin/\/opt\/bin/'    # unreadable
sed 's|/usr/local/bin|/opt/bin|'         # same thing, legible
```

sed by default uses  **Basic** Regular Expressions. In BRE, these are literal characters and must be backslash-escaped to act as metacharacters: `+` `?` `|` `(` `)` `{` `}`.

```bash
sed 's/\(foo\)\+/bar/'      # BRE — backslashes required
sed -E 's/(foo)+/bar/'      # ERE — normal, readable
```

`-E`  switches to Extended REs, which behave the way you expect from awk, grep -E, and Python. **Use `-E` by default.**  

We can also make use of addresses similar to how we did it in awk, a list of exxamples are 


```bash
sed -n '5p'              # line 5
sed -n '$p'              # last line
sed -n '5,10p'           # lines 5 through 10
sed -n '5,$p'            # line 5 to end
sed -n '/error/p'        # lines matching a regex
sed -n '/start/,/stop/p' # from first match to next match, inclusive
sed -n '0~3p'            # every 3rd line
sed -n '/error/,+5p'     # a match and the 5 lines after it 
sed '/^#/d'              # delete comment lines
sed '/^$/d'              # delete blank lines
sed -n '/error/!p'       # ! negates
```

# 3) cut 

cut selects a fixed positional slice from every line, you can pick exactly one of three modes 

| Mode | Slices by                    |
| ---- | ---------------------------- |
| `-f` | fields, split on a delimiter |
| `-c` | character positions          |
| `-b` | byte positions               |

Some usage examples of the cut command are 

```bash
cut -f1        # field 1
cut -f1,3,5    # fields 1, 3 and 5
cut -f2-4      # fields 2 through 4
cut -f3-       # field 3 to end of line
cut -f-3       # start through field 3
cut -f1,4-6,9  # mix and match
```

Four things to remember with the command are 

1. Default delimiter is TAB, not space.
2. The delimiter is a single character.
3. Output order is fixed,
4. Runs of whitespace are NOT collapsed.

# 4) less 

less displays a file, one screen at a time, all the while letting you move around aswell. 

- It **doesn't read the whole file** before displaying.
- It's read-only

The movement related to the less command is 

| Key                 | Does                               |
| ------------------- | ---------------------------------- |
| `SPACE` / `f`       | forward one screen                 |
| `b`                 | back one screen                    |
| `d` / `u`           | forward / back half a screen       |
| `j` / `k` or arrows | one line at a time                 |
| `g`                 | jump to start (or `Ng` for line N) |
| `G`                 | jump to end                        |
| `50p`               | jump to 50% through the file       |
| `q`                 | quit                               |
| `h`                 | help                               |
You can also search in less, using the following commands 

| Key        | Does                        |
| ---------- | --------------------------- |
| `/pattern` | search forward (regex)      |
| `?pattern` | search backward             |
| `n` / `N`  | next / previous match       |
| `ESC-u`    | turn off match highlighting |

Prefixes typed at the **start** of a search pattern change the search's behavior:

- `/!pattern` — find lines **not** matching
- `/^Rpattern` — literal text, no regex interpretation (use when your pattern contains `.` `*` `[`)
- `/^Kpattern` — highlight matches but **don't move** 
- `/&pattern` — hides every line that doesn't match, turning `less` into a live, reversible `grep`

Some additional flags worth knowing for the less command are 

```bash
less -S file        # chop long lines instead of wrapping — essential for wide logs
less -N file        # show line numbers
less -i file        # case-insensitive search (unless pattern has uppercase)
less -R file        # render ANSI colour codes properly
less -X file        # don't clear screen on exit — output stays visible
less -F file        # quit immediately if it fits on one screen
less +G file        # open at the end
less +F file        # open in follow mode
less +/error file   # open at the first match of "error"
less -p error file  # same thing
```

# 5) more 

`more` is the ancestor: forward-only, loads more of the file eagerly, no backward search, and it exits at EOF. `SPACE` pages, `q` quits, `/pattern` searches forward, and that's about it.
# 6) paste 

As the name suggests, this command is used to combine files. The flag -d is used to set the delimiter, an example is 

```bash
paste -d: a.txt b.txt              # single delimiter
paste -d'\n' a.txt b.txt           # newline — interleaves the two files
paste -d',' -s file                # comma
```

Another flag is -s, the transpose flag, Without `-s`, files are pasted in parallel (side by side). With `-s`, each file is collapsed **serially**.

# 7) sort 

As the name suggests, used for sorting data. Some practical flags for the sort command are 

```bash
-o file          # write output to file — SAFE for in-place: sort -o f f works
-c               # check if sorted, don't sort — exit status tells you
-m               # merge already-sorted files without re-sorting
-S 2G            # memory buffer size
-T /path         # temp directory for large sorts
-z               # NUL-terminated lines, for filenames with newlines
--parallel=N     # threads (GNU)
```

Some additional 'sorting' flags are 

| Flag | Sorts by                                               |
| ---- | ------------------------------------------------------ |
| `-n` | numeric value (`9` before `10`)                        |
| `-h` | human-readable sizes (`2K` < `1G`)                     |
| `-V` | version numbers (`1.9` before `1.10`)                  |
| `-M` | month names (JAN … DEC)                                |
| `-g` | general numeric — handles `1.5e3`, scientific notation |
| `-r` | reverse                                                |
| `-f` | fold case                                              |
| `-b` | ignore leading blanks                                  |
| `-R` | random                                                 |

# 8) tail

Used to display the last part of a file, example: 

```bash
tail file              # last 10 lines (default)
tail -n 50 file        # last 50 lines
tail -c 500 file       # last 500 bytes
head -n 50 file        # the inverse — first 50
```

a leading `+` flips the meaning from "last K" to "**starting at** K, counting from the start."

```bash
tail -n +2 file        # everything from line 2 onward
tail -n +1000 file     # from line 1000 to the end
tail -c +100 file      # from byte 100 onward
```

Some other flags used for this command is 


```bash
-q              # suppress the ==> filename <== headers with multiple files
-v              # always show them
--pid=PID       # stop following when process PID exits
-s N            # sleep N seconds between checks (default 1.0)
```

# 9) head 

Opposite of tails, displays from beginning of the file, example: 

```bash
head file              # first 10 lines (default)
head -n 50 file        # first 50
head -n -5 file        # everything EXCEPT the last 5 lines
head -c 100 file       # first 100 bytes
head -q / -v           # suppress / force ==> filename <== headers
```

# 10) uniq 

Used to filter our unique data from a stream. In general uniq compares each line to the one before it.  It has no memory beyond that. Given `a b a b`, it outputs `a b a b` , **`uniq` is nearly always wrong without `sort` in front of it.** That's why the idiom is `sort | uniq -c`, not `uniq -c`.  Some common flags for the uniq command are 

| Flag     | Prints                                                      |
| -------- | ----------------------------------------------------------- |
| _(none)_ | first line of each run, duplicates collapsed                |
| `-c`     | each line prefixed by its count                             |
| `-d`     | **only** lines that appeared more than once (one per group) |
| `-u`     | **only** lines that appeared exactly once                   |
| `-D`     | every copy of every duplicated line                         |
| `-i`     | ignore case when comparing                                  |

That makes up the commands that were prefixed in this module as something to go through, now onwards to the actual module contents 

---

This whole module deals with the concept of Data Manipulation, one method to modify data is to make use of the tr command, which stands for translate. tr translates the character provided in the first argument to the character provided in the second argument. An example is 

```console
hacker@dojo:~$ echo OWN | tr O P
PWN
hacker@dojo:~$
```

It can also handle multiple characters, with the characters in different positions of the first argument replaced with associated characters in the second argument.

```console
hacker@dojo:~$ echo PWM.COLLAGE | tr MA NE
PWN.COLLEGE
hacker@dojo:~$
```

`tr` can also translate characters to nothing (i.e., _delete_ them). This is done via a `-d` flag and an argument of what characters to delete:

```console
hacker@dojo:~$ echo PAWN | tr -d A
PWN
hacker@dojo:~$
```

`tr` can also translate characters to nothing (i.e., _delete_ them). This is done via a `-d` flag and an argument of what characters to delete:

```console
hacker@dojo:~$ echo PAWN | tr -d A
PWN
hacker@dojo:~$
```

You can also remove new lines/line separators by escaping appropriately, an example is 

```console
hacker@dojo:~$ echo "hello_world!" | tr _ "\n"
hello
world!
hacker@dojo:~$
```

As discussed earlier, head command is used to display the first few lines of any files input, by default it shows the first 10 lines but as seen earlier , one can control this using the -n flag. An example is 

```console
hacker@dojo:~$ cat /something/very/long | head -n 2
this
is
hacker@dojo:~$
```

As mentioned earlier, to grab specific columns of data, such as the first column, third column and so on, for this there exists the cut command. For example imagine the following file,

```console
hacker@dojo:~$ cat scores.txt
hacker 78 99 67
root 92 43 89
hacker@dojo:~$
```

You could use `cut` to extract specific columns:

```console
hacker@dojo:~$ cut -d " " -f 1 scores.txt
hacker
root
hacker@dojo:~$ cut -d " " -f 2 scores.txt
78
92
hacker@dojo:~$ cut -d " " -f 3 scores.txt
99
43
hacker@dojo:~$
```

The `-d` argument specifies the column _delimiter_ (how columns are separated). In this case, it's a space character. Of course, it has to be in quotes here so that the shell knows that the space is an argument rather than a space separating other arguments! The `-f` argument specifies the _field_ number (which column to extract).

The sort command, as mentioned earlier, helps in sorting data. It reads line from the input and outputs them in a sorted order. For example:

```console
hacker@dojo:~$ cat names.txt
  hack
  the
  planet
  with
  pwn
  college
hacker@dojo:~$ sort names.txt
  college
  hack
  planet
  pwn
  the
  with
hacker@dojo:~$
```

By default, `sort` orders lines alphabetically. Arguments can change this:
- `-r`: reverse order (Z to A)
- `-n`: numeric sort (for numbers)
- `-u`: unique lines only (remove duplicates)
- `-R`: random order!


