# Security and SSH

## Secure the defaults

The image ships with default passwords and a VNC server running.

```sh
passwd                          # qnxuser
sudo passwd root
sudo vncserv -password=<new>    # VNC
```

`qconn` (remote debug service) is also enabled by default — keep the Pi on the home LAN only.


## SSH from the PC

```sh
ssh qnxuser@qnxpi.local
```

- Use the Pi's IP if `.local` doesn't resolve.
- If SSH complains about MAC algorithms, add `-m hmac-sha2-256` (or put `MACs hmac-sha2-256` in `~/.ssh/config`).
- Tip: give the Pi a DHCP reservation in the router so its IP stays stable.

