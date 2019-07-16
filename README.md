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
            fetch
        # make \
            DISTREL=12.0-RELEASE \
            DISTURL=https://ftp.freebsd.org/pub/FreeBSD/releases/amd64/amd64 \
            image
        # ls mfs*.img

> `DISTURL` can equally be a file:////url

## deployment

To deploy your new image, netboot an existing server using iPXE or
`dd(1)` your new image in over the top. It should be possible to jump
directly to the installer from iPXE, so long as you are using a recent
iPXE and BIOS boot on the server.

### sample iPXE script

```
#!ipxe
echo ========= iPXE ==========
echo mac          ${mac}
echo uuid         ${uuid}
echo serial       ${serial}
echo ip           ${ip}
echo manufacturer ${manufacturer}
echo product      ${product}
echo next-server  ${netX/next-server}
echo filename     ${netX/filename}
echo root-path    ${netX/root-path}
echo default-url  ${default-url}

prompt --key 0x1b --timeout 5000 Press ESC for iPXE shell... && shell ||
imgfree
imgfetch ${default-url}/mfsbsd.img
imgfetch ${default-url}/memdisk
# ready to boot
imgstat
prompt --key 0x1b --timeout 5000 Press ESC for iPXE shell... && shell ||
boot memdisk raw
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
