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

+-----------------------+-----------------------+-------------------------------------------------------------------------+
| Property name         | Python type           | Units                                                                   |
+=======================+=======================+=========================================================================+
| ``acceleration``      | (float, float, float) | x, y, z meter per second per second                                     |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``magnetic``          | (float, float, float) | x, y, z micro-Tesla (uT)                                                |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``orientation``       | (float, float, float) | x, y, z degrees                                                         |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``gyro``              | (float, float, float) | x, y, z radians per second                                              |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``temperature``       | float                 | degrees centigrade                                                      |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``eCO2``              | float                 | equivalent CO2 in ppm                                                   |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``TVOC``              | float                 | Total Volatile Organic Compounds in ppb                                 |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``distance``          | float                 | centimeters                                                             |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``light``             | float                 | non-unit-specific light levels (should be monotonic but is not lux)     |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``lux``               | float                 | SI lux                                                                  |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``pressure``          | float                 | hectopascal (hPa)                                                       |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``relative_humidity`` | float                 | percent                                                                 |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``current``           | float                 | milliamps (mA)                                                          |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``voltage``           | float                 | volts (V)                                                               |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``color``             | int                   | RGB, eight bits per channel (0xff0000 is red)                           |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``alarm``             | (time.struct, str)    | Sample alarm time and string to characterize frequency such as "hourly" |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``datetime``          | time.struct           | date and time                                                           |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``duty_cycle``        | int                   | 16-bit PWM duty cycle (regardless of output resolution)                 |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``frequency``         | int                   | Hertz                                                                   |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``value``             | bool                  | Digital logic                                                           |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``value``             | int                   | 16-bit Analog value, unit-less                                          |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
| ``weight``            | float                 | grams (g)                                                               |
+-----------------------+-----------------------+-------------------------------------------------------------------------+
