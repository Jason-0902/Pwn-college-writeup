# Read Endianness(Python)

## Challenge Overview

In this challenge, we are given a binary located at:

```
/challenge/cimg
```

The binary expects a `.cimg` file and checks for a magic number at the beginning of the file. However, unlike typical ELF binaries, this one does not work with tools like gdb or ltrace.

The challenge hints that the magic number is checked using Python and involves endianness handling — hence the name: 

```
reading-endianness-python.
```

## Initial Observations

Attempting to run tools like gdb or ltrace returns:

not in executable format: file format not recognized


Using xxd to inspect the binary reveals:

```bash
xxd /challenge/cimg | head -n 20
```

Output:

```
00000000: 2321 2f75 7372 2f62 696e 2f65 7865 632d  #!/usr/bin/exec-
00000010: 7375 6964 202d 2d20 2f75 7372 2f62 696e  suid -- /usr/bin
00000020: 2f70 7974 686f 6e33 202d 490a            /python3 -I
...
```

This reveals that `/challenge/cimg` is not a compiled binary, but actually a Python script executed through a suid exec wrapper.

## Strategy

Since the script is plain-text Python, we extracted it using:

```bash
cat /challenge/cimg > cimg_dump.py
```

Reviewing the source code, we found the following logic:

```Python
file = open(path, "rb")
header = file.read1(4)
assert len(header) == 4, "ERROR: Failed to read header!"
assert int.from_bytes(header[:4], "little") == 0x366D3A7B, "ERROR: Invalid magic number!"
```

This means the file:

Reads the first 4 bytes

Interprets them as a little-endian integer

Compares that value to `0x366D3A7B`

## Decoding the Magic Number

We need to convert 0x366D3A7B into a 4-byte little-endian sequenc

## Exploit / Solution

Create the .cimg file with the correct magic number

Then run the program:

```bash
/challenge/cimg flag.cimg
```