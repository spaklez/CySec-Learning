
## Little endian

Intel x86 stores every multi-byte value in Little Endian, i.e. lowest byte first. This module basically deals with working with programs and understanding this endianness further.

Every value in x86 that is larger than a single byte has to be split into individual bytes to live in memory, because memory is addressed one byte at a time. x86 stores the least significant byte at the **lowest address** and the most significant byte at the **highest address**. Some more examples below are:

| size of value | decimal value | hex value | big endian bytes (NOT x86) | little endian bytes (x86) |
| ------------- | ------------- | -------------------- | -------------------------- | ------------------------- |
| 8 (1 byte) | `65` | `0x41` | `41` | `41` |
| 16 (2 bytes) | `4660` | `0x1234` | `12 34` | `34 12` |
| 32 (4 bytes) | `1145258561` | `0x44434241` | `44 43 42 41` | `41 42 43 44` |
| 64 (8 bytes) | `1145258561` | `0x0000000044434241` | `00 00 00 00 44 43 42 41` | `41 42 43 44 00 00 00 00` |

Endianness only matters in memory. As long as you are working one byte at a time in assembly, it never comes up.

---

## Memory order versus register value

Suppose `rdi` points at these eight bytes:

```text
Address    Byte
[rdi+0]    41
[rdi+1]    42
[rdi+2]    43
[rdi+3]    44
[rdi+4]    45
[rdi+5]    46
[rdi+6]    47
[rdi+7]    48
```

A 64-bit load reads those bytes starting at the lowest address:

```asm
mov rax, [rdi]
```

Because x86 is little-endian, `[rdi+0]` becomes the low byte of `rax`, `[rdi+1]` becomes the next byte, and so on. The register value is therefore `0x4847464544434241`. Written as hex, the most-significant byte prints on the left, so the bytes look reversed compared to address order:

```text
memory address order:  41 42 43 44 45 46 47 48
register hex order:    48 47 46 45 44 43 42 41
rax value:             0x4847464544434241
```

The bytes did not move in memory. The CPU interpreted the byte at the lowest address as the least-significant part of the number.

### Is this bad, and how do you avoid getting caught by it?

It is not a bug and it is not something to prevent. It is a display mismatch , and the only fix is knowing which one you are looking at.

Memory is listed in address order, ascending, because that is the only sensible way to list addresses. Numbers are written most significant digit first.
The practical rules that come out of this:

**Know which thing you are reading.** A hex dump, an `objdump` byte column, and `x/8xb` in GDB are all in **address order**. A register value, a `cmp` immediate, and `print $rax` are all in **numeric order**. Every time you move between the two, the bytes appear reversed.

**In GDB, ask for the format you actually want.** `x/8xb $rdi` prints eight bytes in address order and never lies to you about layout. `x/gx $rdi` prints the same eight bytes as one 64-bit number and does apply the endian interpretation. 

**When recovering a string from a multi-byte immediate, reverse it.**  `cmp ax, 0x497a` means the two bytes in memory are `7a 49`, which is `zI`, not `Iz`.

---

## Operand sizes

And also, a refresher on the different sizes in x86 Assembly:

| Name | Bits | Bytes | Partial `rax` Access | Memory Access |
| -------------------- | ---- | ----- | -------------------- | ----------------------------------------- |
| byte | 8 | 1 | `mov al, [rdi]` | `mov BYTE PTR [rdi], 0x11` |
| word | 16 | 2 | `mov ax, [rdi]` | `mov WORD PTR [rdi], 0x1122` |
| doubleword (`dword`) | 32 | 4 | `mov eax, [rdi]` | `mov DWORD PTR [rdi], 0x11223344` |
| quadword (`qword`) | 64 | 8 | `mov rax, [rdi]` | `mov QWORD PTR [rdi], 0x1122334455667788` |


In the "Partial `rax` Access" column, the size comes **from the register**. `mov al, [rdi]` reads one byte because `al` is one byte. `mov eax, [rdi]` reads four because `eax` is four. There is no ambiguity.

