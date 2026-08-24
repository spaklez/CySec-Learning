### Binary

Binary is a number system which is the only one understood by the computer, it is of base-2 and has values of only 1 or 0. A binary digit is called a bit, numbers greater than 1 or 0 require multiple digits to represent them.

Each position in a binary number is a **power of 2**, and they increase right to left. So reading `1101`:

| Bit | 1 | 1 | 0 | 1 |
|---|---|---|---|---|
| Place value | 2³ = 8 | 2² = 4 | 2¹ = 2 | 2⁰ = 1 |
| Contributes | 8 | 4 | 0 | 1 |

8 + 4 + 0 + 1 = **13**. So `0b1101` is 13 in decimal.

The general rule is that **n bits can represent 2ⁿ different values**. A single binary digit (bit) can represent two values (`0` and `1`), two bits can represent four values (`00`, `01`, `10`, and `11`), three bits can represent eight values (`000`, `001`, `010`, `011`, `100`, `101`, `110`, `111`), and four bits can represent sixteen values. This is worth knowing cold, since it's where every size limit in computing comes from — 8 bits gives 256 values, 32 bits gives about 4.3 billion, 64 bits gives roughly 1.8×10¹⁹.

### Decimal

Decimal is base-10, i.e. the system we use day to day, with ten digits `0` through `9`. It works exactly the same way as binary, just with powers of 10 instead of powers of 2:

| Digit | 4 | 2 | 7 |
|---|---|---|---|
| Place value | 10² = 100 | 10¹ = 10 | 10⁰ = 1 |
| Contributes | 400 | 20 | 7 |

Comparatively, a single decimal digit can represent 10 values (from `0` to `9`). Ten values are represented by roughly `log2(10) == 3.3219...` bits, and you get weird situations like binary `1001` being decimal `9`, but binary `1100` (still 4 binary digits) being `12` (_two_ decimal digits!). Another way of expressing this digit desynchronization between decimal and binary is that decimal does not have clean _bit boundaries_.

The lack of bit boundaries makes reasoning about the relationship between decimal and binary complex. For example, it is hard to spot-translate numbers between decimal and binary in general: we can work out that `97` is `1100001`, but it's hard to see that at a glance.

Base-10 is almost certainly just an accident of having ten fingers. There's nothing mathematically special about it, and for computing it's actively inconvenient for the reason above.

Why do computers use Binary? It's because of Logic Gates, as they're easier to build them with 1s (ON) and 0s (OFF).

**Ternary computers** did actually exist. The Soviet **Setun**, built at Moscow State University in 1958, used balanced ternary, i.e. three states of `-1`, `0`, and `+1`. Balanced ternary has real advantages on paper — negative numbers need no special sign handling, rounding is simpler, and base 3 is closer to the theoretically optimal radix *e* (≈2.718) for representing numbers with the fewest total digit-positions.

It lost anyway, and the reason is physical rather than mathematical. Distinguishing two voltage levels is trivially reliable — is there current or isn't there — while distinguishing three means detecting an intermediate level, which is far more sensitive to noise, temperature, and manufacturing variation. Binary components are simpler, cheaper, and more tolerant. Nowadays, however, Binary is the standard.

### Octal

Octal is base-8, using digits `0` through `7`. Each octal digit maps cleanly to exactly **3 bits**, since 2³ = 8.

| Octal | Binary |
|---|---|
| `0` | `000` |
| `3` | `011` |
| `7` | `111` |

Notation is a leading `0` in C, so `0755` is octal, not seven hundred and fifty five. Some languages use `0o755` to make it less ambiguous.

You'll mostly meet octal in one place: **Unix file permissions**. `chmod 755` is octal because each permission triple (read/write/execute) is exactly 3 bits, so one octal digit describes one triple perfectly:

```
7 = 111 = rwx
5 = 101 = r-x
4 = 100 = r--
```

That's the whole reason `chmod` uses those numbers. Outside of permissions and a few legacy systems, octal has largely been replaced by hex, because 3 bits doesn't divide evenly into an 8-bit byte, whereas 4 does.

### Hexadecimal

It's much easier to spot-translate between bases that have more alignment between digits. A single hexadecimal (base 16) digit can represent 16 values (`0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `a`, `b`, `c`, `d`, `e`, `f`): the same number of values that binary can represent in 4 digits! This allows us to have a super simple mapping:

|Hex|Binary|Decimal|
|---|---|---|
|`0`|`0000`|`0`|
|`1`|`0001`|`1`|
|`2`|`0010`|`2`|
|`3`|`0011`|`3`|
|`4`|`0100`|`4`|
|`5`|`0101`|`5`|
|`6`|`0110`|`6`|
|`7`|`0111`|`7`|
|`8`|`1000`|`8`|
|`9`|`1001`|`9`|
|`a`|`1010`|`10`|
|`b`|`1011`|`11`|
|`c`|`1100`|`12`|
|`d`|`1101`|`13`|
|`e`|`1110`|`14`|
|`f`|`1111`|`15`|

