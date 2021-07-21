# Memory Map

The Game Boy has a 16-bit address bus, which is used to address ROM, RAM, and I/O.

Start       | End       | Description                                                      | Notes
------------|-----------|------------------------------------------------------------------|----------
0000        | 3FFF      | 16 KiB ROM bank $00                                              | From cartridge, usually a fixed bank
4000        | 7FFF      | 16 KiB ROM Bank $01–NN                                           | From cartridge, switchable bank via [mapper](#MBCs) (if any)
8000        | 9FFF      | 8 KiB Video RAM (VRAM)                                           | In CGB mode, switchable bank 0/1
A000        | BFFF      | 8 KiB External RAM                                               | From cartridge, switchable bank if any
C000        | CFFF      | 4 KiB Work RAM (WRAM)                                            |
D000        | DFFF      | 4 KiB Work RAM (WRAM)                                            | In CGB mode, switchable bank 1–7
E000        | FDFF      | Mirror of C000–DDFF (Echo RAM)                                   | Nintendo says use of this area is prohibited.
FE00        | FE9F      | Sprite attribute table ([OAM](<#Object Attribute Memory (OAM)>)) |
FEA0        | FEFF      | Not Usable                                                       | Nintendo says use of this area is prohibited.
FF00        | FF7F      | I/O Registers                                                    |
FF80        | FFFE      | High RAM (HRAM)                                                  |
FFFF        | FFFF      | [Interrupt Enable (IE)](<#FFFF - IE - Interrupt Enable (R/W)>) register |

## Jump Vectors in first ROM bank

The following addresses can be used as jump vectors:

- [`rst` instructions](https://rgbds.gbdev.io/docs/v0.5.1/gbz80.7#RST_vec): $0000, $0008, $0010, $0018, $0020, $0028, $0030, $0038
- [Interrupts](#Interrupts): $0040, $0048, $0050, $0058, $0060

However, this memory area ($0000–00FF) may be used for any other purpose if your
program doesn't use all `rst` instructions or interrupts.

## Cartridge Header in first ROM bank

The memory area at $0100–014F contains the [cartridge header](<#The Cartridge Header>).
This area contains information about the program, its entry point, checksums,
information about the used MBC chip, the ROM and RAM sizes, etc.
Most of the bytes in this area have to be specified correctly.

## External Memory and Hardware

The areas 0000–7FFF and A000–BFFF address external hardware on
the cartridge, which is essentially an expansion board.
They usually map to some ROM and RAM, often via a [Memory Bank Controller (MBC)](#MBCs).
The RAM area can be read from and written to normally; writes to the ROM area
control the MBC. Some MBCs allow mapping of other hardware into the RAM area
in this way.

Cartridge RAM is often battery-backed to preserve save files,
high score tables, and other information when the Game Boy is turned off.

## Echo RAM

The range C000–FDFF is mapped to WRAM, but only the lower 13 bits of
the address lines are connected. This causes the address to effectively wrap
around, "repeating" WRAM. All reads and writes to this range have the same
effect as if they were performed to C000–DDFF.

Nintendo prohibits developers from using this memory range.
The behavior is confirmed on all official hardware.

::: warning COMPATIBILITY

Some emulators (such as VisualBoyAdvance up to version 1.8) fail to correctly
emulate Echo RAM.
In some flash cartridges, Echo RAM interferes with SRAM normally at A000–BFFF,
which might damage the cartridge and/or the Game Boy.
**It is therefore advisable to avoid reading from or writing to Echo RAM.**

Software can check if Echo RAM is properly emulated by writing to RAM (avoid
values $00 and $FF) and checking if said value is present in Echo RAM and not
cartrigde SRAM.

:::

## FEA0–FEFF range

Nintendo indicates use of this area is prohibited. This area returns
$FF when OAM is blocked, and otherwise the behavior depends on the
hardware revision.

- On DMG, MGB, SGB, and SGB2, reads [while OAM is locked](<#LCD Status Register>)
  trigger [OAM corruption](<#OAM Corruption Bug>).
  Reads otherwise return $00.
- On CGB revisions 0–D, this area is a unique RAM area, but the address is ANDed
  with a revision-specific mask.
- On CGB revision E, AGB, AGS, and GBP, it returns the high nibble of the
  lower address byte twice, e.g. $FFAx returns $AA, FFBx returns $BB, and so
  forth.

## I/O Ranges

The Game Boy uses the following I/O ranges:

Start   | End     | First appeared | Purpose
--------|---------|----------------|-------------
$FF00   |         |       DMG      | [Controller](<#FF00 - P1/JOYP - Joypad (R/W)>)
$FF01   |  $FF02  |       DMG      | [Communication](<#Serial Data Transfer (Link Cable)>)
$FF04   |  $FF07  |       DMG      | [Divider and Timer](<#Timer and Divider Registers>)
$FF10   |  $FF26  |       DMG      | [Sound](<#Sound Controller>)
$FF30   |  $FF3F  |       DMG      | [Waveform RAM](<#FF30-FF3F - Wave Pattern RAM>)
$FF40   |  $FF4B  |       DMG      | [LCD](<#Rendering Overview>)
$FF4F   |         |       CGB      | [VRAM Bank Select](<#VRAM Banks>)
$FF50   |         |       DMG      | Set to non-zero to disable boot ROM
$FF51   |  $FF55  |       CGB      | [VRAM DMA](<#LCD VRAM DMA Transfers>)
$FF68   |  $FF69  |       CGB      | [BG / OBJ Palettes](<#LCD Color Palettes (CGB only)>)
$FF70   |         |       CGB      | [WRAM Bank Select](<#FF70 - SVBK - CGB Mode Only - WRAM Bank>)
