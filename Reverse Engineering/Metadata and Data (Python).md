# Metadata and Data (Python)

## Challenge Description

Craft a `.cimg` file with correct magic, version, dimensions, and enough data to satisfy the parser and print the flag.

## Findings
- Magic: `....` → hex `.. .. .. ..`
- Version: `..` (little-endian `.. ..`)
- Width: `..` (little-endian `.. .. .. ..`)
- Height: `..` (little-endian `.. .. .. ..`)
- Data length: `width * height = ..` bytes

## Exploit Script

```python
magic = b"...."
version = (..).to_bytes(2, "little")
width = (..).to_bytes(4, "little")
height = (..).to_bytes(4, "little")
data = b".." * (64 * 12)  # adjust if Pixel() rejects some bytes

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