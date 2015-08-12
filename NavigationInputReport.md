# Introduction #

The Move Navigation submits its status and sensor data in HID Input report 0x01 that will typically look like this:

```
01 00 00 00 00 00 7E 80 7D 84 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 03 05 14
FF C8 00 00 23 BF 77 01 C0 02 02 02 01 01 A2 02
00 
```

The following section explains the report's format in detail, e.g. how to interpret the data.


# Details #

Description of directional buttons assumes that the Move Navigation is held with its PS button pointing up and its USB connector pointing towards you.

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0x01) |
| 0x02            | 1                     | Buttons 1 (L3, D-pad) |
| 0x03            | 1                     | Buttons 2 (X, Circle, L1, L2) |
| 0x04            | 1                     | PS button       |
| 0x06            | 1                     | Stick X-axis    |
| 0x07            | 1                     | Stick Y-axis    |
| 0x0E            | 1                     | D-pad UP        |
| 0x0F            | 1                     | D-pad RIGHT     |
| 0x10            | 1                     | D-pad DOWN      |
| 0x11            | 1                     | D-pad LEFT      |
| 0x12            | 1                     | L2 button value |
| 0x14            | 1                     | L1 button value |
| 0x17            | 1                     | Circle button value |
| 0x18            | 1                     | X button value  |
| 0x1E            | 1                     | Battery level   |


## Buttons 1 (L3, D-pad) ##

This is a bit mask that indicates if _L3_ (`0x02`), _D-pad UP_ (`0x10`), _D-pad RIGHT_ (`0x20`), _D-pad DOWN_ (`0x40`) or _D-pad LEFT_ (`0x80`) buttons are pressed. The corresponding bit is set while the button is depressed. The _L3_ button functions when the analog stick is pressed (resulting in a noticeable click).

## Buttons 2 (X, Circle, L1, L2) ##

This is a bit mask that indicates if _L2_ (`0x01`), _L1_ (`0x04`), _Circle_ (`0x20`) or _X_ (`0x40`) buttons are pressed. The corresponding bit is set while the button is depressed.

## PS button ##

The bit `0x01` of this byte is set while the _PS_ button is depressed.

## Stick ##

These fields contain directional data of the analog stick. The manual calls it the _left stick_ (probably in referrence to the PS3 Sixaxis controller). The values are shifted up to always report positive numbers. Subtract `0x80` to obtain signed values and determine direction from the sign. Positive is right/down while negative is left/up, respectively.

## D-pad ##

These fields indicate the state of buttons on the analog pad. The manual calls it the _directional buttons_. Values are ranging from `0x00` (not pressed) to `0xFF` (fully depressed).

## L1, L2 button values ##

These fields indicate the position of the analog _L1_ and _L2_ button, respectively. Values are ranging from `0x00` (not pressed) to `0xFF` (fully depressed).

## Circle, X button values ##

These fields contain data of the _Circle_ and _X_ button, respectively. Since these buttons are only digital, the value is always either `0x00` (not pressed) or `0xFF` (depressed) but never anything in between.

## Battery level ##

This field reports the controller's battery level, a value of `0x05` meaning fully charged. If the controller is connected via USB cable the value is either `0xEE` (charging) or `0xEF` (fully charged).
