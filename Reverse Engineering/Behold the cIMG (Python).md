# Behold the cIMG (Python)

## Challenge Description

The program reads a `.cimg` file, prints the framebuffer, and only reveals the flag if the number of non-space characters is exactly 275.

---

## Key Constraints

- Magic: `b"cIMG"` (4 bytes)
- Version: `1` (little-endian, 2 bytes)
- Width: 4 bytes (little-endian)
- Height: 1 byte
- Data length: `width * height`
- All data bytes must be printable ASCII (`0x20` to `0x7E`)
- `nonspace_count` must equal `275`

---

## Exploit Script

```python
import struct

WIDTH = 25
HEIGHT = 11

header = b"cIMG" + struct.pack("<H", 1) + struct.pack("<I", WIDTH) + struct.pack("<B", HEIGHT)
data = b"#" * 275

with open("behold_cimg_python.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 src/behold_cimg_python.py
/challenge/cimg behold_cimg_python.cimg
```
