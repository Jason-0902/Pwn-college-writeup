# Input Restrictions (Python)

## Challenge Description

The `/challenge/cimg` parser enforces strict input constraints. Besides magic and version, it requires fixed dimensions and restricts the data bytes to printable ASCII.

---

## Key Constraints

- Magic: `b"cIMG"`
- Version: `1` (1 byte)
- Width: `50` (little-endian, 8 bytes)
- Height: `11` (little-endian, 4 bytes)
- Data length: `width * height = 550` bytes
- Data bytes must be printable ASCII: `0x20` to `0x7E`

---

## Exploit Script

```python
import struct

WIDTH = 50
HEIGHT = 11

header = b"cIMG" + bytes([1]) + struct.pack("<Q", WIDTH) + struct.pack("<I", HEIGHT)
data = b"A" * (WIDTH * HEIGHT)

with open("input_restrictions.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

---

## Run

```bash
python3 make_input_restrictions.py
/challenge/cimg input_restrictions.cimg
```
