## Reversing the calculation

```
0000000000401000 <_start>:
  401000:       48 8b 44 24 10          mov    rax,QWORD PTR [rsp+0x10]
  401005:       80 00 2d                add    BYTE PTR [rax],0x2d
  401008:       80 38 9e                cmp    BYTE PTR [rax],0x9e
  40100b:       75 62                   jne    40106f <fail>
```

Here `rax` gets loaded with `argv[1]`, so `[rax]` is the first byte of the input string. The program adds `0x2d` to that byte, then checks whether the result equals `0x9e`. If it doesn't, it jumps to `fail`.

To pass the check, this needs solving backwards. The program computes `input + 0x2d`, and the result has to equal `0x9e`. So:

```
input = 0x9e - 0x2d = 0x71
```

`0x71` is `'q'` in ASCII.

```console
ubuntu@assembly-assortment~reverse-the-calculation:~$ ./hex2ascii.sh 71
q
ubuntu@assembly-assortment~reverse-the-calculation:~$ /challenge/reverse-me q
pwn.college{...REDACTED...}
```

`add` and `cmp` on a single byte both operate mod 256. Whatever operation the binary does to your input *before* comparing it, the reverse-engineering step is to apply the mathematically inverse operation to the target constant. Here the forward operation was addition, so the reverse operation is subtraction.

---
## Reversing the reverse

```
0000000000401000 <_start>:
  401000:       48 8b 44 24 10          mov    rax,QWORD PTR [rsp+0x10]
  401005:       80 28 11                sub    BYTE PTR [rax],0x11
  401008:       80 38 33                cmp    BYTE PTR [rax],0x33
  40100b:       75 62                   jne    40106f <fail>
```

This time the instruction is `sub`, not `add`. The program computes `input - 0x11` and compares it to `0x33`.

Since the forward operation here is subtraction, the reverse operation is addition:

```
input = 0x33 + 0x11 = 0x44
```

`0x44` is `'D'`.

```console
ubuntu@assembly-assortment~reverse-the-reverse:~$ ./hex2ascii.sh 44
D
ubuntu@assembly-assortment~reverse-the-reverse:~$ /challenge/reverse-me D
pwn.college{...REDACTED...}
```

---
## Dealing with bitwise operations

The XOR gate operates on individual bits. The `xor` instruction computes the *exclusive or* of two values. For each bit position, the result is `1` if exactly one of the two input bits is `1`, and `0` otherwise. For example:

```
  01100001  (0x61, 'a')
^ 00101010  (0x2a, 42)
---------
  01001011  (0x4b, 75)
```

A key property of XOR is that it's **its own inverse**: XORing a value with the same value twice gives back the original value:

```
xor    BYTE PTR [rax],0x2a
cmp    BYTE PTR [rax],0x4b
```

the program XORs your input byte with `0x2a` and checks if the result is `0x4b`. To reverse this, XOR the target with the key: `0x4b ^ 0x2a = 0x61`, which is `'a'`.

XORing the key with the target check value gives you exactly the input byte the program is looking for:

```
0000000000401000 <_start>:
  401000:       48 8b 44 24 10          mov    rax,QWORD PTR [rsp+0x10]
  401005:       80 30 51                xor    BYTE PTR [rax],0x51
  401008:       80 38 03                cmp    BYTE PTR [rax],0x3
  40100b:       75 62                   jne    40106f <fail>
```

```
input = 0x03 ^ 0x51 = 0x52
```

`0x52` is `'R'`.

```console
ubuntu@assembly-assortment~dealing-with-bitwise-operations:~$ ./hex2ascii.sh 52
R
ubuntu@assembly-assortment~dealing-with-bitwise-operations:~$ /challenge/reverse-me R
pwn.college{...REDACTED...}
```

---
## Even or odd

The lowest bit in a number is special, it tells you whether the whole number is even or odd. A number is even exactly when its lowest bit is `0`, and odd exactly when its lowest bit is `1`. The `and` instruction can isolate that bit by masking:

