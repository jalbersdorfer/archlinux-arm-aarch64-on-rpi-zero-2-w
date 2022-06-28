# archlinux-arm-aarch64-on-rpi-zero-2-w
Arch Linux ARM AArch64 Installation on Raspberry Pi Zero 2 W 

I took most of the parts from https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-zero-2 - but it is wrong/incomplete in some parts.

## Installation

Replace sdX in the following instructions with the device name for the SD card as it appears on your computer.

1. Start fdisk to partition the SD card:

```
fdisk /dev/sdX
```

2. At the fdisk prompt, delete old partitions and create a new one:  
    - Type **o**. This will clear out any partitions on the drive.  
    - Type **p** to list partitions. There should be no partitions left.  
    - Type **n**, then **p** for primary, **1** for the first partition on the drive,  
      press **ENTER** to accept the default first sector,  
      then type **+200M** for the last sector.  
    - Type **t**, then **c** to set the first partition to type W95 FAT32 (LBA).  
    - Type **n**, then **p** for primary, **2** for the second partition on the drive,   
      and then press **ENTER twice** to accept the default first and last sector.  
    - Write the partition table and exit by typing **w**.  

3. Create and mount the FAT filesystem:

```
mkfs.vfat /dev/sdX1
mkdir boot
mount /dev/sdX1 boot
```

4. Create and mount the ext4 filesystem:

```
mkfs.ext4 /dev/sdX2
mkdir root
mount /dev/sdX2 root
```

5. Download and extract the root filesystem (as root, not via sudo):

```
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-armv7-latest.tar.gz
bsdtar -xpf ArchLinuxARM-rpi-armv7-latest.tar.gz -C root
sync
```

6. Move boot files to the first partition:

```
mv root/boot/* boot
```

6.1. Patch the boot Partition - *This is the Part missing in the official Installation Guide*

```
cd boot
wget https://github.com/raspberrypi/firmware/raw/master/boot/bcm2710-rpi-zero-2-w.dtb
cp bcm2710-rpi-zero-2-w.dtb dtbs/broadcom/bcm283x-rpi-other.dtb
```

7. Unmount the two partitions - *And DO NOT CHANGE the etc/fstab before as mentioned in the official Guide*

```
umount boot root
```

8. Insert the SD card into the Raspberry Pi and apply 5V power.

9. Use the serial console or attach HDMI and a keyboard.
    - Login as the default user alarm with the password alarm.
    - The default root password is root.

10. Initialize the pacman keyring and populate the Arch Linux ARM package signing keys:

```
pacman-key --init
pacman-key --populate archlinuxarm
```

## Further Reading

Where I stole the neccessary hints from:  
https://archlinuxarm.org/forum/viewtopic.php?f=67&t=15695

There was an effort to upgrade the system inside a `chroot` environment before the first boot.  
Maybe this following Guide can help to `chroot` into the SD-Card, because the mentioned   
`arch-chroot` command didn't work for me. It crashed with either with
```
chroot: failed to run command '/bin/bash': Exec format error
```
which can be resolved by installing the following `qemu` packages:
```
pacman -S qemu-user qemu-system-aarch64
```
Then it crashes with:
```
chroot: failed to run command ‘/bin/bash’: No such file or directory
```

which might be resolveable by copying `/usr/bin/qemu-aarch64` or maybe `/usr/bin/qemu-aarch64-static` into the chroot target  

and running `proot` like  

```
proot -R . -q qemu-arm-static
```

as described here:

https://archlinuxarm.org/forum/viewtopic.php?f=30&t=9294
