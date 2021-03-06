# raspi4-setup

1. Install Raspbian on the SD card. I picked the Desktop version of the 32-bit image from [here](https://www.raspberrypi.org/software/operating-systems/).

2. Boot it for the first time (username is `pi`, password is `raspi4`) and follow the tutorial to update the base system.

3. Enable the TPM chip by following the [instructions](https://letstrust.de/archives/20-Mainline.html). Adjust the formating such that it looks like below.

```
# and load the TPM device tree overlay with
dtoverlay=tpm-slb9670
```

4. Install build-time dependencies with `sudo apt-get install autotools-dev autoconf libtool libssl-dev libjson-c-dev libcurl4-openssl-dev uuid-dev libglib2.0-dev
`

5. Install the latest releases of [tpm2-tss](https://github.com/tpm2-software/tpm2-tss), [tpm2-tools](https://github.com/tpm2-software/tpm2-tools), [tpm2-tss-engine](https://github.com/tpm2-software/tpm2-tss-engine), and [tpm2-abrmd](https://github.com/tpm2-software/tpm2-abrmd) in this order using `./configure; make; sudo make install`. Notice that each of these packages has their own dependencies, listed in the corresponding pages. Copy the DBUS authorization file below and create the `tss` group:

```
sudo cp dist/tpm2-abrmd.conf etc/dbus-1/system.d
sudo useradd --system --user-group tss
sudo usermod -a -G tss pi
```

6. Change the permissions for the device nodes to not require root by creating file `/etc/udev/rules.d/tpm-udev.rules` with contents:

```
KERNEL=="tpm[0-9]*", TAG+="systemd", MODE="0660", OWNER="tss", GROUP="tss"
KERNEL=="tpmrm[0-9]*", TAG+="systemd", MODE="0660", OWNER="tss", GROUP="tss"
```
Reboot for the latter changes to have effect.

## WolfSSL

7. Build `wolfssl` and `wolfTPM` with  `/dev/tpmX` support as described [here](https://github.com/wolfssl/wolfTPM).

8. Run a TLS server from `wolfTPM` with `sudo ./examples/tls/tls_server -ecc` and verify it works from a browser in another machine by pointing to the device with port 11111. Notice that you will get a warning about certificate validation that can be eliminated by accepting/installing the CA certificates in the browser.

The many examples in the `examples` folder should be sufficient for writing applications using TLS and keys stored in the TPM.

## OpenSSL

7. The resource management daemon can now be executed with:

```
sudo systemctl enable tpm2-abrmd.service
sudo systemctl start tpm2-abrmd.service
```

8. Follow [this tutorial](https://www.infineon.com/dgdl/Infineon-OPTIGA_TPM_SLx9670_TPM_2.0-ApplicationNotes-v01_00-EN.pdf?fileId=5546d46271bf4f920171c5598a3a0e7b).

## Troubleshooting

After a couple of failed attempts, I usually start getting errors `0x902 (TPM_RC_OBJECT_MEMORY)` which seems to be related to exhaustion of TPM memory space to hold keys. I manage to solve this by running `sudo tpm2_clear` and running the TLS server setup again.
