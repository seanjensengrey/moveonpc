# Introduction #

The PS3 is using an authentication system to identify legit Move controllers. It is implemented as some kind of challege-response scheme using the BT HID Feature reports `0xA0` and `0xA1`.

# Details #

_Note_: The following descriptions of BT reports assume that the first byte is always set to the report ID and is excluded from the stated byte count.

A few seconds after BT connection is established, the PS3 sends 34 random bytes in Feature report `0xA0` as challenge to the Move. A couple of seconds later, it reads back the 22 byte response in Feature report `0xA1`. If the Move did not answer correctly, the PS3 closes the connection and reports error 8012170C. Note that there seems to be only one correct response to each challenge. All legit Move controllers will give the same answer if queried with an identical challenge.

This authentication process appears to be implemented directly in the BT module since communication between the STM32 and the BT module does not contain any trace it.