```
.intel_syntax noprefix
.global solve
solve:

and rdi, 0x1
cmp rdi, 0x1 # checking if number is odd
je odd

mov rax, 1 # even
ret

odd:
mov rax, 0
ret
```

`and rdi, 0x1` zeroes out every bit of `rdi` except the lowest one, leaving either `0` or `1`. The `cmp`/`je` pair branches on that result, land on `odd` and return `0`, fall through and return `1` for even.

```console
ubuntu@assembly-assortment~even-or-odd:~$ vim pwn.s
ubuntu@assembly-assortment~even-or-odd:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~even-or-odd:~$ /challenge/check pwn.so

Let's hand your solve() a series of values and see if it spots the even ones...
hacker@assembly-assortment~even-or-odd:/home/hacker$ /challenge/harness /tmp/your-program.so <value>

[harness] loading shared library /tmp/your-program.so ...
[harness] resolving `solve` symbol ...
[harness] calling solve(0x0) --- your code returns its result in rax
[harness] solve returned 0x1
  ok: solve(0x1) = 0 (odd)
  ok: solve(0x2) = 1 (even)
  ok: solve(0x8000000000000000) = 1 (even)
  ok: solve(0xffffffffffffffff) = 0 (odd)
Every value classified correctly!

Here is your flag!
```

---
## Masking bits

Similarly, `and` can be used to `mask` bits, using specific numbers to keep the bits you want and force the rest to zero. An example is:

```
  1011 0110   (your value)
& 0000 1111   (the mask: keep the low 4 bits)
---------
  0000 0110   (everything above the low 4 bits is gone)
```

A very common masking use is isolating the lowest byte of a value with `0xFF`, since `0xFF` is exactly eight `1` bits:

```
.intel_syntax noprefix
.global LOBYTE
LOBYTE:

mov rax, 0
and rdi, 0xff
mov rax, rdi
ret
```

`and rdi, 0xff` clears every bit above the low 8, leaving only the lowest byte. 

```console
ubuntu@assembly-assortment~masking-bits:~$ vim pwn.s
ubuntu@assembly-assortment~masking-bits:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~masking-bits:~$ /challenge/check pwn.so

Let's call your LOBYTE() on a series of values and keep only the low byte...
hacker@assembly-assortment~masking-bits:/home/hacker$ /challenge/harness /tmp/your-program.so <value>

  ok: solve(0xff) = 0xff
  ok: solve(0x1337) = 0x37
  ok: solve(0xdeadbeef) = 0xef
  ok: solve(0xffffffffffffffff) = 0xff
Every value masked correctly!
```

---
## Lowercase a string

`and` is used for masking bits, while `or` is used for setting bits,  turning specific bits on while leaving the rest alone. Wherever the mask has a `1`, the result bit is forced to `1`; wherever it has a `0`, the original bit passes through. An example is:

```
  0100 0001   ('A', 0x41)
| 0010 0000   (turn on 0x20)
---------
  0110 0001   ('a', 0x61)
```

This is an actual use case, since in ASCII, an uppercase letter and its lowercase partner differ only in the `0x20` case bit, the 6th bit from the right.

```
.intel_syntax noprefix
.global chr_lower
chr_lower:

mov rax, 0
or rdi, 0x20
mov rax, rdi
ret
```

```console
ubuntu@assembly-assortment~lowercase-a-string:~$ vim pwn.s
ubuntu@assembly-assortment~lowercase-a-string:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~lowercase-a-string:~$ /challenge/check pwn.so

Let's hand your chr_lower() uppercase letters and see if it lowercases them...
hacker@assembly-assortment~lowercase-a-string:/home/hacker$ /challenge/harness /tmp/your-program.so <uppercase letter>

  ok: chr_lower('Z') = 0x7a ('z')
  ok: chr_lower('B') = 0x62 ('b')
  ok: chr_lower('Q') = 0x71 ('q')
Every letter lowercased correctly!

Here is your flag!
```

