Uni-T UT70B Communication Protocol 
-------
The Uni-T UT70B supports a standard RS-232 connection and connects at 2400 baud with a character size of 7 bits, odd parity, and 1 stop bit (2400/7-O-1).  Individual measurements are transmitted in an 11 byte long data frame.  Each byte in the data frame is ASCII encoded unless otherwise noted and is decoded simply by subtracting 48 from its value.

Data Frame Structure
--------
| Byte | Value |
|----- | ------ |
| 0 | Multiplier, expressed as the exponent of a power of 10 |
| 1-4 | Raw data value |
| 5 | DMM Mode |
| 6 | Mode specific flags, sign and overload |
| 7 | Unused?, always 0 |
| 8 | Mode specific flags and autoranging |
| 9 | Carriage return, *no encoding* |
| 10 | Line feed, *no encoding* |

Bytes 9,10 can be safely ignored other than to determine if a complete frame has been read.  Currently, byte 7 can also be ignored unless its usage is ever determined.

**PLEASE NOTE:** All values described below are decoded values.

Multiplier - Byte 0
--------------------
Many measurement modes are capable of measuring values across several orders of magnitude.  To accomodate this range of values, the raw value must be scaled by a power of 10.  Byte 0 represents the exponent.  To apply this value:

``` scaled value = raw value * 10^byte0 ```

Raw Data - Bytes 1-4
---------------------
Each of the 4 bytes that make up the raw value is a BCD digit, with byte 1 being the most significant and byte 4 being the least significant.  The value can be extracted as follows:

```raw value = (byte1 - 48)*10^3 + (byte2 - 48)*10^2 + (byte3 - 48)*10 + (byte4 - 48).```

Note that this value *does not* contain a sign or the location of the decimal.

Multimeter Modes - Byte 5
---------------------------------
Most of the modes supported by the multimeter are denoted by a unique ID in byte 5, with three exceptions.  Temperature and Frequency have "submodes" which are detailed under Flags - Byte 6 and AC/DC measurements detailed in Flags - Byte 8.

| Value | Mode |
| --------| ------- |
| 0x1 | Diode testing |
| 0x2 | Frequency measurement |
| 0x3 | Resistance |
| 0x4 | Temperature |
| 0x5 | Continuity testing |
| 0x6 | Capacitance |
| 0x9 | Current in milliamps |
| 0xB | Voltage |
| 0xD | Current in microamps |
| 0xF | Current in amps |

Flags - Byte 6
---------------
Byte 6 contains flags that are applicable to most modes and one that is specific to two different modes.

| Bit | Flag | Description |
| --- | ---- | ----------- |
| 0 | Overload | This bit is set anytime the multimeter has measured a value outside of the range selected (if manual ranged) or outside of its operating specifications (if autoranged).  If this flag is set, the measurement value is considered invalid | |
| 1 | *Unused* | |
| 2 | Sign | 0 if the measured value is positive <br/> 1 if negative |
| 3 | Shared | Mode specific, see below |
|   | Temperature | 0 if meter is set to measure in degrees Fahrenheit <br/> 1 if set to measure in degrees Celsius |
|   | Frequency | 0 if meter is set to measure frequencies <br/> 1 if meter is set to measure RPMs |
| 4 | *Unused* | |
| 5 | *Unused* | |
| 6 | *Unused* | |
| 7 | *Unused* | |

Flags - Byte 7
---------------
The usage of this byte is unknown and is currently considered unused.  It could be used for a low battery indicator or for additional flags that may not be supported by this meter, though this hasn't been verified.
This byte appears to always be zero.

Flags - Byte 8
---------------

| Bit | Flag | Description |
| --- | ---- | ----------- |
| 0 | *Unused* | |
| 1 | Autorange | Set if the meter is in autorange mode. |
| 2 | AC | 1 only if meter is set to measure AC Voltage or Current |
| 3 | DC | 1 only if meter is set to measure DC Voltage or Current |
| 4 | *Unused* | |
| 5 | *Unused* | |
| 6 | *Unused* | |
| 7 | *Unused* | |

Additional Scaling
-------------------
Some modes measure values that are much smaller than can be represented in the raw value passed by the multimeter which results in measured values that are larger than the actual value.  To correct for this behavior:

``` final scaled value = scaled value * mode specific correction ```

The values for each mode are:

| Mode | Mode Specific Correction |
| ---- | --------------------- |
| Voltage | 1.0e-4 |
| Resistance | 0.1 |
| Diode | 1.0e-3 |
| Capacitance | 1.0e-12| 
| Current (uA) | 1.0e-7 |
| Current (mA) | 1.0e-5 |
| Current (A) | 0.01 |

Final Notes
-------------
This specification may contain errors and is most likely incomplete.  If you find any errors or omissions, please submit the needed corrections so this document can be updated.

