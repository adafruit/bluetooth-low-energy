# Work-in-progress!

Please provide feedback via issues or PRs but do not build other things to these protocols yet.

# Adafruit Bluetooth Low Energy (BLE) Protocols

Bluetooth Low Energy (BLE) is a low-power wireless energy protocol. Adafruit provides sample code
and apps to work over BLE. This repo documents the BLE protocols used by this sample code and apps.

## IDs

### Assigned Names

The Bluetooth Special Interest Group (SIG for short) administers identifiers used in the protocol
by individual companies.

Adafruit's Service UUID is **`0xfebb`**. (Confirm [here](https://www.bluetooth.com/specifications/assigned-numbers/16-bit-uuids-for-members/)) It is currently unused.

Adafruit's Company ID is **`0x0822`**. (Confirm [here](https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/)) It is used in the manufacturer data field in advertisments. More info below.

### Base UUID

Adafruit's 128-bit Universally Unique Identifiers (UUID) all share a base UUID. This saves memory in
some BLE stacks because the base UUID is deduplicated.

Adafruit's base UUID is **`ADAFxxxx-C332-42A8-93BD-25E905756CB8`** where the four `xxxx` are replaced with the 16-bit IDs used below.

## Advertising

## Services

### Bluefruit Service

This is the first-gen Adafruit service that packs sensor data and commands into a Nordic UART Service.

### Sensor Services

