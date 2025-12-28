# Input Restrictions (x86)

## Challenge Description

The C parser reads a 14-byte header and then validates magic, version, width, and height. The data size is derived from `width * height`.

---

## Key Constraints

- Magic: `b"cIMG"` (4 bytes)
- Version: `1` (little-endian, 4 bytes)
- Width: `79` (little-endian, 2 bytes)
- Height: `24` (little-endian, 4 bytes)
- Data length: `width * height = 1896` bytes
- Data bytes likely must be printable ASCII (common in prior levels)

---

## Exploit Script

```python
import struct

WIDTH = 79
HEIGHT = 24

header = b"cIMG" + struct.pack("<I", 1) + struct.pack("<H", WIDTH) + struct.pack("<I", HEIGHT)
data = b"A" * (WIDTH * HEIGHT)

with open("input_restrictions_c_v2.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 make_input_restrictions_c_v2.py
/challenge/cimg input_restrictions_c_v2.cimg
```
