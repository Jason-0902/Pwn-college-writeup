# A Basic cIMG (x86)

## Challenge Summary
The binary expects a fixed 16-byte header and a full RGBA framebuffer. It prints the image, then calls `win()` only if all pixels are ASU maroon and every ASCII byte is non-space printable.

## Header Layout
- Magic: 4 bytes, `cIMG`
- Version: 8 bytes, little-endian, value `2`
- Width: 2 bytes, little-endian, value `46`
- Height: 2 bytes, little-endian, value `23`

Data length must be `46 * 23 * 4 = 4232` bytes, arranged as RGBA per pixel.

## Data Constraints
- ASCII byte must be printable: `0x20` to `0x7e`
- Non-space count must be `1058` (so every pixel must be non-space)
- Each pixel RGB must be `(0x8C, 0x1D, 0x40)`

## Build the File
```python
import struct

WIDTH = 46
HEIGHT = 23
VERSION = 2

header = b"cIMG" + struct.pack("<QHH", VERSION, WIDTH, HEIGHT)

# r, g, b, ascii (non-space printable)
pixel = bytes([0x8C, 0x1D, 0x40, ord("A")])

data = pixel * (WIDTH * HEIGHT)

with open("payload.cimg", "wb") as f:
    f.write(header)
    f.write(data)
```

## Run
```bash
python3 src/generate_a_basic_cimg_payload.py
/challenge/cimg payload.cimg
```

## Troubleshooting
- `Failed to read header`: file is too short (needs 16 bytes header).
- `Invalid version`: ensure `VERSION=2` and 8-byte little-endian.
- `Invalid character`: ASCII byte must be `0x20..0x7e` and not space.
- If no flag prints, confirm every pixel is maroon and non-space.
