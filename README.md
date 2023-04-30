Linux support for HP Envy x360 13-bf0xxx
========================================

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

(Please beware that installing a random kernel from the Internet is dangerous.
This repo is simply me writing things down in case I need to install Linux again, and in case this is useful for other 13-bf0xxx users.
If there are enough people who need to run Linux on this machine and need the fixes, I will post the detailed steps here so that you can build your own Linux kernel with patches instead of just installing a package from a stranger.)

Cirrus firmware
---------------

Then, clone `https://github.com/CirrusLogic/linux-firmware.git` and copy the whole subdirectory `linux-firmware/cirrus` to `/lib/firmware/` so that you now have `/lib/firmware/cirrus`.
Reboot.
You should now have working speakers and suspend/resume support.

Suspend/resume
--------------

A workaround is in the kernel package to work around the issue that the laptop internal display is connected to the ports in a way that the kernel doesn't expect.
Booting and waking up confuse the kernel and could result in kernel panics.

Suspend/resume is still a bit broken after installing the patched kernel.
Linux thinks there is both `s2idle` and `s3` sleep support but I've never successfully woken up from `s3` even once.
Fortunately, `s2idle` is the default.
On this laptop, `s2idle` drops the power consumption to about 500mW, which is even lower than some laptops I've seen at `s3` and can sleep for up to 6 days, so it's good enough.

There is still something weird about suspend/resume.
If the laptop goes to sleep by closing the lid, the laptop display will then suffer from randomly dropped frames after waking up.
I found that disabling lid actions helped.
Go to `/etc/systemd/logind.conf` and add the following lines:

```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Save and reboot.
Closing the lid is now completely ignored and will not put the system to sleep, so you need to manually click the suspend button in the OS.
However, now waking up from sleep no longer has frame drop issues.

Speaker issues after waking up (Pipewire)
-----------------------------------------

For reason unknown to me, waking up from suspend also sometimes breaks the speakers.
The speakers still work, but audio is mixed with loud pop noises.
This seems to be a software issue, since uninstalling pipewire and installing pulseaudio makes the problem go away.
