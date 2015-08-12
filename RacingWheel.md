# Introduction #

http://us.playstation.com/ps3/accessories/playstation-move-racing-wheel-ps3.html

For a detailed technical description of the Move's EXT interface see the wiki page on [extension devices](ExtensionDevices.md).


# Configuration data #

```
81 01 00 00 00 00 00 3c 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
a0 02 01 01 00 a0 03 01 01 01 a0 04 01 04 2b a0
05 01 04 2c a0 06 01 04 2d a0 07 01 04 2e a0 08
01 04 2f 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

For an interpretation of this data see the section about [initialization and configuration](ExtensionDevices#Initialization_and_configuration.md) of exension devices.


# Protocol #

The Racing Wheel uses the slave address `0xA0` – the read/write bit is included as LSB, so the read address is `0xA1` and the write address is `0xA0`. The Move sends single-byte feature queries to this address and receives single-byte responses that encode the Racing Wheel's current button states. The Move sends 7 of these queries in a repeating pattern (the overall sequence repeating approximately every 11–13 ms).

The default responses are `0x00`, with the exception of query `0x07` in whose response the bits `0x3C` are always set. But depending on which Raching Wheel buttons are currently pressed, certain additional bits in the responses are set:

| | | | | | | | | **Racing Wheel button state** |
|:|:|:|:|:|:|:|:|:------------------------------|
| **Query** | `0x02` | `0x03` | `0x04` | `0x05` | `0x06` | `0x07` | `0x08` |                               |
| **Response** | `0x01` | | | | | | | _Select_ button pressed       |
| **Response** | `0x08` | | | | | | | _Start_ button pressed        |
| **Response** | `0x10` | | | | | | | _D-pad UP_ pressed            |
| **Response** | `0x20` | | | | | | | _D-pad RIGHT_ pressed         |
| **Response** | `0x40` | | | | | | | _D-pad DOWN_ pressed          |
| **Response** | `0x80` | | | | | | | _D-pad LEFT_ pressed          |
| **Response** | | `0x04` | | | | | | _L1_ button pressed           |
| **Response** | | `0x08` | | | | | | _R1_ button pressed           |
| **Response** | | `0x10` | | | | | | _Triangle_ button pressed     |
| **Response** | | `0x20` | | | | | | _Circle_ button pressed       |
| **Response** | | `0x40` | | | | | | _X_ button pressed            |
| **Response** | | `0x80` | | | | | | _Square_ button pressed       |
| **Response** | | | 0–255 | | | | | _Throttle_ value              |
| **Response** | | | | 0–255 | | | | _L2_ button value             |
| **Response** | | | | | 0–255 | | | _R2_ button value             |
| **Response** | | | | | | `0x01` | | _Left Paddle_ pressed         |
| **Response** | | | | | | `0x02` | | _Right Paddle_ pressed        |


# Accessing the values #

In order to make the Racing Wheel's button state available via Bluetooth, these values are mapped to specific positions in the Move controller's [Input Report](InputReport.md). The following offsets are referring to the Input Report.

## Detecting the device ##

Plugging the Racing Wheel into the EXT socket sets bit `0x10` at [offset 0x04](InputReport#Buttons_3_(PS,_Move,_T)_and_EXT.md). Unplugging clears it again.

## Start/Select ##

Pressing the _Start_ or the _Select_ button has the same effect as pressing the [Move controller's corresponding button](InputReport#Buttons_1_(Start,_Select).md), i.e. both sources cannot be distinguished.

## D-Pad ##

Pressing the directional buttons sets bits at [offset 0x01](InputReport#Buttons_1_(Start,_Select).md) using the same bit mask as in the I²C communication (see table above).

## L1/[R1](https://code.google.com/p/moveonpc/source/detail?r=1) buttons ##

Pressing the _L1_ or _R1_ button sets bits at [offset 0x02](InputReport#Buttons_2_(X,_Square,_Circle,_Triangle).md) using the same bit mask as in the I²C communication (see table above).

## Symbol buttons ##

Pressing the _Triangle_, _Circle_, _X_ or _Square_ button has the same effect as pressing the [Move controller's corresponding button](InputReport#Buttons_2_(X,_Square,_Circle,_Triangle).md), i.e. both sources cannot be distinguished.

## Throttle ##

The position of the analog _Throttle_ grip is mapped to [offset 0x2C](InputReport#External_device_data.md). Values range from `0x00` (not pressed) to `0xFF` (fully depressed).

## L2/[R2](https://code.google.com/p/moveonpc/source/detail?r=2) buttons ##

The position of the analog _L2_ and _R2_ buttons are mapped to [offset 0x2D](InputReport#External_device_data.md) and [offset 0x2E](InputReport#External_device_data.md), respectively. Values range from `0x00` (not pressed) to `0xFF` (fully depressed).

## Paddles ##

Pressing the _Left Paddle_ or _Right Paddle_ sets bits at [offset 0x2F](InputReport#External_device_data.md) using the same bit mask as in the I²C communication (see table above).


