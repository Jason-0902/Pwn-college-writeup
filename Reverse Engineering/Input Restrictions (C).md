# Input Restrictions (C)

## Challenge Description

The C version of `/challenge/cimg` validates a 9-byte header, fixed dimensions, and restricts pixel data to printable ASCII.

---

## Key Constraints

- Magic: `b"cIMG"` (4 bytes)
- Version: `1` (1 byte)
- Width: `71` (little-endian, 2 bytes)
- Height: `21` (little-endian, 2 bytes)
- Data length: `width * height = 1491` bytes
- Data bytes must be printable ASCII: `0x20` to `0x7E`

---

## Exploit Script

```python
import struct

WIDTH = 71
HEIGHT = 21

header = b"cIMG" + bytes([1]) + struct.pack("<H", WIDTH) + struct.pack("<H", HEIGHT)
data = b"A" * (WIDTH * HEIGHT)

with open("input_restrictions_c.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 make_input_restrictions_c.py
/challenge/cimg input_restrictions_c.cimg
```
