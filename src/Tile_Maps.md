
# VRAM Tile Maps

The Game Boy contains two 32×32 tile maps in VRAM at
the memory areas $9800–$9BFF and $9C00–$9FFF. Each of these maps can be used by
the Background or the Window, or both.

Since one tile is 8×8 pixel, each map holds a 256×256 pixel picture.
Only 160×144 of those pixels are displayed on the LCD at any given time.

## Tile Indexes

Each tile map contains the 1-byte IDs of the tiles to be displayed.

Tiles are selected from the $8000–97FF region using either of the two
addressing modes (described in [VRAM Tile Data](<#Tile IDs>)), which
can be selected via [the LCDC register](<#FF40 - LCDC (LCD Control) (R/W)>).

## BG Map Attributes (CGB Mode only)

In CGB Mode, two additional maps, 32×32 bytes each, are stored in VRAM bank 1.

Each byte defines attributes for the corresponding tilemap entry in VRAM bank 0;
that is, 1:9800 defines the attributes for the tile at 0:9800.
Note that, for example, if the map entry at 0:9800 is tile \$2A, the attribute
at 1:9800 doesn't define properties for ALL tiles \$2A on-screen, but only
the one at 0:9800!

```
Bit 7    BG-to-OAM priority         (0=Use OAM Priority bit, 1=BG Priority)
Bit 6    Vertical flip              (0=Normal, 1=Mirror vertically)
Bit 5    Horizontal flip            (0=Normal, 1=Mirror horizontally)
Bit 4    Not used
Bit 3    Tile VRAM bank number      (0=Bank 0, 1=Bank 1)
Bit 2-0  Background palette number  (BG0-7)
```

When bit 7 is set, the corresponding BG tile will have priority above all OBJs
(regardless of the priority bits in OAM memory). There is also a Master Priority
flag in [LCDC register bit 0](<#CGB Mode: BG and Window Master Priority>),
which overrides all other priority bits when cleared.

## Background (BG)

The [SCY and SCX](<#FF42 - SCY (Scroll Y) (R/W), FF43 - SCX (Scroll X) (R/W)>)
registers can be used to scroll the background, specifying the origin of the visible
160×144 pixel area within the total 256×256 pixel background map.
The visible area of the background wraps around the map (that is, when part of
the visible area goes beyond the map edge, it starts displaying the opposite side of the map).

In Non-CGB mode, the background (and the window) can be disabled using
[LCDC bit 0](<#LCDC.0 - BG and Window enable/priority>).

## Window

Besides the background, there is also a window overlaying it.
The contents of the window are not scrollable; it is always displayed starting
at the top left tile of its tile map. The only way to adjust the window
is by modifying its position on the screen, which is done via
[the WX and WY registers](<#FF4A - WY (Window Y Position) (R/W), FF4B - WX (Window X Position + 7) (R/W)>).
The on-screen coordinates of the top left corner of the window are (WX - 7, WY).

The window uses tiles just like the background.

Whether the window is displayed can be toggled using
[LCDC bit 5](<#LCDC.5 - Window enable>). However, in Non-CGB mode, this bit is
ignored if [LCDC bit 0](<#LCDC.0 - BG and Window enable/priority>) is clear.
Enabling the window makes
[Mode 3](<#LCD Status Register>) slightly longer on scanlines where it's visible.
(See [WX and WY](<#FF4A - WY (Window Y Position) (R/W), FF4B - WX (Window X Position + 7) (R/W)>)
for the definition of "Window visibility".)

::: tip Window Internal Line Counter

The window keeps an internal line counter that's functionally similar to
[LY](<#FF44 - LY (LCD Y Coordinate) (R)>), and increments alongside it.
However, it only gets incremented when the window is visible, as described
[here](<#FF4A - WY (Window Y Position) (R/W), FF4B - WX (Window X Position + 7) (R/W)>).

This line counter determines which line of its tilemap is to be rendered on the
current scanline.

:::
