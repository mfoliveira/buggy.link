---
title:	"ppc64le on x86_64: QEMU user-mode emulation"
date:	2018-02-03
---

# ppc64le on x86_64: QEMU user-mode emulation

You can run `ppc64le` binaries on your `x86_64` computer with QEMU user-mode emulation!

This is a quick way to run simple programs.  It needs less resources than
[full-system emulation]({{ site.baseurl }}{% post_url 2018-01-31-ppc64le-on-x86_64-qemu-full-system-emulation %}).
On the other hand, it provides an environment that is less similar to an
original system, of course, and relies on the local filesystem.

Either way, it can be handy to test and debug simple programs locally.

## QEMU

Let's build the QEMU target for user-mode emulation of the `ppc64le` architecture (`ppc64le-linux-user`).

Download (for example, a stable release):

```
$ wget https://download.qemu.org/qemu-2.11.0.tar.xz
$ tar xf qemu-2.11.0.tar.xz
$ cd qemu-2.11.0
```

Build:

```
$ ./configure --target-list=ppc64le-linux-user
...
host CPU          x86_64
host big endian   no
target list       ppc64le-linux-user
...
```

**Or** you can build both the user-mode and full-system emulation targets:

```
$ ./configure --target-list=ppc64le-linux-user,ppc64-softmmu
...
host CPU          x86_64
host big endian   no
target list       ppc64le-linux-user ppc64-softmmu
...
```

And:

```
$ make -j $(nproc)
...
  LINK    ppc64le-linux-user/qemu-ppc64le
```

Run:

```
$ cp ppc64le-linux-user/qemu-ppc64le ~/bin

$ qemu-ppc64le --version
qemu-ppc64le version 2.11.0
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers
```

## Alpine Linux

Now, let’s test that with the `busybox` binary from _Alpine Linux_
(used in the [full-system emulation]({{ site.baseurl }}{% post_url 2018-01-31-ppc64le-on-x86_64-qemu-full-system-emulation %})
post).

Download:

```
$ wget http://dl-cdn.alpinelinux.org/alpine/v3.7/releases/ppc64le/alpine-vanilla-3.7.0-ppc64le.iso
```

Extract `busybox` and the _dynamic loader_ (dependency):

```
$ mkdir mount-iso
$ sudo mount alpine-vanilla-3.7.0-ppc64le.iso mount-iso
$ gzip -dc mount-iso/boot/initramfs-vanilla | cpio -id bin/busybox lib/ld-musl-powerpc64le.so.1
$ sudo umount mount-iso
```

Notice that both files are `ppc64le` binaries:

```
$ file bin/busybox 
bin/busybox: ELF 64-bit LSB shared object, 64-bit PowerPC or cisco 7500, version 1 (SYSV), dynamically linked (uses shared libs), stripped

$ file lib/ld-musl-powerpc64le.so.1 
lib/ld-musl-powerpc64le.so.1: ELF 64-bit LSB shared object, 64-bit PowerPC or cisco 7500, version 1 (SYSV), dynamically linked, stripped
```

Let's try `busybox`'s `uname` function:

```
$ qemu-ppc64le lib/ld-musl-powerpc64le.so.1 bin/busybox uname -sm
Linux ppc64le
```

Cool, it's `ppc64le`!

In order not to include the dynamic loader and `busybox` in commands, you can:

 - Set the `QEMU_LD_PREFIX` environment variable (_ELF interpreter prefix_)  
   to where `lib/ld-musl-powerpc64le.so.1` is (`$PWD)` or `./` in this example):  
   `$ export QEMU_LD_PREFIX=$PWD`

 - _UPDATE (2018-03-17) [the method above is preferred over this one]:_  
   Copy the dynamic loader to `/lib` (does not affect other stuff):  
   `$ sudo cp lib/ld-musl-powerpc64le.so.1 /lib`

 - Create a symlink to `busybox` named after the function:  
   `$ ln -s bin/busybox uname`

It looks better (say, more normal) now:

```
$ qemu-ppc64le ./uname -sm
Linux ppc64le
```

And that runs on a `x86_64` computer!

```
$ uname -sm
Linux x86_64
```

Now let's try `busybox`'s `md5sum` function -- something more complex, processor-instruction wise:

```
$ ln -s bin/busybox md5sum

$ qemu-ppc64le ./md5sum alpine-vanilla-3.7.0-ppc64le.iso
ef574f1fe9b2b021a43c85ebd451b47a  alpine-vanilla-3.7.0-ppc64le.iso
```

It perfectly matches the native `md5sum`:

```
$ md5sum alpine-vanilla-3.7.0-ppc64le.iso
ef574f1fe9b2b021a43c85ebd451b47a  alpine-vanilla-3.7.0-ppc64le.iso
```

Hooray!

So, QEMU user-mode emulation can be of great help with simple programs, like the basics on `ppc64le`.
