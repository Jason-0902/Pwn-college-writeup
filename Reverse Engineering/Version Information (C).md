# Version Information (C)

## Challenge Description

This challenge continues from the previous Python-based version check, but now the cIMG parser is implemented in C. We are required to:

Provide the correct magic number

Specify a version field

Format the input properly in a `.cimg` file

Understand how these values are stored and checked in a compiled C binary


## Reversing the Binary

Using gdb, we disassembled the binary and found the following logic in the `main()` function:

### Step 1: Reading the Header

```asm
mov $0x6, %edx
lea 0x2(%rsp), %rsi
call read_exact
```

The program reads 6 bytes from input, starting at `rsp+2`.

### Step 2: Magic Number Check

```asm
cmpb $0x63, 0x2(%rsp)   ; 'c'
cmpb $0x6d, 0x3(%rsp)   ; 'm'
cmpb $0x36, 0x4(%rsp)   ; '6'
cmpb $0x65, 0x5(%rsp)   ; 'e'
```

This verifies the 4-byte magic number is:

````
"????"
````

### Step 3: Version Number Check

```asm
cmpw $0x87, 0x6(%rsp)
```

`cmpw` compares 2 bytes (a 16-bit word).

The expected version value is `0x87` (decimal 135).

Since this is on x86, it's little-endian, so:

Required bytes = `\x??\x??`

### Crafting the Input File

We need to create a file with the following 6-byte header:

```
Offset	Value
0x00	"????"
0x04	\x??\x??
```

### Python Script

```Python
with open("flag.cimg", "wb") as f:
    f.write(b"????")      # Magic number
    f.write(b"\x??\x??")  # Version number (135)
```

Run it:

```bash
python3 cimg.py
/challenge/cimg flag.cimg
```

You'll get the flag