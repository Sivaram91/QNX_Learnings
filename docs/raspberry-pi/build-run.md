# Build and run from VS Code

## Add the Pi as a target in the QNX Toolkit

In the QNX Toolkit view, add a target connection using `qnxpi.local` (or the Pi's IP). Once connected it shows up as `qnxpi (aarch64le)`.


## Set the build architecture to `aarch64le`

The Toolkit builds for **x86_64** by default. Running that binary on the Pi fails with:

```
Error launching process /tmp/trial
Error: Exec format error
```

`Exec format error` = binary built for the wrong CPU architecture.

**Fix:** VS Code Settings → search `qnx.arch` → set to `aarch64le` (Workspace level). If present, also set `qnx.archList` to `aarch64le` (first entry = default for new projects). Then delete `build/` and rebuild.


## Build and run

1. Status bar → **Build** (Build Active Project).
2. **Run as QNX Application** → the binary is uploaded to `/tmp/<name>` on the Pi and executed there; stdout is piped back to the VS Code terminal.


## Verifying code runs on the Pi

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
