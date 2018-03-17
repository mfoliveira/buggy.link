---
title:	'ppc64le on x86_64: Cross Compilers'
date:	2018-03-17
---

# ppc64le on x86_64: cross compilers

You can build `ppc64le` executables on your `x86_64` computer with a [_cross compiler_][wikipedia-cross-compiler].  

That is, a compiler that runs on a certain _host_ platform (e.g., `x86_64`) and generates
executable code for a different _target_ platform (e.g., `ppc64le`).

This can be useful to build software such as simple programs, the Linux kernel,
and the OPAL firmware.

And then you run it locally with an emulator (e.g., QEMU [full-system][qemu-full] or
[user-mode][qemu-user] emulation)
or remotely in real hardware (most people don't have `ppc64le` systems at home _yet!_).

## Cross Compiler Packages and Tarballs

Several Linux distributions provide cross compiler packages for `ppc64le` (and `ppc64` -- big-endian,
which is required sometimes).

For distros that don't, there are cross compiler _tarballs_ available from Bootlin (read on)!  
And, of course, you can build your own, but that's not covered here for simplicity's sake. :-)

The cross compiler packages and commands are usually named after the _target_ platform.  
Packages usually have the tool name first, and commands the tool name last; for example:
- `ppc64le`:
  - Package: `gcc-powerpc64le-linux-gnu-gcc`
  - Command: `powerpc64le-linux-gnu-gcc`
- `ppc64`:
  - Package: `gcc-powerpc64-linux-gnu-gcc`
  - Command: `powerpc64-linux-gnu-gcc`

Check if your distro is listed here and install its packages, otherwise use the tarball for any distro.

### Packages for Ubuntu 16.04 LTS

Search:

```
$ apt-cache search --names-only powerpc64 | grep -v -- '-[0-9]' | sort
binutils-powerpc64le-linux-gnu - GNU binary utilities, for powerpc64le-linux-gnu target
binutils-powerpc64-linux-gnu - GNU binary utilities, for powerpc64-linux-gnu target
...
gcc-powerpc64le-linux-gnu - GNU C compiler for the ppc64el architecture
gcc-powerpc64-linux-gnu - GNU C compiler for the ppc64 architecture
...
g++-powerpc64le-linux-gnu - GNU C++ compiler for the ppc64el architecture
g++-powerpc64-linux-gnu - GNU C++ compiler for the ppc64 architecture
...
```

Install:

```
$ sudo apt-get install gcc-powerpc64le-linux-gnu gcc-powerpc64-linux-gnu
```

Check:

```
$ which powerpc64{le,}-linux-gnu-{gcc,ld,objdump}
/usr/bin/powerpc64le-linux-gnu-gcc
/usr/bin/powerpc64le-linux-gnu-ld
/usr/bin/powerpc64le-linux-gnu-objdump
/usr/bin/powerpc64-linux-gnu-gcc
/usr/bin/powerpc64-linux-gnu-ld
/usr/bin/powerpc64-linux-gnu-objdump
```

### Packages for Fedora 27

Search:

```
$ sudo dnf search powerpc64
...
gcc-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
gcc-c++-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
gcc-powerpc64le-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64le-linux-gnu
binutils-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
gcc-c++-powerpc64le-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64le-linux-gnu
binutils-powerpc64le-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64le-linux-gnu
```

Install:

```
$ sudo dnf install gcc-powerpc64le-linux-gnu gcc-powerpc64-linux-gnu
```

Check:

```
$ which powerpc64{le,}-linux-gnu-{gcc,ld,objdump}
/usr/bin/powerpc64le-linux-gnu-gcc
/usr/bin/powerpc64le-linux-gnu-ld
/usr/bin/powerpc64le-linux-gnu-objdump
/usr/bin/powerpc64-linux-gnu-gcc
/usr/bin/powerpc64-linux-gnu-ld
/usr/bin/powerpc64-linux-gnu-objdump
```

