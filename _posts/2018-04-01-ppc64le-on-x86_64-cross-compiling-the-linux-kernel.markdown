---
title:	'ppc64le on x86_64: Cross crompiling the Linux kernel'
date:	2018-04-01
---

# ppc64le on x86_64: cross compiling the Linux kernel

Let's use the `ppc64le` [cross-compilers][post-cross-compilers] on your `x86_64` computer to build the Linux kernel!  
And, obviously, boot-test it with QEMU [full-sytem][post-qemu-full] emulation!


# Build the Linux kernel

To cross compile the Linux kernel, just set the `ARCH` and `CROSS_COMPILE` variables for `make`.  

For example:  
`ARCH=powerpc CROSS_COMPILE=powerpc64le-linux-gnu- make -j $(nproc)`  

Note: you can specify the _path_ to the cross compiler if it's not in `$PATH`.  
Note: there **is** a trailing _dash_ after the target-platform triplet.

You can also `export` the variables and not specify them in `make` commands.  
For example, this is equivalent to the command line above:

```
$ export ARCH=powerpc
$ export CROSS_COMPILE=powerpc64le-linux-gnu-

$ make -j $(nproc)
```

So, let's build the Linux kernel with that.

_Note:_ These variables can be set either _before_ `make` (as environment variables) or _after_ it (as variable overrides), as in this [documentation][make-variables] -- but apparently not all build variables can be set in both ways.


Download and extract:

```
$ wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.15.10.tar.xz
$ tar xf linux-4.15.10.tar.xz
```

Configure (this doesn't need `CROSS_COMPILE` -- no _target_ code involved -- but it won't hurt):  
`$ make ARCH=powerpc pseries_le_defconfig`

Build:  

- Ubuntu 16.04 LTS, Fedora 27:  
`$ ARCH=powerpc CROSS_COMPILE=powerpc64le-linux-gnu- make -j $(nproc)`

- RHEL/CentOS 7:  
`$ ARCH=powerpc CROSS_COMPILE=powerpc64-linux-gnu- make -j $(nproc)`

- Any Distro (Tarball):  
`$ ARCH=powerpc CROSS_COMPILE=~/bin/powerpc64le-power8--glibc--stable-2018.02-2/bin/powerpc64le-linux- make -j $(nproc)`

Check:

```
$ file vmlinux
vmlinux: ELF 64-bit LSB executable, 64-bit PowerPC [...]
```

Cool.  

Well, it doesn't make sense to `make install` that on your local system.  
But we'll need the `modules_install` target to get the kernel modules to test.  
So let's use the `INSTALL_MOD_PATH` variable.

Install modules:
```
$ ARCH=powerpc CROSS_COMPILE=powerpc64-linux-gnu- \
  INSTALL_MOD_PATH=mod-install-dir \
  make -j $(nproc) modules_install
...
  DEPMOD  4.15.10
```

And the usual `/lib/modules/<version>` directory structure is there:

```
$ find mod-install-dir/ -name 4.15.10
mod-install-dir/lib/modules/4.15.10
```

Nice, let's test it.

# Install Alpine Linux

Let's install [Alpine Linux][alpine-linux] to a disk image with [QEMU full-system emulation][post-qemu-full].  

Download the ISO image:  
`$ wget http://dl-cdn.alpinelinux.org/alpine/v3.7/releases/ppc64le/alpine-vanilla-3.7.0-ppc64le.iso`

Create an empty disk image:  
`$ dd if=/dev/zero of=alpine-disk.img bs=1 seek=1G count=0`

Start QEMU with the <F12>ISO image, disk image, networking, and 1 gigabyte of RAM (for temporary files):

```
$ qemu-system-ppc64 \
    -machine pseries \
    -cpu power8 \
    -m 1G \
    -nodefaults \
    -nographic \
    -serial stdio \
    -cdrom alpine-vanilla-3.7.0-ppc64le.iso \
    -drive file=alpine-disk.img \
    -net nic \
    -net user
```

The devices `sda` and `eth0` should be available:

```
localhost login: root
Welcome to Alpine!

...

localhost:~# uname -srm
Linux 4.9.65 ppc64le

localhost:~# ls /dev/sd*
/dev/sda

localhost:~# ip link list
...
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff

```

Run `setup-alpine` to perform the installation to disk:

