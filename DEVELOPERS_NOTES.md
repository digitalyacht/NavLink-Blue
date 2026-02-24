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

### 2) Single PGN command

Format:

`$PDGY,SET_PGN,PGN,ON|OFF`

- `PGN`: decimal NMEA 2000 PGN
- `ON`: allow TX of this PGN
- `OFF`: block TX of this PGN

Examples:

- `$PDGY,SET_PGN,129029,ON`
- `$PDGY,SET_PGN,129029,OFF`

## NMEA RAW Protocol Notes

- RX to app (notifications from ABF2): `!PDGY,...` sentences
- TX from app to bus (write to ABF2): RAW protocol payload accepted and forwarded to NMEA 2000 processing

For binary PGN field definitions, refer to NMEA 2000 Appendix A/B from NMEA.

