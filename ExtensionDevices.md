# Introduction #

The Move features an [Extension Socket](HardwareAndFirmware#Extension_Socket.md) (EXT) which allows additional devices to be connected and communicate with the Move using the [I²C](http://en.wikipedia.org/wiki/I%C2%B2C) interface, with the Move acting as _master node_ and a bus speed of 400 kbit/s (Fast Mode). This socket also acts as power supply for the extension device.

These are the known official extension devices for the Move:

  * [Sharp Shooter](SharpShooter.md)
  * [Racing Wheel](RacingWheel.md)


# Hardware #

An extension device is connected to the Move via the [Extension Socket](HardwareAndFirmware#Extension_Socket.md) on the Connector Board. Interestingly, the official extension devices use a 6-pin plug, so the socket's first and last pin are not used.

<table align='center'>
<tr>
<blockquote><td><img src='http://wiki.moveonpc.googlecode.com/git/extension_port.jpg' /></td>
<td width='360'><img src='http://wiki.moveonpc.googlecode.com/git/sharpshooter_extension_plug.jpg' /></td>
</tr>
<tr>
<td>Move's EXT Port (with pins subsequently labeled)</td>
<td>Sharp Shooter's EXT Plug (with pins subsequently labeled)</td>
</tr>
</table></blockquote>

| **[Move's EXT pin (J15 in schematic)](HardwareAndFirmware#Extension_Socket.md)** | **Device's EXT pin** | **Function** |
|:---------------------------------------------------------------------------------|:---------------------|:-------------|
| 1                                                                                | —                    | —            |
| 2                                                                                | 1                    | VCC (3.2 V)  |
| 3                                                                                | 2                    | ~ENABLE (signals the Move that an EXT device was plugged in/out) |
| 4                                                                                | 3                    | _unknown_    |
| 5                                                                                | 4                    | I²C Clock (SCL) |
| 6                                                                                | 5                    | I²C Data (SDA) |
| 7                                                                                | 6                    | GND          |
| 8                                                                                | —                    | —            |

The Move's EXT interface is not enabled by default, presumably to save battery power. It will only be switched on if the Move is connected to a host via Bluetooth and if its ~ENABLE pin is pulled to GND. By default, it is pulled high by the Move's internal pull-up resistors. The Sharp Shooter and Racing Wheel have this pin hard-wired to GND, so the EXT interface will be enabled as soon as these extension devices are plugged into the Move. Unplugging them will disable the EXT interface again.


# Protocol #

The descriptions in this section assume that you are familiar with I²C. The following notation will be used:

| **Symbol** | **Description** |
|:-----------|:----------------|
| S          | Start Condition |
| Rs         | Repeated Start Condition |
| P          |  Stop Condition |
| SLA        | 7-bit Slave Address |
| R/W        | Read/Write bit  |
| A/A̅       | ACK/NACK        |


## Initialization and configuration ##

After enabling the EXT interface, the Move will perform some sort of reset sequence on the I²C bus: it will toggle SCL a couple of times at relatively low speed while SDA is HIGH, thus issuing several Stop Conditions. After a short delay, the actual I²C communication starts at full speed:

```
[ S ][ SLA+W ][ A ][ Data ][ A ][ Rs ][ SLA+R ][ A ][ Data ... ][ A ][ Data ][ A̅ ][ P ]
        0xA0         0x00                0xA1       |      255 ×      8 bits     |
                                                    | (8 bits + ACK)             |
                                                    |    device's config data    |
```

The Move requests 256 bytes from feature `0x00` of the extension device which must answer to slave address `0xA0` – the read/write bit is included as LSB in this notation, so the read address is `0xA1` and the write address is `0xA0`.

These 256 bytes of data allow the extension device to configure the Move. It can also be retrieved by the host (the PS3, for example) by using [Feature Report 0xE0](FeatureReport0xE0.md). The data is organized in 3 blocks:

| **Byte offset** | **Name** | **Description** |
|:----------------|:---------|:----------------|
| 0x00            | _ExtInfo_ | extension device ID, remaining data _unknown_ |
| 0x40            | _ExtOut_ | config data for receiving from the Move |
| 0xA0            | _ExtIn_  | config data for sending to the Move |


### ExtInfo ###

This part of the configuration does not seem to influence the following exchange of data between Move and extension device (see next sections) at all. No resulting changes could be observed when manipulating the data in our experiments.

The first two bytes form a device ID which allows the host (the PS3) to identify the different extension devices. Using a list of known IDs, the host can determine which extension device exactly is currently connected. The Move itself does not check this ID.


### ExtOut ###

This part of the configuration gives the Move blobs of arbitrary data, each of which to send only once to an arbitrary device on the same I²C bus as the extension device. This is done after configuration data has been received and before the periodical reading of feature data from the extension device starts.

The _ExtOut_ block is a list of variable-length descriptor items, each item comprising the following fields:

| **Byte offset** | **Name** | **Description** |
|:----------------|:---------|:----------------|
| 0x00            | _slaveAddr_ | target I²C slave address |
| 0x01            | _featureId_ | numerical identifier of the feature |
| 0x02            | _dataLen_ | number of bytes in the next field |
| 0x03            | _data_   | _dataLen_ number of bytes |

The byte following the last descriptor item must be set to `0x00` so that the Move knows where to stop reading.

The _slaveAddr_ field tells the Move which slave on the I²C bus to send the data to. The Move expects an 8-bit address, i.e. the LSB is assumed to be the read/write bit and should be set to `0`.

The _featureId_ field contains the first byte of data to send to the target slave device. It is not treated differently from the following data to be sent.

The _dataLen_ field tells the Move how many more bytes of data this descriptor item contains. The valid range for this value is `0` ≤ <i>dataLen</i> ≤ `0x28`.

The _data_ field contains _dataLen_ number of bytes which the Move should send to the target slave device.


### ExtIn ###

This part of the configuration tells the Move which data to request from the extension device and what to do with it.

In order to make the extension device's feature data (e.g. the Sharp Shooter's button states) available via Bluetooth, these values are later mapped to specific positions in the Move controller's [Input Report](InputReport.md). In the _ExtIn_ config data the extension device announces a list of its available features and the associated mappings.

The _ExtIn_ block is an array of descriptor items, each item having a fixed size of 5 bytes:

| **Byte offset** | **Name** | **Description** |
|:----------------|:---------|:----------------|
| 0x00            | _slaveAddr_ | target I²C slave address |
| 0x01            | _featureId_ | numerical identifier of the feature |
| 0x02            | _dataLen_ | number of bytes in response |
| 0x03            | _mergeMode_ | how to merge into the Input Report |
| 0x04            | _dstOffset_ | where to merge into the Input Report |

The block size of 96 bytes limits the number of descriptor items (and thus the number of readable features) to 19. The byte following the last descriptor item must be set to `0x00` so that the Move knows where to stop reading.

The _slaveAddr_ field tells the Move which slave on the I²C bus to ask for the specified feature. The Move expects an 8-bit address, i.e. the LSB is assumed to be the read/write bit and should be set to `0`.

The _featureId_ field uniquely identifies each readable feature offered by the extension device. The Move will use this number to request a certain feature.

The _dataLen_ field tells the Move how many bytes to expect as answer to this feature request.

The _mergeMode_ field sets the operator used for merging the feature data with the regular data in the Input Report:

| **Value** | **Description** |
|:----------|:----------------|
| `0x00`    | NOP (do not merge) |
| `0x01`    | OR              |
| `0x02`    | AND             |
| `0x03`    | XOR             |
| `0x04`    | COPY (overwrites existing value) |

The byte offset given by _dstOffset_ is relative to the **second** byte in the Move's [Input Report](InputReport.md). Since the first byte always contains the fixed Report ID, writable data starts at the second byte. The Input Report buffer size limits valid offsets to the range `0` ≤ <i>dstOffset</i> ≤ `0x2F`.


## Sending data ##

Following initialization and configuration is the actual data exchange between Move and extension device.

If at least one non-zero descriptor item was specified in the _ExtOut_ config, the Move will send the data given in the desciptors to the extension device. It sends this data only once. It does not expect the extension device to send back any response (apart from ACKs, of course).

It is hard to say what this functionality was intended for since none of the official extension devices uses this feature. But it could actually be used to make the Move send some custom init data to the EXT device or any other I²C device (on the same bus as the extension device) of our choice.
Think of something that needs to be woken up or configured once before it starts proper operation. So, this block seems to be nothing but input for some kind of I²C "echo" service.


## Reading feature data ##

The Move then enters polling mode where it periodically requests the features specified in the _ExtIn_ config from the extension device. It reads back the responses and merges them into its own [Input Report](InputReport.md) as detailed in the _ExtIn_ section above.

I²C communication for a single feature request looks as follows (with N as _dataLen_, i.e. the number of bytes to report back):

```
[ S ][ SLA+W ][ A ][ Data ][ A ][ Rs ][ SLA+R ][ A ][ Data ... ][ A ][ Data ][ A̅ ][ P ]
                    8 bits                          |      N-1 ×      8 bits     |
                  Feature ID                        | (8 bits + ACK)             |
                                                    |    device's feature data   |
```

The list of features is processed in the order in which they appeared in the _ExtIn_ config.

The frequency of polling varies slightly. For instance, an update to all of the Sharp Shooter's 6 features is performed approximately every 11–13 ms.
