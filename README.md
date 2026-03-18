# SSD1306 MicroPython Driver
MicroPython driver for the SSD1306 OLED display on ESP32, with no external library dependencies.
Comments and methods were originally written in Brazil Protuguese. Version in english in future.


## Supported Hardware
- **Microcontroller:** ESP32 DevKit (tested on ESP32-D0WD-V3 rev 3.1)
- **Display:** SSD1306 OLED 128x32 via I2C
- **MicroPython:** v1.27.0

> **Note:** This driver is configured for **128x32** displays. For 128x64 displays, adjust `init_seq` (`0xA8, 0x3F` and `0x22, 0x00, 0x07`) and the framebuffer size (1024 bytes).

## Utilization examples
<image src="examples/Images/Umidity and temp.jpg" width="100">  Example of usage for temperature and umidity meter. ANT10 is necessary.

<image src="examples/Images/8x8_font_4_lines.jpg" width="100">  4 lines with 8x8 font

<image src="examples/Images/8x16_font.jpg" width="100">  3 lines with 8x16 font

<image src="examples/Images/flash_method.jpg" width="100">  The flash method turns on and off all pixels in the display.

<image src="examples/Images/vert_scroll.jpg" width="100">  Vertical scroll

<image src="examples/Images/horiz_scroll.jpg" width="100">  Horizontal scroll

## Wiring
| SSD1306 | ESP32  |
|---------|--------|
| GND     | GND    |
| VDD     | 3.3V   |
| SCK     | GPIO22 |
| SDA     | GPIO21 |


## Installation
Copy the following file to your ESP32 (`File → Save as → MicroPython device`):

```
ssd1306.py
```
> Note: The driver was copied to ESP32 and tested using Thonny (https://thonny.org/).


## Basic Usage
```python
from machine import Pin, SoftI2C
from ssd1306 import SSD1306

sda = Pin(21)
scl = Pin(22)
i2c = SoftI2C(scl=scl, sda=sda, freq=400_000)

display = SSD1306(i2c)

display.texto('Hello World', 0, 24, 8)
display.texto('28.3°C  73%', 0, 8, 16)
display.exibe()
```

## API
### `__init__(bus, addr=0x3C)`
Initializes the display and clears the screen.

### `limpa_tela()`
Clears all pixels and updates the display.

### `pixel(x, y, status=True)`
Turns a pixel on (`True`) or off (`False`) at position (x, y).
- x: 0 to 127
- y: 0 to 31 (0 = bottom, 31 = top)

### `texto(str_texto, x, y, tamanho=8)`
Renders a string on the display.
- `tamanho=8`: 8x8 pixel font
- `tamanho=16`: 8x16 pixel font
- Supports the `°` (degree) symbol in addition to standard ASCII

### `exibe()`
Sends the framebuffer to the display. Must be called after any `pixel()` or `texto()` operations.

### `flash(count=3, on=300, off=100)`
Flashes the screen `count` times. `on` and `off` in milliseconds.

### `envia_comando(bytearray)`
Sends raw SSD1306 commands. For advanced use — example: horizontal scroll:

```python
display.envia_comando(bytearray([
    0x2E,        # deactivate scroll
    0x26,        # scroll right
    0x00,        # dummy byte
    0x00,        # start page
    0x00,        # speed
    0x03,        # end page
    0x00, 0xFF,  # dummy bytes
    0x2F,        # activate scroll
]))
```

## Included Fonts

Two bitmap fonts are included as Python dictionaries:
- **`fonte_8x8`** — full ASCII + `°` symbol, transposed vertical column format
- **`fonte_8x16`** — full ASCII + `°` symbol, two pages per character

Unmapped characters fall back to space automatically.


## Internal Design

The driver maintains a **framebuffer** of 512 bytes in RAM (128 columns × 4 pages × 8 bits). All drawing operations modify the local framebuffer. Data is only transferred to the display when `exibe()` is called, avoiding unnecessary I2C transmissions.

Memory layout:
```
Page 3 (y=24..31) ← yellow zone on bicolor display
Page 2 (y=16..23) ← blue zone
Page 1 (y= 8..15) ← blue zone
Page 0 (y= 0.. 7) ← blue zone
```

## Changelog

| Version | Description |
|---------|-------------|
| v0      | Basic SSD1306 controls — initialization, framebuffer, pixel |
| v1      | 8x16 font support, text renderer improvements |
| v2      | Fonts converted from lists to dictionaries, `°` symbol added |


## License
MIT License — free to use, modify, and distribute.
