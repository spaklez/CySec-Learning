
## Disassembly with `objdump`

The opposite of what we've been doing so far is known as disassembly, i.e. the process of converting the binary machine code in an executable *back* into human-readable assembly instructions.

One of the most common tools for disassembly: `objdump`. Given a binary, `objdump -d` will disassemble the executable sections and show you the assembly instructions. By default, `objdump` uses AT&T syntax rather than the Intel syntax we've been writing in, which is why we pass the `-M intel` option.

```console
hacker@dojo:~$ objdump -d -M intel /tmp/your-program

/tmp/your-program:     file format elf64-x86-64


Disassembly of section .text:

0000000000401000 <_start>:
  401000:	48 c7 c7 39 05 00 00 	mov    rdi,0x539
  401007:	48 c7 c7 00 00 00 00 	mov    rdi,0
  40100e:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  401015:	0f 05                	syscall
```

Reading the columns left to right: the address the instruction lives at, the raw bytes of the instruction, and then the disassembled instruction itself.

Second, `objdump` displays the raw bytes of each instruction (e.g., the *hexadecimal* values `0f 05` is the `syscall` instruction). Third, the values that are being moved into registers are *also* represented as hexadecimal. Remember that the Intel Architecture stores this Hex output as Little Endian, the steps it follows are,

- The human-readable number is `0x539`.

- The immediate field of this instruction is 4 bytes (32 bits) wide, so it is padded with zeros: `0x00000539`. (The destination `rdi` is still a 64-bit register — `mov r64, imm32` encodes a 32-bit immediate and sign-extends it into the full register. The register is 8 bytes; the encoded constant is 4.)

- If we split that into individual bytes, we get: `00`, `00`, `05`, `39`.

- Little-Endian storage flips the order of those bytes when writing them to memory: `39`, `05`, `00`, `00`.

Which is exactly what you see sitting in the byte column: `48 c7 c7` is the opcode plus the REX.W prefix plus the ModR/M byte selecting `rdi`, and then `39 05 00 00` is the constant, backwards.

```console
ubuntu@introspecting~disassembling-programs:~$ objdump -d -M intel /challenge/disassemble-me

/challenge/disassemble-me:     file format elf64-x86-64


Disassembly of section .text:

0000000000401000 <__bss_start-0x1000>:
  401000:       48 c7 c7 2c 51 00 00    mov    rdi,0x512c
  401007:       48 c7 c7 00 00 00 00    mov    rdi,0x0
  40100e:       48 c7 c0 3c 00 00 00    mov    rax,0x3c
  401015:       0f 05                   syscall
ubuntu@introspecting~disassembling-programs:~$ /challenge/submit-number 0x512c
CORRECT! Here is your flag:
```

Note the byte column again: `2c 51 00 00` on disk, `0x512c` when read as a little-endian 32-bit value. Same thing written two ways.

Also note the label  `<__bss_start-0x1000>` instead of `<_start>`. That is `objdump` telling you the binary has no symbol for that address, so the best it can do is describe the location relative to a symbol it *does* know about. Stripped binaries look like this everywhere.

---

## Tracing syscalls with `strace`

There also exists a command to trace syscalls, known as `strace`, an example usage of it is as below:

```console
hacker@dojo:~$ strace /tmp/your-program
execve("/tmp/your-program", ["/tmp/your-program"], 0x7ffd48ae28b0 /* 53 vars */) = 0
exit(42)                                 = ?
+++ exited with 42 +++
hacker@dojo:~$
```

The syntax used here for output is `system_call(parameter, parameter, parameter, ...) = return_value`.

In this example, `strace` reports two system calls: the second is the `exit` system call that your program uses to request its own termination, and you can see the parameter you passed to it (42). The first is an `execve` system call.

### What the `execve` line actually is

`execve` is the syscall that loads a program and starts running it. It is not something my program called, it is the call that *created* my program's running state. When you type `strace /tmp/your-program`, `strace` starts first, forks a child, tells the kernel it wants to trace that child, and then the child calls `execve` to replace its own memory image with the binary I asked for. So the first line of every `strace` output is `execve`, because that is the moment the process stopped being a copy of `strace` and started being my program.

Its three parameters, in order:

- `"/tmp/your-program"` — the path to the binary being loaded.
- `["/tmp/your-program"]` — the `argv` array. Here it only has one element, `argv[0]`, which is the program name. If I had passed arguments they'd show up as more elements in that list.
- `0x7ffd48ae28b0 /* 53 vars */` — the `envp` array, i.e. the environment. `strace` doesn't print all 53 environment variables, it just prints the pointer and tells you in a comment how many there were.

