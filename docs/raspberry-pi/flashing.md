# Flash and first boot

## Flash the SD card

Use **Raspberry Pi Imager**:

| Setting | Value |
|---|---|
| Device | Raspberry Pi 4B |
| Operating System | Use custom → select the `.img` |
| Storage | the SD card |

Click NEXT → WRITE to confirm writing the SD card
The QNX image does its own configuration; Imager customisation doesn't apply.


## First boot

1. Insert the card, connect HDMI 0 (next to the USB-C port) and a USB keyboard, power up.
2. First boot runs one-time provisioning and takes few minutes. Don't power-cycle it.
3. Log in with the default `qnxuser` / `qnxuser` (root: `root` / `root`).

