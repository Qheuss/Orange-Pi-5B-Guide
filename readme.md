# Orange Pi 5B eMMC Installation Guide

This guide documents the process of installing Armbian on an Orange Pi 5B's eMMC storage.

## Prerequisites

- Orange Pi 5B with 32GB/64GB/128GB/256GB eMMC
- 8GB or larger SD card (use a class 10 or above)
- Armbian image for Orange Pi 5 (https://www.armbian.com/orangepi-5/)

## Step 1: Prepare the SD Card

1. Download Armbian image for Orange Pi 5 (not 5B)
2. Flash the image to your SD card using Balena Etcher or Rufus
3. Insert the SD card into your Orange Pi 5B and boot

## Step 2: Modify Device Tree File

After booting from the SD card, modify the device tree file to properly support Orange Pi 5B:

```bash
sudo nano /boot/armbianEnv.txt
```

Change the `fdtfile` line to:

```
fdtfile=rockchip/rk3588s-orangepi-5b.dtb
```

Save the file and reboot.

## Step 3: Initialize eMMC

If the eMMC doesn't appear after reboot, use RKDevTool to initialize it:

1. Connect your Orange Pi 5B to a Windows computer
2. Use RKDevTool to flash a compatible image to the eMMC (https://www.youtube.com/watch?v=5q_tytwmseg)
3. Boot again from the SD card

## Step 4: Prepare eMMC for Installation

After booting from SD card, verify that eMMC is visible:

```bash
lsblk
```

The eMMC should appear as mmcblk0 with a capacity of around 233GB (if you have the 256GB version).

Create partitions on eMMC:

```bash
sudo parted /dev/mmcblk0 mklabel gpt
sudo parted /dev/mmcblk0 mkpart primary fat32 32MiB 512MiB
sudo parted /dev/mmcblk0 mkpart primary ext4 512MiB 100%
sudo mkfs.vfat -n BOOT /dev/mmcblk0p1
sudo mkfs.ext4 -L rootfs /dev/mmcblk0p2
```

## Step 5: Copy System to eMMC

Mount the partitions:

```bash
sudo mkdir -p /mnt/emmc-boot /mnt/emmc-root
sudo mount /dev/mmcblk0p1 /mnt/emmc-boot
sudo mount /dev/mmcblk0p2 /mnt/emmc-root
```

Copy the boot files, following symbolic links:

```bash
sudo cp -rL /boot/* /mnt/emmc-boot/
```

Copy the root filesystem:

```bash
sudo rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/var/log.hdd/*","/boot/*"} / /mnt/emmc-root/
```

Create necessary directories in the new root:

```bash
sudo mkdir -p /mnt/emmc-root/{dev,proc,sys,tmp,run,mnt,media,boot}
sudo chmod 1777 /mnt/emmc-root/tmp
```

## Step 6: Configure Boot Settings

Edit the boot configuration on eMMC:

```bash
sudo nano /mnt/emmc-boot/armbianEnv.txt
```

Make sure it contains:

```
fdtfile=rockchip/rk3588s-orangepi-5b.dtb
rootdev=/dev/mmcblk0p2
rootfstype=ext4
```

## Step 7: Set Up File System Table

Configure proper mounting in the new system:

```bash
sudo nano /mnt/emmc-root/etc/fstab
```

Replace the contents with:

```
/dev/mmcblk0p2  /           ext4    defaults,noatime,commit=600,errors=remount-ro    0 1
/dev/mmcblk0p1  /boot       vfat    defaults                                         0 2
tmpfs           /tmp        tmpfs   defaults,nosuid                                  0 0
```

## Step 8: Finalize Installation

Unmount the partitions and shut down:

```bash
sudo umount /mnt/emmc-boot
sudo umount /mnt/emmc-root
sudo sync
sudo shutdown -h now
```

Remove the SD card and power on the Orange Pi 5B. It should now boot from the eMMC.

## Troubleshooting

If the system doesn't boot from eMMC:

1. Reinsert the SD card and boot from it
2. Verify that all steps were completed correctly
3. Check the boot configuration and fstab on the eMMC
4. Ensure the device tree file is correct for Orange Pi 5B

## What's Next?

With Armbian now running from eMMC, you can set up:

- Samba/NFS for file sharing
- Docker for containerized applications
- Minecraft server
- Or anything you want :)
