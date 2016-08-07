The [Intel Precise Touch (IPTS)](http://www.intelfreepress.com/news/universal-stylus-initiative/9678/) Linux HID driver for compatible touchscreens; such a device is found in the Microsoft Surface Pro 4.

## Related Links

 * [upstream driver](https://github.com/ipts-linux-org/ipts-linux) and its [wiki page](https://github.com/ipts-linux-org/ipts-linux/wiki)
  * [UEXS002 — Fluid and Flowing Natural Writing with Intel® Precise Touch Technology](http://myeventagenda.com/sessions/0B9F4191-1C29-408A-8B61-65D7520025A8/7/5)
 * [Installing Debian on the Microsoft Surface Pro 4](https://github.com/jimdigriz/debian-mssp4)

# Preflight

    git clone https://github.com/jimdigriz/ipts-linux.git
    cd ipts-linux
    git checkout ipts-v4.6.x

    cat <<'EOF' | sudo tee /etc/apt/sources.list.d/debian-backports.list >/dev/null
    deb http://http.debian.net/debian jessie-backports main non-free contrib
    #deb-src http://http.debian.net/debian jessie-backports main non-free contrib
    EOF
    sudo apt-get update
    sudo apt-get install linux-source-4.6

You also need to run the following to prep the `i915` driver:

    cat <<'EOF' | sudo tee /etc/modprobe.d/i915.conf > /dev/null
    options i915 enable_guc_submission=Y guc_log_level=3
    
    blacklist mei_itouch_hid
    EOF

**N.B.** we blacklist the `mei_itouch_hid` (touchscreen) driver as it may make your system unstable and so you do not want to load it automatically on boot

You will need the OpenCL kernel binaries that are located in your Windows partition at `%WINDIR%\INF\PreciseTouch` and you need to copy the contents of it all to `/itouch` and add the following symbolic links:

    sudo ln -s iaPreciseTouchDescriptor.bin /itouch/integ_descriptor.bin
    sudo ln -s SurfaceTouchServicingSFTConfigMSHW0078.bin /itouch/integ_sft_cfg_skl.bin
    sudo ln -s SurfaceTouchServicingDescriptorMSHW0078.bin /itouch/vendor_descriptor.bin
    sudo ln -s SurfaceTouchServicingKernelSKLMSHW0078.bin /itouch/vendor_kernel_skl.bin

Once done, the directory structure should look like:

    $ tree /itouch
    /itouch
    +-- iaPreciseTouchDescriptor.bin
    +-- integ_descriptor.bin -> iaPreciseTouchDescriptor.bin
    +-- integ_sft_cfg_skl.bin -> SurfaceTouchServicingSFTConfigMSHW0078.bin
    +-- SurfaceTouchServicingDescriptorMSHW0076.bin
    +-- SurfaceTouchServicingDescriptorMSHW0078.bin
    +-- SurfaceTouchServicingKernelSKLMSHW0076.bin
    +-- SurfaceTouchServicingKernelSKLMSHW0078.bin
    +-- SurfaceTouchServicingSFTConfigMSHW0076.bin
    +-- SurfaceTouchServicingSFTConfigMSHW0078.bin
    +-- SurfaceTouchServicingTouchBlobMSHW0076.bin
    +-- SurfaceTouchServicingTouchBlobMSHW0078.bin
    +-- vendor_descriptor.bin -> SurfaceTouchServicingDescriptorMSHW0078.bin
    \-- vendor_kernel_skl.bin -> SurfaceTouchServicingKernelSKLMSHW0078.bin

# Build

    xzcat /usr/src/linux-config-4.6/config.amd64_none_amd64.xz > .config
    cat <<'EOF' >> .config
    CONFIG_BLK_DEV_NVME=y
    CONFIG_MODULE_SIG=n
    CONFIG_SYSTEM_TRUSTED_KEYRING=n
    CONFIG_INTEL_MEI_ITOUCH=m
    EOF
    make oldconfig
    CONCURRENCY_LEVEL=`getconf _NPROCESSORS_ONLN` fakeroot make-kpkg --initrd --append-to-version=-mssp4 kernel_image

# Install

    sudo dpkg -i ../linux-image-4.6.4-mssp4+_4.6.4-mssp4+-10.00.Custom_amd64.deb

Now reboot.

# Usage

    sudo modprobe mei_itouch_hid

# Troubleshooting

Edit `drivers/misc/itouch/itouch.h` to enabled debugging, recompile and after a reboot, look at the output of `dmesg -w`.
