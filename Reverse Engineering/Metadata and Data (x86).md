# Metadata and Data (x86)

## Challenge Description

Craft a `.cimg` file with correct magic, version, dimensions, and enough data to satisfy the parser and print the flag.

## Findings
- Magic: `"(Nnr"` hex `28 4e 6e 72`
- Version: `1` (little-endian 2 bytes: `01 00`)
- Width: `64` (little-endian 4 bytes: `40 00 00 00`)
- Height: `12` (little-endian 4 bytes: `0c 00 00 00`)
- Data length: `width * height = 768` bytes

## Exploit Script

```python
magic = b"(Nnr"
version = (1).to_bytes(2, "little")
width = (64).to_bytes(4, "little")
height = (12).to_bytes(4, "little")
data = b"A" * (64 * 12)

with open("flag.cimg", "wb") as f:
    f.write(magic)
    f.write(version)
    f.write(width)
    f.write(height)
    f.write(data)
```

Run it:

```bash
python3 make_cimg.py
/challenge/cimg flag.cimg
```
