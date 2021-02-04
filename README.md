# raspi4-setup

1. Install Raspbian on the SD card. I picked the Desktop version of the 32-bit image from [here](https://www.raspberrypi.org/software/operating-systems/).

2. Boot it for the first time (username is `pi`, password is `raspi4`) and follow the tutorial to update the base system.

3. Enable the TPM chip by following the [instructions](https://letstrust.de/archives/20-Mainline.html). Adjust the formating such that it looks like below.

```
# and load the TPM device tree overlay with
dtoverlay=tpm-slb9670
```

4. Install build-time dependencies with `sudo apt-get install autotools-dev autoconf libtool`

5. Build `wolfssl` and `wolfTPM` with  `/dev/tpmX` support as described (here)[https://github.com/wolfssl/wolfTPM]. Use `configure --enable-devtpm` instead for `wolfTPM`.ifconfig

6. Run a TLS server from `wolfTPM` with `sudo ./examples/tls/tls_server -ecc` and verify it works from a browser by pointing to device with port 11111. Notice that you will get a warning about certificate validation that can be eliminated by installing the CA certificates in the browser.
