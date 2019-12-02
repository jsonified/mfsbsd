# mfsBSD

Copyright (c) 2018 Martin Matuska <mm at FreeBSD.org>

Version 2.3

## Description

This is a set of scripts that generates a bootable image, ISO file or boot
files only, that create a working minimal installation of FreeBSD. This
minimal installation gets completely loaded into memory.

The image may be written directly using dd(1) onto any bootable block device,
e.g. a hard disk or a USB stick e.g. /dev/da0, or a bootable slice only,
e.g. /dev/ada0s1

## Build-time requirements
 - FreeBSD 10 or higher installed, tested on i386 or amd64
 - Base and kernel from a FreeBSD 10 or higher distribution
   (release or snapshots, e.g mounted CDROM disc1 or ISO file)

## Runtime requirements
 - a minimum of 512MB system memory

## Other information

See [BUILD](./BUILD.md) and [INSTALL](./INSTALL.md) for building and installation instructions.

Project homepage: http://mfsbsd.vx.sk

This project is based on the ideas of the depenguinator project:
http://www.daemonology.net/depenguinator/

## [packet.net] changes

This branch contains a number of tweaks to make working with [packet.net]
servers feasible. See the branch history for details.

To prepare your image:

- clone this repo on an appropriate FreeBSD system
- check out this branch
- replace `./conf/authorized_keys` with a more appropriate set
- fetch packages and dependencies and then rename the directory, these
    will be embedded in the ramdisk image

        pkg fetch -r FreeBSD -o . -Uyd tmux rsync
        mv All packages

- include the MANIFEST, base.txz, kernel.txz and other files as required
    in `customfiles/usr/freebsd-dist/` for the installer to find them
- alternatively, delete the MANIFEST, and host all of these files on
    a webserver, and during the install, use a custom mirror
- build the image via something like this:

        sudo make WRKDIR=(mktemp -d -t mfsbsd) BASE=/downloads/BSD/current/latest/ftp
        ls mfs*.img


## deployment

To deploy your new image, netboot an existing server using iPXE and
dd your new image in over the top. It should be possible to jump
directly to the installer from packet.net's systems like so, with a
FreeBSD 11.1 OS and this custom iPXE script:

```
#!/bin/sh
sysctl kern.geom.debugflags=16
mount -t tmpfs tmpfs /tmp
cd /tmp
fetch --no-verify-peer https://example.org/ipxe/r339417/mfsbsd-12.1-RELEASE-amd64.img
dd if=mfsbsd-12.1-RELEASE-amd64.img of=/dev/ada0 conv=sync bs=1m
reboot -n -q -l
```

[packet.net]: https://packet.net/

[koans]

```
sudo make WRKDIR=(mktemp -d -t mfsbsd) BASE=/downloads/BSD/12.1-RELEASE koans
make dist
```

# overwrite the existing partition from a ramdisk


```
mount -t tmpfs tmpfs /tmp
sysctl kern.geom.debugflags=0x10
rsync -Phrivazcl --inplace mfsbsd-*.img root@...:/tmp/
```

[koans]: https://koan-ci.com/
