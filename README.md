# ESP32 IR Web Remote (Wi-Fi Setup Portal + HTTP API + Web UI)

This project turns an **ESP32** into a small **IR transmitter** that you can control from a browser or via HTTP API calls.

Features:
- Wi-Fi setup portal (AP page on first setup / when Wi-Fi fails)
- Runs an HTTP server when connected to your Wi-Fi
- Web UI with:
  - configurable NEC **address** + **repeats**
  - preset buttons + custom command send
  - live log/status
  - device settings page to update SSID/password/device name
- mDNS support (unique hostname per device): `http://<name>-<suffix>.local/`
- Designed for running multiple devices (unique suffix prevents name collisions)

---

## Tested hardware

✅ Tested with:
- **AZ-Delivery KY-005 IR infrared transmitter transceiver module**
- **AZ-Delivery ESP32 Dev Kit C V4 NodeMCU WLAN WiFi Development Board** (USB-C)

---

## Libraries

- **Arduino-IRremote** (Armin Joachimsmeyer)
- ESP32 Arduino core libraries:
  - `WiFi.h`
  - `WebServer.h`
  - `Preferences.h`
  - `ESPmDNS.h`

Install Arduino-IRremote from **Arduino IDE → Library Manager**.

---

## Wiring
![Diagram](docs/images/diagram.png)
### Recommended (best range)
ESP32 GPIO pins cannot drive an IR LED at high current for good range. For best results, drive the IR LED with a transistor.

Typical wiring:
- ESP32 **IR_SEND_PIN** → resistor (e.g. **1kΩ**) → transistor base
- transistor emitter → GND
- transistor collector → IR LED cathode (–)
- IR LED anode (+) → 3.3V through a resistor (e.g. **100–220Ω**)
- common GND

> This project is tested with the **KY-005** module. Connect its signal input to the configured `IR_SEND_PIN` and power it according to the module specs.

---

## Setup flow

1. Boot device:
   - It tries to connect to the previously saved Wi-Fi.
2. If it cannot connect (or BOOT button held at boot):
   - It starts an **open** AP with an SSID like:  
     `ESP32-Setup-<suffix>`
   - Connect to it and open:  
     `http://192.168.4.1`
   - Enter:
     - SSID
     - password (optional)
     - device name (optional)
3. Device reboots and connects to your Wi-Fi.
4. Open the main UI:
   - via IP shown in serial log, **or**
   - via mDNS:  
     `http://<device-name>-<suffix>.local/`

---

## Web UI

When connected to Wi-Fi, open the device in your browser:
- `http://<device-ip>/`  
or
- `http://<mdns-name>.local/`

The UI provides:
- Device Settings:
  - change SSID/password/device name (reboots after saving)
- IR Settings:
  - NEC Address
  - repeats
- Send Presets + Send Custom command
- Live log/status

---

## HTTP API

### Send NEC command (address selectable)
Send an NEC command (values can be decimal or hex like `0x1B`):

```
GET /api/ir/send?addr=0x01&cmd=0x1B&repeats=0
```

Parameters:
- `cmd` (required): 0–255
- `addr` (optional, default is firmware’s `NEC_ADDR_DEFAULT`): 0–255
- `repeats` (optional, default is firmware’s `NEC_REPEATS_DEFAULT`): 0–255

Examples:
```
/api/ir/send?addr=1&cmd=27
/api/ir/send?addr=0x01&cmd=0x1B&repeats=3
```

### Status / logs
```
GET /api/status
```

### Update Wi-Fi + device name (STA mode)
```
POST /api/wifi/set
Content-Type: application/x-www-form-urlencoded

ssid=<...>&pass=<...>&name=<...>
```

### Forget Wi-Fi and reboot to portal
```
GET /api/wifi/forget
```

---

## Notes

- Credentials are stored in ESP32 flash (NVS) using `Preferences`, so they persist across resets and firmware uploads.
- mDNS behavior depends on your OS:
  - works well on iOS/macOS
  - on Windows it may depend on installed mDNS/Bonjour support

---

## License

MIT License.
