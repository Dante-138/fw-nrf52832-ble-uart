# fw-nrf52832-ble-uart

BLE to UART bridge for the nRF52832 DK using the Nordic UART Service (NUS). Zephyr RTOS, nRF Connect SDK. It is the nRF52832 port of `fw-nrf52840-ble-uart`.

## What it does

Two-way text link between a serial terminal on a PC and a phone running nRF Connect:

```
PC terminal <--UART--> nRF52832 DK <--BLE--> Phone (nRF Connect)
```

- Text typed in the terminal is sent to the phone as a NUS notification when Enter is pressed.
- Text written by the phone to the NUS RX characteristic appears in the terminal.
- The device advertises as `nRF52832_BLE_UART`.

LEDs: LED1 blinks while the firmware runs, LED2 is on while a BLE central is connected.

Design:

- UART to BLE: UART RX uses the async API with DMA. Characters accumulate until `\r` or `\n`, then go through a FIFO to `ble_write_thread`, which calls `bt_nus_send()`.
- BLE to UART: `bt_receive_cb()` calls `uart_tx()` directly.

Differences from the nRF52840 version, because of the 64 KB RAM:

| | nRF52840 | nRF52832 |
|---|---|---|
| Board | PCA10056 | PCA10040 |
| Flash / RAM | 1 MB / 256 KB | 512 KB / 64 KB |
| ACL buffer | 251 B | 128 B |
| L2CAP MTU | 247 | 124 |
| West board | `nrf52840dk/nrf52840` | `nrf52dk/nrf52832` |

Limitations: no local echo in the terminal, messages are sent only on Enter, about 20 characters per message at the default MTU.

## Hardware

| Item | Detail |
|---|---|
| Board | nRF52832 DK (PCA10040) |
| MCU | nRF52832, Cortex-M4F, 64 MHz, 512 KB flash, 64 KB RAM |
| UART | UART0 through the J-Link VCOM, 115200 8N1 (`nordic,nus-uart = &uart0` in `app.overlay`) |
| Phone | Bluetooth 4.0 or newer, nRF Connect app |

## Build

Requires nRF Connect SDK v3.x. Use a project path without spaces.

```bash
west build -b nrf52dk/nrf52832 --pristine
```

## Flash

```bash
west flash --runner jlink
```

## Test

1. Open a serial terminal on the DK's COM port: 115200 baud, 8N1, no flow control.
2. In nRF Connect on the phone, scan and connect to `nRF52832_BLE_UART`. LED2 turns on.
3. Open the Nordic UART Service and enable notifications on the TX characteristic (`6E400003-B5A3-F393-E0A9-E50E24DCCA9E`).
4. Type text plus Enter in the terminal: it shows up on the TX characteristic.
5. Write text to the RX characteristic (`6E400002-B5A3-F393-E0A9-E50E24DCCA9E`): it shows up in the terminal.

## License

MIT, see [LICENSE](LICENSE).
