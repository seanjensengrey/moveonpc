# Hardware #

## Extension Socket ##

On its bottom side, the Move controller features an 8-pin extension socket (labeled _EXT_ on the plastic casing) which allows external devices such as the [Sharp Shooter](SharpShooter.md) to be connected to the controller. The socket is mounted on a small separate board that is connected to the main board by a 9-pin flat ribbon cable.

<table align='center'>
<tr>
<blockquote><td><img src='http://wiki.moveonpc.googlecode.com/git/extension_port.jpg' /></td>
<td><img src='http://wiki.moveonpc.googlecode.com/git/connector_board.jpg' /></td>
</tr>
<tr>
<td>Move's EXT Port (with pins subsequently labeled)</td>
<td>Move's Connector Board</td>
</tr>
</table></blockquote>

While the socket looks very similar to a micro USB jack at first glance, it is something different. It is used by Nikon (and other manufacturers) for its digital cameras and goes by the name of _UC-E6_. Note that you will often find this connector mistakenly labeled as "mini USB 8 pin" or "micro USB 8 pin", even though it is _not_ a USB connector.

### Resources ###

  * <a href='http://wiki.moveonpc.googlecode.com/git/connector_board_schematic.pdf'>Connector Board schematic as PDF</a>


# Firmware #

For some reason, Sony chose not to lock access to the Move's firmware. You can either read/write the Flash memory by connecting a debugger to the JTAG or SWD pins on the Move's main board. Or you can, more comfortably, use the Device Firmware Update (DFU) interface which works over USB. To do this, you need to enable the Move's USB DFU mode.

## Enabling USB DFU mode ##

There are two ways to enabled USB DFU mode. Either use the [writable USB/BT Feature report 0xF2](WriteFeatureReport0xF2.md) or, if you enjoy tinkering with hardware, do the following:

**Warning**: This can break your controller if you short the wrong pins or something. Stick with the USB/BT approach if you want to play it safe. You have been warned!

Tie the [EXT port's](#Extension_Socket.md) pins 5 and 6 to GND, press and hold the PS button, and then reset the Move using the small Reset button on the backside of the controller. The easiest way to achieve the first part, is to use a thin paper clip or something similar that fits into the Move's EXT port. The socket's shield is connected to GND, so you can use the paper clip to connect pins 5 and 6 with it. Be sure not to short any other pins though! Use another paper clip or a sharp pen for pressing the Reset button.

Once the little red LED near the Move's USB socket starts flashing in an unfamiliar way, you are done. You can then remove the paper clip from the EXT port and release the PS button.

## Accessing Flash memory ##

In USB DFU mode, the Move will no longer identify as "Motion Controller" USB device but as "SCE STDFU 0200" instead (the number is the protocol revision). This new USB device's PID is 0x04DC.

Unfortunately, the MCU's vendor, STMicroelectronics, implemented its own flavour of the DFU protocol (called DfuSe) which breaks compatibility with the DFU standard. Luckily, this extended DFU is documented by ST and we even have an open-source tool which speaks DfuSe: [dfu-util](http://dfu-util.gnumonks.org).

Older versions of `dfu-util` were not yet able to handle DfuSe, but I tried the recent version 0.7 and that works like a charm.

Saving the Move's 128 kB Flash memory to file `move.bin` on your computer:
```
dfu-util -d 054c:04dc -U move.bin -a 0 -s 0x08000000:131072
```
You most certainly need to run this as root.

Writing the Flash works in a similar fashion, but I will not put this here for simple copy&paste since this is guaranteed to brick your Move if you do not know _exactly_ what you are doing. You have been warned!