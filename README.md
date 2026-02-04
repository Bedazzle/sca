# Screen Animation Format (.sca)

## General Description

The `.sca` format is designed for storing frame-based animations on the ZX Spectrum. The base format (type 0) is a sequence of uncompressed full-screen frames, each 6912 bytes in size. Frame delays are specified in units of vertical interrupts (1/50 second). The file consists of a header, a delay table, and frame data.

**Byte order:** little-endian (least significant byte first).

---

## Header Structure

| Offset   | Size    | Description |
|----------|---------|-------------|
| 00–02    | 3 bytes | Marker: ASCII string `"SCA"` |
| 03       | 1 byte  | Version (format version) |
| 04–05    | 2 bytes | Frame width in pixels |
| 06–07    | 2 bytes | Frame height in pixels (max 192) |
| 08       | 1 byte  | Recommended border color |
| 09–10    | 2 bytes | Frame count |
| 11       | 1 byte  | Payload type |
| 12–13    | 2 bytes | Payload offset (from start of file) |

---

## Payload

The structure of the payload depends on the specified type.

### Payload Type 0 – Unpacked 6912-byte Screens

#### Delay Table

Contains `N` bytes, where `N` is the number of frames.  
Each byte represents the delay for the corresponding frame in units of vertical interrupts (1/50 second).

#### Frames

Each frame is 6912 bytes of ZX Spectrum screen memory in `SCREEN$` format (bitmap + attributes).  
Frames are stored sequentially with no padding.

---

### Payload Type 1 – Multicolor Attribute Screens (53c/127c)

This payload type is designed for multicolor modes where the bitmap pattern is constant across all cells, and only attributes change per frame. The visual illusion of additional colors is achieved through dithering patterns.

#### Delay Table

Contains `N` bytes, where `N` is the number of frames.
Each byte represents the delay for the corresponding frame in units of vertical interrupts (1/50 second).

#### Fill Pattern

8 bytes defining the bitmap pattern for each character cell.
This pattern is repeated for all 768 cells on screen (32×24).

Common patterns:
- **53c (checkerboard):** `AA 55 AA 55 AA 55 AA 55` — 50% pixels set, creates ~53 perceived colors
- **127c:** `DD 77 DD 77 DD 77 DD 77` — unequal pixel distribution for ~127 perceived colors

Here's a visual representation of the 8x8 cell patterns:

  53c Checkerboard (AA 55 alternating):

  AA = 10101010  ▓░▓░▓░▓░
  55 = 01010101  ░▓░▓░▓░▓
  AA = 10101010  ▓░▓░▓░▓░
  55 = 01010101  ░▓░▓░▓░▓
  AA = 10101010  ▓░▓░▓░▓░
  55 = 01010101  ░▓░▓░▓░▓
  AA = 10101010  ▓░▓░▓░▓░
  55 = 01010101  ░▓░▓░▓░▓
  50% ink, 50% paper → visually blends colors equally

  127c Pattern (DD 77 alternating):

  DD = 11011101  ▓▓░▓▓▓░▓
  77 = 01110111  ░▓▓▓░▓▓▓
  DD = 11011101  ▓▓░▓▓▓░▓
  77 = 01110111  ░▓▓▓░▓▓▓
  DD = 11011101  ▓▓░▓▓▓░▓
  77 = 01110111  ░▓▓▓░▓▓▓
  DD = 11011101  ▓▓░▓▓▓░▓
  77 = 01110111  ░▓▓▓░▓▓▓
  75% ink, 25% paper → color leans toward ink

  With 127c you'd use both DD/77 (75% ink) and its inverse 22/88 (25% ink) patterns across different cells to get more color gradations. The 8-byte fill field allows any custom pattern.


#### Frames

Each frame is 768 bytes of ZX Spectrum attribute data only.
Frames are stored sequentially with no padding.

---
