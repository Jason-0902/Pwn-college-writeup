# File Formats: Directives (x86)

## Challenge Summary
This x86 challenge keeps the same cIMG directive model but changes the directive code and expected framebuffer size. The binary reads a packed header, executes directives, renders the framebuffer, and calls `win()` only if the rendered framebuffer matches `desired_output`.

## Header Layout
The header is 12 bytes:
- Magic: 4 bytes, `cIMG`
- Version: 2 bytes, little-endian, value `3`
- Width: 1 byte
- Height: 1 byte
- Remaining directives: 4 bytes, little-endian

Structure:
```c
struct cimg_header {
    char magic[4];
    uint16_t version;
    uint8_t width;
    uint8_t height;
    uint32_t remaining_directives;
} __attribute__((packed));
```

## Main Checks from Disassembly
Key checks in `main`:
- `cmpl $0x474d4963` -> magic must be `cIMG`
- `cmpw $0x3` -> version must be `3`
- `cmp $0x2174,%cx` -> directive code must be `0x2174` (`8564`)
- `cmp $0x522,%r14d` -> framebuffer pixel count must be `0x522` (`1314`)

## Directive Logic
Each directive starts with a 2-byte code:
- Required code: `8564`
- File encoding: little-endian `74 21`

Only this code reaches the handler:
```c
case 8564:
    handle_8564(&cimg);
    break;
```

## Validation Rules
After rendering:
1. `num_pixels` must equal `sizeof(desired_output) / 24`
2. `str.c` must match for each entry
3. If char is not space and not newline, full 24-byte `memcmp` must match

So the solve strategy is reconstructive, not guesswork.

## Reconstruction Strategy
Parse `desired_output` from `/challenge/cimg.c`, remove trailing NUL, split into 24-byte terminal cells, and recover each pixel:
- `r = int(chunk[7:10])`
- `g = int(chunk[11:14])`
- `b = int(chunk[15:18])`
- `ascii = chunk[19]`

Then write:
- Header (`<4sHBBI`)
- One directive (`<H`, value `8564`)
- Reconstructed pixel bytes (`r,g,b,ascii`)

## Build the File
```python
#!/usr/bin/env python3
import re
import struct

SRC = "/challenge/cimg.c"
OUT = "payload.cimg"


def extract_desired_output_bytes(src_path: str) -> bytes:
    with open(src_path, "r", encoding="latin1") as f:
        src = f.read()

    m = re.search(r'char desired_output\[\]\s*=\s*"((?:\\.|[^"])*)";', src, re.S)
    if not m:
        raise RuntimeError("desired_output string not found")

    s = bytes(m.group(1), "utf-8").decode("unicode_escape").encode("latin1")
    if s.endswith(b"\x00"):
        s = s[:-1]

    if len(s) % 24 != 0:
        raise RuntimeError(f"desired_output length is not a multiple of 24: {len(s)}")

    return s


def pick_dimensions(pixels: int) -> tuple[int, int]:
    for h in range(255, 0, -1):
        if pixels % h == 0:
            w = pixels // h
            if 1 <= w <= 255:
                return w, h
    raise RuntimeError(f"cannot fit pixel count into uint8 dimensions: {pixels}")


def main():
    desired = extract_desired_output_bytes(SRC)
    pixels = len(desired) // 24

    width, height = pick_dimensions(pixels)
    if width * height != pixels:
        raise RuntimeError(f"width*height != pixels ({width}*{height} != {pixels})")

    header = struct.pack("<4sHBBI", b"cIMG", 3, width, height, 1)
    directive = struct.pack("<H", 8564)

    pixel_data = bytearray()
    for i in range(pixels):
        chunk = desired[i * 24 : (i + 1) * 24]
        r = int(chunk[7:10].decode("ascii"))
        g = int(chunk[11:14].decode("ascii"))
        b = int(chunk[15:18].decode("ascii"))
        a = chunk[19]
        pixel_data.extend(bytes([r, g, b, a]))

    with open(OUT, "wb") as f:
        f.write(header)
        f.write(directive)
        f.write(pixel_data)

    print(f"[+] wrote {OUT}")
    print(f"[+] pixels={pixels}, width={width}, height={height}, data_len={len(pixel_data)}")


if __name__ == "__main__":
    main()
```

## Run
```bash
python3 payload.py
/challenge/cimg ./payload.cimg
```

## Troubleshooting
- `ERROR: Invalid magic number!`: first 4 bytes must be `63 49 4d 47`
- `ERROR: Unsupported version!`: version must be `03 00`
- `ERROR: invalid directive_code`: directive must be `74 21`
- `ERROR: Failed to read data!`: data must be exactly `width * height * 4`
- No flag: ensure trailing NUL was removed before splitting into 24-byte cells
