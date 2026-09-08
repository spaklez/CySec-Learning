## Reaching into the caller's frame

The stack stores scratch data and return addresses, along with the local variables of functions.

When you `call solve`, two things happen: the return address gets pushed onto the stack (8 bytes), and execution jumps to `solve`. The push is the important part, pushing **decrements** `rsp`, so the return address ends up at a smaller address than whatever was already on the stack. The stack grows toward smaller addresses, not larger ones. `push` is `sub rsp, 8` followed by a write; `pop` is a read followed by `add rsp, 8`.

At the moment `solve` starts running, `rsp` points at the return address that was just pushed. Anything the *caller* had sitting in its own local frame lives at addresses larger than the current `rsp`. To read it, you use a **positive** offset, like `[rsp+0x40]`. Anything at a **negative** offset, at addresses smaller than `rsp`, is unallocated.
```text
smaller addresses
  ...
  [rsp - N]     unclaimed space (or stale data from a function that already returned)
  [rsp]         ← current stack pointer
  [rsp + 0x40]  caller's locals — the flag lives here
  ...
larger addresses
```

```
.intel_syntax noprefix
.global solve
solve:

mov rsi, rsp
add rsi, 0x40
mov rdi, 1
mov rdx, 10000 # writing a large number of bytes
mov rax, 1
syscall
ret
```

```console
ubuntu@the-stack-revisited~reaching-into-the-callers-frame:~$ vim pwn.s
ubuntu@the-stack-revisited~reaching-into-the-callers-frame:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@the-stack-revisited~reaching-into-the-callers-frame:~$ /challenge/check pwn.so

Let's see if your solve reads the flag out of the caller's frame and writes it to stdout...

hacker@the-stack-revisited~reaching-into-the-callers-frame:/home/hacker$ /challenge/harness /tmp/your-program.so

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] found solve at 0x782e1559e000
[harness] reading 128 bytes of flag from stdin ...
[harness] calling caller(solve, flag_buf) --- caller stashes the flag in its local frame at [rsp+0x40] (from solve's view) and then calls into your solve
[harness] your `solve` should `write` 128 bytes from [rsp+0x40] to stdout:

Here is your flag!
```

---
## Loading stale stack data

When your code calls a function, that callee can move `rsp` left and use stack memory of its own. When it returns, it moves `rsp` back right, but the bytes it wrote are not automatically erased. When the leftover data is sensitive, skipping that erasure can become a vulnerability. This level starts with the smallest version of that issue, one stale 8-byte value.

```
.intel_syntax noprefix
.global solve
solve:

call rdi
mov rax, qword ptr [rsp-0x10]
ret
```

`call rdi` calls the function pointer `load_secret`. While it runs, `load_secret` pushes its own return address and uses  scratch space below *its* `rsp`, space that sits below (leftward of) `solve`'s original `rsp`. When `load_secret` returns, `rsp` moves back to where it was, but the bytes it wrote at that lower address are still there. `[rsp-0x10]` reaches 16 bytes below `solve`'s current `rsp` to read exactly the spot `load_secret` left its secret in.

```console
ubuntu@the-stack-revisited~loading-stale-stack-data:~$ vim pwn.s
ubuntu@the-stack-revisited~loading-stale-stack-data:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@the-stack-revisited~loading-stale-stack-data:~$ /challenge/check pwn.so

Let's see if your solve calls load_secret, then returns the stale 8-byte value it leaves behind...

hacker@the-stack-revisited~loading-stale-stack-data:/home/hacker$ /challenge/harness /tmp/your-program.so

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] calling solve(load_secret) --- your code receives the load_secret function pointer in rdi
[harness] load_secret returns 0, but leaves an 8-byte stale value at [rsp-0x10] from solve's view
[harness] call load_secret, then return qword ptr [rsp-0x10] in rax:
[harness] match! return value == stale secret.

Your function returned the stale secret. The checker will print the flag now!

Here is your flag!
```

---
## Stealing stale stack data

The above was an example of loading a stale 8-byte value from an old callee frame. Similarly, one can steal data from the stack too, just a bigger stale region and read out with `write` instead of returned in `rax`.

```
.intel_syntax noprefix
.global solve
solve:

call rdi
mov rdi, 1
lea rsi, [rsp-0x88]
mov rdx, 100
mov rax, 1
syscall
```