This mapping from a hex digit to 4 bits is something that's easily memorizable (most important: memorize `1`, `2`, `4`, and `8`, and you can quickly derive the rest). Better yet, two hex digits is **8** bits, which is one byte! Unlike decimal, where you'd have to memorize 16 mappings for 4 bits and 256 mappings for 8 bits, with hexadecimal, you only have to memorize 16 mappings for 4 bits and the same amount of mappings for 8 bits, since it's just two hexadecimal digits concatenated! Some examples:

|Hex|Binary|Decimal|
|---|---|---|
|`00`|`0000 0000`|`0`|
|`0e`|`0000 1110`|`14`|
|`3e`|`0011 1110`|`62`|
|`e3`|`1110 0011`|`227`|
|`ee`|`1110 1110`|`238`|

Now you're starting to see the beauty. This gets even more obvious when you expand beyond one byte of input, but we'll let you find that out through future challenges!

**Converting by grouping** — because 4 bits is exactly one hex digit, converting is purely mechanical. Split the binary into groups of 4, **starting from the right**, and translate each group independently:

```
0b110000110
     ↓ group from the right, pad the leftmost group with zeros
  0001 1000 0110
     ↓ translate each group
    1    8    6
  = 0x186
```

Starting from the right matters, since that's where the least significant bit is. If you group from the left and the total isn't a multiple of 4, every group is wrong.

Going the other way is the same in reverse, i.e. each hex digit expands to its 4 bits:

```
0x3e  →  0011 1110  →  0b111110
```

**Pros of hex over binary:** far shorter to write, aligns perfectly with bytes and nibbles, and stays trivially convertible to binary in your head. A 64-bit value is 16 hex digits instead of 64 binary ones.

**Cons:** you can't see individual bits at a glance, so if you're doing bit-level work like checking whether a specific flag bit is set, binary is more readable. And arithmetic in hex is harder for humans than decimal, since we don't have the multiplication tables memorised.

**Notation.** How do you differentiate `11` in decimal, `11` in binary (which equals `3` in decimal), and `11` in hex (which equals `17` in decimal)? For numerical constants, we sometimes prepend binary data with `0b`, hexadecimal with `0x`, and keep decimal as is, resulting in `11 == 0b1011 == 0xb`, `3 == 0b11 == 0x3`, and `17 == 0b10001 == 0x11`.

### Encoding

Bits/Binary is mostly used for encoding images, text, assembly code and so on.

**ASCII** (American Standard Code for Information Interchange, 1963) is the mapping from numbers to characters. It's a **7-bit** encoding, so 128 possible characters, which was enough for English letters, digits, punctuation and control codes. The eighth bit was originally used for parity checking, or just left as 0.

For the most part, Uppercase letters use 0x40 + letter index in hex, and for lowercase it's 0x60 + letter index in hex.

| Char | Hex | Decimal |
|---|---|---|
| `A` | `0x41` | 65 |
| `Z` | `0x5a` | 90 |
| `a` | `0x61` | 97 |
| `z` | `0x7a` | 122 |
| `0` | `0x30` | 48 |
| `9` | `0x39` | 57 |

Two things that fall out of this and are genuinely useful:

The difference between an uppercase and lowercase letter is exactly `0x20`, i.e. a single bit (bit 5). So case conversion is just flipping one bit — `'A' | 0x20` gives `'a'`, and `'a' & ~0x20` gives `'A'`. That's why `tolower()` is so cheap.

Digit characters start at `0x30`, so converting the character `'7'` to the number 7 is just subtracting `0x30`. This is why `pwn 3` in the previous module exited with 51, i.e. `0x33`, and not 3.

Characters lower than 0x20 are control characters, i.e. not printable — newline is `0x0a`, tab is `0x09`, null is `0x00`, carriage return is `0x0d`.

Eventually ASCII evolved into **UTF-8**. UTF-8 also makes use of 8 bits, and if the top bit is set, it reads more bytes, and if those top bits are set, it reads more and more bytes.

The design is genuinely elegant. If the top bit is `0`, it's a plain single-byte ASCII character, so **all valid ASCII is already valid UTF-8** and nothing broke when the world switched over. If the top bit is `1`, the number of leading 1s tells you how many bytes this character occupies:

| Leading bits | Total bytes | Range covered |
|---|---|---|
| `0xxxxxxx` | 1 | ASCII, U+0000–U+007F |
| `110xxxxx` | 2 | U+0080–U+07FF |
| `1110xxxx` | 3 | U+0800–U+FFFF |
| `11110xxx` | 4 | U+10000–U+10FFFF |

Continuation bytes always start `10xxxxxx`, which means you can jump into the middle of a UTF-8 stream and immediately tell whether you're at a character boundary. That property is why it won over the alternatives.

**Why hex is used for this** — because a byte is exactly two hex digits, dumping raw memory or a file in hex gives you a clean grid where every byte is visually separate. Look at any `xxd` or hex-editor output and you're reading bytes directly. In binary the same dump would be 8× wider and unreadable; in decimal the columns wouldn't line up.

