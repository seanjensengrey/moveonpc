# Introduction #

The Move submits its calibration data in HID report 0x10. Unlike other reports from the controller, this one has to be read _three times_ to get everything. The results will typically look like this:

```
10 00 46 07 73 7f 9f 7f e8 8f 85 6e ca 7f f4 7e
55 7f f1 7f 1a 6e 98 90 0a 80 5b 7f 67 7f c9 90
4c 7f b7 7f e6 6e 06 7f 87 08 a5 7f 06 80 80 81
76

10 01 07 a8 7f 00 80 8b 81 00 00 00 00 00 00 00
01 00 01 92 08 e0 01 e5 94 e6 7f a1 81 e0 01 4b
7f d6 91 59 81 e0 01 a1 7f 2c 80 a6 95 89 07 36
0c

10 82 50 c3 5b c8 fb 42 68 3d b5 42 cd 13 7d 3f
93 5b 81 3f 70 ea 76 3f ff 80 4f 3f c6 ea 9c 3d
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00
```

The following section explains the report's format in detail, e.g. how to interpret the data. There is, however, a lot of guessing involved. If you figured out one of the missing parts, please let us know or add it to this page yourself! Raw calibration data read from different controllers can be found at bottom of this page.


# Details #

<img src='http://moveonpc.googlecode.com/svn/wiki/movelocalcoordinates.png' align='right' />

Fields that cover more than one byte are stored as little-endian, i.e. the least significant byte comes first. The same notation is used in the table below! Description of the axes assumes that the Move is held with the buttons pointing up and the glowing ball pointing away from you:
  * X-axis pointing to the right
  * Y-axis pointing up
  * Z-axis pointing away from you.

Since calibration data is split across three parts it has to be reassembled first. In each of the reports, the second byte's lower nibble identifies which of the three blocks we are dealing with. Note that the first report retrieved might as well be the last one in order. The following table assumes that the three reports are chained together in correct order, skipping the first two bytes (report ID and block index) of the second and third block!

| **Byte offset** | **Length (in bytes)** | **Description** |
|:----------------|:----------------------|:----------------|
| 0x00            | 1                     | HID Report ID (always 0x10) |
| 0x01            | 1                     | Index to keep blocks in order |
| 0x02            | 2                     | Temperature header (?) |
| 0x04            | 6                     | Accelerometer, orientation #1 (max. Y) |
| 0x0A            | 6                     | Accelerometer, orientation #2 (min. X) |
| 0x10            | 6                     | Accelerometer, orientation #3 (min. Y) |
| 0x16            | 6                     | Accelerometer, orientation #4 (max. X) |
| 0x1C            | 6                     | Accelerometer, orientation #5 (max. Z) |
| 0x22            | 6                     | Accelerometer, orientation #6 (min. Z) |
| 0x28            | 2                     | Temperature header (?) |
| 0x2A            | 6                     | Gyro bias (?)   |
| 0x30            | 2                     | Temperature header (?) |
| 0x32            | 6                     | Gyro bias (?)   |
| 0x38            | 7                     | all 0x00        |
| 0x3F            | 1                     | always either 0x00 or 0x01 (fixed for each controller) |
| 0x40            | 2                     | always 0x0100   |
| 0x42            | 2                     | Temperature header (?) |
| 0x44            | 2                     | always 0x01E0   |
| 0x46            | 6                     | Gyroscopes rotation X (80rpm) |
| 0x4C            | 2                     | always 0x01E0   |
| 0x4E            | 6                     | Gyroscopes rotation Z (80rpm) |
| 0x54            | 2                     | always 0x01E0   |
| 0x56            | 6                     | Gyroscopes rotation Y (80rpm) |
| 0x5C            | 2                     | Temperature header (?) |
| 0x5E            | 12                    | Unknown vector 1 (float) |
| 0x6A            | 12                    | Unknown vector 2 (float) |
| 0x76            | 4                     | Unknown value 1 (float) |
| 0x7A            | 4                     | Unknown value 2 (float) |
| 0x7E            | 17                    | all 0x00        |


## Index to keep blocks in order ##

The lower nibble of this byte indicates where to put this report in the reassembled block. It can either be 0x00, 0x01, or 0x82. Not sure what the additional high bit for the third blocks means. It probably only indicates that this is the last one.


## Temperature header (?) ##

These fields act as some kind of header for other data. Effectively, only 12 bits are used and also the values look like the temperature as seen in the [Input report](InputReport.md). So, this seems to indicate the temperature at which the following values were determined. This would also make sense since the internal sensors will change bias or gain depending on temperature.


## Accelerometer, orientation #n ##

