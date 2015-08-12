| **Byte offset** | **Description** |
|:----------------|:----------------|
| 0x00            | HID Report ID (always 0xF7) |
| 0x01            | battery OSD status (same as [battery level in Input Report](InputReport#Battery_level.md)) |
| 0x02            | lower byte of battery voltage (as measured by the ADC) |
| 0x03            | higher byte of battery voltage (as measured by the ADC) |
| 0x04            | lower byte of [temperature value](InputReport#Temperature.md) (raw ADC value plus adjustable offset) |
| 0x05            | higher byte of [temperature value](InputReport#Temperature.md) (raw ADC value plus adjustable offset) |
| 0x06            | `0x00` if output pin `PC12` is set, `0x01` otherwise |
| 0x07            | `0x01` if input pin `PC10` is set, `0x02` if input pin `PC11` is set, `0x03` if both are set, `0x00` otherwise |
| 0x08            | always `0x00`   |
| 0x09            | always `0x00`   |
| 0x0A            | always `0x00`   |
| 0x0B            | always `0x00`   |

# bq24080 Li-Ion charger IC #

The MCU's output pin `PC12` is connected to the _charge enable_ input of the bq24080 Li-Ion charger IC. Pulling this input low enables charging the battery. Byte `0x06` in the Feature Report will thus be zero if no charging is in progress.

The MCU's input pins `PC10` and `PC11` are connected to the bq24080's charge status outputs. The following table translates the values in the Feature Report into the corresponding status:

| **Report's byte 0x07** | **Charge status** |
|:-----------------------|:------------------|
| `0x00`                 | Precharge in progress |
| `0x01`                 | Charge done       |
| `0x02`                 | Fast charge in progress |
| `0x03`                 | Timer fault or sleep mode |