```console
ubuntu@the-stack-revisited~stealing-stale-stack-data:~$ vim pwn.s
ubuntu@the-stack-revisited~stealing-stale-stack-data:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@the-stack-revisited~stealing-stale-stack-data:~$ /challenge/check pwn.so

Let's see if your solve calls read_flag, then steals the stale stack bytes it leaves behind...

hacker@the-stack-revisited~stealing-stale-stack-data:/home/hacker$ /challenge/harness /tmp/your-program.so

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] reading a 128-byte padded flag buffer into the harness ...
[harness] calling solve(read_flag) --- your code receives the read_flag function pointer in rdi
[harness] read_flag returns 0, but leaves a 128-byte stale flag buffer at [rsp-0x88] from solve's view
[harness] call read_flag, then write those stale bytes to stdout:
```

`lea rsi, [rsp-0x88]` computes the address of the stale buffer without dereferencing it, then hands that address straight to `write` as the source pointer.

---
## Reserving your own frame

Up to this point, everything's fit in registers. But sometimes a function needs more scratch space than registers can hold, so it makes its own room on the stack by moving `rsp` down, `sub rsp, 256` claims 256 bytes, addressable as `[rsp]` through `[rsp+255]`. Everything that was already on the stack is still there, just at bigger offsets now since `rsp` moved.

This space isn't zeroed for you. It's ordinary stack memory that might still hold bytes from whatever ran before. 

```asm
sub rsp, 256       # allocate a 256-byte frame
    ...            # initialize and use [rsp] through [rsp+255]
add rsp, 256       # deallocate the frame
ret
```

 `ret` pops its return address from `[rsp]`, if `rsp` isn't back where it started before `ret` runs. 

```
.intel_syntax noprefix
.global solve
solve:

sub rsp, 256
mov rcx, 0
loop:
mov byte ptr [rsp+rcx], 0
cmp rcx, 0xFF
je done
inc rcx
jmp loop

done:
add rsp, 256
ret
```

```console
ubuntu@the-stack-revisited~reserving-your-own-frame:~$ vim pwn.s
ubuntu@the-stack-revisited~reserving-your-own-frame:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@the-stack-revisited~reserving-your-own-frame:~$ /challenge/check pwn.so

Checking that your solve() reserves and restores a stack frame...
Your code makes room on the stack and puts rsp back.

Let's pre-fill your frame with nonzero bytes and see if solve() clears them...
hacker@the-stack-revisited~reserving-your-own-frame:/home/hacker$ /challenge/harness /tmp/your-program.so

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] calling solve() on a temporary stack with 256 nonzero bytes where its frame will be
[harness] solve cleared the whole 256-byte frame and restored rsp
Every byte in the reserved frame was cleared.

Here is your flag!
```

---
## Using your own frame

A byte only has 256 possible values, so the reserved 256-byte frame doubles as a presence table, one slot per possible byte value, indexed by the value itself.

```
.intel_syntax noprefix
.global solve
solve:

sub rsp, 256
mov rcx, 0

clear_loop: # clearing up the stack frame
mov byte ptr [rsp+rcx], 0
inc rcx
cmp rcx, 256
jne clear_loop

mark_loop: # marking value in stack frame
cmp rsi, 0
je setup_count
mov rcx, 0
mov cl, byte ptr [rdi] # moving values from challenge's pointer to rcx, byte by byte
mov byte ptr [rsp+rcx], 1
inc rdi
dec rsi
jmp mark_loop

setup_count:
mov rcx, 0
mov rax, 0

count_loop: # counting values from stack table
mov r10b, byte ptr [rsp+rcx]
cmp r10b, 0x0
je skip
inc rax

skip:
inc rcx
cmp rcx, 256
jne count_loop

done:
add rsp, 256
ret
```

`clear_loop` zeroes the whole 256-byte table first. `mark_loop` walks the input buffer byte by byte, using each byte's own value as an index (`mov cl, [rdi]` then `[rsp+rcx]`) and setting that slot to `1`. `count_loop` then just walks all 256 slots and counts how many are non-zero. 

