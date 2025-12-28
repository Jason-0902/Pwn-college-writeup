# Behold the cIMG (C)

## Challenge Description

The C version validates a 14-byte header, displays the framebuffer, and only calls `win()` if the number of non-space characters equals 275.

---

## Key Constraints

- Magic: `b"cIMG"` (4 bytes)
- Version: `1` (little-endian, 4 bytes)
- Width: 4 bytes (little-endian)
- Height: 2 bytes (little-endian)
- Data length: `width * height`
- All data bytes must be printable ASCII (`0x20` to `0x7E`)
- `nonspace_count` must be `275`

---

## Exploit Script

```python
import struct

WIDTH = 25
HEIGHT = 11

header = b"cIMG" + struct.pack("<I", 1) + struct.pack("<I", WIDTH) + struct.pack("<H", HEIGHT)
data = b"#" * (WIDTH * HEIGHT)

with open("behold_cimg_c.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 src/behold_cimg_c.py
/challenge/cimg behold_cimg_c.cimg
```
