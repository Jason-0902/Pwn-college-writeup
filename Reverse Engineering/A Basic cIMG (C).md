# A Basic cIMG (C)

## Challenge Summary
The binary expects a fixed 18-byte header and a full RGBA framebuffer. It prints the image, then calls `win()` only if all pixels are ASU maroon and every ASCII byte is non-space printable.

## Header Layout
- Magic: 4 bytes, `cIMG`
- Version: 4 bytes, little-endian, value `2`
- Width: 8 bytes, little-endian, value `73`
- Height: 2 bytes, little-endian, value `20`

Data length must be `73 * 20 * 4 = 5840` bytes, arranged as RGBA per pixel.

## Data Constraints
- ASCII byte must be printable: `0x20` to `0x7e`
- Non-space count must be `1460` (so every pixel must be non-space)
- Each pixel RGB must be `(0x8C, 0x1D, 0x40)`

## Build the File
```python
import struct

WIDTH = 73
HEIGHT = 20
VERSION = 2

header = b"cIMG" + struct.pack("<IQH", VERSION, WIDTH, HEIGHT)

# r, g, b, ascii (non-space printable)
pixel = bytes([0x8C, 0x1D, 0x40, ord("A")])

data = pixel * (WIDTH * HEIGHT)

with open("a_basic_cimg_c.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

## Run
```bash
python3 src/generate_a_basic_cimg_c.py
/challenge/cimg a_basic_cimg_c.cimg
```

## Troubleshooting
- `Failed to read header`: file is too short (needs 18 bytes header).
- `Invalid version`: ensure `VERSION=2` and 4-byte little-endian.
- `Invalid character`: ASCII byte must be `0x20..0x7e` and not space.
- If no flag prints, confirm every pixel is maroon and non-space.
