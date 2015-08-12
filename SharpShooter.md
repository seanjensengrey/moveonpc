# Introduction #

http://us.playstation.com/ps3/accessories/playstation-move-sharp-shooter-ps3.html

For a detailed technical description of the Move's EXT interface see the wiki page on [extension devices](ExtensionDevices.md).


# Configuration data #

```
80 81 00 00 00 00 00 01 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
a0 02 01 01 02 a0 03 01 01 04 a0 04 01 01 03 a0
05 01 01 05 a0 06 01 01 01 a0 07 01 04 2b 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

For an interpretation of this data see the section about [initialization and configuration](ExtensionDevices#Initialization_and_configuration.md) of exension devices.


# Protocol #

The Sharp Shooter uses the slave address `0xA0` – the read/write bit is included as LSB, so the read address is `0xA1` and the write address is `0xA0`. The Move sends single-byte feature queries to this address and receives single-byte responses that encode the Sharp Shooter's current button states. The Move sends 6 of these queries in a repeating pattern (the overall sequence repeating approximately every 11–13 ms).

The default responses are `0x00`. But depending on which Sharp Shooter buttons are currently pressed, certain bits in the responses are set:

| | | | | | | | **Sharp Shooter button state** |
|:|:|:|:|:|:|:|:-------------------------------|
| **Query** | `0x02` | `0x03` | `0x04` | `0x05` | `0x06` | `0x07` |                                |
| **Response** | | | | | | `0x01` | _Weapon 1_ selected            |
| **Response** | | | | | | `0x02` | _Weapon 2_ selected            |
| **Response** | | | | | | `0x04` | _Weapon 3_ selected            |
| **Response** | | | | | | `0x80` | _Reload_ button pressed        |
| **Response** | `0x08` | | `0x40` | | | | _Move_ button pressed          |
| **Response** | `0x10` | `0xFF` | `0x80` | `0xFF` | | `0x40` | _Trigger_ pressed              |
| **Response** | | | | | `0x10` | | _Triangle_ button pressed      |
| **Response** | | | | | `0x80` | | _Square_ button pressed        |


# Accessing the values #

In order to make the Sharp Shooter's button state available via Bluetooth, these values are mapped to specific positions in the Move controller's [Input Report](InputReport.md). The following offsets are referring to the Input Report.

Note that the Sharp Shooter's pump-action grip is connected to the Move's T button. See the _Trigger_ button's description below for more details.

## Detecting the device ##

Plugging the Sharp Shooter into the EXT socket sets bit `0x10` at [offset 0x04](InputReport#Buttons_3_(PS,_Move,_T)_and_EXT.md). Unplugging clears it again.

## Weapon selection ##

The selected weapon is mapped to a single byte at [offset 0x2C](InputReport#External_device_data.md) using the same bit mask as in the I²C communication (see table above).

## Reload button ##

Pressing the _Reload_ button sets bit `0x80` at [offset 0x2C](InputReport#External_device_data.md).

## Move button ##

Pressing the _Move_ button has the same effect as pressing the [Move controller's Move button](InputReport#Buttons_3_(PS,_Move,_T)_and_EXT.md), i.e. both sources cannot be distinguished.

## Trigger ##

Pulling the _Trigger_ sets the bit 0x40 at offset 0x2C. Additionally it sets the [T button values](InputReport#T_button_values.md) and [T button bits](InputReport#Buttons_3_(PS,_Move,_T)_and_EXT.md) in the Input Report just as fully depressing the Move controller's T button.

Pulling the pump-action grip has exactly the same effect as pressing the Move controller's T button, i.e. both sources cannot be distinguished. Note that in this case the additional bit 0x40 at offset 0x2C is _not_ set, which makes it possible to distinguish between pulling the _Trigger_ and pulling the pump-action grip.

## Triangle and Square buttons ##

Pressing the _Triangle_ or the _Square_ button has the same effect as pressing the [Move controller's corresponding button](InputReport#Buttons_2_(X,_Square,_Circle,_Triangle).md), i.e. both sources cannot be distinguished.


# Hardware #

For the EXT interface's pinout and description see [ExtensionDevices#Hardware](ExtensionDevices#Hardware.md).

<table align='center'>
<tr>
<blockquote><td width='360'><img src='http://wiki.moveonpc.googlecode.com/git/sharpshooter_board_top.jpg' /></td>
<td width='360'><img src='http://wiki.moveonpc.googlecode.com/git/sharpshooter_board_bottom_labeled.jpg' /></td>
</tr>
<tr>
<td>Sharp Shooter board, top side</td>
<td>Sharp Shooter board, bottom side (with test pads subsequently labeled, see schematic below)</td>
</tr>
</table></blockquote>

## Resources ##

  * <a href='http://wiki.moveonpc.googlecode.com/git/sharpshooter_board_schematic.pdf'>Sharp Shooter Board schematic as PDF</a>