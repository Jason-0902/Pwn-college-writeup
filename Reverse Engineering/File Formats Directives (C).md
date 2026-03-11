# File Formats: Directives (C)

## Challenge Summary
This challenge introduces directive-driven parsing in the cIMG format. The binary reads a packed header, executes a sequence of directives, renders a framebuffer, and calls `win()` only when the final framebuffer matches `desired_output` under strict comparison rules.

## Header Layout
The binary reads 12 bytes into a packed header:
- Magic: 4 bytes, `cIMG`
- Version: 2 bytes, little-endian, value `3`
- Width: 1 byte
- Height: 1 byte
- Remaining directives: 4 bytes, little-endian

Struct model:
```c
struct cimg_header {
    char magic[4];
    uint16_t version;
    uint8_t width;
    uint8_t height;
    uint32_t remaining_directives;
} __attribute__((packed));
```

## Directive Logic
In `main`, each directive begins with a 2-byte code:
- Valid code: `17571` (`0x44a3`)
- Encoding in file: little-endian `a3 44`

Only this code is accepted:
```c
switch (directive_code) {
    case 17571:
        handle_17571(&cimg);
        break;
    default:
        exit(-1);
}
```

## Pixel Format and Validation
`handle_17571` reads `width * height * 4` bytes where each pixel is:
- `r` (1 byte)
- `g` (1 byte)
- `b` (1 byte)
- `ascii` (1 byte)

The ASCII byte must be printable: `0x20..0x7e`.

Each pixel is transformed into a fixed 24-byte terminal string:
`\x1b[38;2;%03d;%03d;%03dm%c\x1b[0m`

## Win Condition
After all directives:
1. `num_pixels` must equal `sizeof(desired_output) / 24`
2. For each pixel entry, `str.c` must match
3. If `str.c` is not space and not newline, full 24-byte entry must match

From disassembly:
- `cmp $0x339, %r14d` means required pixel count is `0x339 = 825`

So dimensions must satisfy:
`width * height = 825`

A valid choice is:
- `width = 55`
- `height = 15`

## Reconstruction Strategy
Do not guess the image manually. Parse `desired_output` from source, split into 24-byte cells, and recover each target pixel as:
- `r = int(chunk[7:10])`
- `g = int(chunk[11:14])`
- `b = int(chunk[15:18])`
- `ascii = chunk[19]`

Then build a `.cimg` file with:
- Correct header
- One directive (`17571`)
- Reconstructed pixel stream

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


def main():
    desired = extract_desired_output_bytes(SRC)
    pixels = len(desired) // 24

    width, height = 55, 15
    if width * height != pixels:
        raise RuntimeError(f"width*height != pixels ({width}*{height} != {pixels})")

    header = struct.pack("<4sHBBI", b"cIMG", 3, width, height, 1)
    directive = struct.pack("<H", 17571)

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
    print(f"[+] pixels={pixels}, data_len={len(pixel_data)}")


if __name__ == "__main__":
    main()
```

## Run
```bash
python3 payload.py
/challenge/cimg ./payload.cimg
```

## Troubleshooting
- `ERROR: Invalid magic number!`: verify first 4 bytes are exactly `63 49 4d 47`
- `ERROR: Unsupported version!`: version must be `03 00`
- `ERROR: invalid directive_code`: directive bytes must be `a3 44`
- `ERROR: Failed to read data!`: data length must be exactly `width * height * 4`
- No flag after render: check that you removed the trailing NUL from `desired_output` before 24-byte chunking
