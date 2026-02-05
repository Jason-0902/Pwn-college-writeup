# Internal State Mini (x86)

## Challenge Overview

The challenge provides a program named cimg which parses a custom image-like file format.
The program:

Reads a fixed-size header

Reads pixel data

Converts the pixels into an internal framebuffer using `display()`

Compares the framebuffer with a built-in reference (desired_output)

Calls `win()` if they match

Our goal is to construct a valid input file that produces the same internal state as `desired_output`.

1. Header format

From the beginning of main:

```asm
mov    %rsp,%r12
mov    $0x8,%edx
mov    %r12,%rsi
call   read_exact
```

The header size is 8 bytes.

The first check is:

```asm
cmpl   $0x474d4963,(%rsp)
```

Because the program runs on little-endian x86, this corresponds to the ASCII string:

```
63 49 4d 47  ->  "cIMG"
```

Next:

```asm
cmpw   $0x2,0x4(%rsp)
```

So the 16-bit field at offset 4 must be equal to 2.

Then:

```asm
movzbl 0x6(%rsp),%ebx   ; width
movzbl 0x7(%rsp),%edx   ; height
```

Therefore the header structure is:

```C
struct header {
    char     magic[4];   // "cIMG"
    uint16_t version;    // must be 2
    uint8_t  width;
    uint8_t  height;
};
```

2. Pixel data layout

Immediately after reading the header, the program allocates memory for the pixels:

```asm
imul   %edx,%ebx      ; width * height
shl    $0x2,%rbx      ; * 4
call   malloc
```

So each pixel consists of 4 bytes, and the total size is:

```
width × height × 4 bytes
```

We can treat each pixel as:

```C
struct pixel {
    uint8_t a;
    uint8_t b;
    uint8_t c;
    uint8_t d;
};
```

3. Character constraint

Before calling `display()`, the program checks the fourth byte of every pixel:

```asm
movzbl 0x3(%rbp,%rax,4),%ecx
lea    -0x20(%rcx),%esi
cmp    $0x5e,%sil
```

This enforces:

```
pixel[i].d ∈ [0x20, 0x7e]
```

In other words, the last byte must be a printable ASCII character.

4. What display() actually does

The function is called as:

```C
display(header, pixels);
```

Inside display() we find a call to __snprintf_chk with the following format string:

```
"\033[38;2;%03d;%03d;%03dm%c\033[0m"
```

Which is equivalent to:

```
"\x1b[38;2;%03d;%03d;%03dm%c\x1b[0m"
```

This is a standard ANSI true-color escape sequence:

```
ESC[38;2;R;G;BmCESC[0m
```

By examining how arguments are passed to `snprintf`, the mapping is:

| Format field | Source   |
| ------------ | -------- |
| %03d (R)     | pixel[0] |
| %03d (G)     | pixel[1] |
| %03d (B)     | pixel[2] |
| %c  (char)   | pixel[3] |

So each pixel produces a formatted string of exactly 24 bytes:

```
ESC[38;2;RRR;GGG;BBBmCESC[0m
```

5. Framebuffer layout

Later in main, the program compares the generated framebuffer against a reference:

```asm
mov    $0x18,%edx
call   memcmp
```

`0x18 = 24`, so each framebuffer entry is 24 bytes.

At most four entries are compared.

6. Dumping the reference output

The reference framebuffer is located at:

```
0x404020 <desired_output>
```

Dumping it in GDB:

```bash
x/96bx 0x404020
```

yields four consecutive 24-byte cells.

Each cell has the form:

```
ESC[38;2;RRR;GGG;BBBmCESC[0m
```

We can directly decode the target pixel values from these strings.

Cell 0

```
ESC[38;2;089;157;130mcESC[0m → (89, 157, 130, 'c')
```

Cell 1

```
ESC[38;2;017;078;037mIESC[0m → (17, 78, 37, 'I')
```
Cell 2

```
ESC[38;2;039;225;183mMESC[0m → (39, 225, 183, 'M')
```

Cell 3

```
ESC[38;2;234;109;205mGESC[0m → (234, 109, 205, 'G')
```

7. Avoiding the internal index remapping

`display()` contains a non-trivial index remapping when writing into the framebuffer.

However, if we choose:

```
width  = 4
height = 1
```

the mapping degenerates to a simple linear order.

This allows us to place pixels in the same order as the framebuffer cells without
reversing the remapping logic.

8. Final file format

Header (8 bytes)

```
63 49 4d 47    "cIMG"
02 00          version = 2
04             width  = 4
01             height = 1
```

### Pixel data

Four pixels, in order:

| Index | R   | G   | B   | C   |
| ----: | --- | --- | --- | --- |
|     0 | 89  | 157 | 130 | 'c' |
|     1 | 17  | 78  | 37  | 'I' |
|     2 | 39  | 225 | 183 | 'M' |
|     3 | 234 | 109 | 205 | 'G' |

9. File generator

```python
with open("solve.cimg", "wb") as f:
    # header
    f.write(b"cIMG")
    f.write((2).to_bytes(2, "little"))
    f.write(bytes([4, 1]))   # width = 4, height = 1

    pixels = [
        (89, 157, 130, ord('c')),
        (17, 78,  37,  ord('I')),
        (39, 225, 183, ord('M')),
        (234,109,205, ord('G')),
    ]

    for p in pixels:
        f.write(bytes(p))
```

Run:

```bash
python3 gen.py
/challenge/cimg < solve.cimg
```

This produces the correct internal framebuffer and triggers `win()`.