### Packages for RHEL/CentOS 7

Search:

```
$ sudo yum search powerpc64
binutils-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
binutils-powerpc64le-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64le-linux-gnu
gcc-c++-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
gcc-powerpc64-linux-gnu.x86_64 : Cross-build binary utilities for powerpc64-linux-gnu
```

Note: those compilers are _bi-endian_ (i.e., compile _both_ little-endian and big-endian),
but the utilities (e.g., assembler, linker, object dump) are _endian-specific_.
So, it's required to install the _little-endian_ binutils too.

Install:

```
$ sudo yum install gcc-powerpc64-linux-gnu binutils-powerpc64le-linux-gnu.x86_64
```

Check:

```
$ which powerpc64-linux-gnu-gcc powerpc64{le,}-linux-gnu-{ld,objdump}
/usr/bin/powerpc64-linux-gnu-gcc
/usr/bin/powerpc64le-linux-gnu-ld
/usr/bin/powerpc64le-linux-gnu-objdump
/usr/bin/powerpc64-linux-gnu-ld
/usr/bin/powerpc64-linux-gnu-objdump
```



### Tarballs for Any Distro

You can use the cross compilers provided by [Bootlin](https://bootlin.com) (formerly Free Electrons)
at [toolchains.bootlin.com](https://toolchains.bootlin.com).

It's possible to select some parameters of the cross compiler to download (**awesome!**):
- Architecture (e.g., `powerpc64le-power8`, `powerpc64-power8`)
- C library (e.g.., `glibc`, `musl`)
- Version (e.g., `stable`, `bleeding-edge`)

Download from the generated link, and extract; for example:

```
$ cd ~/bin
$ wget https://toolchains.bootlin.com/downloads/releases/toolchains/powerpc64le-power8/tarballs/powerpc64le-power8--glibc--stable-2018.02-2.tar.bz2
$ tar xf powerpc64le-power8--glibc--stable-2018.02-2.tar.bz2

$ # optional, just for `which`
$ export PATH=$PATH:~/bin/powerpc64le-power8--glibc--stable-2018.02-2/bin/
```

Check:

Note: these cross compiler commands don't have `gnu` in the file name.

```
$ which powerpc64le-linux-{gcc,ld,objdump}
~/bin/powerpc64le-power8--glibc--stable-2018.02-2/bin/powerpc64le-linux-gcc
~/bin/powerpc64le-power8--glibc--stable-2018.02-2/bin/powerpc64le-linux-ld
~/bin/powerpc64le-power8--glibc--stable-2018.02-2/bin/powerpc64le-linux-objdump
```

## Example: Assembly Testcase and QEMU user-mode emulation

Consider this simple test program to just return `42`.  
It's written in assembly, so it doesn't need the C standard library (to avoid distro differences, for starters).

__Note__: This is _not yet_ an introduction to PowerPC assembly :) That will come later with more details.  

```
$ cat return_42.s

	.abiversion 2		# use ELFv2 ABI (for little-endian)
	.globl _start		# set global visibility for symbol '_start'

_start:				# implement symbol '_start' (program entry point)
	li	%r0, 1		# load register 0 (system call number); syscall 1 is 'exit'
	li	%r3, 42		# load register 3 (function parameter): return value
	sc			# exec system call instruction
```

Cross-compile it with the GNU assembler (`as`) and the GNU linker (`ld`):
```
$ powerpc64le-linux-gnu-as return_42.s -o return_42.o
$ powerpc64le-linux-gnu-ld return_42.o -o return_42
```

Check it:

```
$ file return_42
return_42: ELF 64-bit LSB executable, 64-bit PowerPC or cisco 7500, version 1 (SYSV), statically linked, not stripped
```

Run it with QEMU [user-mode emulation][qemu-user]:

```
$ qemu-ppc64le return_42

$ echo $?
42
```

Excellent!  
Now let's move to C.


## Example: C Testcase and QEMU user-mode emulation


Consider this simple test program that just returns `42`:  
```
$ echo 'int main() { return 42; }' > return_42.c
```

Compile (natively):
```
$ gcc -o return_42 return_42.c

$ file return_42
return_42: ELF 64-bit LSB executable, x86-64, [...]
```

Run it (natively):
```
$ ./return_42 

$ echo $?
42
```

Okay, it works.

Now, let's cross-compile and run it.  
This varies with the distro as each distro provide more or less content (e.g., library support) in its _cross_ packages.

### Cross-compile on Ubuntu 16.04 LTS

Let's start with Ubuntu 16.04 as it provides a lot of cool, cross stuff. :)

Cross-compile the program with GNU C Compiler (`gcc`):

```
$ powerpc64le-linux-gnu-gcc -o return_42 return_42.c
```

Check it's a `ppc64le` program:

```
$ file return_42
return_42: ELF 64-bit LSB executable, 64-bit PowerPC [...] 
```

Now, let's run it with QEMU user-mode emulation:

```
$ qemu-ppc64le return_42
/lib64/ld64.so.2: No such file or directory
```

Uh-oh. It's missing the dynamic loader, from the C library.  
There are 2 options: 1) built it _statically-linked_; 2) get the dynamic loader.  
Let's do both.

