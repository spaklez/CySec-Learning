Our programs in general have a small region of memory ready for temporary storage, this region is known as the Stack. The stack is pointed to by the `rsp` register, when a program starts, `rsp` points to data that represents the number of command-line arguments passed to the program (including the program name itself). Remember that pushing onto the stack decreases the `rsp` value, and popping from the stack increases the `rsp` value.

Reading/Dereferencing the `rsp` value gives you the argument count of the program that `rsp` is pointing to,

```assembly
.intel_syntax noprefix
.global _start
_start:
mov rdi, [rsp]
mov rax, 60
syscall
```

```console
hacker@the-stack~the-stack:/home/hacker$ pwn arg0 arg1 arg2
hacker@the-stack~the-stack:/home/hacker$ echo $?
4
```

Note the count includes the program name itself, so three arguments gives `argc` of 4.

Remember that `rsp` points to the top of the stack, you can access other values of the stack by offsetting from `rsp`. In general, `[rsp+N]` reads memory at the address `rsp+N` where N is in bytes.

So the stack from the top looks like:

| Expression | What's there |
|---|---|
| `[rsp]` | `argc`, the argument count |
| `[rsp+8]` | pointer to `argv[0]`, the program name |
| `[rsp+16]` | pointer to `argv[1]`, the first real argument |
| `[rsp+24]` | pointer to `argv[2]` |
| ... | ... |
| | a NULL to mark the end of `argv` |
| | then the environment variables, `envp` |

You'll notice these offsets go in multiples of 8. That's because many values on the stack, such as numbers or memory addresses, tend to be 8 bytes (64 bits) wide, so consecutive values are 8 bytes apart.

This is the same byte-addressing point from the previous module, i.e. one address holds one byte, so an 8-byte value spans 8 addresses and the *next* value starts 8 further along. `[rsp+1]` would land you one byte into `argc` rather than at the next item, which gives garbage.

An example of reading a value from a specific offset is

```assembly
.intel_syntax noprefix
.global _start
_start:
mov rdi, [rsp+128]
mov rax, 60
syscall
```

In this case, the 128 offset means 128 bytes past `rsp`, which is `128/8 = 16` slots along, i.e. the 17th value on the stack counting `[rsp]` itself as the first.

Right after the argument count, the stack stores _pointers_ to each program argument. These are _addresses_ stored in memory: `[rsp+16]` doesn't contain the argument text directly --- it contains the _address_ where that text lives.

To get the actual argument data, you need to dereference twice: once to get the pointer from the stack, and once to follow it to the data.

For example, if your program is run as `/tmp/your-program Hi`:

```text
     Register │ Contents
   +───────────────────────────+
   │ rsp      │ 1337000        │─┐
   +───────────────────────────+ │
                                 │
  ┌──────────────────────────────┘
  │
  │    Address    │ Contents
  │  +────────────────────────+
  │  │ ...        │ ...       │
  │  +────────────────────────+
  └▸ │ 1337000    │ 2         │  ◀── the ARGument Count (termed "argc")
     +────────────────────────+
     │ 1337008    │ 1234000   │──────┐
     +────────────────────────+      │
     │ 1337016    │ 1234560   │────┐ │
     +────────────────────────+    │ │
     │ 1337024    │ 0         │    │ │
     +────────────────────────+    │ │
	                               │ │
   ┌───────────────────────────────┘ │
   │                                 │
   │   Address   │ Contents          │
   │ +──────────────────────────+    │
   │ │ 1234000   │ "/tmp/..."   │◀───┘ the program name
   │ +──────────────────────────+
   │ │ ...       │ ...          │
   │ +──────────────────────────+
   └▸│ 1234560   │ "Hi"         │ the first argument!
     +──────────────────────────+
```

```assembly
mov rdi, [rsp+16]   # load the first argument pointer (e.g., 1234560) from the stack
mov rdi, [rdi]      # follow the pointer to read the actual data (e.g., "Hi")
```

```console
hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ pwn 3
hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ echo $?
51

hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ pwn p
hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ echo $?
112

hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ pwn l
hacker@the-stack~program-arguments-on-the-stack:/home/hacker$ echo $?
108
```

Worth looking at those exit codes, `3` gives 51, `p` gives 112, `l` gives 108. Those are the **ASCII values** of the characters, i.e. `'3'` is 0x33 = 51, `'p'` is 0x70 = 112. The second dereference loads 8 bytes starting at the string, but since the exit code is only the low byte of `rdi`, what comes out is the first character. Arguments arrive as text, never as numbers.

To pop from the stack, we make use of the `pop` instruction, so `pop`'ing `rdi` does two things,

1. Reads the value at `[rsp]` into `rdi` (just like `mov rdi, [rsp]`).
2. Adds 8 to `rsp`, advancing the stack pointer to the next value.

```text
hacker@dojo:~$ /tmp/your-program hello world
```

Before the `pop rdi`:

```text
    Address    │ Contents
  +───────────────────────+
  │ ...        │ ...      │
  +───────────────────────+
┌▸│ 1337000    │ 3        │  ◀── the argument count
│ +───────────────────────+
│ | 1337008    | ???      |
│ +───────────────────────+
│
└────────────────────────────┐
                             │
   Register │ Contents       │
  +────────────────────────+ │
  │ rsp     │ 1337000      │─┘
  +────────────────────────+
  │ rdi     │ 0            │
  +────────────────────────+
```

After the `pop rdi`:

```text
    Address    │ Contents
  +───────────────────────+
  │ ...        │ ...      │
  +───────────────────────+
  │ 1337000    │ 3        │
  +───────────────────────+
┌▸| 1337008    | ???      |
│ +───────────────────────+
│
└────────────────────────────┐
                             │
   Register │ Contents       │
  +────────────────────────+ │
  │ rsp     │ 1337008      │─┘
  +────────────────────────+
  │ rdi     │ 3            │
  +────────────────────────+
```

The value `3` was popped off the top of the stack into `rdi`, and `rsp` advanced by 8 bytes to point to the next value. The data at `1337000` is still there in memory, but as far as the stack is concerned, it's been removed: `rsp` has moved past it.

That last point matters more than it looks. `pop` doesn't erase anything, it just moves the pointer. The old value sits there until something else pushes over it, which is why "freed" stack data is still readable if you go looking for it, and why uninitialised local variables in C contain leftover junk from whatever ran before.

Diagrams often draw the stack vertically, but the numbers themselves still have a simple left-to-right order:

```text
smaller addresses                                      larger addresses
...  rsp-0x10      rsp-0x08      rsp      rsp+0x08      rsp+0x10  ...
```

Positive offsets from `rsp`, such as `[rsp+8]`, read bytes at larger addresses. Negative offsets, such as `[rsp-8]`, read bytes at smaller addresses.

When a program starts, the kernel has already placed launch data at the starting `rsp` and at larger addresses to its right on this number line.