---
## Uppercase a string

Since lowercasing involves setting the `0x20` (6th-from-the-right) bit, uppercasing is the opposite, i.e. clearing it. That's done by `and`ing with the flipped mask. `0x20` flipped is `0xDF`:

```
  0110 0001   ('a', 0x61)
& 1101 1111   (0xDF: keep every bit except 0x20)
---------
  0100 0001   ('A', 0x41)
```

An example of uppercasing a string: 

```
.intel_syntax noprefix
.global str_upper
str_upper:

loop:
mov al, BYTE PTR [rdi] # moving the characters
cmp al, 0x0 # checking if last char
je done
and al, 0xDF
mov BYTE PTR [rdi], al  # moving char back in its place
inc rdi
jmp loop

done:
ret
```

Each pass through `loop` reads one byte at `[rdi]`, checks it against `0x0` (the C-string null terminator) to know when to stop, clears the case bit with `and al, 0xDF`, writes the byte back in place, then advances `rdi` by one and repeats. `inc rdi` is what walks the string.

```console
ubuntu@assembly-assortment~uppercase-a-string:~$ vim pwn.s
ubuntu@assembly-assortment~uppercase-a-string:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~uppercase-a-string:~$ /challenge/check pwn.so

Let's hand your str_upper() some lowercase strings and read back what it writes...
hacker@assembly-assortment~uppercase-a-string:/home/hacker$ /challenge/harness /tmp/your-program.so <hex bytes>

  ok: 'hello' -> 'HELLO'
  ok: 'iipsssyzevjozsqldbglcjuntglipwayvglvbww' -> 'IIPSSSYZEVJOZSQLDBGLCJUNTGLIPWAYVGLVBWW'
Every letter uppercased correctly --- your loop walked the whole string!

Here is your flag!
```

---

## Swap case

So overall: `or` always sets the case bit (forcing lowercase); `and` always clears it (forcing uppercase). Neither one swaps case. To flip bits, you use `xor`. Since `xor` sets a bit exactly when its two inputs differ, XORing a bit with `1` inverts it, and XORing with `0` leaves it alone. So XORing a letter with `0x20` flips its case bit.

```
  0110 0001   ('a', 0x61)        0100 0001   ('A', 0x41)
^ 0010 0000   (flip 0x20)      ^ 0010 0000   (flip 0x20)
---------                      ---------
  0100 0001   ('A', 0x41)        0110 0001   ('a', 0x61)
```

Remember the `0x20` bit only means "case" for letters. XORing a digit or punctuation byte with `0x20` still flips that bit, it just doesn't mean anything as a case swap there.

```
.intel_syntax noprefix
.global str_swapcase
str_swapcase:

loop:
mov al, BYTE PTR [rdi] # moving the characters
cmp al, 0x0 # checking if last char
je done
xor al, 0x20
mov BYTE PTR [rdi], al  # moving char back in its place
inc rdi
jmp loop

done:
ret
```

```console
ubuntu@assembly-assortment~swap-case:~$ vim pwn.s
ubuntu@assembly-assortment~swap-case:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~swap-case:~$ /challenge/check pwn.so

Let's hand your str_swapcase() some mixed-case strings and read back what it writes...
hacker@assembly-assortment~swap-case:/home/hacker$ /challenge/harness /tmp/your-program.so <hex bytes>

  ok: 'A' -> 'a'
  ok: 'HeLLo' -> 'hEllO'
  ok: 'uLJPsPvCWASQZiuJtfAeAsyRGeqaatfMrdWQAM' -> 'UljpSpVcwasqzIUjTFaEaSYrgEQAATFmRDwqam'
Every letter's case flipped correctly --- your loop walked the whole string!

Here is your flag!
```

---
## Shifting left