- 1) Build it statically-linked:

```
$ powerpc64le-linux-gnu-gcc -static -o return_42 return_42.c

$ file return_42
return_42: ELF 64-bit LSB executable, 64-bit PowerPC [...] statically linked [...]

$ qemu-ppc64le return_42

$ echo $?
42
```

- 2) Get the dynamic loader:

Download and extract the `libc6` package for the `ppc64el` architecture (_not_ a typo on Ubuntu/Debian!).  
Then fix its `/lib64/ld64.so.2` symlink to point to an existing file in the extracted contents.

```
$ wget http://ports.ubuntu.com/pool/main/g/glibc/libc6_2.23-0ubuntu10_ppc64el.deb

$ dpkg-deb -x libc6_*_ppc64el.deb ~/bin/cross-libc/

$ cd ~/bin/cross-libc/

$ # the symlink target does not exist (absolute addressing to /lib)
$ ls -l lib64/ld64.so.2 
[...] lib64/ld64.so.2 -> /lib/powerpc64le-linux-gnu/ld-2.23.so

$ # fix it (relative addressing to ../lib):
$ ln -sf ../lib/powerpc64le-linux-gnu/ld-2.23.so lib64/ld64.so.2

$ # okay now!
$ ls -l lib64/ld64.so.2
[...] lib64/ld64.so.2 -> ../lib/powerpc64le-linux-gnu/ld-2.23.so

$ cd -
```

Now, let's try again:

```
$ powerpc64le-linux-gnu-gcc -o return_42 return_42.c

$ file return_42
return_42: ELF 64-bit LSB executable, 64-bit PowerPC [...] dynamically linked [...]

$ export QEMU_LD_PREFIX=~/bin/cross-libc/

$ qemu-ppc64le return_42

$ echo $?
42
```

And it runs fine now! ;-)

### Cross-compile on Fedora 27 and RHEL/CentOS 7


Cross-compile the program with GNU C Compiler (`gcc`):

- Fedora 27:  
`$ powerpc64le-linux-gnu-gcc -o return_42 return_42.c`

- RHEL/CentOS 7 (_bi-endian_ cross-compiler; specify `-mlittle-endian`):  
`$ powerpc64-linux-gnu-gcc -mlittle-endian return_42.c -o return_42`

Both distros fail in the linker (`ld`) stage due to the lack of some C runtime (`crt`) pieces:
```
/usr/bin/powerpc64le-linux-gnu-ld: cannot find crt1.o: No such file or directory
/usr/bin/powerpc64le-linux-gnu-ld: cannot find crti.o: No such file or directory
/usr/bin/powerpc64le-linux-gnu-ld: cannot find -lc
/usr/bin/powerpc64le-linux-gnu-ld: cannot find crtn.o: No such file or directory
collect2: error: ld returned 1 exit status
```