Starting at offset 0x04, there are accelerometer values for 6 different orientations. (6-point tumble test, read more: http://www.gcdataconcepts.com/calibration.html) Order of the axes is X-Z-Y  for each orientation (see the [Input report](InputReport.md) to decode). From these values alone the bias vector and the gain matrix can be calculated with a multidimensional optimization algorithm, e.g. the downhill simplex algorithm.

Do not confuse them with the maximum output these sensors can report (e.g. by violently shaking the controller)!

The description in the above table contains information which axis is reporting minimum or maximum. The order is a little strange. Reading the Y-member of orientation #1 yields the maximum for the y-axis, the X-member of orientation #2 yields the minimum for the x-axis, and so on.

To further illustrate, these are the decoded min/max accelerometer values found in the sample calibration data at the top of this page.

| **Byte offset** | **X-member** | **Z-member** | **Y-member** |
|:----------------|:-------------|:-------------|:-------------|
| 0x04            | −141         | −97          | **4072**     |
| 0x0A            | **−4475**    | −54          | −268         |
| 0x10            | −171         | −15          | **−4582**    |
| 0x16            | **4248**     | 10           | −165         |
| 0x1C            | −153         | **4297**     | −180         |
| 0x22            |  −73         | **−4378**    | −250         |


## Vectors at 0x2A and 0x32 ##

Not sure what these are. Perhaps minimum and maximum resting noise, although these values are greater than the values I observed when reading gyroscope data from a static controller.

Maybe they are something temperature related.

But there is one sure thing: if we subtract them from the gyro readings, we get very accurate values for 80 rpm rotation. (I dont know which one to subtract, but there is not such a big difference.)


## Byte at 0x3F ##

No idea what (if anything) this means. It is 0x00 for 6 out of 8 different controllers and 0x01 for the remaining two.


## Gyroscopes ##

These are 3 vectors, 3 elements each, 2 bytes per element, each vector preceeded by 0xE001 (in little-endian). The order is X-Z-Y again.

These are 80 rpm readings, but we have to subtract one of the vectors at 2A or 32 to get the exact values.

An accurate gain matrix can be calculated from these measures, also with a multidimensional optimization algorithm.

The bias must be compensated by the orientation algorithm, it's probably not part of the calibration data, because it always drifts.

## Unknown vector 1 at 0x5E ##

This is a vector, each element coded in float32 like this: http://en.wikipedia.org/wiki/Single_precision_floating-point_format

The order is probably in X-Z-Y, as anywhere else.

I don't know, what it is, but there are negative and positive values as well, so it can be a bias. But it's not even near to the gyro, acc or mag biases I measured.


## Unknown vector 2 at 0x6A ##

Coded as the one before. The values are very near to 1.0.

I observed, that when I multiply this vector with the gyro bias vector before subtracting from the gyro 80rpm measures, I get a better calibration.

So in order to get the accurate 80rpm measures:
GyroMeasure80rpm-(GyroBias1\*UnknownVector2)
or
GyroMeasure80rpm-(GyroBias2\*UnknownVector2)


## Unknown values at 0x76 and 7A ##

Coded in float32, as the elements of the previous vectors.

No idea, what these values can be. They are similar values in each calibration data we have (?).


# Calibration data #

This section contains calibration data read from 8 different Move controllers. Feel free to add more.

```
10 00 6d 08 5d 7f fd 7e fd 8f af 6e cf 7e ec 7e
ea 7f ca 7e 23 6e b3 90 11 7f 52 7f be 7f c1 8f
24 7f c8 7f cc 6d 4c 7f c4 08 02 7e 5d 80 02 80
85 07 06 7e 59 80 09 80 00 00 00 00 00 00 00 00
00 01 c1 08 e0 01 d8 92 8c 80 0d 80 e0 01 fa 7d
af 8f c4 7f e0 01 18 7e 8e 80 39 95 9b 07 10 ef
69 41 67 56 be 40 d2 5a bc c2 5d cf 76 3f 79 f8
7c 3f 1e d9 74 3f 12 37 3e 3f bd cd 0b 3e 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 46 07 73 7f 9f 7f e8 8f 85 6e ca 7f f4 7e
55 7f f1 7f 1a 6e 98 90 0a 80 5b 7f 67 7f c9 90
4c 7f b7 7f e6 6e 06 7f 87 08 a5 7f 06 80 80 81
76 07 a8 7f 00 80 8b 81 00 00 00 00 00 00 00 01
00 01 92 08 e0 01 e5 94 e6 7f a1 81 e0 01 4b 7f
d6 91 59 81 e0 01 a1 7f 2c 80 a6 95 89 07 36 0c
50 c3 5b c8 fb 42 68 3d b5 42 cd 13 7d 3f 93 5b
81 3f 70 ea 76 3f ff 80 4f 3f c6 ea 9c 3d 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 30 08 69 7f b4 7f 63 90 b1 6e 84 7f 41 7f
00 80 8f 7f 68 6e dd 90 d8 7f b2 7f cf 7f 82 90
6c 7f e5 7f 9b 6e a9 7f ed 08 4d 7f 48 80 44 80
ba 07 3e 7f 4d 80 45 80 00 00 00 00 00 00 00 00
00 01 ed 08 e0 01 70 93 59 80 6c 80 e0 01 53 7f
20 8f 40 80 e0 01 bf 7f 4c 80 17 95 df 07 ff c9
4a c3 e9 84 c3 41 a1 fd 5f 42 27 bc 72 3f 9f 5f
7d 3f 60 99 75 3f 7e 3f d0 3f 15 d3 11 3e 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 d0 07 5c 7f a7 7e c1 8f 72 6e 9e 7e 75 7e
69 7f c4 7e e3 6d 87 90 f6 7e 59 7f 45 7f cd 8f
33 7f 8e 7f b1 6d d3 7e 28 09 20 80 42 80 fc 7f
27 08 1f 80 45 80 03 80 00 00 00 00 00 00 00 01
00 01 24 09 e0 01 bf 93 6f 80 13 80 e0 01 c2 7f
33 92 e3 7f e0 01 13 80 65 80 bf 94 40 08 3d 21
41 c3 28 56 ff 41 64 48 68 42 eb 8c 70 3f a2 8b
7b 3f 34 ae 71 3f 8c d7 82 3f 1a d8 56 3e 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 84 07 ef 7e 30 7f cd 90 06 6e 0e 7f c5 7f
23 7f 00 7f 7c 6e 43 90 5d 7f c2 7f 21 7f 20 90
ed 7f 50 7f e5 6d cd 7f bf 08 0f 7f 1c 80 47 7f
b4 07 11 7f 1c 80 4e 7f 00 00 00 00 00 00 00 00
00 01 bf 08 e0 01 21 95 fe 7f 28 7f e0 01 0d 7f
91 90 24 7f e0 01 4a 7f 19 80 0f 93 d6 07 68 55
02 c3 00 2b d6 c2 b1 31 76 c1 1b 5d 72 3f ed 13
7a 3f 60 8a 73 3f 63 53 29 3f 21 5a b9 3d 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 cb 07 61 7f c7 7f ba 8f 65 6e 10 80 e5 7e
7e 7f f0 7f d3 6d 90 90 d9 7f db 7e 98 7f 93 90
ff 7e 55 7f 23 6f b9 7e 0d 09 61 7f 52 80 25 80
10 08 5e 7f 52 80 29 80 00 00 00 00 00 00 00 00
00 01 09 09 e0 01 27 96 5d 80 3c 80 e0 01 87 7f
a3 90 16 80 e0 01 7f 7f 4b 80 fa 94 23 08 46 d9
a2 43 04 43 87 43 e2 14 be c2 f8 d6 72 3f 07 8b
80 3f 32 1f 74 3f b8 a6 fd 3f 8e e0 68 3e 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 b3 07 af 7f 96 7f 36 90 dc 6e 7e 7f 11 7f
cf 7f c7 7f fd 6d cd 90 0e 80 7e 7f 8e 7f ab 90
a4 7f 1d 80 d8 6e ea 7e ed 08 97 7f 3d 80 c6 80
d4 07 97 7f 3f 80 d8 80 00 00 00 00 00 00 00 00
00 01 ec 08 e0 01 59 93 3f 80 d9 81 e0 01 a0 7f
de 8e de 80 e0 01 d2 7e 3f 80 36 95 f9 07 a8 c2
d5 c2 42 d5 46 43 a6 b4 93 41 c1 f7 72 3f 97 e9
80 3f 6f d8 71 3f ff f2 6a 3f d3 04 f3 3d 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
10 00 ab 07 a8 7f a3 7f ca 90 89 6e 98 7f 88 7f
9c 7f b3 7f 6c 6e 98 90 f8 7f b4 7f 6a 7f b4 90
bf 7f ae 7f a9 6e 8c 7f 1f 09 0c 7f cf 7f fb 7f
17 08 03 7f b5 7f fe 7f 00 00 00 00 00 00 00 00
00 01 1c 09 e0 01 3d 97 cc 7f fe 7f e0 01 37 7f
ff 8f 0e 80 e0 01 00 7f c9 7f f5 93 36 08 41 13
13 40 26 a1 7e 42 3b 2f f6 42 54 dc 73 3f de d3
7d 3f 9f 9c 72 3f 7d a3 56 3f f1 85 b9 3d 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```