### Groupings

A standard-size grouping of bits is known as a byte. As you know, bytes are what is actually stored in your computer's memory. As you might also know, computers think in binary: just a bunch of ones and zeroes. For historical reasons, we express these ones and zeroes ("bits") in groups of 8, and each group of 8 is a "byte". This number is purely arbitrary: early computers (pre-1960s or so) didn't have this grouping at all, or had other arbitrary groupings. It is very feasible for there to be an alternate universe in which a byte is 16, 32, or really any number of bits (though for math reasons, it'll likely remain a power-of-2).

The 8-bit byte was popularised by IBM with the System/360 in 1964, and every modern architecture now uses 8-bit bytes. However, since modern architectures are 64-bit, we make use of Words. Words are groupings of 8-bit bytes.

| Name | Bits | Bytes | Hex digits |
|---|---|---|---|
| **bit** | 1 | — | — |
| **nibble** | 4 | ½ | 1 |
| **byte** | 8 | 1 | 2 |
| **word** | 16 | 2 | 4 |
| **dword** (double word) | 32 | 4 | 8 |
| **qword** (quad word) | 64 | 8 | 16 |

A **nibble** is half a byte, i.e. exactly one hex digit. That's why hex works so well — one hex character *is* one nibble.

The confusing part, which came up in the memory module: **"word" here means 16 bits, not the machine's native width.** In general computer-architecture usage a "word" is whatever the architecture's natural unit is, so 64 bits on x86_64. But in x86 terminology specifically, `word` was fixed at 16 bits by the original 8086 and never changed, so `dword` is 32 and `qword` is 64. This is why the assembler wants `mov QWORD PTR [addr], 42` for a 64-bit write. Just remember x86 names are historical, not descriptive.

You'll also see **hword** (half word) in ARM documentation, which is 16 bits there since ARM's word is 32.

### Significance and ordering

**Most significant bit (MSB)** is the bit with the largest place value, i.e. the **leftmost** one when written normally. **Least significant bit (LSB)** is the smallest place value, i.e. the **rightmost**.

```
0b1101 0110
  ↑        ↑
  MSB      LSB
  (128)    (1)
```

The same terms apply at byte level. In `0x11223344`, the most significant byte is `0x11` and the least significant is `0x44`.

| Term | Position | Also called |
|---|---|---|
| Most significant | leftmost | high bit / high byte / high-order |
| Least significant | rightmost | low bit / low byte / low-order |

This vocabulary connects directly to things you've already used. The partial registers, i.e. `al` is the low byte of `ax` and `ah` is the high byte. Little-endian storage, i.e. the *least* significant byte goes at the *lowest* address. And the sign bit below, which is just the MSB given a special meaning.

### Representing negative numbers

Binary on its own has no concept of a minus sign, so you need a convention.

**Sign-magnitude** is the obvious first idea: reserve the MSB as a sign flag, `0` for positive and `1` for negative, and let the remaining bits hold the magnitude.

```
0000 0101 = +5
1000 0101 = -5
```

It's intuitive but it fails badly in practice. You get **two zeros**, `0000 0000` and `1000 0000` are both zero, which means every equality check needs a special case. And addition doesn't work — adding `+5` and `-5` with normal binary addition gives `1000 1010`, which is `-10`, not zero. So the CPU would need separate circuitry for signed arithmetic.

**Two's complement** is what's actually used, everywhere, and it solves both problems.

To negate a number: **invert all the bits, then add 1.**

```
+5  = 0000 0101
      1111 1010    ← invert
      1111 1011    ← add 1
-5  = 1111 1011
```

Now check the addition:

```
  0000 0101   (+5)
+ 1111 1011   (-5)
─────────────
1 0000 0000   ← the carry out falls off the end
  0000 0000   = 0  ✓
```

That's the whole point. **Normal binary addition just works for signed numbers**, so the CPU needs only one adder circuit for both signed and unsigned arithmetic. There's also exactly one zero.

The MSB still tells you the sign, i.e. `1` means negative, but it isn't a separate flag — it carries place value like every other bit, just a *negative* one. For 8 bits the MSB is worth −128:

```
1111 1011 = -128 + 64 + 32 + 16 + 8 + 0 + 2 + 1 = -5 ✓
```

Range for n bits is **−2ⁿ⁻¹ to 2ⁿ⁻¹−1**, so 8 bits covers −128 to +127. Note the asymmetry, one more negative value than positive, because zero occupies a positive-side slot.

Two consequences worth carrying forward:

The same bit pattern means different things depending on whether you read it as signed or unsigned. `0xff` is either 255 or −1. Nothing in the bits themselves tells you which — that's determined entirely by which instruction the CPU uses. This is exactly what the `movsx` versus `movzx` distinction was about in the register module, and it's a genuine source of security bugs, i.e. a length check that treats attacker-controlled `-1` as a huge positive number.

And overflow wraps around silently. `127 + 1` in 8-bit signed gives `1000 0000`, which is −128. No error, no warning, just a wrong answer that propagates.

---