Unfortunately Fedora 27 and RHEL/CentOS 7 don't seem to provide packages with a
cross-compiled C standard library -- just the cross-compilers. (or maybe I didn't search it correctly.)

Well, let's work around that to have some fun.

Build it without the C standard library (`-nostdlib`):

- Fedora 27:  
`$ powerpc64le-linux-gnu-gcc -nostdlib -o return_42 return_42.c`

- RHEL/CentOS 7 (_bi-endian_ cross-compiler; specify `-mlittle-endian`):  
`$ powerpc64-linux-gnu-gcc -mlittle-endian -nostdlib return_42.c -o return_42`

Now, both distros print this warning:  
`/usr/bin/powerpc64le-linux-gnu-ld: warning: cannot find entry symbol _start; defaulting to <address>`

That's because the `_start` symbol (the _real_ program entry point -- that in turn calls the `main()`
function in C) is part of the C standard library.

And apparently a program does not run well without it:
```
$ qemu-ppc64le return_42
qemu: uncaught target signal 11 (Segmentation fault) - core dumped
Segmentation fault (core dumped)
```

So, let's write a minimal `_start` routine in assembly to just call `main()` and the `exit`
system call.  
(very similar to/compare with the assembly testcase in the example above).

```
$ cat start.s

	.abiversion 2		# use ELFv2 ABI (for little-endian)
	.globl _start		# set global visibility for symbol '_start'

_start:				# implement symbol '_start' (program entry point)
	bl	main		# branch to the 'main' symbol (set link register for return)
	li	%r0, 1		# load register 0 (system call number); syscall 1 is 'exit'
				# keep register 3 (return value too) from 'main'
	sc			# exec system call instruction
```

And include it in the compilation:

- Fedora 27:  
`$ powerpc64le-linux-gnu-gcc -nostdlib start.s return_42.c -o return_42`

- RHEL/CentOS 7 (_bi-endian_ and _bi-ABI_ cross-compiler, also specify `-mabi=elfv2`):  
`$ powerpc64-linux-gnu-gcc -mlittle-endian -mabi=elfv2 -nostdlib start.s return_42.c -o return_42`

Great, no warning messages about `_start` this time!

```
$ file return_42
return_42: ELF 64-bit LSB executable, 64-bit PowerPC or cisco 7500, version 1 (SYSV), statically linked, BuildID[sha1]=87aa5afc0db809012621beb9d4587fe1b8fcaa59, not stripped
```

The `file` output looks good, so let's run it:


```
$ qemu-ppc64le return_42

$ echo $?
42
```

Success!


## Considerations

Most cross compilers (particularly the ones provided by Linux distros) usually lack the C standard
library and other libraries.  This limits what can be compiled -- for example, what about `printf()` for debugging?

However, that is an expected restriction. The Linux distros usually provide only enough cross-compiler
support in order to build specific stuff that doesn't need external libraries (for example, Linux kernel,
GRUB2 bootloader), not really to run them on emulators on other architectures (our _great_ use case).

Fortunately, that type of software (without libraries; together with simple programs for learning purposes)
is exactly the type of software we should be looking at here.

For software that needs libraries, you can still run a _native_ `ppc64le` compiler in a virtual machine
on your `x86_64` computer with [QEMU full-system emulation][qemu-full].

In the next posts we'll cross compile and run the Linux kernel and the OPAL firmware on QEMU. Yay!

[wikipedia-cross-compiler]: https://en.wikipedia.org/wiki/Cross_compiler
[qemu-full]: {{ site.baseurl }}{% post_url 2018-01-31-ppc64le-on-x86_64-qemu-full-system-emulation %}
[qemu-user]: {{ site.baseurl }}{% post_url 2018-02-03-ppc64le-on-x86_64-qemu-user-mode-emulation %}


