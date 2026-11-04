# QNX 8.0 on Raspberry Pi

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

## Pages

1. [Host setup](host-setup.md): license, SDP, image
2. [Flash and first boot](flashing.md)
3. [WiFi](wifi.md)
4. [Security and SSH](security-ssh.md)
5. [Build and run](build-run.md)

## Next steps

- [ ] Commit the working Trial project as a known-good baseline
- [ ] Get on-target debugging working (breakpoint + step from VS Code)

## References

- [QNX Toolkit contribution points (`qnx.arch`)](https://www.qnx.com/developers/docs/8.0/com.qnx.doc.qnxtoolkit.user_guide/topic/qnx_toolkit_contribution_points.html)
- [QNX Everywhere: Interacting with the system](https://www.qnx.com/developers/docs/qnxeverywhere/com.qnx.doc.qnxeverywhere/topic/qsti/interacting-with-the-system.html)
