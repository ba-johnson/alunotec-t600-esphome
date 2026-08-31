# T600 RS485 protocol notes

These notes describe only the commands used by `esphome/pergola.yaml`. They are based on a working installation and AlunoTec's supplied *RS485 Pavilion Communication Protocol*, V1.0 dated 2024-05-09. The source PDF is not redistributed here.

## Serial parameters

- 9600 baud
- 8 data bits
- no parity
- 1 stop bit

## Ten-byte frame

| Offset | Field | Value used here |
|---:|---|---|
| 0 | Header | `0x55` transmit; documentation says `0xAA` reply |
| 1 | Frame number | Incrementing `0x01`-`0xFE` |
| 2 | Device ID | `0xFE` |
| 3 | Motor address | Installation/channel dependent |
| 4 | Device type | `0x00` motor, `0x01` RGB, `0x02` white light |
| 5 | Command | `0x01` read or `0x03` control |
| 6 | Data address | Function/register |
| 7 | Data | Function value |
| 8 | CRC low byte | Modbus CRC-16 over bytes 0-7 |
| 9 | CRC high byte | Modbus CRC-16 over bytes 0-7 |

The YAML increments its frame number after every transmission and skips `0x00` and `0xFF`.

## Commands implemented

| Function | Motor address | Device type | Command | Data address | Data |
|---|---:|---:|---:|---:|---|
| Motor open | channel | `0x00` | `0x03` | `0x01` | `0x01` |
| Motor close | channel | `0x00` | `0x03` | `0x01` | `0x02` |
| Motor stop | channel | `0x00` | `0x03` | `0x01` | `0x00` |
| Light power | `0xFE` | `0x01` RGB / `0x02` white | `0x03` | `0x02` | `0x00` off / `0x01` on |
| Light brightness | `0xFE` | `0x01` / `0x02` | `0x03` | `0x03` | 0-100 |
| White-light mode | `0xFE` | `0x02` | `0x03` | `0x04` | `0x00` cool / `0x01` warm |
| RGB preset | `0xFE` | `0x01` | `0x03` | `0x05` | Field-tested `0x00`-`0x07` |

## Read commands implemented

| Query | Motor address | Device type | Command | Data address |
|---|---:|---:|---:|---:|
| Roof status | `0x11` | `0x00` | `0x01` | `0x30` |
| Light power | `0xFE` | `0x01` / `0x02` | `0x01` | `0x31` |
| Light brightness | `0xFE` | `0x01` / `0x02` | `0x01` | `0x32` |
| RGB colour | `0xFE` | `0x01` | `0x01` | `0x34` |

Replies are visible in ESPHome's UART debug log but are not parsed into Home Assistant state.

## Documented-versus-observed RGB values

The supplied V1.0 protocol table states `0x01` Red through `0x08` Gradient. The installed controller required `0x00` Red through `0x07` Gradient. The repository deliberately follows the observed working behaviour.

Do not assume that unimplemented parameter-setting, limit-setting or factory-reset commands are safe. They are intentionally omitted.

