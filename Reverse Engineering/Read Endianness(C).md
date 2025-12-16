# Read Endianness(C)

## Challenge Description

We are given a binary located at:


The goal is to reverse engineer the binary and craft a `.cimg` file with the correct **magic number** at the beginning, such that the binary accepts the file and prints the flag.

The twist this time: the binary is written in **C**, and the magic number is compared as a **little-endian integer**, just like in real-world file parsers.

---

## Tools Used

- `gdb` – for disassembling and analyzing the binary
- `xxd` – to inspect the binary content
- `echo -ne` – to craft the `.cimg` file
- Python (optional) – for converting integers to little-endian byte sequences

---

## Step-by-Step Solution

### 1. Triggering the Magic Number Check

We begin by creating a dummy `.cimg` file and testing the binary:

```bash
echo -ne '\x00\x00\x00\x00' > test.cimg
/challenge/cimg test.cimg
```

Output:

```
ERROR: Invalid magic number!
```

So the binary is clearly checking the first 4 bytes.

### 2. Disassembling the Binary

We load the binary in gdb:

gdb /challenge/cimg


Inside gdb, we run:

```bash
set args test.cimg
break main
run
disassemble main
```

We find the following key instruction:

```asm
0x4012d4 <main+144>: cmpl $0x366d6e7b, 0x4(%rsp)
```

This tells us:

* The program reads 4 bytes into `[rsp+0x4]`

* Then compares that value against `0x366D6E7B`

* If equal, it jumps to a call to win() and prints the flag

### 3. Convert Magic Number to Little-Endian Bytes

Since the binary compares the number in memory, and x86 is little-endian, we must write the little-endian form of the number into the file

### 4. Craft the .cimg File

```bash
echo -ne '????????' > flag.cimg
xxd flag.cimg
```

### 5. Run the Binary and Capture the Flag

```bash
/challenge/cimg flag.cimg
```

You should now receive the flag