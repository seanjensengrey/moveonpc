The RGB LEDs in the Move controller are dimmed using pulse-width modulation (PWM). This Feature report lets you modify the PWM frequency. The default is around 188 kHz and can also be restored by resetting the controller (using the small reset button on the back). The report is structured as follows:

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0x03) |
| 0x01            | 1                     | Magic value (always 0x41) |
| 0x02            | 1                     | Command byte    |
| 0x03            | 4                     | Frequency (32-bit unsigned integer, Little-Endian) |

The command byte lets you either set the PWM frequency to one of 4 default values or to a custom frequency. The following values are possible:

| **Command byte** | **Function** |
|:-----------------|:-------------|
| 0x00             | set PWM frequency to custom frequency |
| 0x01             | set PWM frequency to 153.6 kHz |
| 0x02             | set PWM frequency to 230.4 kHz |
| 0x03             | set PWM frequency to 307.2 kHz |
| 0x04             | set PWM frequency to 384.0 kHz |

If the custom mode is selected, the desired PWM frequency must be specified in the report at offset `0x03`. The value is interpreted by the Move as 32-bit unsigned integer in Little-Endian byte order (i.e. least significant byte first). The valid range is 733 Hz to 24 MHz.

**Note**: Even though the controller lets you increase the frequency to several Megahertz, there is usually not much use in operating at such extreme rates. Additionally, you will even lose resolution at these rates, i.e. the number of distinct LED intensities between "off" and "fully lit" decreases. For example: at 7 MHz there are only 5 different intensities left instead of the usual 256. This is not a feature of PWM per se but is rather due to software/hardware limitations of the Move controller.

**Note**: Make sure to switch off the LEDs prior to sending the report. If you do not do this, changing the PWM frequency will switch off the LEDs and keep them off until the Bluetooth connection has been teared down and then reestablished.
