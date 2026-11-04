# QNX 8.0 on Raspberry Pi — Dev Setup Notes

Working setup as of 2026-11-04  
Build on Windows PC (VS Code + QNX Toolkit)  
Deploy and run on Raspberry Pi 4 running QNX 8.0.

## Current state (verified working)

- [x] QNX 8.0 image flashed and booting on the Pi
- [x] Pi connected to WiFi
- [x] SSH from PC to Pi works
- [x] QNX Toolkit detects the Pi as target `qnxpi` (aarch64le)
- [x] Build on PC → upload → run on Pi via **Run as QNX Application**
- [ ] Debugging (breakpoints / stepping on target) — next step

## Environment

| Item | Value |
|---|---|
| Host | Windows, VS Code + QNX Toolkit extension |
| SDP | QNX SDP 8.0 at `%USERPROFILE%\qnx800` |
| Board | Raspberry Pi 4 |
| Target image | QNX 8.0 Quick Start Target Image (QSTI) for RPi4 |
| Target name | `qnxpi` |
| Target arch | `aarch64le` |
| Test project | `Trial` (CMake) |

---

## Setup steps

### 1. Host: license and SDP

1. Create a myQNX account and get the free non-commercial **QNX SDP 8.0** license at [qnx.com/getqnx](https://qnx.com/getqnx). Accept and deploy it to the corresponding email ID
2. Install the **QNX Software Center (QSC)**, login with same email ID in QSC
3. In QSC, install **QNX SDP 8.0.5** (`com.qnx.qnx800`). (Note: Earlier versions shown above might be inaccessible, scroll down to the latest version and install it)
4. Install **VS Code** and the **QNX Toolkit** extension, and point it at the SDP install.

### 2. Get the Pi image

In QSC, install the RPi4 quick start image package `com.qnx.qnx800.quickstart.rpi4`
(Updates → QNX Software Development Platform → Reference Images).

It lands in `%USERPROFILE%\qnx800\images` (Windows) or `~/qnx800/images` (Linux), follow the Readme.md to generate the img file

### 3. Flash the SD card

Use **Raspberry Pi Imager**:

| Setting | Value |
|---|---|
| Device | Raspberry Pi 4B |
| Operating System | Use custom → select the `.img` |
| Storage | the SD card |

Click NEXT → WRITE to confirm writing the SD card
The QNX image does its own configuration; Imager customisation doesn't apply.

### 4. First boot

1. Insert the card, connect HDMI 0 (next to the USB-C port) and a USB keyboard, power up.
2. First boot runs one-time provisioning and takes few minutes. Don't power-cycle it.
3. Log in with the default `qnxuser` / `qnxuser` (root: `root` / `root`).

### 5. Configure WiFi

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

#### Option A — on the PC (via the SD card)

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

#### Option B — on the Pi (with commands)

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

#### Check the connection

```sh
ifconfig bcm0     # want: status: associated + an inet address
```

Then sync the clock (it falls back to the image build date, which breaks TLS):

```sh
sudo ntpdate pool.ntp.org
date              # shows UTC — correct
```

### 6. Secure the defaults

The image ships with default passwords and a VNC server running.

```sh
passwd                          # qnxuser
sudo passwd root
sudo vncserv -password=<new>    # VNC
```

`qconn` (remote debug service) is also enabled by default — keep the Pi on the home LAN only.

### 7. SSH from the PC

```sh
ssh qnxuser@qnxpi.local
```

- Use the Pi's IP if `.local` doesn't resolve.
- If SSH complains about MAC algorithms, add `-m hmac-sha2-256` (or put `MACs hmac-sha2-256` in `~/.ssh/config`).
- Tip: give the Pi a DHCP reservation in the router so its IP stays stable.

### 8. Add the Pi as a target in the QNX Toolkit

In the QNX Toolkit view, add a target connection using `qnxpi.local` (or the Pi's IP). Once connected it shows up as `qnxpi (aarch64le)`.

### 9. Set the build architecture to `aarch64le`

The Toolkit builds for **x86_64** by default. Running that binary on the Pi fails with:

```
Error launching process /tmp/trial
Error: Exec format error
```

`Exec format error` = binary built for the wrong CPU architecture.

**Fix:** VS Code Settings → search `qnx.arch` → set to `aarch64le` (Workspace level). If present, also set `qnx.archList` to `aarch64le` (first entry = default for new projects). Then delete `build/` and rebuild.

### 10. Build and run

1. Status bar → **Build** (Build Active Project).
2. **Run as QNX Application** → the binary is uploaded to `/tmp/<name>` on the Pi and executed there; stdout is piped back to the VS Code terminal.

---

## Verifying code actually runs on the Pi

```cpp
#include <sys/utsname.h>
#include <cstdio>

int main() {
    utsname u;
    uname(&u);
    printf("%s %s %s\n", u.sysname, u.nodename, u.machine);  // QNX <hostname> aarch64le
}
```

On the Pi via SSH:

```sh
ls -l /tmp/trial          # uploaded binary
pidin | grep trial        # running process (add a sleep() to catch it)
```

## Next steps

- [ ] Commit the working Trial project as a known-good baseline
- [ ] Get on-target debugging working (breakpoint + step from VS Code)

## References

- [QNX Toolkit contribution points (`qnx.arch`)](https://www.qnx.com/developers/docs/8.0/com.qnx.doc.qnxtoolkit.user_guide/topic/qnx_toolkit_contribution_points.html)
- [QNX Everywhere — Interacting with the system](https://www.qnx.com/developers/docs/qnxeverywhere/com.qnx.doc.qnxeverywhere/topic/qsti/interacting-with-the-system.html)