```console
ubuntu@the-stack-revisited~using-your-own-frame:~$ vim pwn.s
ubuntu@the-stack-revisited~using-your-own-frame:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@the-stack-revisited~using-your-own-frame:~$ /challenge/check pwn.so

Let's hand your solve() some byte strings and count the distinct values it finds...
hacker@the-stack-revisited~using-your-own-frame:/home/hacker$ /challenge/harness /tmp/your-program.so <hex bytes>

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] calling solve(data, 1) --- your code returns the count of distinct byte values in rax
[harness] solve returned 1
  ok: 1 byte -> 1 distinct
  ok: 4 bytes -> 1 distinct
  ok: 5 bytes -> 5 distinct
  ok: 6 bytes -> 3 distinct
  ok: 256 bytes -> 256 distinct
  ok: 29 bytes -> 29 distinct
  ok: 35 bytes -> 34 distinct
  ok: 28 bytes -> 26 distinct
  ok: 3 bytes -> 3 distinct
Every count was right --- your frame held up!
```

---
## Environment variables on the stack

The stack doesn't only store the program's arguments — it also stores an array of pointers to strings, the environment variables. Each string carries both the name and value in one line, like `PATH=/usr/bin:...`, `HOME=/home/hacker`, or `PWN=COLLEGE`.

For a program called with no arguments and a single environment variable `FLAG`, the layout at process start looks like this:

```text
rsp + 0   →  argc               (1)
rsp + 8   →  argv[0]            (pointer to the program name string)
rsp + 16  →  NULL               (end of argv)
rsp + 24  →  envp[0]            (pointer to "FLAG=..." string)
rsp + 32  →  NULL               (end of envp)
```

Two things worth noticing:

1. Both `argv` and `envp` are **NULL-terminated** pointer lists, the kernel writes a `NULL` pointer at the end of each, so you know where to stop walking. That's the `NULL` at `rsp+16` and the one at `rsp+32`.
2. Each `envp` entry is a pointer into a `NAME=VALUE` string, e.g. `envp[0]` points at the start of `FLAG=...`.

```
.intel_syntax noprefix
.global start
start:

mov rdi, 1
mov rsi, [rsp+24]
mov rdx, 100
mov rax, 1
syscall
mov rdi, 0
mov rax, 60
syscall
```

`[rsp+24]` is `envp[0]`, a pointer, so `mov rsi, [rsp+24]` loads the *address* the string lives at, not the string itself. That address goes straight into `write` as the source pointer.

```console
ubuntu@the-stack-revisited~environment-variables-on-the-stack:~$ vim pwn.s
ubuntu@the-stack-revisited~environment-variables-on-the-stack:~$ ./build.sh pwn
/nix/store/7fpa17hpgqqs084lk08j0mwmc5xpbyf4-binutils-2.46/bin/ld.bfd: warning: cannot find entry symbol _start; defaulting to 0000000000401000
Successfully built pwn!
ubuntu@the-stack-revisited~environment-variables-on-the-stack:~$ /challenge/check pwn

Checking that your assembly reads envp[0] and writes its bytes to stdout...
Your assembly looks correct! Let's see what it prints...

Running your program with FLAG set in the environment...

hacker@the-stack-revisited~environment-variables-on-the-stack:/home/hacker$ env -i 'FLAG=<the flag, padded to 128 bytes>' /tmp/your-program

FLAG=pwn.college{REDACTED}===================================
If your program is right, the flag is printed above!
```

---
## Aligning the stack through the environment

In the previous level you read `envp[0]`, a pointer the kernel placed on the stack, pointing into the strings region above the pointer tables. The same layout applies here, `argc` at `rsp+0`, `argv[0]` pointer at `rsp+8`, NULL at `rsp+16`, `envp[0]` pointer at `rsp+24`, NULL at `rsp+32`, then the actual strings further up.

The interesting question is where the actual addresses come from. When a program launches, the kernel fills the stack **backwards** from some starting address: it lays down the environment strings first (growing toward smaller addresses), then the argument strings, then other metadata, then the `envp[]` and `argv[]` pointer tables, and finally `argc`, which is where `rsp` ends up.

That has a direct consequence: the more bytes stuffed into the environment (or the arguments), the further left the kernel has to push everything else. One extra byte in an environment variable means `rsp` lands one byte lower, the strings region shifts one byte lower with it, and `argv[0]` holds a value one smaller.

`env -i` runs a command with an **empty** environment, any `NAME=VALUE` pairs listed after it are the only environment strings the child receives. That matters here because if your own shell's variables were also present, they'd add unpredictable bytes to the count and throw the target address off. The technique is to take a clean baseline with `env -i /challenge/program` to see what address it wants `argv[0]` at, then re-run with exactly one padded environment variable until `argv[0]` lands on that address.

