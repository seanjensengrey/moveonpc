# Introduction #

The Move submits its status and sensor data in HID reports that will typically look like this:

```
01 00 00 00 06 00 00 7f 7f 7f 7f c0 05 2d 89 a8
7f 77 8d 1e 89 a0 7f 6f 8d f8 7f 02 80 fb 7f f8
7f 03 80 fb 7f 7d 10 9e fa 10 55 0a 00 00 00 00
00
```

The following section explains the report's format in detail, e.g. how to interpret the data.



# Details #

<img src='http://moveonpc.googlecode.com/svn/wiki/movelocalcoordinates.png' align='right' />

Fields that cover more than one byte are stored as little-endian, i.e. the least significant byte comes first. Description of the axes assumes that the Move is held with the buttons pointing up and the glowing ball pointing away from you:
  * X-axis pointing to the right
  * Y-axis pointing up
  * Z-axis pointing away from you.

Accelerometers, gyroscopes and the analog T button each report two values per report. The first value (subsequently denoted _1st half-frame_ ) contains data that is slightly older than the second one. This effectively doubles the reporting rate for these sensors.

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0x01) |
| 0x01            | 1                     | Buttons 1 (Start, Select) |
| 0x02            | 1                     | Buttons 2 (X, Square, Circle, Triangle) |
| 0x03            | 2_<sup>note 1</sup>_  | Buttons 3 (PS, Move, T) and EXT |
| 0x04            | 1_<sup>note 2</sup>_  | Sequence number |
| 0x05            | 1                     | T button values (1st half-frame) |
| 0x06            | 1                     | T button values (2nd half-frame) |
| 0x07            | 4                     | always 0x7F7F7F7F |
| 0x0B            | 1                     | Timestamp (upper byte) |
| 0x0C            | 1                     | Battery level   |
| 0x0D            | 2                     | X-axis accelerometer (1st half-frame) |
| 0x0F            | 2                     | Z-axis accelerometer (1st half-frame) |
| 0x11            | 2                     | Y-axis accelerometer (1st half-frame) |
| 0x13            | 2                     | X-axis accelerometer (2nd half-frame) |
| 0x15            | 2                     | Z-axis accelerometer (2nd half-frame) |
| 0x17            | 2                     | Y-axis accelerometer (2nd half-frame) |
| 0x19            | 2                     | X-axis gyroscope (1st half-frame) |
| 0x1B            | 2                     | Z-axis gyroscope (1st half-frame) |
| 0x1D            | 2                     | Y-axis gyroscope (1st half-frame) |
| 0x1F            | 2                     | X-axis gyroscope (2nd half-frame) |
| 0x21            | 2                     | Z-axis gyroscope (2nd half-frame) |
| 0x23            | 2                     | Y-axis gyroscope (2nd half-frame) |
| 0x25            | 2_<sup>note 1</sup>_  | Temperature     |
| 0x26            | 2_<sup>note 3</sup>_  | X-axis magnetometer |
| 0x28            | 2_<sup>note 1</sup>_  | Z-axis magnetometer |
| 0x29            | 2_<sup>note 3</sup>_  | Y-axis magnetometer |
| 0x2B            | 1                     | Timestamp (lower byte) |
| 0x2C            | 5                     | External device data |

_note 1_: This field actually covers 12 bits. Only the _upper_ nibble of the byte at the specified offset+1 belongs to this field. The lower nibble belongs to the following field.

_note 2_: This field actually covers 4 bits. Only the _lower_ nibble of the byte at the specified offset belongs to this field. The upper nibble belongs to the previous field.

_note 3_: This field actually covers 12 bits. Only the _lower_ nibble of the byte at the specified offset belongs to this field. The upper nibble belongs to the previous field.



## Buttons 1 (Start, Select) ##

This is a bit mask that indicates if _Select_ (value `0x01`) or _Start_ (value `0x08`) are pressed. The corresponding bit is set while the button is depressed.


## Buttons 2 (X, Square, Circle, Triangle) ##

This is a bit mask that indicates if _X_ (`0x40`), _Square_ (`0x80`), _Circle_ (`0x20`) or _Triangle_ (`0x10`) are pressed. The corresponding bit is set while the button is depressed.


## Buttons 3 (PS, Move, T) and EXT ##

This is a bit mask that indicates if _PS_ (`0x0001`), _Move_ (`0x4008`) or the analog _T_ button (`0x8010`) are pressed. The corresponding bits are set while the button is depressed. If set, the bit `0x1000` indicates that an [extension device](HardwareAndFirmware#Extension_Socket.md), such as the [Sharp Shooter attachment](SharpShooter.md), is currently attached, has successfully sent its config data and the Move sent the specified [ExtOut data](ExtensionDevices#Sending_data.md) in reply.

Note that the upper byte of this bit mask is applied to the byte at offset `0x04` while the lower byte of this bit mask is applied to the byte at offset `0x03`.


## Sequence number ##

This field has only 4-bit resolution. It increases with every received report and overflows at value `0x0F`. It can be used as a simple mechanism to detect missed reports.


## T button values ##

This field indicates the analog T button's position, ranging from `0x00` (not pressed) to `0xFF` (fully depressed).


## Timestamp ##

This is a 16-bit counter that continuously increases with every report and overflows at value `0xFFFF`. Its lower byte is located at offset `0x2B`, its upper byte at offset `0x0B`. The value is build from an internal counter and could be used to measure the time interval between two consecutive reports.


## Battery level ##

This field reports the controller's battery level, a value of `0x05` meaning fully charged. If the controller is connected via USB cable the value is either `0xEE` (charging) or `0xEF` (fully charged).


## Accelerometer ##

These fields contain 16-bit data of the tri-axis accelerometer. The values are shifted up to always report positive numbers. Subtract `0x8000` (the 0 rad<sup>-1</sup> level) to obtain signed values and determine direction from the sign.


## Gyroscope ##

These fields contain 16-bit data of the tri-axis gyrometer. The values are shifted up to always report positive numbers. Subtract `0x8000` (the ideal zero G level) to obtain signed values and determine direction from the sign.


## Temperature ##

This field reports temperature as a 12-bit value. It contains the raw ADC value (plus an offset) measured at a voltage divider which contains a thermistor. Unless we find out which thermistor exactly is used (extremely unlikely considering the unlabeled part on the PCB), the only way to derive a temperature from these values is conducting our own measurements.


## Magnetometer ##

These fields contain 12-bit data of the tri-axis magnetometer. The value range is -2048..+2047 (assuming two's complement).

To convert this to a signed value, you can use the two's complement:

```
int decode_12bit_signed(int x) {
    return (((x) & 0x800)?(-(((~(x)) & 0xFFF) + 1)):(x));
}
```


## External device data ##

Devices plugged into the [EXT socket](HardwareAndFirmware#Extension_Socket.md), such as the [Sharp Shooter attachment](SharpShooter.md), report some of their data using these bytes.
