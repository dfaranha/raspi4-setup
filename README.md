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

6. Run a TLS server from `wolfTPM` with `sudo ./examples/tls/tls_server -ecc` and verify it works from a browser in another machine by pointing to the device with port 11111. Notice that you will get a warning about certificate validation that can be eliminated by accepting/installing the CA certificates in the browser.

The many examples in the `examples` folder should be sufficient for writing applications using TLS and keys stored in the TPM.

## Troubleshooting

After a couple of failed attempts, I usually start getting errors `0x902 (TPM_RC_OBJECT_MEMORY)` which seems to be related to exhaustion of TPM memory space to hold keys. I manage to solve this by running `sudo tpm2_clear` and running the TLS server setup again.