The `= 0` at the end is the return value. On success `execve` doesn't actually return anywhere, because the code that called it no longer exists — it got overwritten by the new program. `strace` prints `= 0` to mean "this succeeded". The `exit` call shows `= ?` for the opposite reason: the process is gone, so there is nothing left to return into. The `+++ exited with 42 +++` line is `strace` reporting the process's final status, not a syscall.

### Reading the parameters and return values in general

Every traced line has the same shape. Here is a busier example, a program reading a file:

```console
hacker@dojo:~$ strace ./read-a-file
execve("./read-a-file", ["./read-a-file"], 0x7ffc9f2b1e30 /* 53 vars */) = 0
openat(AT_FDCWD, "/etc/hostname", O_RDONLY) = 3
read(3, "dojo\n", 4096)                  = 5
write(1, "dojo\n", 5)                    = 5
close(3)                                 = 0
exit_group(0)                            = ?
+++ exited with 0 +++
```

- `openat(AT_FDCWD, "/etc/hostname", O_RDONLY) = 3` — three parameters going in (relative to the current directory, this path, read-only) and a file descriptor `3` coming back. 0, 1 and 2 are already taken by stdin, stdout and stderr, so the first file a program opens is almost always 3.
- `read(3, "dojo\n", 4096) = 5` — read from fd 3, into a buffer, at most 4096 bytes. The return value 5 is how many bytes actually came back. `strace` is showing the buffer contents *after* the call, which is why the string is already filled in even though it's an output parameter.
- `write(1, "dojo\n", 5) = 5` — fd 1 is stdout. Five bytes in, five bytes written.
- Failures show up as a negative return with the errno name attached, e.g. `openat(AT_FDCWD, "/nope", O_RDONLY) = -1 ENOENT (No such file or directory)`. That is how you spot a program failing without it printing anything.

The point is that `strace` shows you the boundary between the program and the kernel. Anything the program does purely in registers and its own memory is invisible here; anything it wants the OS to do for it shows up as a line.


```console
ubuntu@introspecting~tracing-syscalls:~$ /challenge/trace-me
ubuntu@introspecting~tracing-syscalls:~$ strace /challenge/trace-me
execve("/challenge/trace-me", ["/challenge/trace-me"], 0x7ffca9728e70 /* 16 vars */) = 0
alarm(23983)                            = 0
exit(0)                                 = ?
+++ exited with 0 +++
ubuntu@introspecting~tracing-syscalls:~$ /challenge/submit-number 23983
CORRECT! Here is your flag:
```

Running it normally prints nothing at all — the program produces no output. Under `strace` the `alarm` syscall and its parameter are right there. `alarm(n)` asks the kernel to deliver a `SIGALRM` after `n` seconds, and it returns the number of seconds left on any previously-set alarm, which is 0 here because there wasn't one. The program never lives long enough for the alarm to fire; the syscall was only ever there to be observed.

---

## GDB

GDB stands for the **G**NU **D**e**b**ugger, and it is typically used to hunt down and understand bugs. More specifically, a debugger is a tool that enables the close monitoring and introspection of another process. GDB is by far the most common debugger used on Linux, to launch it, you just use the command `gdb` specifying the program that you want to debug. To exit from GDB, you either use the `quit` command or just `q`.

GDB observes the behaviour of the program at runtime, thus to start the program in GDB, we use the `starti` command. `starti` **start**s the program at the very first **i**nstruction. Once the program is running, you can use other GDB commands to inspect its actual runtime state. You can also disassemble the program using the `disassemble` command, this is similar to the `objdump` command and dumps the assembly code. So when one launches GDB, you have to start the program, and then after that you have to disassemble it (if needed).

The difference between `objdump -d` and GDB's `disassemble` is that GDB is disassembling the code *as it exists in memory right now*, and it marks the current instruction pointer with `=>`. `objdump` reads the file on disk and has no notion of "current".