Starting with the CircuitPlayground Bluefruit, sensor data is made available via Services to be read
and configured by external devices over a dedicated connection using [a standard
firmware](https://learn.adafruit.com/bluefruit-playground-app/firmware) which is an Arduino sketch.

These services share two common characteristics:

**Measurement Period** - `0x0001`
Stores the number of milliseconds between measurements as a signed 32-bit int. `-1` stops a
measurement and `0` indicates as fast as possible. Both readable and writable.

**Service Version** - `0x0002`
Readable uint32 version number. Assume 1 if the characteristic is missing.

The third characteristic is specific to the data being sensed. It is always readable and notifiable.
The table below includes the 16-bit UUID (combined with the base ID above) and a description of the
data format including units. This table mirrors the measurement types used in CircuitPython drivers
as defined [here](https://circuitpython.readthedocs.io/en/latest/docs/design_guide.html#sensor-properties-and-units).

| Property name         | Python type           | Units                                                                   |
| --------------------- | --------------------- | ----------------------------------------------------------------------- |
| ``acceleration``      | (float, float, float) | x, y, z meter per second per second                                     |
| ``magnetic``          | (float, float, float) | x, y, z micro-Tesla (uT)                                                |
| ``orientation``       | (float, float, float) | x, y, z degrees                                                         |
| ``gyro``              | (float, float, float) | x, y, z radians per second                                              |
| ``temperature``       | float                 | degrees centigrade                                                      |
| ``eCO2``              | float                 | equivalent CO2 in ppm                                                   |
| ``TVOC``              | float                 | Total Volatile Organic Compounds in ppb                                 |
| ``distance``          | float                 | centimeters                                                             |
| ``light``             | float                 | non-unit-specific light levels (should be monotonic but is not lux)     |
| ``lux``               | float                 | SI lux                                                                  |
| ``pressure``          | float                 | hectopascal (hPa)                                                       |
| ``relative_humidity`` | float                 | percent                                                                 |
| ``current``           | float                 | milliamps (mA)                                                          |
| ``voltage``           | float                 | volts (V)                                                               |
| ``color``             | int                   | RGB, eight bits per channel (0xff0000 is red)                           |
| ``alarm``             | (time.struct, str)    | Sample alarm time and string to characterize frequency such as "hourly" |
| ``datetime``          | time.struct           | date and time                                                           |
| ``duty_cycle``        | int                   | 16-bit PWM duty cycle (regardless of output resolution)                 |
| ``frequency``         | int                   | Hertz                                                                   |
| ``value``             | bool                  | Digital logic                                                           |
| ``value``             | int                   | 16-bit Analog value, unit-less                                          |
| ``weight``            | float                 | grams (g)                                                               |

#### Temperature Service - `0x0100`
 - **Temperature** - `0x0101`
Reads the current temperature in degrees Celsius as a 32-bit float. Both readable and notifiable.

#### Accelerometer Service - `0x0200`
 - **Acceleration** - `0x0201`
Reads the x, y, and z acceleration values in meters per second squared as three 32-bit floats. Both readable and notifiable.

#### Light Sensor Service - `0x0300`
 - **Light Level** - `0x0301`
Reads the current light level as a 32-bit float. Both readable and notifiable.

#### Gyroscope Service - `0x0400`
 - **Gyro** - `0x0401`
Reads the x, y, and z values in radians as three 32-bit floats. Both readable and notifiable.

#### Magnetometer Service - `0x0500`
 - **Magnetic** - `0x0501`
Reads the x, y, and z values in micro-Tesla (uT) as three 32-bit floats. Both readable and notifiable.

#### Board Button Service - `0x0600`
 - **Pressed** - `0x0601`
Reads the state of the buttons and switches as an unsigned 32-bit int.
   - **bit 0** is the slide switch with `1` representing left and `0` representing right.
   - **bit 1** is button A with `1` representing a pressed state. 
   - **bit 2** is button B with `1` representing a pressed state.
Other bits available for future buttons. Both readable and notifiable.

#### Humidity Sensor Service - `0x0700`
 - **Humid** - `0x0701`
Reads the relative humidity in percentage as a 32-bit float. Both readable and notifiable.

#### Barometric Pressure Sensor Service - `0x0800`
 - **Pressure** - `0x0801`
Reads the barometric pressure in hectoPascal (hPa) as a 32-bit float. Both readable and notifiable.

#### Addressable Pixel Service - `0x0900`
General service for any kind of addressable pixels, including WS2812 (NeoPixel), WS2801, APA102 (DotStar),
etc. It is the client’s responsibility to know the number of colors, color order, etc. and to fill the buffer
appropriately. The server is only concerned with the electrical protocol.

 - **Pixel Pin** - `0x0901`
Stores the pin to send data out on as an unsigned 8-bit int. Both readable and writable.
 - **Pixel Pin Type** - `0x0902`
Stores the type of pin to send data out on as an unsigned 8-bit int. `0` represents a WS2812 (NeoPixel)
running at 800Hz and `1` represents SPI-based APA201 (DotStar). More protocols to be added in
the future. Both readable and writable.
 - **Pixel Data** - `0x0903`
Writable-only variable length data that represents the pixel data.
   - **First part** is an unsigned 16-bit int that represents the byte-index (not pixel-index) for where to start
writing data into the buffer.
   - **Second part** is an unsigned 8-bit integer that represents the flags.
     - **bit 0** is when to write the data with `1` representing to immediately write to pixels and `0` representing
to not write pixel data yet.
   - **Third part** is the raw pixel data itself. This is a string of unsigned 8-bit ints with each byte representing the
red, green, and blue components of each addressable LED. The proper color order depends on the pixels themselves.

 - **Pixel Buffer Size** - `0x0904`
Stores the buffer size in bytes needed to hold data for entire pixel string as an unsigned 16-bit int. Both readable and writable.

#### Color Sensor Service - `0x0A00`
 - **Color** - `0x0A01`
Reads the Red, Green, and Blue values in radians as three unsigned 16-bit ints. Both readable and notifiable.

#### Sound (Microphone) Service - `0x0B00`
 - **Sound Samples** - `0x0B01`
 Reads the level sample values as an array of signed 16-bit ints. The length of this field varies by the sampling
 period. Both readable and notifiable.
 - **Number of Channels** - `0x0B02`
 Reads the number of channels as an unsigned 8-bit integer. Readable only.

#### Tone Service - `0x0C00`
 - **Tone** - `0x0C01`
 Instructs the device what frequency to play for the specified duration. Writable only.
    - **First part** is an unsigned 16-bit int that represents the frequency on hertz. Use 0 for no tone.
    - **Second part** is an unsigned 32-bit integer that represents the duration in milliseconds. Use 0 for non-stop.

#### Quaternion Service - `0x0D00`
 - **Quaternions** - `0x0D01`
Reads the W, X, Y, and Z quaternion values expressed as four 32-bit floats. Both readable and notifiable.
 - **Calibration In** - `0x0D02`
Reads the calibration data for the accelerometer, gyroscope, and magnetometer. The data is returned as nine 32-bit floats. 
Accel + Gyro + Magnetic data should only enable when performing sensor calibration and disable when done. Both 
readable and notifiable.
   - **First part** is three 32-bit floats representing the accelerometer's X, Y, and Z stored calibration values.
   - **Second part** is three 32-bit floats representing the gyroscope's X, Y, and Z stored calibration values.
   - **Third part** is three 32-bit floats representing the magnetometer's X, Y, and Z stored calibration values.
 - **Calibration Out** - `0x0D03`
Writeable only field with the calibration data for the accelerometer, gyroscope, and magnetometer. Calibrated data
is computed by mobile and written to nRF52 for saving to flash as result for calibration process. The expected data
is in the following parts:
   - **First part** is three 32-bit floats representing the accelerometer's **accel_zerog** values.
   - **Second part** is three 32-bit floats representing the gyroscope's **gyro_zerorate** values.
   - **Third part** is three 32-bit floats representing the magnetometer's **mag_hardiron** values.
   - **Fourth part** is one 32-bit float representing the magnetometer's **mag_field** value.
   - **Fifth part** is nine 32-bit floats representing the magnetometer's **mag_softiron** values.

#### Proximity Service - `0x0E00`
 - **Proximity** - `0x0E01`
 Reads the proximity as an unsigned 16-bit int. The higher number the closer object to sensor. Both readable and notifiable.
