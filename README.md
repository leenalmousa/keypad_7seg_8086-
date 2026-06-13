# 4×3 Keypad → 7-Segment Display (8086 + 8255 PPI)

An 8086 assembly project that scans a **4×3 matrix keypad** through the **8255 Programmable Peripheral Interface (PPI)** and drives a **common-cathode 7-segment display** to show the pressed digit. The 8255 is address-decoded with a **74LS138** so it sits at a fixed I/O range, and the whole system is simulated in **Proteus**.

[![Assembly](https://img.shields.io/badge/Language-8086%20Assembly-525252?logo=assemblyscript&logoColor=white)](Assembly_code.asm)
[![Assembler](https://img.shields.io/badge/Assembler-MASM32-7B68EE)](https://en.wikipedia.org/wiki/Microsoft_Macro_Assembler)
[![Simulation](https://img.shields.io/badge/Simulator-Proteus-1E6FBA)](https://www.labcenter.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

![Circuit schematic](https://github.com/user-attachments/assets/2145c6f0-a91e-4649-af0a-8187518102bc)

## Hardware

| Component        | Role |
|------------------|------|
| **Intel 8086**   | Sends scan codes to columns and reads back rows; drives the 7-segment pattern |
| **8255 PPI**     | Port A → display, Port B → keypad columns (output), Port C lower → keypad rows (input) |
| **74LS138 (3→8 decoder)** | Address-decodes the 8255 so its ports map to fixed I/O addresses |
| **4×3 matrix keypad** | 3 columns driven, 4 rows sensed |
| **Common-cathode 7-segment display** | Driven directly by Port A |

### 8255 port map

The 74LS138 places the 8255 at I/O addresses `68h` and above:

| Port           | Address | Direction | Connected to                     |
|----------------|---------|-----------|----------------------------------|
| Port A         | `68h`   | output    | 7-segment cathodes (a..g)        |
| Port B         | `6Ah`   | output    | Keypad column drives (one-hot)   |
| Port C (lower) | `6Ch`   | input     | Keypad row sense lines (PC0..PC3)|
| Control word   | `6Eh`   | —         | `88h` — A out, B out, C upper in |

## How it works

1. **Init** &mdash; write `88h` to the 8255 control register (Mode 0; Port A output, Port B output, Port C upper input).
2. **Scan loop** &mdash; for each column 1..3, drive a one-hot pattern on Port B (`100b`, `010b`, `001b`), then read Port C and mask the low nibble. A non-zero result means a key in that column is pressed.
3. **Resolve the key** &mdash; jump to per-column handler (`column1` / `column2` / `column3`), which compares the row pattern (`0001b`, `0010b`, `0100b`, `1000b`) to identify the exact row → digit.
4. **Display** &mdash; write the matching 7-segment pattern to Port A and resume scanning.

### Keypad → 7-segment pattern map

| Key | Column drive (PB) | Row sense (PC) | Pattern on PA (gfedcba) | Hex   | Shown |
|-----|-------------------|----------------|-------------------------|-------|-------|
| 1   | `100b`            | `0001b`        | `0000110`               | `06h` | `1`   |
| 4   | `100b`            | `0010b`        | `1100110`               | `66h` | `4`   |
| 7   | `100b`            | `0100b`        | `0000111`               | `07h` | `7`   |
| 2   | `010b`            | `0001b`        | `1011011`               | `5Bh` | `2`   |
| 5   | `010b`            | `0010b`        | `1101101`               | `6Dh` | `5`   |
| 8   | `010b`            | `0100b`        | `1111111`               | `7Fh` | `8`   |
| 0   | `010b`            | `1000b`        | `0111111`               | `3Fh` | `0`   |
| 3   | `001b`            | `0001b`        | `1001111`               | `4Fh` | `3`   |
| 6   | `001b`            | `0010b`        | `1111101`               | `7Dh` | `6`   |
| 9   | `001b`            | `0100b`        | `1101111`               | `6Fh` | `9`   |

> The keypad's `*` and `#` positions are intentionally unmapped &mdash; the scanner falls through and continues polling.

## Files

```
.
├── Assembly_code.asm        # 8086 source (MASM32 syntax)
├── Hardware_project.pdsprj  # Proteus simulation (8086 + 8255 + 74LS138 + keypad + display)
├── Report.docx              # Written report
├── README.md
└── LICENSE
```

## Run the simulation

1. Install [Proteus](https://www.labcenter.com/) (8.x).
2. Open `Hardware_project.pdsprj` &mdash; the assembled firmware is already bundled inside the project (Proteus stores it as `8086/Debug/Debug.exe`).
3. Hit ▶ **Run** and click keypad buttons &mdash; the digit appears on the 7-segment display.

> Want to rebuild from source? Install MASM32, then open the source pane in Proteus VSM Studio and edit `main.asm` (or just replace the bundled file with `Assembly_code.asm` from the repo root &mdash; they are identical). Proteus' 8086 model also requires **Internal Memory Size** = `0x10000` in its properties.

## Course context

Built for the Microprocessors course at PSUT. Demonstrates I/O port handling on the 8086, address decoding with the 74LS138, matrix-keypad scanning, and 7-segment encoding; without using any library calls.

## Author

**Leen Almousa** &mdash; [github.com/leenalmousa](https://github.com/leenalmousa)

## License

Released under the [MIT License](LICENSE).
