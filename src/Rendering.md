
# Rendering Overview

The Game Boy outputs graphics to a 160×144 pixel LCD, using a fairly complex
mechanism to facilitate rendering.

::: warning

Graphics terminology tends to vary a lot among different platforms, consoles,
users and communities. You may be familiar with slightly different definitions.
Keep also in mind that some of the definitions refers to lower-level (hardware)
tools and some others to more abstract concepts.

:::

## Tiles

Similarly to other retro systems, pixels are not manipulated
individually, as this would be expensive CPU-wise. Instead, pixels are grouped
in 8×8 squares, called *tiles* (or sometimes "patterns"), often considered as
the base unit in Game Boy graphics.

A tile does not encode color information. Instead, a tile assigns a *color ID*
to each of its pixels, ranging from 0 to 3. For this reason, Game Boy graphics
are also called *2bpp* (2 bits per pixel). When a tile is used in the
Background or Window, colors are associated to these IDs through a *palette*.
When a tile is used in an OBJ, IDs 1 to 3 are also associated via a palette,
but ID 0 is ignored and always means "transparent".

## Palettes

A palette consists of an array of colors, 4 in the Game Boy's case.
Palettes are stored differently in monochrome and color versions of the console.

Modifying palettes enables graphical effects such as quickly flashing some graphics (damage,
invulnerability, thunderstorm, etc.), fading the screen, "palette swaps", and more.

## Layers

The Game Boy has three "layers", from back to front: the Background, the Window,
and the Objects. Some features and behaviors break this "layer" abstraction,
but it works for the most part.

### Background

The background is composed of a *tilemap*. A tilemap is a
large grid of tiles. However, tiles aren't directly written to tilemaps,
they merely contain references to the tiles.
This makes reusing tiles very cheap, both in CPU time and in
required memory space, and it is the main mechanism that helps working around the
paltry 8 KiB of video RAM.

The background can be made to scroll as a whole, by writing to
[two hardware registers](<#FF42 - SCY (Scroll Y) (R/W), FF43 - SCX (Scroll X) (R/W)>).
This makes scrolling very cheap.

### Window

The window is sort of a second background layer on top of the background.
It is fairly limited: it has no transparency, it's always a
rectangle and only the position of the top-left pixel can be controlled.

Possible usage include a fixed status bar in an otherwise scrolling game
(such as in *Super Mario Land 2*).

### Objects

The background is useful for elements scrolling as a whole, but it's
impractical for things that need to move independently, such as the player.

*Objects* are designed to fill this gap: they are made of 1 or 2
stacked tiles (8×8 or 8×16 pixels) and can be displayed anywhere on the screen.

::: tip NOTE

Several objects can be combined to draw a larger graphical element, usually
called "meta-sprite". Originally, the term "sprites" referred to fixed-sized
objects composited together with a background by hardware.
Use of the term has since become more general.

:::

To summarise:

- **Tile**, an 8x8-pixel chunk of graphics.
- **Object**, an entry in object attribute memory, composed of 1 or 2
  tiles. Independent from the background.