In the "Memory Access" column, both operands are a memory address and a bare number. Neither one has a size. `mov [rdi], 0x11` could mean store one byte, two, four or eight, and `0x11` fits all of them. 
The assembler needs to know the width, and it will take that width from a register operand if there is one, or from a size directive if there is not.

---

## Sign extension

A smaller value can be copied into a larger register as either unsigned or signed. If the byte is unsigned, filling the high bytes with zero is fine: `0x7f` becomes `0x000000000000007f`. But a signed byte uses two's complement. The byte `0xff` is `-1`, so extending it to 64 bits must fill the new high bits with `1`s: `0xffffffffffffffff`. This is known as sign extension and copies the sign bit into the new high bits. On x86-64, the instruction used is:

```asm
movsx rax, BYTE PTR [rdi]
```

This reads one byte from the address in `rdi`, treats that byte as signed, and returns the 64-bit signed value in `rax`.

| Instruction | `0x7f` becomes | `0xff` becomes |
| --- | --- | --- |
| `movzx rax, BYTE PTR [rdi]` | `0x000000000000007f` (127) | `0x00000000000000ff` (255) |
| `movsx rax, BYTE PTR [rdi]` | `0x000000000000007f` (127) | `0xffffffffffffffff` (-1) |

The split happens at `0x80`. Any byte with its top bit set is negative under a signed reading, so `0x00` through `0x7f` behave identically either way and `0x80` through `0xff` diverge.

Remember that the `ret` instruction doesn't take any arguments.

### Worked example: a sign-extending function

The function takes a pointer to one byte in `rdi`, loads that byte as a signed 8-bit value, sign-extends it to 64 bits, and returns it in `rax`.

```asm
.intel_syntax noprefix
.global solve
solve:

movsx rax, BYTE PTR [rdi]
ret
```

```console
ubuntu@endian-escapades~sign-extension:~$ /challenge/check pwn.so

Let's call your solve() on signed bytes and check the 64-bit results...
  ok: byte 0x1 -> 1
  ok: byte 0x7f -> 127
  ok: byte 0x80 -> -128
  ok: byte 0x81 -> -127
  ok: byte 0xff -> -1
  ok: byte 0x5a -> 90
  ...
Every byte sign-extended correctly!
```

---

## `movabs` and the 64-bit immediate

```
  401005:       48 bb 6f 44 74 44 77    movabs rbx,0x5a354b774474446f
  40100c:       4b 35 5a
```

`movabs` is `mov` with a **full 64-bit immediate**. The ordinary `mov r64, imm32` form used everywhere so far encodes only 4 bytes of constant and sign-extends them into the register, which is fine for small numbers and useless for a value like this one. `movabs` is the only form that carries all 8 bytes.

That makes it a 10-byte instruction. The important part for reversing, the eight immediate bytes are printed in the byte column **in memory order**, and printed in the instruction text **as a number**. Look at both:

```
byte column:  6f 44 74 44 77 4b 35 5a
immediate:    0x5a354b774474446f
```

And because the comparison is against a qword loaded from the password buffer, it is the **byte column** that tells you what characters the password contains, in order:

| Byte | `6f` | `44` | `74` | `44` | `77` | `4b` | `35` | `5a` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Char | `o` | `D` | `t` | `D` | `w` | `K` | `5` | `Z` |

Which gives `oDtDwK5Z`.

You can read it either way. Take the byte column left to right, or take the immediate and reverse it in pairs from the right. They are the same operation. Reading the byte column is less error-prone because there is nothing to reverse.

---

## The one thing to get straight about endianness

**Endianness is a rule about how one multi-byte value is spread across consecutive addresses. It is applied at the instant of a multi-byte load or store, and nowhere else.**

**Registers have no endianness.** A register is not byte-addressable. 
**Memory has no endianness either.** Memory holds bytes at addresses. That is all. The bytes `41 42 43 44` sitting at some address are neither little endian nor big endian by themselves; they are four bytes.