```
localhost:~# setup-alpine
Available keyboard layouts:
...
Select keyboard layout [none]: us
Available variants: <...>
Select variant []: us-intl
 * Caching service dependencies ... [ ok ]
 * Setting keymap ... [ ok ]
Enter system hostname (short form, e.g. 'foo') [localhost]: alpine
...
Which one do you want to initialize? (or '?' or 'done') [eth0] 
Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp] 
Do you want to do any manual network configuration? [no] 
...
New password: 
...
passwd: password for root changed by root
Which timezone are you in? ('?' for list) [UTC] 
...
HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] 

Available mirrors:
1) dl-cdn.alpinelinux.org
...
Enter mirror number (1-28) or URL to add (or r/f/e/done) [f]: 1
...
Which SSH server? ('openssh', 'dropbear' or 'none') [openssh] 
...
Which NTP client to run? ('busybox', 'openntpd', 'chrony' or 'none') [chrony] 
...
Available disks are:
  sda	(1.1 GB QEMU     QEMU HARDDISK   )
Which disk(s) would you like to use? (or '?' for help or 'none') [none] sda
...
The following disk is selected:
  sda	(1.1 GB QEMU     QEMU HARDDISK   )
How would you like to use it? ('sys', 'data', 'lvm' or '?' for help) [?] sys
WARNING: The following disk(s) will be erased:
  sda	(1.1 GB QEMU     QEMU HARDDISK   )
WARNING: Erase the above disk(s) and continue? [y/N]: y
Creating file systems...
Installing system on /dev/sda4:
Installing grub on /dev/sda1
Installing for powerpc-ieee1275 platform.
...
Installation is complete. Please reboot.

alpine:~# reboot
...
reboot: Restarting system
```

And it boots from the disk image:

```
alpine login: root
Password: 
Welcome to Alpine!
...

alpine:~# mount | grep ' / '
/dev/sda4 on / type ext4 (rw,relatime,data=ordered)

alpine:~# poweroff
```

# Install the Kernel

Setup the disk image as loop device, load partitions, mount the root partition, and copy kernel files.

```
$ sudo losetup --find --show alpine-disk.img 
/dev/loop1

$ sudo kpartx -av /dev/loop1 
add map loop1p1 (253:3): 0 16384 linear /dev/loop1 2048
add map loop1p2 (253:4): 0 204800 linear /dev/loop1 18432
add map loop1p3 (253:5): 0 524288 linear /dev/loop1 223232
add map loop1p4 (253:6): 0 1349632 linear /dev/loop1 747520

$ mkdir disk
$ sudo mount /dev/mapper/loop1p4 disk/

$ sudo mkdir disk/linux
$ sudo cp -r \
  linux-4.15.10/vmlinux \
  linux-4.15.10/System.map \
  linux-4.15.10/mod-install-dir/ \
  disk/linux/

$ sudo umount disk/

$ sudo kpartx -dv /dev/loop1 
del devmap : loop1p4
del devmap : loop1p3
del devmap : loop1p2
del devmap : loop1p1

$ sudo losetup -d /dev/loop1
```

Now boot Alpine again with QEMU again (the ISO image and more RAM is not required anymore):

```
$ qemu-system-ppc64 \
    -machine pseries \
    -cpu power8 \
    -nodefaults \
    -nographic \
    -serial stdio \
    -drive file=alpine-disk.img \
    -net nic \
    -net user
```

Copy the kernel files to `/boot` and `/lib/modules`, create an initramfs, and update grub config:

```
alpine:~# mv /linux/vmlinux /boot/vmlinux-4.15.10
alpine:~# mv /linux/System.map /boot/System.map-4.15.10
alpine:~# mv /linux/mod-install-dir/lib/modules/4.15.10/ /lib/modules/

alpine:~# mkinitfs -o /boot/initramfs-4.15.10 4.15.10
==> initramfs: creating /boot/initramfs-4.15.10

alpine:~# touch /etc/update-extlinux.conf   #  only once.

alpine:~# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.15.10
done

alpine:~# reboot
```

And it goes on to boot the new kernel:

```
                             GNU GRUB  version 2.02

 +----------------------------------------------------------------------------+
 |*GNU/Linux                                                                  | 
 | Advanced options for GNU/Linux                                             |
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

...

  Booting `GNU/Linux'

Loading Linux 4.15.10 ...
Loading initial ramdisk ...
OF stdout device is: /vdevice/vty@71000000
Preparing to boot Linux version 4.15.10 (mauricfo@t470.localdomain) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)) #1 SMP Sun Mar 18 13:56:26 -03 2018
```

And there it is!

```
alpine:~# uname -a
Linux alpine 4.15.10 #1 SMP Sun Mar 18 13:56:26 -03 2018 ppc64le Linux
```

The `ppc64le` kernel cross-compiled on `x86_64` just booted! Cheers!



[post-cross-compilers]: {{ site.baseurl }}{% post_url 2018-03-17-ppc64le-on-x86_64-cross-compilers %}
[post-qemu-full]: {{ site.baseurl }}{% post_url 2018-01-31-ppc64le-on-x86_64-qemu-full-system-emulation %}
[make-variables]: https://ftp.gnu.org/old-gnu/Manuals/make-3.79.1/html_chapter/make_6.html#SEC63
[alpine-linux]: https://alpinelinux.org
