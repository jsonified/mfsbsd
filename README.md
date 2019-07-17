# mfsBSD

## upstream

- Copyright (c) 2018 Martin Matuska <mm at FreeBSD.org>
- Version 2.3
- https://github.com/mmatuska/mfsbsd | http://mfsbsd.vx.sk

# groupon

To prepare your image:

- clone this repo on an appropriate FreeBSD system
- amend files in `./conf/*` as required
- build the image, all listed parameters are optional:

        # make \
            DISTREL=12.0-RELEASE \
            DISTURL=https://ftp.freebsd.org/pub/FreeBSD/releases/amd64/amd64 \
            fetch pkg
        # make \
            DISTREL=12.0-RELEASE \
            DISTURL=https://ftp.freebsd.org/pub/FreeBSD/releases/amd64/amd64 \
            gzimage
        # ls mfs*

> `DISTURL` can equally be a file:////url

## deployment

To deploy your new image, netboot an existing server using iPXE, or
simply unzip and dd this image in over the top. It should be possible
to jump directly to the installer from iPXE, so long as you are using a
BIOS-based boot on your server, and iPXE with HTTP and BZIMAGE support.

### sample iPXE script

```
#!ipxe
echo _____________ iPXE __________________
echo mac______________${mac}
echo uuid_____________${uuid}
echo serial___________${serial}
echo ip_______________${ip}
echo manufacturer_____${manufacturer}
echo product__________${product}
echo next-server______${netX/next-server}
echo filename_________${netX/filename}
echo config-server____${netX/default-url}

prompt --key 0x1b --timeout 5000 Press ESC for iPXE shell... && shell ||
# log info via GET and ignore any HTTP response
imgfetch ${default-url}/log?uuid=${uuid}&mac=${mac:hexhyp}&domain=${domain}&hostname=${hostname}&serial=${serial} ||
imgfree
imgfetch ${default-url}/mfsbsd.img.gz
imgfetch ${default-url}/memdisk
# ready to boot
imgstat
prompt --key 0x1b --timeout 5000 Press ESC for iPXE shell... && shell ||
boot memdisk raw keeppxe ||
prompt --key 0x1b --timeout 9999 Press ESC for iPXE shell... && shell ||
reboot
```

## deploying on top of an existing system

If you already have shell access, then it's possible to slip mfsBSD on
there while zfs isn't looking - ada1 seems to be the drive they actually
boot from at BIOS level, so split it off from zfs and then dd over
active ada0 before zfs notices!

```
# mount -t tmpfs tmpfs /mnt
# cd /mnt
# curl -#LO http://your/mfsbsd.img
# sha256 mfsbsd.img
SHA256 (mfsbsd.img) = e87f9bfabfc1cc9bc7e42c6c38fa5c0c17680b508c641fc0db2153179e8389f5
# zpool offline zroot ada1
# dd if=mfsbsd.img of=/dev/ada1 conv=sync bs=1m
# sysctl kern.geom.debugflags=16
# dd if=mfsbsd.img of=/dev/ada0 conv=sync bs=1m
# reboot -n -q -l && /bin/pray
```