**The instruction is where endianness happens.** `mov rax, [rdi]` is the moment a decision gets made.

**Byte-at-a-time access never invokes it.** `mov al, [rdi]`, `mov al, [rdi+1]`, and so on, read one byte each. One byte has no order. 

`rdi` itself holds a **pointer**, a single 64-bit number. It has no endianness, because it is a register.

The **bytes it points at** are just bytes at addresses. They also have no endianness on their own.

Consider:

```
rdi -> 11 22 33 44 55 66 77 88
```

Those bytes were placed there by the operating system when it copied your command-line argument in, one byte at a time, in the order you typed them. No multi-byte store happened, so no endian convention was applied. They are in typed order and that is the only order they have.

Then:

```asm
mov rsi, [rdi]      // rsi = 0x8877665544332211
```

*This* instruction applies the convention. It says "treat those eight bytes as one 64-bit number", and the little-endian rule makes `11` the least significant byte. Reading the same buffer as two 32-bit values:

```asm
mov esi, [rdi]      // esi = 0x44332211
mov edx, [rdi+4]    // edx = 0x88776655
```

So the answer is: **the bytes are not in any endianness. The load chose one.**

---
## Two helper scripts

Two small scripts that I made

```bash
#!/bin/bash
objdump -d -M intel $1
```

`dump.sh` , just saves typing `-d -M intel` sixteen times

```bash
#!/bin/bash
# Remove "0x" if it exists, then convert the raw hex to ASCII
echo "$1" | sed 's/0x//' | xxd -r -p
echo ""
```

`hex2ascii.sh`, converts hex2ascii `xxd -r -p` is doing the real work here, `-r` reverses (hex to binary rather than binary to hex) and `-p` is plain mode, meaning a bare hex stream with no offsets or address column. The `sed` strips a leading `0x` so the same script accepts either format. The trailing `echo ""` adds a newline, since `xxd` emits exactly the bytes and nothing else, and without it the shell prompt lands on the same line as the output.

---

## Reversing, qword by qword

```
  401000:       48 8b 7c 24 10          mov    rdi,QWORD PTR [rsp+0x10]
  401005:       48 bb 57 4f 6b 36 34    movabs rbx,0x71613234366b4f57
  40100c:       32 61 71
  40100f:       48 8b 07                mov    rax,QWORD PTR [rdi]
  401012:       48 39 d8                cmp    rax,rbx
  401015:       75 75                   jne    40108c <fail>
  401017:       48 bb 34 58 61 43 54    movabs rbx,0x7430775443615834
  40101e:       77 30 74
  401021:       48 8b 47 08             mov    rax,QWORD PTR [rdi+0x8]
  401025:       48 39 d8                cmp    rax,rbx
  401028:       75 62                   jne    40108c <fail>
```

Two qwords, so sixteen characters, checked eight bytes at a time.

| Field | Byte column | Characters |
| --- | --- | --- |
| `[rdi]` | `57 4f 6b 36 34 32 61 71` | `WOk642aq` |
| `[rdi+8]` | `34 58 61 43 54 77 30 74` | `4XaCTw0t` |

So the password is `WOk642aq4XaCTw0t`.

```console
ubuntu@endian-escapades~qword-by-qword:~$ ./hex2ascii.sh 0x574f6b36343261713458614354773074
WOk642aq4XaCTw0t
ubuntu@endian-escapades~qword-by-qword:~$ /challenge/reverse-me WOk642aq4XaCTw0t
```

---

## Dword by dword

```
  401005:       8b 07                   mov    eax,DWORD PTR [rdi]
  401007:       3d 43 63 36 67          cmp    eax,0x67366343
  401012:       8b 47 04                mov    eax,DWORD PTR [rdi+0x4]
  401015:       3d 44 37 39 70          cmp    eax,0x70393744
  40101c:       8b 47 08                mov    eax,DWORD PTR [rdi+0x8]
  40101f:       3d 68 6e 6c 7a          cmp    eax,0x7a6c6e68
  401026:       8b 47 0c                mov    eax,DWORD PTR [rdi+0xc]
  401029:       3d 68 54 72 78          cmp    eax,0x78725468
```

