SMILE Plug initramfs firmware updater
===================

Description
------
* **mkuinit**: bash script to create an LZMA-compressed initramfs image from these sources
* **emmc/init**: ash script that gets run as init
* **emmc/bin/busybox**: a statically compiled (armv7-a vfpv3-d16 hard) busybox (version 1.22.1)
* **emmc/bin/mkfs.ext4**: a statically compiled (armv7-a vfpv3-d16 hard) mkfs.ext4 utility

U-Boot Environment
------
Set these values for bootcmd and bootargs:
```
setenv bootcmd 'mmcinfo; usb start; fatload mmc 0:1 0x6400000 uImage; if fatload usb 0 0x6000000 uinit-emmc; then fatload usb 0 0x6400000 uImage-emmc; bootm 0x6400000 0x6000000; else bootm 0x6400000; fi'
setenv bootargs 'console=ttyS0,115200 root=/dev/mmcblk0p2 rw rootwait'
saveenv
```

Files on Micro SD Card (eMMC model)
------
* Root filesystem: rootfs-emmc.tgz
* Initramfs image: uinit-emmc
* (optional) Kernel image: uImage-emmc

Updater Operation
------
* All updater files above should be placed in the root directory on the first partition of a FAT formatted micro SD card.
* The system image updater is contained within the initramfs image.  U-Boot will check for *uinit-emmc* on the micro SD card, and if present boot the kernel with it.
* By default, U-Boot will load the uImage currently on the eMMC on the first partition.  If this gets corrupted or you wish to boot using a different uImage, one can be supplied on the SD card with the name *uImage-emmc*.  If this is present on the card, it will get loaded in place of the uImage from eMMC.
* When the updater runs, it will first pause for 5 seconds to allow USB devices to register with the kernel.  Since the SD card is on the USB bus, this is mandatory.
* The LEDs will change from the U-Boot default of green eyes and red frown to blinking green eyes and smile, signifying you are in the updater script.
* It will check for *rootfs-emmc.tgz* on the SD card, and if present it will format both partitions of the eMMC, extract the tarball, and move the kernel files to the first partition.
* Upon success, *uinit-emmc* on the SD card will be renamed *uinit-emmc.pass*, and upon fail will be renamed *uinit-emmc.fail*.  This renaming will prevent U-Boot from reloading the initramfs image and starting the updater again.
* When finished, the updater will reboot the device, and it will boot normally from eMMC.
