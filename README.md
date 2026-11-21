## K.R. Mangalam University
# School of Engineering & Technology
# Nand to Tetris I — Capstone Project
## Make the Computer & Draw a Box

**Course code:** ETCCNT301  
**Submitted by:** Gauri Sharma  
**Roll number:** 2501730108  
**K value:** 08 (last two digits of roll number)

> This project builds the Hack computer, implements a Python assembler, and draws a programmable box on the Hack screen.

## Project Requirements

| Specification | Value |
|---|---|
| Box height | 20 rows |
| Box width | 16 pixels (1 screen word) |
| Starting row | 8 |
| Starting position | Left edge of the screen |
| Assembler extra feature | Duplicate-label warning (`K mod 3 = 2`) |

## Personal Values Derived from K

- **Starting row:** `K = 8`, so the box covers rows **8–27**.
- **Width:** `16 × ((8 mod 4) + 1) = 16 × 1 = 16 pixels`, exactly one 16-bit screen word.
- **Start address:** `SCREEN + (32 × 8) = 16384 + 256 = 16640`.
- **Last row address:** `16640 + (32 × 19) = 17248`.
- **Assembler feature:** `8 mod 3 = 2`, so the extra feature is a warning for duplicate labels.

## 1. Building the Computer

I constructed the CPU, Memory, and top-level Computer chips using the hardware from the Nand to Tetris lab projects.

### CPU

- The A-register input is selected between the ALU output and the instruction using the **A-instruction** bit.
- The ALU’s second input is selected between the A-register and `inM` using the **`a` bit**.
- Control signals determine whether the ALU output is written to the A-register, D-register, or RAM.
- Jump control uses the ALU status outputs and the instruction’s jump bits.

### Memory

The 15-bit address is divided by its top two bits:

| Address range | Hardware component |
|---|---|
| `0–16383` | RAM16K |
| `16384–24575` | Screen memory map |
| `24576` | Keyboard memory map |

### Computer

The top-level Computer chip connects:

- `ROM32K` for instructions
- `CPU` for execution
- `Memory` for RAM, screen, and keyboard access

The CPU’s `outM`, `writeM`, and `addressM` outputs are connected to the Memory chip, while the CPU’s `pc` output provides the ROM address.

### Test Results

#### CPU

**Figure 1.** `CPU.hdl` passes the CPU test; the generated output is identical to the compare file.

#### Memory

**Figure 2.** `Memory.hdl` passes the Memory test for RAM, Screen, and Keyboard addressing.

#### Computer

**Figure 3.** `Computer.hdl` runs the Add program successfully.

**Figure 4.** `Computer.hdl` runs the Max program successfully.

**Figure 5.** `Computer.hdl` runs the Rect program successfully.

## 2. Python Assembler

The assembler translates Hack assembly source into 16-bit machine code using a two-pass process.

### Pass 1 — Labels

- Removes comments and blank lines.
- Records every label, such as `(LOOP)`, with the ROM address of the next instruction.
- Keeps track of the instruction address separately from the source line number.

### Pass 2 — Translation

- **A-instructions** are converted to a `0` followed by a 15-bit address or variable value.
- **C-instructions** are converted into 16-bit binary instructions.
- New variables begin at RAM address `16` and increase for each newly encountered variable.
- Predefined symbols and labels are resolved to their corresponding addresses.

### Extra Feature — Duplicate Labels

Because `K mod 3 = 2`, the assembler includes a duplicate-label warning.

While processing labels in the first pass, the assembler checks whether a label already exists:

- A warning is printed with the source line number.
- The original label definition is retained.

```text
Warning: Line 5: label '(LOOP)' is defined more than once
```

### Test Results

The assembler was tested with `Add.asm`, `Max.asm`, `Rect.asm`, `Pong.asm`, `Sample.asm`, and a duplicate-label test file.

- **Figure 6.** The assembled `Add.hack` matches the official output.
- **Figure 7.** The assembled `Max.hack` matches the official output.
- **Figure 8.** The assembled `Rect.hack` matches the official output.
- **Figure 9.** The assembled `Pong.hack` matches the official output across 27,483 lines.
- **Figure 10.** The terminal output shows successful test runs and the duplicate-label warning.

## 3. Box Program

`Box.asm` creates a 20-row by 16-pixel box on the left edge of the Hack screen.

### Program Logic

1. Read the keyboard memory address `KBD`.
   - If a key is pressed, set the color to `-1` (black).
   - If no key is pressed, set the color to `0` (white).
2. Set `addr` to `16640` (`SCREEN + 32 × 8`).
3. Set `count` to `20`.
4. Repeat 20 times:
   - Write the selected color to the 16-bit screen word at `addr`.
   - Add `32` to `addr` to move to the corresponding word in the next row.
   - Decrease `count`.
5. Restart the loop indefinitely, continuously updating the screen while a key is held.

Because the box is exactly one word wide, each row requires only one memory write.

### Emulator Results

- **Figure 11.** With no key pressed, the color is `0` and the screen remains blank.
- **Figure 12.** With a key held down, the color is `-1` and the box is drawn at the left edge.

## Mistake Fixed

The initial `Box.asm` tried to move to the next row using:

```assembly
M=M+D
```

This produced a `KeyError: 'M+D'` because the assembler’s supported C-instruction computations do not include `M+D`. The correct Hack syntax uses `D+M`:

```assembly
M=D+M
```

This is why `D+M` was used for moving to the next screen word in the final implementation.

## Submitted Files

The `capstone/` directory contains:

- `CPU.hdl`
- `Memory.hdl`
- `Computer.hdl`
- `assembler.py`
- `Box.asm`
- `Box.hack`
- Test programs and supporting files