| Offset | Byte column | Immediate | Characters |
| --- | --- | --- | --- |
| `[rdi]` | `43 63 36 67` | `0x67366343` | `Cc6g` |
| `[rdi+4]` | `44 37 39 70` | `0x70393744` | `D79p` |
| `[rdi+8]` | `68 6e 6c 7a` | `0x7a6c6e68` | `hnlz` |
| `[rdi+0xc]` | `68 54 72 78` | `0x78725468` | `hTrx` |

`Cc6gD79phnlzhTrx`.


```console
ubuntu@endian-escapades~dword-by-dword:~$ ./hex2ascii.sh 0x4363366744373970686e6c7a68547278
Cc6gD79phnlzhTrx
```

The endian reversal is per-group, and the groups themselves stay in offset order. 

---

## Word by word

```
  401005:       66 8b 07                mov    ax,WORD PTR [rdi]
  401008:       66 3d 7a 49             cmp    ax,0x497a
  401012:       66 8b 47 02             mov    ax,WORD PTR [rdi+0x2]
  401016:       66 3d 50 6b             cmp    ax,0x6b50
  ...
```

Each immediate is two bytes, so each one reverses to two characters:

| Offset | Immediate | Bytes | Chars |
| --- | --- | --- | --- |
| `[rdi]` | `0x497a` | `7a 49` | `zI` |
| `+2` | `0x6b50` | `50 6b` | `Pk` |
| `+4` | `0x6735` | `35 67` | `5g` |
| `+6` | `0x4a39` | `39 4a` | `9J` |
| `+8` | `0x3645` | `45 36` | `E6` |
| `+0xa` | `0x5275` | `75 52` | `uR` |
| `+0xc` | `0x4d33` | `33 4d` | `3M` |
| `+0xe` | `0x6e75` | `75 6e` | `un` |

`zIPk5g9JE6uR3Mun`.


```console
ubuntu@endian-escapades~word-by-word:~$ ./hex2ascii.sh 7a49506b3567394a45367552334d76e56e
zIPk5g9JE6uR3Mvn
```

---

## Byte by byte

A single byte has no order to reverse, so the immediates *are* the characters, already in order.

```
  401005:       8a 07                   mov    al,BYTE PTR [rdi]
  401007:       3c 4a                   cmp    al,0x4a
  40100f:       8a 47 01                mov    al,BYTE PTR [rdi+0x1]
  401012:       3c 6d                   cmp    al,0x6d
  ...
```

```console
ubuntu@endian-escapades~byte-by-byte:~$ ./hex2ascii.sh 0x4a6d7537754a62673559574a55695137
Jmu7uJbg5YWJUiQ7
```

---

## Structs

Real programs rarely read a buffer at one uniform size. They read *structs*, a handful of fields of *different* sizes, laid out one after another in memory.

This is what the structure in C would be defined as:

```c
struct { uint64_t a; uint32_t b; uint16_t c; uint8_t d; uint8_t e; };
```

| Field | C type     | Instruction size | Offset | Bytes |
| ----- | ---------- | ---------------- | ------ | ----- |
| `a`   | `uint64_t` | qword            | 0      | 8     |
| `b`   | `uint32_t` | dword            | 8      | 4     |
| `c`   | `uint16_t` | word             | 12     | 2     |
| `d`   | `uint8_t`  | byte             | 14     | 1     |
| `e`   | `uint8_t`  | byte             | 15     | 1     |

### Cracking a struct

