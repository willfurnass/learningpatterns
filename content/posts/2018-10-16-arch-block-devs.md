+++
title = "Arch Linux: where have my drivers gone?"
date = 2018-10-16T18:12:41Z
tags  = ["arch", "linux", "drivers", "kernel"]
featured_image = ""
description = ""
+++

I've been running [Arch Linux][arch] on my personal and work laptops for a couple of years and love it: 
it's very configurable, 
cutting edge,
(surprisingly) stable and
has [awesome documentation][arch-wiki] (useful for *all* Linux users).

However, there's one aspect of it that really bugs me: 
files related to the current kernel and [kernel modules][kern-mods] (inc. device drivers) are 
aggressively purged as soon as you upgrade the [linux](https://www.archlinux.org/packages/core/x86_64/linux/) kernel package i.e.
**the kernel modules built against the running kernel are no longer on disk so cannot be dynamically loaded**.
This issue has a variety of effects.

I learned about this Arch-specific issue some time ago when I plugged in a USB stick.  
It wasn't automounted, either by the neat [udiskie][udiskie] media automounter/manager or by Gnome's [nautilus][nautilus] file manager:

I then ran [lsblk][lsblk] to see if the [block device][blk-dev] corresponding to the USB stick was visible to the OS:

```
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme0n1        259:0    0 953.9G  0 disk  
├─nvme0n1p1    259:1    0   512M  0 part  /boot
└─nvme0n1p3    259:2    0 953.1G  0 part  
  └─luks       254:0    0 953.1G  0 crypt 
    ├─vg0-swap 254:1    0    48G  0 lvm   [SWAP]
    ├─vg0-root 254:2    0   200G  0 lvm   /
    └─vg0-home 254:3    0   690G  0 lvm   /home
```

Nope.  However, running [lsusb][lsusb] showed that the device was visible on the USB bus:

```
Bus 001 Device 003: ID 04f3:21d5 Elan Microelectronics Corp. 
Bus 001 Device 002: ID 0cf3:e300 Qualcomm Atheros Communications 
Bus 001 Device 004: ID 0c45:6713 Microdia 
Bus 001 Device 019: ID 058f:1234 Alcor Micro Corp. Flash Drive
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Yep: *Alcor Micro Corp. Flash Drive* was my USB stick.

It turned out that no block device had been created as the relevant kernel driver couldn't be found: 
I'd upgraded the kernel OS package since booting the machine and all old kernel files inc. kernel modules had been deleted.

If you find that you can't mount a USB stick, mount a [CIFS][cifs] share or have some other reason to believe that some device driver is unavailable then check that:

```sh
uname -r
```

(the release of the currently running kernel) is equal to

```sh
pacman -Q linux
```

which is the version of the current `linux` OS package (kernel files and kernel module files on disk).
**If not**, you'll need to reboot to load the newer kernel.

**Take-home message**: If [pacman][pacman] (the Arch package management utility) tells you that you need to upgrade your kernel, reboot ASAP afterwards.
One of several reasons why Arch Linux would make for a very poor choice on a server host OS, despite being an awesome desktop OS!

Note that other Linux OSs are more conservative when it comes to keeping older kernels around: 
Debian, for example, keeps many older kernels on disk post-upgrade, which has its own issues 
(boot partitions getting full being the main one).  
However, that approach does allow you to continue using all stuff built against the current kernel until you reboot plus gives you fallback options if you encounter problems with a new kernel. 
The Arch devs assume that you'll never want to revert to an older kernel.  Saying that, that's yet to be a problem for me on my laptops!

Thanks to *Namarrgon* in [#archlinux on Freenode IRC][arch-irc] for pointing me at the root cause of this issue!


[arch-irc]: https://wiki.archlinux.org/index.php/IRC_channel
[arch-wiki]: https://wiki.archlinux.org/
[arch]: https://www.archlinux.org/
[blk-dev]: https://en.wikipedia.org/wiki/Device_file#Block_devices
[cifs]: https://linux.die.net/man/8/mount.cifs
[kern-mods]: https://wiki.archlinux.org/index.php/Kernel_module
[lsblk]: https://linux.die.net/man/8/lsblk
[lsusb]: https://linux.die.net/man/8/lsusb
[nautilus]: https://en.wikipedia.org/wiki/GNOME_Files
[udiskie]: https://github.com/coldfix/udiskie
[pacman]: https://wiki.archlinux.org/index.php/pacman
