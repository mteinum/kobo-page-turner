# Kobo Page Turner

[![Arduino Build](https://github.com/mteinum/kobo-page-turner/actions/workflows/build.yml/badge.svg)](https://github.com/mteinum/kobo-page-turner/actions/workflows/build.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/platform-ESP32-blue.svg)](https://www.espressif.com/en/products/socs/esp32)

A BLE keyboard controller for M5Stack AtomS3 that enables wireless page turning on Kobo e-readers.

![Kobo Page Turner](docs/IMG_8010.jpeg)

## Author

Morten Teinum (morten.teinum@gmail.com)

## Hardware

- **Device**: M5Stack AtomS3
- **Connection**: Bluetooth Low Energy (BLE)

## Features

- **Short Press**: Next page (Right Arrow key)
- **Long Press** (≥600ms): Previous page (Left Arrow key)
- **Visual Status Indicator**: 
  - Green filled square: BLE connected
  - Red outline square: BLE disconnected

## Setup

### Prerequisites

1. Install Arduino IDE or Arduino CLI
2. Install ESP32 board support
3. Install required libraries:
   - [M5Unified](https://github.com/m5stack/M5Unified)
   - [ESP32 BLE Keyboard](https://github.com/T-vK/ESP32-BLE-Keyboard)

### ESP32 BLE Keyboard Library Modifications

The project requires a modified version of the ESP32 BLE Keyboard library. Apply the following patch to `BleKeyboard.cpp`:

<details>
<summary>Click to view the required patch</summary>

```diff
diff BleKeyboard.cpp ~/Documents/Arduino/libraries/ESP32_BLE_Keyboard/BleKeyboard.cpp
0a1,2
> #include "BleKeyboard.h"
>
17d18
< #include "BleKeyboard.h"
105c106
<   BLEDevice::init(deviceName);
---
>   BLEDevice::init(String(deviceName.c_str()));
116c117
<   hid->manufacturer()->setValue(deviceManufacturer);
---
>   hid->manufacturer()->setValue(String(deviceManufacturer.c_str()));
118c119
<   hid->pnp(0x02, 0xe502, 0xa111, 0x0210);
---
>   hid->pnp(0x02, vid, pid, version);
120a122,128
>
> #if defined(USE_NIMBLE)
>
>   BLEDevice::setSecurityAuth(true, true, true);
>
> #else
>
121a130
>   pSecurity->setAuthenticationMode(ESP_LE_AUTH_REQ_SC_MITM_BOND);
123c132
<   pSecurity->setAuthenticationMode(ESP_LE_AUTH_BOND);
---
> #endif // USE_NIMBLE
167a177,188
> void BleKeyboard::set_vendor_id(uint16_t vid) {
> 	this->vid = vid;
> }
>
> void BleKeyboard::set_product_id(uint16_t pid) {
> 	this->pid = pid;
> }
>
> void BleKeyboard::set_version(uint16_t version) {
> 	this->version = version;
> }
>
482a504,513
>
> #if !defined(USE_NIMBLE)
>
>   BLE2902* desc = (BLE2902*)this->inputKeyboard->getDescriptorByUUID(BLEUUID((uint16_t)0x2902));
>   desc->setNotifications(true);
>   desc = (BLE2902*)this->inputMediaKeys->getDescriptorByUUID(BLEUUID((uint16_t)0x2902));
>   desc->setNotifications(true);
>
> #endif // !USE_NIMBLE
>
486a518
>
487a520,525
>
>   BLE2902* desc = (BLE2902*)this->inputKeyboard->getDescriptorByUUID(BLEUUID((uint16_t)0x2902));
>   desc->setNotifications(false);
>   desc = (BLE2902*)this->inputMediaKeys->getDescriptorByUUID(BLEUUID((uint16_t)0x2902));
>   desc->setNotifications(false);
>
489c527,528
< #endif  // !USE_NIMBLE
---
>
> #endif // !USE_NIMBLE
```

</details>

These modifications enhance the BLE security and compatibility with Kobo e-readers.

### Upload Instructions

2. Upload the sketch to your M5Stack AtomS3

3. Pair with your Kobo e-reader:
   - Device name: `Kobo PageTurner`
   - Look for it in Bluetooth settings on your Kobo

## Usage

Once paired, simply press the button on the AtomS3:
- **Quick tap**: Turn to next page
- **Press and hold**: Go back to previous page

The display shows the current connection status and button functions.

## Configuration

You can modify these settings in the code:

- `BLE_DEVICE_NAME`: Change the Bluetooth device name
- `LONG_PRESS_MS`: Adjust the long press threshold (default: 600ms)

## License

MIT License - see [LICENSE](LICENSE) file for details.
