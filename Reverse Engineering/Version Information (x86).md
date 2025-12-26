# Version Information (x86)

## Challenge Description

This challenge tests your ability to reverse engineer a binary that validates both a magic number and a version number from a .cimg file. The validation is done purely in assembly (x86), and the goal is to understand how the binary checks the file format and version.

## Reversing the Binary

We used gdb to disassemble the binary:

```bash
gdb /challenge/cimg
(gdb) disassemble main
```

### Observations:

From the disassembly, we noted the following comparisons:

```asm
cmpb   $0x28,0xc(%rsp)
cmpb   $0x4e,0xd(%rsp)
cmpb   $0x6d,0xe(%rsp)
cmpb   $0x67,0xf(%rsp)
cmpq   $0x74,0x10(%rsp)
```

This shows the program is reading 5 bytes

Solution Strategy

We wrote a Python script to generate the required binary .cimg file.

### Python Script:

```python
# generate_pass_cimg.py

with open("pass.cimg", "wb") as f:
    f.write(b"\x..\x..\x..\x..\x..")
    f.write(b"\x00" * 7) 
```

Run the script:

```bash
python3 generate_pass_cimg.py
```

After generating the file, we ran:

```bash
/challenge/cimg pass.cimg
```