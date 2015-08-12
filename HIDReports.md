# Move controller #

## Readable reports ##

  * BT Input report 0x01: [Sensor data](InputReport.md)
  * BT Feature report 0x04: Bluetooth addresses
  * BT Feature report 0x10: [Calibration data](CalibrationData.md)
  * BT Feature report 0x11
  * BT Feature report 0xA1: "Response" in [authentication scheme with the PS3](Authentication.md)
  * BT Feature report 0xE0: [external device access control](FeatureReport0xE0.md) (EXT connector)
  * USB/BT Feature report 0xF1: read access to 1024 bytes of internal Flash memory (containing things such as the controller's [calibration data](CalibrationData.md))
  * BT Feature report 0xF7: [miscellaneous debug data](FeatureReport0xF7.md)
  * BT Feature report 0xF8
  * BT Feature report 0xF9: Firmware version
  * BT Feature report 0xFB: Debug data? (state of an internal variable and the output pins PB0–15, PE0–15)

## Writable reports ##

  * BT Output report 0x02: Sphere color and rumble, expects a 49 byte buffer (inluding Report ID)
  * BT Feature report 0x03: [PWM frequency](FeatureReport0x03.md) used for driving the sphere's color LEDs
  * USB Feature report 0x05: Bluetooth master address
  * BT Output report 0x06: Same as Output report 0x02, but expects a 9 byte buffer (inluding Report ID)
  * BT Feature report 0xA0: "Challenge" in [authentication scheme with the PS3](Authentication.md)
  * BT Feature report 0xE0: [external device access control](FeatureReport0xE0.md) (EXT connector)
  * USB/BT Feature report 0xF1: write access to 1024 bytes of internal Flash memory (containing things such as the controller's [calibration data](CalibrationData.md))
  * USB Feature report 0xF2: [Controller's operation mode](WriteFeatureReport0xF2.md)
  * USB Feature report 0xFA: Sphere color (keeps glowing without need to refresh)
  * USB report 0xFB: Debug data? (set state of the output pins PB0–15, PE0–15 and a related internal variable)


# Move Navigation controller #

## Readable reports ##

  * BT Input report 0x01: [Button data](NavigationInputReport.md)
  * USB/BT Feature report 0xF2: controller's own Bluetooth device address
    * the address 11:22:33:44:55:66 is reported as `ff ff 00 11 22 33 44 55 66 ?? ?? ?? ?? ?? ?? ?? ??`, i.e. the address starts at offset `0x03`
    * NOTE: unlike with the Move controller, the address is not stored backwards in the report
  * USB/BT Feature report 0xF5: host's Bluetooth device address
    * the address aa:bb:cc:dd:ee:ff is reported as `01 00 aa bb cc dd ee ff`, i.e. the address starts at offset `0x02`
    * NOTE: unlike with the Move controller, the address is not stored backwards in the report

## Writable reports ##

  * USB Feature report 0xF5: host's Bluetooth device address
    * the address aa:bb:cc:dd:ee:ff must be written as `01 00 AA BB CC DD EE FF`, i.e. with the address starting at offset `0x02`
    * NOTE: unlike with the Move controller, the address is not to be written backwards in the report
