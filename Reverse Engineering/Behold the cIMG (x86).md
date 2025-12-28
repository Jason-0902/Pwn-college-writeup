# Behold the cIMG (x86)

## Challenge Description

The x86 version reads a 28-byte header, displays the framebuffer, and only calls `win()` if the number of non-space characters equals 275.

---

## Key Constraints

- Magic: `b"cIMG"` (4 bytes)
- Version: `1` (little-endian, 8 bytes)
- Width: 8 bytes (little-endian)
- Height: 8 bytes (little-endian)
- Data length: `width * height`
- All data bytes must be printable ASCII (`0x20` to `0x7E`)
- `nonspace_count` must be `275`

---

## Exploit Script

```python
import struct

WIDTH = 25
HEIGHT = 11

header = b"cIMG" + struct.pack("<Q", 1) + struct.pack("<Q", WIDTH) + struct.pack("<Q", HEIGHT)
data = b"#" * (WIDTH * HEIGHT)

with open("behold_cimg_x86.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 src/behold_cimg_x86.py
/challenge/cimg behold_cimg_x86.cimg
```
