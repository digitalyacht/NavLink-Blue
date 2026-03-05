# NavLink Blue

Welcome to our NavLink Blue Developer's Guide (SDK) on GitHub. This guide is intended to help developers quickly get to grips with our NavLink Blue Low Energy Wireless NMEA2000 Gateway and implement it within their Apps.
<br>
<img src="https://github.com/digitalyacht/NavLink-Blue/blob/main/Images/NAVLinkBlue_with_cable_on_White.jpg" width=70%>
<br>
The NavLink Blue NMEA2000 Gateway is designed to allow an Application Developer to support bi-directional communication with an NMEA2000 network, through simple serial messages. Any individual or company that wishes to integrate NavLink Blue will need as a minimum the NMEA2000 Appendix A+B which is available from [http://nmea.org](http://nmea.org) for both NMEA members and non-members.

NavLink Blue is based on our existing NAVLink2 and iKonvert wired NMEA2000 gateways and they share the exact same serial protocol, so once you support one device, supporting the others is easy.

<img src="https://github.com/digitalyacht/NavLink-Blue/blob/main/Images/Data_Monitor.png" width=70%>

Included in this repository are:-

*1.  Developers Application Note on NAVLink Blue*

*2.  The latest NavLink Blue firmware*

*3.  The NavLink 2 Developers Guide V1.04 (pdf)*

*4.  Latest User Manual for NavLink Blue*

No special libraries are required to integrate NavLink Blue and it supports BTLE.

A full developer's guide for iKonvert, much of which is applicable to NavLink 2 is provided in the [associated Wiki](https://github.com/digitalyacht/iKonvert/wiki).

# NAVLink Blue – BLE Gateway for NMEA 2000

NAVLink Blue is an ESP32 firmware that bridges NMEA 2000 (CAN bus) with Bluetooth Low Energy clients, plus Wi-Fi web tools for configuration and updates.

## Core Features

- Full-duplex NMEA 2000 ↔ BLE data path
- BLE server mode and BLE sensor client mode
- PGN filtering by category and by single PGN
- Embedded web UI, REST API, WebSocket events
- OTA firmware update and SPIFFS web asset update
- Device and configuration persistence in NVS

## BLE Service and Characteristics

- Service UUID: `0xABF0`
- Characteristics:
  - `ABF2` (`0xABF2`): NMEA stream out (notify) + client writes to send NMEA data to bus
  - `ABF3` (`0xABF3`): command channel (ASCII commands)
  - `ABF4` (`0xABF4`): additional notify/read characteristic (kept for compatibility but not active)

In BLE server mode, advertising is connectable and automatically restarted after a successful connection when free slots are available.

## BLE Modes

BLE server (mobile/tablet app connects to NAVLink Blue)
BLE client (NAVLink Blue scans/connects to saved sensors)

Supported sensor integrations include RuuviTag, Mopeka, Calypso, and Minew MSL01.

## Calypso Wind Behavior (latest)

- Calypso wind notifications update wind speed, direction, and battery.
- If no Calypso wind message is received for more than 5 seconds, NAVLink Blue sends wind data as unavailable (`0xFF` bytes for wind speed and angle fields in PGN 130306 output payload).
- Battery instance is set to 0x10. 

## ABF3 Command Set

Write commands as ASCII text to `ABF3`, with mandatory prefix:

`$PDGY,`

### 1) Group filter command

Format:

`$PDGY,SET_FILTER,0,0,SAMPLE_RATE_MS,ACTION`

- `SAMPLE_RATE_MS`: transmit cycle in milliseconds
- `ACTION` values:
  - `AIS`
  - `ELECTRICAL`
  - `ENGINE`
  - `ENVIRONMENT`
  - `GENERAL`
  - `NAVIGATION`
  - `FILTERS_ON`
  - `FILTERS_OFF`

Behavior:

- Category actions (`AIS`...`NAVIGATION`) toggle that PGN group ON/OFF
- `FILTERS_ON` disables all PGN TX (global OFF)
- `FILTERS_OFF` enables all PGN TX (global ON)

Examples:

- `$PDGY,SET_FILTER,0,0,1000,AIS`
- `$PDGY,SET_FILTER,0,0,1000,ENGINE`
- `$PDGY,SET_FILTER,0,0,1000,FILTERS_ON`
- `$PDGY,SET_FILTER,0,0,1000,FILTERS_OFF`

The 0,0,1000 are just padding reserved for future use. 
### 2) Single PGN command

Format:

`$PDGY,SET_PGN,PGN,ON|OFF`

- `PGN`: decimal NMEA 2000 PGN
- `ON`: allow TX of this PGN
- `OFF`: block TX of this PGN

Examples:

- `$PDGY,SET_PGN,129029,ON`
- `$PDGY,SET_PGN,129029,OFF`

ABF3 Response
After every write to ABF3, the device stores a response that can be retrieved by reading ABF3:

$PDGY,ACK — command was recognized and executed successfully
$PDGY,NACK — command was unknown, malformed, or missing required arguments
Example: receive only the wind PGN (130306)

Disable all PGN TX:
Write $PDGY,SET_FILTER,0,0,1000,FILTERS_ON → read ABF3: $PDGY,ACK

Enable wind PGN only:
Write $PDGY,SET_PGN,130306,ON → read ABF3: $PDGY,ACK

List of supported PGN by the BLE connection {59392,59904,60160,60416,60928,65240,126208,126464,126983,126984,126985,
			126986,126987,126988,126992,126993,126996,126998,127233,127237,127245,127250,127251,127252,127257,
			127258,127488,127489,127493,127496,127497,127498,127500,127501,127502,127503,127504,127505,127506,
			127507,127508,127509,127510,127511,127512,127513,127514,127744,127745,127746,127747,127748,127749,
			127750,127751,128259,128267,128275,128520,129025,129026,129027,129028,129029,129033,129038,129039,
			129040,129041,129044,129045,129283,129284,129285,129291,129301,129302,129538,129539,129540,129541,
			129542,129545,129546,129547,129549,129550,129551,129556,129792,129793,129794,129795,129796,129797,
			129798,129799,129800,129801,129802,129803,129804,129805,129806,129807,129808,129809,129810,129811,
			129812,129813,130052,130053,130054,130060,130061,130064,130065,130066,130067,130068,130069,130070,
			130071,130072,130073,130074,130306,130310,130311,130312,130313,130314,130315,130316,130320,130321,
			130322,130323,130324,130560,130567,130569,130570,130571,130572,130573,130574,130576,130577,130578,
			130580,130581,130582,130583,130584,130585,128780,128000,128006,128007,128008,128538,128768,128769,
			128776,128777,128778,126720};
      
Please contact us if you wish to add new PGN.

Note that a device reset using the reset button will also reset the filters 

## NMEA RAW Protocol Notes

- RX to app (notifications from ABF2): `!PDGY,...` sentences
- TX from app to bus (write to ABF2): RAW protocol payload accepted and forwarded to NMEA 2000 processing

For binary PGN field definitions, refer to NMEA 2000 Appendix A/B from NMEA.


## OTA Update 

OTA Updates files are in the ZIP Folder 

Upload first file from the unzipped file that end by F in the settings page 
Upload second file that end with S in the settings page 

Firmware version in the settings page should be 2.00



