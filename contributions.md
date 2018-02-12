---
title: open source contributions
---

# Open Source Contributions

## Upstream

 - [linux](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/log/?qt=author&q=mauricfo%40linux.vnet.ibm.com)
 - [multipath-tools](https://git.opensvc.com/gitweb.cgi?p=multipath-tools%2F.git&a=search&h=HEAD&st=author&s=mauricfo%40linux.vnet.ibm.com)
 - [klibc](https://git.kernel.org/pub/scm/libs/klibc/klibc.git/commit/?id=6e2976b9d103fb37a6a06e5bccb323fa5ab1c53b)
 - libaio:
   [1](https://pagure.io/libaio/c/451979cbf799bc321dc5c2040ba4142b67998d7d),
   [2](https://pagure.io/libaio/c/98127d2b408879ae71349dbb1e0a59cb7c2cce06),
   [3](https://pagure.io/libaio/blob/272ea61121fab2255f4a67770bf2575c2c4723f0/f/harness/cases/20.t#_4)
 - ppc64-diag:
   [1](https://sourceforge.net/p/linux-diag/ppc64-diag/ci/efb9a4df3f8860093eba61e823430f30fe5168e7/),
   [2](https://sourceforge.net/p/linux-diag/ppc64-diag/ci/cf718b32bcbd11098020d7d676392c78bb456a84/)
 - systemd:
   [1](https://github.com/systemd/systemd/commit/c51d5a48a1dbdb51e7009a3baf42987c2fd2cdef),
   [2](https://github.com/systemd/systemd/pull/2731)
   (not upstream per [policy](https://github.com/systemd/systemd/pull/2731#issuecomment-190349441),
   but accepted [downstream](https://git.centos.org/commitdiff/rpms!systemd/0e8dd16e3f8047929c504ef5ae650c42e003a3ec#n82e8f7883a492be8f2b12b23784dc44b3821bed9))

## Downstream -- Debian

 - The bootstrap and early development of the Debian `ppc64el` 
   [port](https://www.debian.org/ports/ppc64el/)  
   together with a few more [colleagues](https://wiki.debian.org/ppc64el#Development_Team);
   later with significant help and  
   take-over by the Debian community and Canonical for Ubuntu.

 - `linux` Source Package  

   - [Patch Submission](https://lists.debian.org/debian-kernel/2014/06/msg00060.html) 
   (`[PATCH V2 0/9] Add ppc64el support to src:linux`)
   - [GIT commits](https://anonscm.debian.org/cgit/kernel/linux.git/log/?qt=grep&q=mauricfo%40linux.vnet.ibm.com)

 - [Bugs](https://bugs.debian.org/cgi-bin/pkgreport.cgi?archive=both;include=tags%3Apatch;submitter=mauricfo%40linux.vnet.ibm.com) submitted with patches

## Downstream -- Ubuntu

 - Contributions more focused on storage, such as support for  
   Multipath I/O, Fibre Channel, NVMe, Software RAID, and the  
   respective installer, partitioner, and boot loader support.

 - Source Packages  
   (search for `Mauricio Faria de Oliveira`)
   - `multipath-tools` [changelog](https://launchpad.net/ubuntu/+source/multipath-tools/+changelog?batch=300)
   - `grub-installer` [changelog](https://launchpad.net/ubuntu/+source/grub-installer/+changelog?batch=300)
   - `partman-prep` [changelog](https://launchpad.net/ubuntu/+source/partman-prep/+changelog?batch=300)

 - [Bugs](https://bugs.launchpad.net/bugs/+bugs?&field.status=FIXRELEASED&field.bug_commenter=mauricfo&field.tag=patch) commented with patches

