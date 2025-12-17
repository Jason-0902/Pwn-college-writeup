# Read Endianness(x86)

## Challenge Description

In this challenge, we are given a binary located at: `/challenge/cimg`

## Tools Used

- `gdb` – for disassembly and reverse engineering
- `echo -ne` – to create binary files
- Python (optional) – to convert integers to little-endian byte sequences

##  Step-by-Step Solution

### 1. Disassembling the Binary

We open the binary in gdb:

```bash
gdb /challenge/cimg
```

Inside gdb, we run:

```bash
set args test.cimg
break main
run
disassemble main
```

### 2. Locating the Magic Number Check

In the disassembly output, we find the key instruction:

```asm
cmpl $0x726e6f28, 0x4(%rsp)
```

This tells us:

The program reads 4 bytes into memory at `[rsp+0x4]`

It compares that value against the constant 

If the comparison succeeds, execution continues to the win() function, which prints the flag

### 4. Handling Endianness

Because the architecture is little-endian, the value must be written to the file in little-endian byte order.

### 5. Crafting the .cimg File

```bash
echo -ne 'Hidden value' > flag.cimg
```

### 6. Retrieving the Flag

Finally, run the program with the crafted file:

```bash
/challenge/cimg flag.cimg
```

