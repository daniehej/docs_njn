# Guide to Flashing QSPI-NOR on the Nvidia Jetson Nano Developer Kit

Since Jetpack 4.5, the boot firmware is moved from the SD card to the internal QSPI-NOR flash memory. Among other things, this enables the Jetson Nano to boot from USB. However, this means that the QSPI-NOR must be flashed in order to boot the new SD card format.

The QSPI bootloader needs to be updated for Jetpack > 4.4 to boot. When using an original image, this will automatically be done upon first boot, after which the files for teh update are deleted. In order to distribute our own image, we therefore have to flash the QSPI-NOR.

After flashing QSPI-NOR, the Jetson Nano will no longer be able to boot Jetpack versions < 4.5.

In order to flash a Jetson, first put it into recovery mode by shorting the recovery pin on the board to ground while plugging in the power. Then let go of the recovery pin and the Jetson Nano will boot into recovery mode. 

The recovery pin is in the 8-pin pin header J40 in the corner of the board. If the Micro SD side of the Jetson Nano is facing towards you, the pins you need to bridge are ⣿ -> ⣭. The recovery pin is also marked on the bottom of the board as FR.

Connect with Micro-USB (can be done at any time)

If the Jetson Nano is booted into recovery mode, the command lsusb will return

NVIDIA Corp. APX

as a connected device. If it is not in recovery, you will se a different Nvidia device.

On a linux machine, navigate to Linux_For_Tegra

sudo ./flash.sh jetson-nano-qspi mmcblk0p1

If the device is not in recovery mode, the command will fail, therefore when updating many devices, it is not necessary to use lsusb every time.

After the QSPI-NOR is flashed, you can insert and boot Jetpack > 4.5 SD cards.