```
  401005:       48 bb 71 61 4f 32 4a    movabs rbx,0x6455694a324f6171
  40100c:       69 55 64
  40100f:       48 8b 07                mov    rax,QWORD PTR [rdi]
  401012:       48 39 d8                cmp    rax,rbx
  40101b:       8b 47 08                mov    eax,DWORD PTR [rdi+0x8]
  40101e:       3d 78 74 6f 6d          cmp    eax,0x6d6f7478
  401025:       66 8b 47 0c             mov    ax,WORD PTR [rdi+0xc]
  401029:       66 3d 4e 61             cmp    ax,0x614e
  40102f:       8a 47 0e                mov    al,BYTE PTR [rdi+0xe]
  401032:       3c 35                   cmp    al,0x35
  401036:       8a 47 0f                mov    al,BYTE PTR [rdi+0xf]
  401039:       3c 32                   cmp    al,0x32
```

Each field reverses at its own width, independently.

| Offset | Size | Byte column | Chars |
| --- | --- | --- | --- |
| 0 | qword | `71 61 4f 32 4a 69 55 64` | `qaO2JiUd` |
| 8 | dword | `78 74 6f 6d` | `xtom` |
| 12 | word | `4e 61` | `Na` |
| 14 | byte | `35` | `5` |
| 15 | byte | `32` | `2` |

`qaO2JiUdxtomNa52`.

---

## Scrambled structs

The last struct read its fields top to bottom, in memory order, so reading the disassembly straight down handed you the password already in order. A program can check a struct's fields in *any* order it likes.

 Layout is fixed by the struct definition, so field `c` is at offset 12 no matter what. Access order is whatever the code does, and the code is free to check the last field first. 

**So the offset is the only thing that tells you where a field goes.** Not the position of the `cmp` in the listing. When reversing, read the offset out of every instruction and sort by that, rather than transcribing top to bottom.


```
  401005:       8a 47 0e                mov    al,BYTE PTR [rdi+0xe]
  401008:       3c 4f                   cmp    al,0x4f
  401010:       48 bb 59 72 42 38 30    movabs rbx,0x674d523038427259
  401017:       52 4d 67
  40101a:       48 8b 07                mov    rax,QWORD PTR [rdi]
  40101d:       48 39 d8                cmp    rax,rbx
  401022:       8a 47 0f                mov    al,BYTE PTR [rdi+0xf]
  401025:       3c 74                   cmp    al,0x74
  401029:       66 8b 47 0c             mov    ax,WORD PTR [rdi+0xc]
  40102d:       66 3d 48 48             cmp    ax,0x4848
  401033:       8b 47 08                mov    eax,DWORD PTR [rdi+0x8]
  401036:       3d 55 76 6d 45          cmp    eax,0x456d7655
```

Check order in the code: 14, 0, 15, 12, 8. Same five fields as before, same layout, deliberately shuffled.

Sorting by offset instead:

| Offset | Size | Byte column | Chars |
| --- | --- | --- | --- |
| 0 | qword | `59 72 42 38 30 52 4d 67` | `YrB80RMg` |
| 8 | dword | `55 76 6d 45` | `UvmE` |
| 12 | word | `48 48` | `HH` |
| 14 | byte | `4f` | `O` |
| 15 | byte | `74` | `t` |

`YrB80RMgUvmEHHOt`.

The first attempt was another transcription slip:

---

## Command reference

| Command                  | What it does                                 |
| ------------------------ | -------------------------------------------- |
| `xxd -r -p`              | convert a plain hex string to raw bytes      |
| `x/8xb $rdi`             | GDB: eight bytes in **address order**        |
| `x/gx $rdi`              | GDB: the same bytes as one 64-bit **number** |
| `as --64 x.s -o x.o`     | assemble                                     |
| `ld -shared x.o -o x.so` | link as a shared library                     |


| Instructon                      | What it does                                      |
| ------------------------------- | ------------------------------------------------- |
| `movabs rbx, imm64`             | `mov` with a full 8-byte immediate, 10 bytes long |
| `movsx rax, BYTE PTR [rdi]`     | load a byte, sign-extend into 64 bits             |
| `movzx rax, BYTE PTR [rdi]`     | load a byte, zero-extend into 64 bits             |
| `mov al` / `ax` / `eax` / `rax` | 1 / 2 / 4 / 8 byte loads                          |


---
