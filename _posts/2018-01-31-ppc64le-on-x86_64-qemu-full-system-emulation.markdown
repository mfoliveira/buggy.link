---
title:	"ppc64le on x86_64: QEMU full-system emulation"
date:	2018-01-31
---

# ppc64le on x86_64: QEMU full-system emulation

You can run a `ppc64le` virtual machine on your `x86_64` computer with QEMU full-system emulation!

## QEMU

Let's build the QEMU target for full-system emulation of the `ppc64le` architecture (`ppc64-softmmu`).

Download (for example, a stable release):

```
$ wget https://download.qemu.org/qemu-2.11.0.tar.xz
$ tar xf qemu-2.11.0.tar.xz 
$ cd qemu-2.11.0
```

Build:

```
$ ./configure --target-list=ppc64-softmmu
...
host CPU          x86_64
host big endian   no
target list       ppc64-softmmu
...

$ make -j$(nproc)
...
  LINK    ppc64-softmmu/qemu-system-ppc64
```

Run:

```
$ cp ppc64-softmmu/qemu-system-ppc64 ~/bin

$ qemu-system-ppc64 --version
QEMU emulator version 2.11.0
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers
```

## QEMU machines

QEMU provides a number of _machine_ types for PowerPC (for the various PowerPC _platforms_).

The `ppc64le` architecture currently runs on 2 platforms:

 - `pseries`: the virtualized environment traditionally provided by the PowerVM hypervisor
    for LPARs (logical partitions -- virtual machines), and later introduced in QEMU for
    KVM guests.

 - `powernv`: the non-virtualized (`nv`) environment provided by the OPAL firmware, to run
    without a hypervisor and/or as a hypervisor (KVM host), also known as _bare-metal_ mode.  
    (note: _originally_ non-virtualized, more recently introduced in QEMU; so sort of  _virtualized_ here :-)

The QEMU machines for those are:

```
$ qemu-system-ppc64 -machine help
Supported machines are:
...
powernv              IBM PowerNV (Non-Virtualized)
...
pseries              pSeries Logical Partition (PAPR compliant) (alias of pseries-2.11)
...
```

## QEMU processors

Similarly, QEMU provides a number of _processor_ types for PowerPC.

The `ppc64le` architecture currently runs on PowerPC _server_-class processors,
also known as "POWER" processors (originally manufactured by IBM), on POWER8 and later.

The QEMU processors range from POWER5+ to POWER9, currently:

```
$ qemu-system-ppc64 -cpu help | grep power[0-9] | grep alias
PowerPC power5+          (alias for power5+_v2.1)
PowerPC power5gs         (alias for power5+_v2.1)
PowerPC power7           (alias for power7_v2.3)
PowerPC power7+          (alias for power7+_v2.1)
PowerPC power8e          (alias for power8e_v2.1)
PowerPC power8nvl        (alias for power8nvl_v1.0)
PowerPC power8           (alias for power8_v2.0)
PowerPC power9           (alias for power9_v2.0)
```

## QEMU files and options for PowerPC platforms