```console
ubuntu@the-stack-revisited~aligning-the-stack-through-the-environment:~$ env -i /challenge/program
argv[0] is at 0x7fffffffefd5; I want it at 0x7fffffffef55.
That's 128 bytes lower --- adjust your env padding.
ubuntu@the-stack-revisited~aligning-the-stack-through-the-environment:~$ env -i FOO=$(python3 -c "print('A'*120)") /challenge/program
argv[0] is at 0x7fffffffef58; I want it at 0x7fffffffef55.
That's 3 bytes lower --- adjust your env padding.
ubuntu@the-stack-revisited~aligning-the-stack-through-the-environment:~$ env -i FOO=$(python3 -c "print('A'*123)") /challenge/program
```

---

## Aligning the stack through GDB

A common snafu: by default, GDB passes its own environment to the program it's debugging. your shell's variables plus a few GDB adds on its own, and those extra variables shift the stack left compared to running the program straight from the shell. 

```console
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb:~$ /challenge/bin/gdb /challenge/program
(gdb) run
Starting program: /challenge/program
process 164 is executing new program: /proc/164/exe
argv[0] is at 0x7fffffffed1e (running under GDB) --- saved as your target.
Quit gdb, then run `/challenge/program` from your shell with environment
padding (`FOO=xxxxxxxx /challenge/program`) until argv[0] lands here.
[Inferior 1 (process 164) exited with code 01]
(gdb) q
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb:~$ /challenge/program
argv[0] is at 0x7fffffffed72 (running in the shell); I want it at 0x7fffffffed1e (where gdb put it).
That's 84 bytes lower --- pad your shell environment to shift it there.
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb:~$ python3 -c "print('a'*81)"
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb:~$ A=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa /challenge/program
```

---
## Aligning the stack through GDB, generalized

```console
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb-generalized:~$ /challenge/bin/gdb /challenge/program
(gdb) run
Starting program: /challenge/program
process 144 is executing new program: /proc/144/exe
argv[0] is at 0x7fffffffeee1 (running under GDB) --- saved as your target.
Quit gdb. My environment in here is nothing like your shell's, so you
can't pad your shell to match it. Outside gdb I require exactly ONE
environment variable: wipe your environment and rebuild it with one
padded variable ---
    env -i FOO=xxxxxxxx /challenge/program
--- then grow the x's until argv[0] lands here.
[Inferior 1 (process 144) exited with code 01]
(gdb) q
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb-generalized:~$ env -i SOMENAME=value /challenge/program
argv[0] is at 0x7fffffffefc6 (one env var); I want it at 0x7fffffffeee1 (where gdb put it).
That's 229 bytes lower --- add 229 more characters to your one variable.
ubuntu@the-stack-revisited~aligning-the-stack-through-gdb-generalized:~$ env -i A=aaa...(241 a's)... /challenge/program
```

---

## Things to remember

- The stack grows toward **smaller** addresses. `push` = `sub rsp, 8` then write; `pop` = read then `add rsp, 8`.
- From inside a function, **positive** offsets from `rsp` reach into the caller's frame (rightward/upward in address terms). **Negative** offsets reach into unallocated space or stale data from a function that already returned and moved `rsp` back (leftward/downward).
- Stack memory is never pre-zeroed. Any frame you reserve with `sub rsp, N` needs explicit initialization.
- `sub rsp, N` / `add rsp, N` must be balanced before `ret`, since `ret` reads its return address from `[rsp]`.
- Stack layout at process start: `argc` at `rsp+0`, then `argv[]` pointers (NULL-terminated), then `envp[]` pointers (also NULL-terminated), then the actual argument and environment strings further up in memory.
- The kernel fills the stack backward from a fixed top address. More bytes in the environment or arguments pushes everything, including `rsp` itself,  to smaller addresses.
- `env -i` runs a command with a completely empty environment, so only the variables listed after it are present. Necessary whenever exact stack alignment matters and your own shell's variables would otherwise add unpredictable padding.
- GDB injects its own environment into debugged programs, which shifts stack addresses relative to a normal shell run. Reproducing a GDB-observed address outside GDB generally requires `env -i` plus one controlled variable, not just padding your existing shell environment.