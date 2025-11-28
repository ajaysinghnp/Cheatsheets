# Generic - BK7231N (Tuya QFN32)

## Info and Flashing Guide

| Parameter    | Value                             |
| ------------ | --------------------------------- |
| Board Code   | generic-bk7231n-qfn32-tuya        |
| MCU          | BK7231N                           |
| Manufacturer | Beken                             |
| Series       | BK72XX                            |
| Frequency    | 120 MHz                           |
| Flash Size   | 2 MiB                             |
| RAM Size     | 256 KiB                           |
| Voltage      | 3.0V - 3.6V                       |
| I/O          | 19x GPIO, 6x PWM, 2x UART, 1x ADC |
| Wi-Fi        | 802.11 b/g/n                      |
| Bluetooth    | BLE v5.1                          |

## Usage

Board Code: `generic-bk7231n-qfn32-tuya`

In `platformio.ini`:

```toml
[env:generic-bk7231n-qfn32-tuya]
platform = libretiny
board = generic-bk7231n-qfn32-tuya
framework = arduino
```

In ESPHome Yaml:

```yaml
bk72xx:
  board: generic-bk7231n-qfn32-tuya
```

## Pin Functions

| Name(s) | UART | I^2^C | SPI  | PWM  | Other     |
| ------- | ---- | ----- | ---- | ---- | --------- |
| P0      | TX2  | SCL2  |      |      |           |
| P1      | RX2  | SDA2  |      |      |           |
| P10     | RX1  |       |      |      |           |
| P11     | TX1  |       |      |      |           |
| P14     |      |       | SCK  |      |           |
| P15     |      |       | CS   |      |           |
| P16     |      |       | MOSI |      |           |
| P17     |      |       | MISO |      |           |
| P20     |      | SCL1  |      |      | TCK       |
| P21     |      | SDA1  |      |      | TMS       |
| P22     |      |       |      |      | TDI       |
| P23     |      |       |      |      | ADC3, TDO |
| P24     |      |       |      | PWM4 |           |
| P26     |      |       |      | PWM5 |           |
| P28     |      |       |      |      |           |
| P6      |      |       |      | PWM0 |           |
| P7      |      |       |      | PWM1 |           |
| P8      |      |       |      | PWM2 |           |
| P9      |      |       |      | PWM3 |           |

## Flash Memory Map

Flash Size: 2MiB / 2,097,152 B / 0x200000

Hex values are in bytes.

| Name            | Start    | Length             | End      |
| --------------- | -------- | ------------------ | -------- |
| Bootloader      | 0x00000  | 68 KiB / 0x11000   | 0x11000  |
| App Image       | 0x11000  | 1.1 MiB / 0x119000 | 0x12A000 |
| OTA Image       | 0x12A000 | 664 KiB / 0xA6000  | 0x1D0000 |
| Calibration     | 0x1D0000 | 4 KiB / 0x1000     | 0x1D1000 |
| Network Data    | 0x1D1000 | 4 KiB / 0x1000     | 0x1D2000 |
| TLV Store       | 0x1D2000 | 4 KiB / 0x1000     | 0x1D3000 |
| Key-Value Store | 0x1D3000 | 32 KiB / 0x8000    | 0x1DB000 |
| User Data       | 0x1DB000 | 148 KiB / 0x25000  | 0x200000 |
| Tuya Storage    | 0x200000 | 76 KiB / 0x13000   | 0x213000 |

Bootloader and app partitions contain CRC16 sums every 32 bytes. That results in the actual flash offsets/sizes not aligned to sector boundaries. To simplify calculations, the values shown in the table (extrcted from bootloader's partition table) were aligned to 4096 bytes.
