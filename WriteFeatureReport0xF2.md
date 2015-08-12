Using this USB/BT Feature report, you can set the Move controller's operation mode. The first byte, as usual, must be the report ID. The second byte controls what to do. The following values are possible:

| **Control byte** | **Function** |
|:-----------------|:-------------|
| 0x41             | _unknown_    |
| 0x42             | set USB DFU mode |
| 0x43 or 0x44     | set BT DFU mode |
| 0x45             | _unknown_    |
| 0x46             | _unknown_    |
| 0x47             | _unknown_    |
| 0x48             | _unknown_    |
| 0x49             | _unknown_    |

After changing the operation mode, the small red LED near the Move's USB socket will start blinking in a certain pattern to indicate the current operation mode:

| **Operation mode** | **LED sequence** |
|:-------------------|:-----------------|
| default            | always ON (if connected to BT host) |
| USB DFU            | 0101111111       |
| BT DFU             | 0111111111       |

In the above table, a `1` indicates that the LED is lit while a `0` indicates that the LED is off. The patterns repeat continuously. So, for instance, in USB DFU mode, the LED will be on most of the time and only shortly flash twice in a row.

The most useful function, at the moment, is setting USB DFU mode since this allows you to comfortably [access the Move's firmware](HardwareAndFirmware#Firmware.md) via USB.

Changing the operation mode will make the Move identify as different USB device. In USB DFU mode, its PID will be 0x04DC and it appears as "SCE STDFU 0200". In BT DFU mode, its PID will be 0x03D6 and it appears as "SCE BTDFU 0200".