A bit shift moves every bit the same number of positions. A left shift moves bits toward the high end (i.e. the MSB) and drops the bits that leave that end, inserting zeros at the low end. An example below:

```text
before:                    1011 0010  (178)
dropped from high end:     10.. ....
inserted at low end:       .... ..00
after shifting left by 2:  1100 1000  (200)
```

A logical right shift does the opposite, it moves bits toward the low end (i.e. the LSB), it drops the low bits, and inserts zeros at the high end.

```text
before:                     1101 1011  (219)
dropped from low end:       .... ..11
inserted at high end:       00.. ....
after shifting right by 2:  0011 0110  (54)
```

Moving every bit left by one doubles an unsigned value, and shifting left by `n` multiplies by `2^n`, as long as no `1` bit is lost past the high end. Moving every bit right by one halves an unsigned value, discarding any remainder, so shifting right by `n` divides by `2^n`.

On 64-bit x86, `shl` shifts left and `shr` performs the zero-filling right shift:

```asm
shl rax, 2
shr rax, 2
```

```
.intel_syntax noprefix
.global solve
solve:

shl rdi, 4
mov rax, rdi
ret
```

```console
ubuntu@assembly-assortment~shifting-left:~$ vim pwn.s
ubuntu@assembly-assortment~shifting-left:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~shifting-left:~$ /challenge/check pwn.so

Let's call your solve() and watch it shift each value left by 4 bits...
hacker@assembly-assortment~shifting-left:/home/hacker$ /challenge/harness /tmp/your-program.so <value>

  ok: solve(0x1) = 0x10
  ok: solve(0xff) = 0xff0
  ok: solve(0x1234) = 0x12340
  ok: solve(0xac81e1fc7cd91d) = 0xac81e1fc7cd91d0
Every value shifted correctly!

Here is your flag!
```

---
## Shifting right

This one asks for the second-lowest byte of a value, bits 8 through 15, sometimes called "byte 1" (byte 0 being the lowest).

```
.intel_syntax noprefix
.global solve
solve:

shr rdi, 8
mov rax, rdi
and rax, 0xff # masking everything else 0 apart from the lowest byte
ret
```

`shr rdi, 8` shifts the whole value right by one byte, which moves byte 1 down into the byte 0 position (and drops the original byte 0 entirely). At that point byte 1 is sitting at the bottom, but everything above it is still there too, so `and rax, 0xff` masks it down to just that one byte.

```console
ubuntu@assembly-assortment~shifting-right:~$ vim pwn.s
ubuntu@assembly-assortment~shifting-right:~$ ./build-shared.sh pwn
Successfully built pwn.so!
ubuntu@assembly-assortment~shifting-right:~$ /challenge/check pwn.so

Let's call your solve() and pull byte 1 (bits 8-15) out of each value...
hacker@assembly-assortment~shifting-right:/home/hacker$ /challenge/harness /tmp/your-program.so <value>

  ok: solve(0x1300) = 0x13
  ok: solve(0xabcd) = 0xab
  ok: solve(0xdeadbeef) = 0xbe
  ok: solve(0xcd5cfd1d2bc9aeb7) = 0xae
Every byte extracted correctly!

Here is your flag!
```

---

---
## Instruction reference

| Instruction | What it does |
| --- | --- |
| `and dst, src` | Bitwise AND — masking. Clears every bit where `src` has a `0`, leaves the rest of `dst` untouched. |
| `or dst, src` | Bitwise OR — setting bits. Forces every bit to `1` wherever `src` has a `1`. |
| `xor dst, src` | Bitwise XOR — flips every bit where `src` has a `1`. Self-inverse: XORing the same value twice returns the original. |
| `shl dst, n` | Shift left by `n`, zero-filling the low end. Equivalent to `× 2^n` if no `1` bit is lost off the top. |
| `shr dst, n` | Logical shift right by `n`, zero-filling the high end. Equivalent to `÷ 2^n`, remainder discarded. |

---
