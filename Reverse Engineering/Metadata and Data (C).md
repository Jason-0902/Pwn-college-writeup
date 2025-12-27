# Metadata and Data (C)

## Challenge Description

Craft a `.cimg` file with correct magic, version, dimensions, and enough data to satisfy the parser and print the flag.

## Findings
- Magic: `"[M4G"` hex `5b 4d 34 47`
- Version: `1` (little-endian 8 bytes: `01 00 00 00 00 00 00 00`)
- Width: `40` (little-endian 2 bytes: `28 00`)
- Height: `14` (1 byte: `0e`)
- Data length: `width * height = 560` bytes

## Exploit Script

```python
magic = b"[M4G"
version = (1).to_bytes(8, "little")
width = (40).to_bytes(2, "little")
height = (14).to_bytes(1, "little")
data = b"A" * (40 * 14)

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