The QEMU machine for the `pseries` platform requires 2 files:

 - `spapr-rtas.bin`:
    _Server_ POWER Architecture Platform Requirements -
    [Run-Time Abstraction Services](https://en.wikipedia.org/wiki/Run-Time_Abstraction_Services)  
    (otherwise, `qemu-system-ppc64: Could not find LPAR rtas 'spapr-rtas.bin'`)

 - `slof.bin`:
    [Slimline](https://www.openfirmware.info/SLOF)
    [Open Firmware](https://en.wikipedia.org/wiki/Open_Firmware)  
    (otherwise, `qemu-system-ppc64: Could not find LPAR firmware 'slof.bin'`)

Both files ship with QEMU, and should be automatically installed at the right places with `make install`.

However, for our local QEMU binary (for testing purposes) that is not done, so copy the files to the
current directory when QEMU runs:

```
$ cp qemu-2.11.0/pc-bios/spapr-rtas.bin .
$ cp qemu-2.11.0/pc-bios/slof.bin .
```

And these QEMU options allow you to run without a couple of other files:

 - `-nodefaults`  
    (otherwise, `qemu-system-ppc64: Initialization of device VGA failed: failed to find romfile "vgabios-stdvga.bin"`)

 - `-nographic`  
    (otherwise, `Could not read keymap file: 'en-us'`)

## QEMU command line

Finally, this QEMU command line is good for starters: `pseries` platform, `power8` processor,
and text console on terminal (tip: do not press `ctrl-c` for now :-) 

```
$ qemu-system-ppc64 \
    -machine pseries \
    -cpu power8 \
    -nodefaults \
    -nographic \
    -serial stdio
```

## Alpine Linux

Now, let's test that with something real, _Alpine Linux_.

Download:
```
$ wget http://dl-cdn.alpinelinux.org/alpine/v3.7/releases/ppc64le/alpine-vanilla-3.7.0-ppc64le.iso
```

Run:
```
$ qemu-system-ppc64 \
    -machine pseries \
    -cpu power8 \
    -nodefaults \
    -nographic \
    -serial stdio \
    -cdrom alpine-vanilla-3.7.0-ppc64le.iso
```

And you can (slowly) watch SLOF load, populate the PowerPC device tree with
the devices emulated by QEMU, search for boot devices, and boot one.

```
SLOF **********************************************************************
QEMU Starting
 Build Date = Jul 24 2017 15:15:46
 FW Version = git-89f519f09bf85091
 Press "s" to enter Open Firmware.

Populating /vdevice methods
Populating /vdevice/vty@71000000
Populating /vdevice/nvram@71000001
Populating /vdevice/v-scsi@71000002
       SCSI: Looking for devices
          8200000000000000 CD-ROM   : "QEMU     QEMU CD-ROM      2.5+"
Populating /pci@800000020000000
No NVRAM common partition, re-initializing...
Scanning USB 
Using default console: /vdevice/vty@71000000
     
  Welcome to Open Firmware

  Copyright (c) 2004, 2017 IBM Corporation All rights reserved.
  This program and the accompanying materials are made available
  under the terms of the BSD License available at
  http://www.opensource.org/licenses/bsd-license.php


Trying to load:  from: disk ... 
E3405: No such device
Trying to load:  from: /vdevice/v-scsi@71000002/disk@8200000000000000 ...   Successfully loaded
```

And GRUB2 comes to the screen!

```
Welcome to GRUB!




                             GNU GRUB  version 2.02

 +----------------------------------------------------------------------------+
 |*Linux vanilla                                                              | 
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            | 
 +----------------------------------------------------------------------------+

      Use the ^ and v keys to select which entry is highlighted.          
      Press enter to boot the selected OS, `e' to edit the commands       
      before booting or `c' for a command-line.
```

You can edit the boot entry to remove the `quiet` option to watch the kernel
booting in more detail (as that is a bit slow); press `ctrl-x` to boot.

```
                             GNU GRUB  version 2.02

 +----------------------------------------------------------------------------+
 |setparams 'Linux vanilla'                                                   | 
 |                                                                            |
 |linux        /boot/vmlinuz modules=loop,squashfs,sd-mod,usb-storage nomodes\|
 |et console=hvc0                                                             |
 |initrd        /boot/initramfs-vanilla                                       |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            | 
 +----------------------------------------------------------------------------+

      Minimum Emacs-like screen editing is supported. TAB lists           
      completions. Press Ctrl-x or F10 to boot, Ctrl-c or F2 for          
      a command-line or ESC to discard edits and return to the GRUB menu. 
```

And it boots Linux:

```
  Booting a command list

OF stdout device is: /vdevice/vty@71000000
Preparing to boot Linux version 4.9.65 (buildozer@build-3-7-ppc64le) (gcc version 6.4.0 (Alpine 6.4.0) ) #1-Alpine SMP Fri Nov 24 17:48:26 GMT 2017
...
instantiating rtas at 0x000000001daf0000... done
...
Booting Linux via __start() @ 0x0000000002000000 ...
...
Linux version 4.9.65 (buildozer@build-3-7-ppc64le) (gcc version 6.4.0 (Alpine 6.4.0) ) #1-Alpine SMP Fri Nov 24 17:48:26 GMT 2017
Found initrd at 0xc000000002d00000:0xc0000000033c6535
Using pSeries machine description
...
Alpine Init 3.2.0-r0
 * Loading boot drivers: squashfs: version 4.0 (2009/01/31) Phillip Lougher
...
 * Mounting boot media: ibmvscsi 71000002: SRP_VERSION: 16.a
...
 * Installing packages to root filesystem: (1/16) Installing musl (1.1.18-r2)
...
   OpenRC 0.24.1.a941ee4a0b is starting up Linux 4.9.65 (ppc64le)
...

Welcome to Alpine Linux 3.7
Kernel 4.9.65 on an ppc64le (/dev/hvc0)

localhost login:
```

And you can check the environment around:

```
localhost login: root
Welcome to Alpine!
...

localhost:~# uname -sm
Linux ppc64le

localhost:~# cat /proc/cpuinfo
processor	: 0
cpu		: POWER8 (architected), altivec supported
clock		: 1000.000000MHz
revision	: 2.0 (pvr 004d 0200)

timebase	: 512000000
platform	: pSeries
model		: IBM pSeries (emulated by qemu)
machine		: CHRP IBM pSeries (emulated by qemu)
```

Yeehaw!

It's indeed a bit slow, as that is full-system emulation, but it can very well
be used to run small programs which are not processor intensive, just like the
ones to learn and practice the basics about `ppc64le`. ;-)
