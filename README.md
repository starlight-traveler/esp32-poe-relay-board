# ESP32 PoE Relay System

This firmware starts a local Wi-Fi access point for emergency relay control and also brings up the wired PoE Ethernet interface.

## Network Behavior

- Wi-Fi AP SSID: `NDAIMM-EMERGENCY-SHUTOFF`
- Wi-Fi AP IP: `192.168.4.1`
- Ethernet: starts separately and gets its own IP from the wired network
- The built-in web server is reachable from either interface
- The firmware does not bridge or NAT Wi-Fi AP clients onto the wired network

## Boot Flow

`src/MAIN_WIFI_AP.ino` starts:

1. GPIO, I2C, RTC, SD, and serial
2. Wi-Fi AP and web server
3. Bluetooth
4. PoE Ethernet
5. DIN input handling
6. Relay control task

## Relay Web API

The web server runs on port `80`.

### Read relay state

```text
GET /api/relays
```

Example response:

```json
{"relays":[0,1,0,0,0,0,0,0]}
```

### Set one relay on or off

```text
GET /api/relay?channel=1&state=on
```

Accepted `state` values:

- `on` / `off`
- `open` / `close`
- `1` / `0`
- `true` / `false`

Example response:

```json
{"channel":1,"state":1,"relays":[1,1,0,0,0,0,0,0]}
```

### Set all relays on or off

```text
GET /api/relays/all?state=off
```

Example response:

```json
{"state":0,"relays":[0,0,0,0,0,0,0,0]}
```

## Legacy UI Routes

These routes still exist for the built-in web page:

- `/`
- `/getData`
- `/Switch1` through `/Switch8`
- `/AllOn`
- `/AllOff`
- `/RTC_Event`

## Build

If `pio` is not on your shell `PATH`, this environment also works with:

```sh
~/.platformio/penv/bin/pio run
```

## Source Files

- `src/MAIN_WIFI_AP.ino`: boot sequence
- `src/WS_WIFI.cpp`: AP setup, HTTP routes, relay API
- `src/WS_ETH.cpp`: Ethernet startup and DHCP IP handling
- `src/WS_Relay.cpp`: low-level relay control
