This is the old summary / start page that has been in place before May 2012 and the GSoC project changes.


# Supporting the PlayStation Move controller on the PC #

In October/September 2010 Sony released the PlayStation Move – a motion-sensing game controller for the PlayStation 3. The Move controller is an excellent input device which can be used for very accurate input but all software development for PC support has to be done from scratch.

The project idea developed out of a thread at http://forums.motioninjoy.com/viewtopic.php?f=33&t=929.

Update 02-12-2011:
I was asked to add a link to another open source PS Move API project for Mac OS X and Linux - check out
http://thp.io/2010/psmove/

Update 16-12-2011: Another spin-off project for Windows that does not need the MotionInJoy driver but uses the Windows Bluetooth stack can be found at https://code.google.com/p/moveframework/

## Please add ideas, comments, code snippets! ##

Any help or advice is greatly appreciated!

In case you want to join or contribute to the moveonpc opensource project, send an email to _hkaufmann@gmail.com_.


## Tasks ##

This list should give an overview of the work which needs to be done to get Move support on the PC:

  1. Bluetooth interface/driver for the controller itself
  1. Eye camera driver
  1. Image processing: optical tracking of position using the glowing sphere
  1. Algorithms to calculate orientation from sensors
  1. Combination of data from the inertial sensors and optical tracking (data fusion)
  1. Interface for other software to read the data and configure 1–5


## Current status ##

Parts of the above tasks have already been completed. The [PS3 Move Tracker](PS3MoveTracker.md) (see screenshot below) demonstrates the currently available features in an interactive real-time demo. The following sections give detailed information on each task and its current status.

<img src='http://moveonpc.googlecode.com/svn/wiki/ps3movetracker_small.jpg' align='center' />

### 1. Bluetooth interface ###

Communication via Bluetooth is currently possible using MotioninJoy's filter driver and the Windows HID API. Sphere color can be changed, button presses detected and even controlling the rumble is possible. While this setup works fine so far, it has a few downsides: The MiJ driver doesn't allow direct access to the controller. It has to be polled constantly to receive data, and MiJ makes it impossible to directly access the Move's calibration data.

A small demo project for **native** device support exists in the SVN repository (project _nativemove_). However, initially setting up the BT connection withouth MiJ is currently very much trial and error. Extremely annoying and not very comfortable to use. This is the actual reason why the Windows implementation still depends on MiJ's filter driver. Once this connection issue is solved there are no more dependencies on MiJ!

On Linux there already is Bluetooth support for the Move (see linmctool at http://pabr.org/).

### 2. Camera driver ###

The _CL Eye Platform Driver_ for Windows is freely available at http://codelaboratories.com/.

On Linux the PlayStation Eye is natively supported since kernel version 2.6.29.

### 3. Image processing ###

Blob detection, circle fitting and calculation of the glowing sphere's X-Y-Z coordinates from camera images is implemented in the [PS3 Move Tracker](PS3MoveTracker.md). Although this works reasonably well there is much room for improvement. See the wiki page for details.

### 4. Orientation from inertial sensors ###

Data can be read from the controller via Bluetooth or USB (about 60 updates per second). See the wiki page on [the input report](InputReport.md) for details. The [PS3 Move Tracker](PS3MoveTracker.md) implements a very simple approach for calculating the orientation from the inertial sensors. A lot of improvement is needed here! Again, see the wiki page for details.

The data reported by the Move controller is (more or less) the raw sensor data. Every Move controller stores [calibration data](CalibrationData.md) in its internal Flash memory that must be used to correctly adjust the measured values. While some parts of this calibration data have been figured out already, there are still many unknowns. See the wiki page for details.

### 5. Data fusion ###

Good and reasonable data fusion is needed to further improve the accuracy of the the position data aquired in image processing. The [PS3 Move Tracker](PS3MoveTracker.md) uses information about position and orientation separately – no real fusion is done so far. Also lacking, but very important: a prediction filter to fill position gaps whenever the glowing sphere is occluded – with data acquired by the inertial sensor (dead reckoning).

### 6. Interface for other software ###

The interface itself is not such a problem, there is software used in the research community for about 10 years such as OpenTracker or VRPN (and a couple others such as GlovePie, ...) and there are standardized protocols to talk to software. However there will also be need for a (driver) GUI to configure parameters and that also has to be implemented.