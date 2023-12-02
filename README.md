Linux support for HP Envy x360 13" 2022 (13-bf0xxx)
===================================================

HP Envy x360 13-bf0xxx comes with terrible Linux support.
The default Linux kernel 6.1 under Debian 12 Bookworm does not work properly with the speakers.
Suspend/resume support is also not there.
This is a summary of all the hacks I found online.
The problem is not Linux not supporting the devices on Envy x360 properly, but rather disgusting BIOS and configuration errors that do not follow the standard.
Booting and working under Windows do not mean proper driver support, but unfortunately many vendors do think so, and proper standard-conforming support is simply the lowest priority when it just works on Windows.

Installing the patched kernel
-----------------------------

This repo contains README.md and a single deb package of a patched Linux kernel for Debian 12 Bookworm.
The package should theoretically also work for Ubuntu, but no testing is done.
Download the deb package and just do `sudo dpkg -i linux-image-6.1.26-fast+_6.1.26-fast+-2_amd64.deb` to install the new kernel.
Reboot.

Please beware that installing a random kernel from the Internet is dangerous.
If you do not trust this kernel image, see below.

Patching and compiling your own kernel
--------------------------------------

If you do not trust this kernel image, then you can apply the 3 patches (in this order)
```
0001-Cirrus-hack.patch
0002-drm-i915-pps-Add-get_pps_idx-hook-as-part-of-pps_get.patch
0003-drm-i915-edp-Fix-warning-as-vdd-went-down-without-dr.patch
```
in this repo to kernel 6.1.62 and then compile and install it on your own.
There are plenty of resources online on how to patch, compile and install your own kernel.
If there are a lot of requests from people, I can show the details of all the steps here for Debian 12, and Ubuntu should be similar.

Cirrus firmware
---------------

Then, clone `https://github.com/CirrusLogic/linux-firmware.git` and copy the whole subdirectory `linux-firmware/cirrus` to `/lib/firmware/` so that you now have `/lib/firmware/cirrus`.
Reboot.
You should now have working speakers and suspend/resume support.

Suspend/resume
--------------

A workaround is in the kernel package to work around the issue that the laptop internal display is connected to the ports in a way that the kernel doesn't expect.
Booting and waking up confuse the kernel and could result in kernel panics.
Fortunately, if you install the kernel deb package in this repo or compile your own kernel after applying the patches, then this workaround is already included.

Suspend/resume is still a bit broken after installing the patched kernel.
Linux thinks there is both `s2idle` and `s3` sleep support but I've never successfully woken up from `s3` even once.
Fortunately, `s2idle` is the default.
On this laptop, `s2idle` drops the power consumption to about 300mW, which is even lower than some laptops I've seen at `s3` and can sleep for more than a week, so it's good enough.
