# NAVLink Blue - Bluetooth Low Energy (BLE) Gateway for NMEA 2000

## Overview
**NAVLink Blue** is a Bluetooth Low Energy (BLE) device that connects the NMEA 2000 (CAN BUS) network to Bluetooth clients in a full-duplex mode. It allows seamless data transfer, configuration, and diagnostics between marine electronics and compatible applications.

## Features
- **BLE Characteristics**:
  - `ABF2`: Streams NMEA 2000 data and receives data from clients.
  - `ABF3`: Command channel for configuring filters and sample rate.
  - `ABF4`: Low sample rate for battery optimization and boat monitoring.
- **Switch between Bluetooth & Wi-Fi**: Supports Wi-Fi mode for OTA updates, diagnostics, and filter configuration.
- **Over-the-Air (OTA) updates**: Easily update firmware via Wi-Fi.
- **Filter and Sample Rate Management**: Customizable data filters and sample rates for efficient monitoring and control.

## BLE Service and Characteristics

- **Service UUID**: `ABF0`
- **Characteristics**:
  - `ABF2`:
    - Streams NMEA 2000 data via notifications in a RAW protocol format.
    - Receives data from the client via Write operations and forwards it to the bus.
  - `ABF3`: Command channel for configuring filters and setting sample rates.
  - `ABF4`: Low sample rate notifications for battery-efficient applications and boat monitoring.


## Getting Started

### BLE Advertisement
- **Device Name**: `NAVLinkBlue-XXXX` (where `XXXX` is the unique device ID)
- **Manufacturer Data**: Encoded in ASCII as `DYACHTXXXX`, where `XXXX` is the device's unique ID.
- **Service UUID**: `ABF0`
- **Connectable** : YES

### Switching Between Bluetooth and Wi-Fi
Our gateway can be switched to a Wi-Fi mode like a lot of our product. This Wifi mode can be used to perform software updates, debug the NMEA network and configure filters.  
1. **Switch Mode**: Hold the reset button for 5 seconds. LEDs will light up in sequence to indicate the switch.
2. **Wi-Fi Mode Details**:
   - Network: `SSID: NAVLinkBlue-XXXX`
   - Password: `PASS-XXXX` (where `XXXX` is the last four characters of the SSID)
   - IP Address: `192.168.1.1` (for OTA updates and configuration)

### OTA Firmware Update
1. Switch to Wi-Fi mode.
2. Connect to the Wi-Fi network created by the device and use the password `PASS-XXXX`.
3. Access the device by navigating to `192.168.1.1` in your web browser.
4. Upload the firmware file (`NAVLinkBlue_VXXX.bin`) from the settings page.

### Receiving NMEA Data
- Subscribe to notifications from `ABF2`, and the server will send NMEA 2000 data to the client in **RAW protocol format** (Base64-encoded). 
- -Some filters can be applied to the data so the client will only receive certain PGN. The sample rate is the same for all of the data sent and will use the last NMEA Message received if a PGN has a higher sample rate than what you have chosen. 
- [TX PGN Format](https://github.com/digitalyacht/iKonvert/wiki/4.-Serial-Protocol#42-tx-pgn-sentence)
### Sending Data to NMEA 2000
1. To send data from a client to the NAVLink Blue server, write data to the `ABF2` characteristic using a BLE write operation.This will transfer the data from the client to the NAVLink Blue device. The device will then process this data and, if applicable, send it to the NMEA 2000 CAN Bus.
2. This data must be formatted in **RAW protocol** for compatibility with the NMEA 2000 CAN Bus.
- [RX PGN Format](https://github.com/digitalyacht/iKonvert/wiki/4.-Serial-Protocol#41-rx-pgn-sentence)

You can use the Light Blue App to write on the ABF2 characteristic and send the data to the bus:

| ![Light Blue App example](https://github.com/user-attachments/assets/54b5b856-b369-4deb-a00a-8488c90b531d) | **Creating a Message:**<br>The message is made as described in our Serial Protocol RX Sentence. First start with !PDGY followed by the PGN Number and the destination Address. The NMEA2000 data is then encoded in base64 and sent|
|---|---|



## Command Structure via `ABF3`
Commands follow this format:

$PDGY,ACTION,0,0,SAMPLE_RATE,ACTION_TYPE

![image](https://github.com/user-attachments/assets/3e39dcc0-d6b6-4f4f-a96c-5ff6e182e965)


Commands are sent in the format:
NB : The Light Blue app can be used to test commands (Figure 1)


$PDGY,ACTION,0,0,SAMPLE_RATE,ACTION_TYPE

- ACTION: Specifies the command. Supported option:
  - SET_FILTER
  
- SAMPLE_RATE: The rate in milliseconds (e.g., 1000 for 1 Hz).

- ACTION_TYPE: Defines the filter to toggle. The following filters are supported:
  - AIS: Includes data related to AIS (Automatic Identification System), covering position reports, navigation aids, and various types of vessel and static data.
  - ELECTRICAL: Includes data related to electrical systems on board, such as battery status, power usage, and electrical measurements.
  - ENGINE: Includes data related to engine performance and status, including RPM, engine temperature, fuel consumption, and other engine diagnostics.
  - ENVIRONMENT: Includes environmental data such as wind conditions, water depth, and atmospheric pressure.
  - GENERAL: Includes general system information such as product information, configuration, and diagnostics.
  - NAVIGATION: Includes navigation-related data like heading, course, position, speed, and waypoint information.
  - ALLOFF: Turns off all filters.

Please note that the `0,0` in the command format are mandatory and must be included.

If you want to activate multiple categories, resend a command with the same sample rate for each category. By default, NAVLink Blue will send all data without filtering, using the NMEA Bus sample rate.


