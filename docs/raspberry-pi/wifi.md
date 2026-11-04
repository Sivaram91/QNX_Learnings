# WiFi

## Configure WiFi

The WiFi config lives in `wpa_supplicant.conf` on the FAT32 boot partition — `/boot/wpa_supplicant.conf` on the running Pi.

> **Edit the `/boot` copy only.** `/data/var/etc/settings/wpa_supplicant.conf` is a runtime copy, regenerated from `/boot` at every boot; changes there are lost.

Config block (same for both options):

```
network={
    ssid="YourWiFiName"
    key_mgmt=WPA-PSK
    psk="YourPassword"
}
```

- SSID is case-sensitive; spaces count. Prefer the **2.4 GHz** network.

### Option A — on the PC (via the SD card)

1. Shut down the Pi (`sudo shutdown`), or do this right after flashing, before first boot.
2. Insert the SD card in the PC. The boot partition may **not** show up in Explorer automatically. If it doesn't, assign it a drive letter:
   1. Press `Win + X` → **Disk Management**.
   2. Find the SD card disk (match it by size) and its small **FAT32** partition.
   3. Right-click that partition → **Change Drive Letter and Paths…** → **Add** → pick a letter → OK.
   4. The partition now appears in Explorer.

   > If Windows offers to **format** the disk, click **Cancel**. Don't touch the other (non-FAT) partitions on the card — they hold the QNX system.
3. Edit (or create) `wpa_supplicant.conf` there and add the config block.
4. Put the card back in the Pi and boot.

No keyboard-layout trouble — the easiest option.

### Option B — on the Pi (with commands)

1. Edit the config:
   ```sh
   sudo vi /boot/wpa_supplicant.conf
   ```
   (`i` = insert, `Esc` then `:wq` = save and quit.)
2. Restrict access, since it holds the WiFi password:
   ```sh
   sudo chmod 600 /boot/wpa_supplicant.conf
   ```
3. Reboot so the system starts WiFi with the new config:
   ```sh
   sudo shutdown       # reboots by default on QNX
   ```

To test without rebooting, start it by hand (wpa_supplicant only reads its config at startup):

```sh
sudo slay wpa_supplicant-2.11          # stop the running instance
sudo wpa_supplicant-2.11 -B -i bcm0 -c /boot/wpa_supplicant.conf
sudo dhcpcd bcm0                       # "/etc/dhcpcd.conf: No such file" is harmless
```

If it doesn't connect, run it in the foreground (drop `-B`, add `-d`) to see the error:

| Message | Meaning |
|---|---|
| `CTRL-EVENT-CONNECTED` | Success |
| `WRONG_KEY` / `4-Way Handshake failed` | Wrong password |
| `No suitable network found` | Wrong SSID or 5 GHz-only network |
| `Failed to read or parse configuration` | Syntax error (missing `"` or `}`) |

Notes for Option B:

- The image uses a US keyboard layout. On a German QWERTZ keyboard, `y`/`z` are swapped and symbols (`"`, `=`, `{`, `}`) sit on their US positions.
- Use absolute paths (`/boot/...`). A relative path makes `wpa_supplicant -B` fail silently.
- `wpa_cli` and `wpa_passphrase` are not on the image.

### Check the connection

```sh
ifconfig bcm0     # want: status: associated + an inet address
```

Then sync the clock (it falls back to the image build date, which breaks TLS):

```sh
sudo ntpdate pool.ntp.org
date              # shows UTC — correct
```

