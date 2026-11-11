# Arduino Bitmap Converter

## Overview

A small C# command-line tool that converts a bitmap (`.bmp`) image into a compact C++ header that an Arduino (or similar microcontroller) can use to draw the image on an attached TFT display.  This was created because images can quickly consume the flash/RAM of an 8-bit microcontroller.

```
abc <input_bitmap> <output_file> [transparency_color]
```

## How It Works

- **Color reduction:** each 24-bit RGB pixel is converted to the 16-bit RGB565 format used by the display controller.
- **Palette + run-length encoding:** the image is scanned row by row and collapsed into horizontal runs. Each run is packed into a single 16-bit word: a 6-bit index into a generated color palette and a 10-bit run length.  A large flat-colored icon shrinks from thousands of pixels to a few hundred words.
- **Flash storage:** the stroke and palette arrays are emitted as `PROGMEM` constants, so the image data stays in flash and uses no SRAM.
- **Fast rendering:** the generated `Bitmap` base class draws each run with a single `drawFastHLine` call on the RA8875 display driver instead of plotting pixels one at a time.
- **Transparency and inversion:** the transparency color (magenta by default) is skipped while drawing, so images can sit on top of other content.  Colors can also be inverted at draw time by indexing the palette in reverse.
- **Code generation:** the tool writes a shared `Bitmap.h` base class plus one header per image. Each header contains a subclass that holds the image's dimensions/data, so drawing it on the device takes one line: `Bitmap::draw(myLogo, x, y)`.
