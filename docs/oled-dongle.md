# TOTEM OLED dongle

This build adds an ELEGOO 0.96-inch, 128x64 SSD1306 I2C OLED to the existing
XIAO nRF52840 dongle. It displays the active layer, USB/Bluetooth output state,
and the battery level of both keyboard halves.

## Wiring

Follow the labels printed on the OLED module; do not rely on physical pin order.

| OLED | XIAO nRF52840 |
| --- | --- |
| GND | GND |
| VCC | 3V3 |
| SCL | D5 |
| SDA | D4 |

For the two-button prototype, connect each normally-open momentary button
between its GPIO and ground:

| Button | XIAO connections | Behavior |
| --- | --- | --- |
| PC | D0 and GND | Toggle PC mode (off/default is Mac mode) |
| SYM | D1 and GND | Toggle the `SYM` layer |

Press a button again to turn its layer off. PC mode is intentionally not
persistent, so every reboot starts in Mac mode. If `SYM` is also active, it
appears and takes priority because it is the higher layer.

Either dongle button also wakes the OLED. If the keyboard otherwise remains
idle, the screen blanks again after the normal display timeout.

Disconnect USB power before changing any wire. Power the OLED from 3V3 even
though the breakout also accepts 5V, so its I2C pull-ups remain safe for the
XIAO's 3.3V GPIO.

## Firmware artifacts

- `totem_dongle_oled-zmk.uf2`: OLED-enabled dashboard firmware.
- `totem_dongle-zmk.uf2`: unchanged display-free fallback firmware.
- The left and right firmware artifacts are unchanged.

Only the dongle needs to be flashed for the display change. Flashing the OLED
build should not require clearing bonds or reflashing the keyboard halves.

## Expected display

The standard OLED build keeps the compact information-dense layout:

- Top left: active USB/Bluetooth output.
- Top right: one battery row for each half.
- Bottom right: highest active layer (`BASE`, `PC`, `NUM`, `SYM`, `ADJ`, or
  conditional `NAV`).

The experimental YADS-inspired build uses a compact dashboard layout:

- Top left: a large active-layer name.
- Top right: a shared slot showing `CAPS` for Caps Lock or `CW` for Caps Word.
- Bottom: separate `L` and `R` percentages with battery bars.

The dashboard uses grid-aligned Micro 5 rasterizations: 44 px for the central
layer (a true 4x pixel grid) and 22 px for status text (a true 2x grid). The
font is embedded locally under its SIL Open Font License so cloud builds remain
reproducible.

Each bottom widget shows `OFF` when that half is disconnected, `...` while a
newly connected half is awaiting its first battery report, and its percentage
value and bar once the report arrives. The percent sign is omitted; the left
value is left-aligned and the right value is right-aligned. The bars mirror one
another: the left drains from the right edge and the right drains from the left.

The battery rows follow peripheral pairing order. The current pairing should
remain intact. If bonds ever have to be cleared, pair the left half first and
the right half second so the rows remain left-to-right.

## Assumptions and recovery

- The OLED uses the usual SSD1306 I2C address `0x3C`.
- If the screen stays blank, first verify VCC/GND and SDA/SCL. Then scan for an
  unexpected `0x3D` address before changing firmware.
- If the dongle misbehaves, flash the display-free `totem_dongle-zmk.uf2`.
