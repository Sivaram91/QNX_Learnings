# Host setup: license, SDP and image

## Host: license and SDP

1. Create a myQNX account and get the free non-commercial **QNX SDP 8.0** license at [qnx.com/getqnx](https://qnx.com/getqnx). Accept and deploy it to the corresponding email ID
2. Install the **QNX Software Center (QSC)**, login with same email ID in QSC
3. In QSC, install **QNX SDP 8.0.5** (`com.qnx.qnx800`). (Note: Earlier versions shown above might be inaccessible, scroll down to the latest version and install it)
4. Install **VS Code** and the **QNX Toolkit** extension, and point it at the SDP install.


## Get the Pi image

In QSC, install the RPi4 quick start image package `com.qnx.qnx800.quickstart.rpi4`
(Updates → QNX Software Development Platform → Reference Images).

It lands in `%USERPROFILE%\qnx800\images` (Windows) or `~/qnx800/images` (Linux), follow the Readme.md to generate the img file