```console
ubuntu@introspecting~disassembling-in-gdb:~$ gdb /challenge/debug-me
...
Reading symbols from /challenge/debug-me...
(No debugging symbols found in /challenge/debug-me)

(gdb) starti

HACKER: You successfully started your program!
HACKER: Now, run the 'disassemble' command to view the assembly code.

0x0000000000401000 in main ()
(gdb) disassemble
Dump of assembler code for function main:
=> 0x0000000000401000 <+0>:     mov    rdi,0x7021
   0x0000000000401007 <+7>:     mov    rdi,0x0
   0x000000000040100e <+14>:    mov    rax,0x3c
   0x0000000000401015 <+21>:    syscall
End of assembler dump.
(gdb) q
ubuntu@introspecting~disassembling-in-gdb:~$ /challenge/submit-number 0x7021
```

The `<+0>`, `<+7>`, `<+14>` are byte offsets from the start of the function, and they line up with the instruction lengths: the two 7-byte `mov r64, imm32` instructions, then another 7, then `syscall` at +21.

---

## Stepping and reading registers

To execute a single instruction in GDB, use the `stepi` command (**step** one **i**nstruction, also abbreviated `si`). To read a register value during these steps, you use the `print` command along with the register name prepended with a `$`, for example `print $rdi`.

```console
(gdb) starti

Dump of assembler code for function main:
=> 0x0000000000401000 <+0>:     mov    rdi,CENSORED
   0x0000000000401007 <+7>:     mov    rdi,0x0
   0x000000000040100e <+14>:    mov    rax,0x3c
   0x0000000000401015 <+21>:    syscall
End of assembler dump.

0x0000000000401000 in main ()
(gdb) print $rdi
$1 = 0
(gdb) stepi

   0x0000000000401000 <+0>:     mov    rdi,CENSORED
=> 0x0000000000401007 <+7>:     mov    rdi,0x0
   ...
0x0000000000401007 in main ()
(gdb) print $rdi
$2 = 2197
(gdb) stepi
...
(gdb) print $rdi
$3 = 0
```

- Before any instruction has run, `rdi` is 0.
- After **one** `stepi`, the censored `mov` has executed and `rdi` holds 2197. This is the only moment the value exists.
- After a **second** `stepi`, `mov rdi, 0x0` has executed and the value is gone. GDB happily prints 0 and nothing warns you that you destroyed what you were looking for.

The `=>` marker points at the instruction that is *about to execute*, not the one that just ran. That's the thing to keep straight: when `=>` is on line `<+7>`, line `<+0>` has already happened.

`print` also outputs in decimal by default. 2197 in hex is `0x895`; if the challenge wants hex, `print/x $rdi`.

---

## Setting register values

Similar to reading the values of registers using the `print` command, we can set their values using the `set` command, more specifically `set $rdi = 42`. An example usage is:


```console
(gdb) starti

Dump of assembler code for function main:
=> 0x0000000000401000 <+0>:     mov    rdi,0x0
   0x0000000000401007 <+7>:     add    rdi,CENSORED
   0x000000000040100e <+14>:    mov    rdi,0x0
   0x0000000000401015 <+21>:    mov    rax,0x3c
   0x000000000040101c <+28>:    syscall
End of assembler dump.

0x0000000000401000 in main ()
(gdb) stepi
...
HACKER: The first instruction set rdi to 0.
HACKER: Use 'set $rdi = 1337' to change it, then use 'stepi' once more.
0x0000000000401007 in main ()
(gdb) set $rdi = 1337
(gdb) stepi
...
0x000000000040100e in main ()
(gdb) print $rdi
$1 = 9672
```

Note also that `set $rdi = 1337` and `set $foo = 1337` are the same command form. If you typo the register name you don't get an error, you just quietly create a convenience variable and the register stays untouched.

---
## pop

```console
(gdb) starti

Dump of assembler code for function main:
=> 0x0000000000401000 <+0>:     pop    rdi
   0x0000000000401001 <+1>:     mov    rdi,0x0
   0x0000000000401008 <+8>:     mov    rax,0x3c
   0x000000000040100f <+15>:    syscall
End of assembler dump.

0x0000000000401000 in main ()
(gdb) print $rid
$1 = void
(gdb) print $rdi
$2 = 0
(gdb) stepi

   0x0000000000401000 <+0>:     pop    rdi
=> 0x0000000000401001 <+1>:     mov    rdi,0x0
   ...
0x0000000000401001 in main ()
(gdb) print $rdi
$3 = 62
```

`pop rdi` does two things in one instruction: it reads the 8 bytes at `[rsp]` into `rdi`, then adds 8 to `rsp`. Nothing is erased from memory — the value is still sitting there — but the stack pointer has moved past it, so as far as the program is