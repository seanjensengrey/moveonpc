[Extension devices](ExtensionDevices.md) initially send [config data](ExtensionDevices#Initialization_and_configuration.md) which, among other things, contains a device ID for identifying the different possible extension devices. This feature lets you retrieve this data.

Retrieving data from an extension device requires two steps: sending report 0xE0 with the desired read configuration, and then reading report 0xE0 to get the actual data.

# Sending the report #

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0xE0) |
| 0x01            | 1                     | Mode selector   |
| 0x02            | 1                     | Target extension device's I²C slave address |
| 0x03            | 1                     | Offset          |
| 0x04            | 1                     | Length          |
| 0x05            | 44                    | _unused_ (set it to zeros) |

Setting the _Mode selector_ != 0 sets up a read operation of _Length_ bytes starting at _Offset_ in the 256 bytes of the [extension device's config data](ExtensionDevices#Initialization_and_configuration.md). The I²C slave address for Sony's official extension devices is always `0xA0`.

Since the data buffer in the read result (see next section) is limited to 40 bytes, it does not make any sense to specify _Length_ > 40. If you do, it will internally be limited to 40.

# Reading the report #

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0xE0) |
| 0x01            | 1                     | Error flag      |
| 0x02            | 1                     | Target extension device's I²C slave address |
| 0x03            | 1                     | Offset          |
| 0x04            | 1                     | Length          |
| 0x05            | 4                     | _unknown_       |
| 0x09            | 40                    | Data            |

The _Error flag_ will be zero if the operation was successful, otherwise it will be non-zero (if no extension device is connected, for example). The _Slave address_, _Offset_, and _Length_ fields will contain the values specified in the previous sending of this report. Starting at offset `0x09` is the actual data retrieved from the extension device. If less than 40 bytes have been requested, the remaining bytes will be zero.
