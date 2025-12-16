# File Formats & Magic Numbers (x86 Version)

##　Challenge Description

We are given a binary located at /challenge/cimg, which expects a file with a .cimg extension as input. The program validates the input file based on a specific magic number — a set of bytes at the beginning of the file that identifies the file format.

Our goal is to reverse engineer the binary, determine the expected magic number, and create a valid .cimg file that passes the check and reveals the flag.

## Tools Used

* ltrace

* gdb (with TUI/CLI mode)

* xxd

* Bash utilities (echo, python3)

## Step-by-Step Solution

1. Initial Recon

We tested the binary with a dummy file:

```bash
touch test.cimg
/challenge/cimg test.cimg
```

This returned:

```bash
ERROR: Invalid magic number!
```

So the binary is clearly checking for a specific signature at the start of the file.

2. Using ltrace to Inspect File Operations

We traced the program’s library calls:

```bash
ltrace /challenge/cimg test.cimg
```

Output:
```python
open("test.cimg", 0, 00)  = 3
dup2(3, 0)                = 0
read(0, "", 4)            = 4
puts("ERROR: Invalid magic number!") = ...
```

This confirmed that:

* The program opens the file,

* Redirects it to standard input,

* Reads the first 4 bytes,

* Then validates those bytes.

3. Analyzing with GDB

We launched the binary in GDB to inspect the validation logic:

```bash
gdb /challenge/cimg
```

Set a breakpoint:

```bash
set args test.cimg
break main
run
```

Then we disassembled main:

disassemble main

We found the following magic number check:

```asm
movzbl -0x1c(%rbp),%eax
cmp    $0x5b,%al     ; '['
movzbl -0x1b(%rbp),%eax
cmp    $0x4f,%al     ; 'O'
movzbl -0x1a(%rbp),%eax
cmp    $0x4e,%al     ; 'N'
movzbl -0x19(%rbp),%eax
cmp    $0x72,%al     ; 'r'
```

This indicates that the program expects the first 4 bytes of the file

4. Crafting a Valid .cimg File

5. Getting the Flag

Finally, we ran the program with the valid file:

```bash
/challenge/cimg flag.cimg
```