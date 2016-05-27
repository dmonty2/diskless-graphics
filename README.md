# diskless-graphics
Allow diskless stations to support open &amp; closed source graphics drivers with multiple versions nvidia/ati/intel/etc

Assuming you already have a *buntu 16.04 32bit and 64bit diskless with free graphics drivers already up and running.   Our diskless roots are installed as lxc containers so we can manage software from the server.

## remake_squashfs
remake_squashfs will compress usr and opt partitions which greatly increases the speed of diskless station.  This has to be run after installing/upgrading packages.  Each nbd share should have a unique name that is referenced in the pxe menu below. e.g.

```
[xenial64usr]
        exportname = /var/lib/lxc/xenial64/rootfs/squashed/usr.sfs
        readonly = true
[xenial64opt]
        exportname = /var/lib/lxc/xenial64/rootfs/squashed/opt.sfs
        readonly = true

```

## enable_squashfs
Once the opt_new.sfs and usr_new.sfs are finished compressing.  I usually do:
```
echo enable_squashfs | at 10PM
```
This will:
1. Shutdown all diskless stations - they will crash if you reload nbd-server with an updated squashfs.
2. cd /squashed/ mv opt_new.sfs opt.sfs; mv usr_new.sfs usr.sfs
3. reload nbd-server

## aufs_drivers
This creates separate aufs overlay folders that only contain the nvidia-304 nvidia-340 and nvidia-361 drivers while leaving the base diskless root diskless folder as-is with open source drivers.

## initramfs-tools
/etc/initramfs-tools files are to configure the diskless clients to
* use nbd-squashfs.
* auto-detect nvidia driver version to aufs-mount on-top of the file-system.

These are enabled by passing the following arguments to your pxeboot menu:
```
LABEL linux
  MENU LABEL Please wait...
  KERNEL xenial32/vmlinuz-4.4.0-22-generic
  APPEND initrd=xenial32/initrd.img-4.4.0-22-generic rw nfsroot=10.10.0.1:/var/lib/lxc/xenial32/rootfs,rsize=65536,wsize=65536,timeo=600,retrans=5 root=/dev/nfs aufs=tmpfs nbdsquashserver=10.10.0.1 nbdsquashusr=xenial32usr nbdsquashopt=xenial32opt nosplash
```
* aufs=tmpfs -  overlays a tmpfs on the read-only nfs or nbd-squashfs folders.  This is required for all other options.
* nbdsquashserver=10.10.0.1 - IP of nbd-server - nbd + squashfs is a big speed boost for diskless clients.
* nbdsquashusr=xenial32usr - name of the nbd-server's usr share (noted above)
* nbdsquashopt=xenial32opt - name of the nbd-server's opt share (noted above)
