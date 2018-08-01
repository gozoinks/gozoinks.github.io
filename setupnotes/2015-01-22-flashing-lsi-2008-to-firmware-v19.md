---
title: Flashing LSI 2004 to firmware v19
slug: flashing-lsi-2008-to-firmware-v19
date_published: 2015-01-22T06:12:57.989Z
date_updated:   2015-01-24T19:50:39.575Z
---

I came at this from several directions, trying to avoid the need to lay hands on the machine. But if you want to get on with it already, just prepare a FreeDOS USB flash boot disk, copy the firmware, ROM, and flash utility to it, and flash the firmware with that. 

This procedure generally applies to all LSI 2000-series controllers. I have a four-lane controller, the 9211-4i, which uses the "LSISAS2004" chip. It's effectively the same for, say, an 9211-8i, which would use the LSISAS2008" chip.

#Prepare the boot disk
1. Download a pre-built FreeDOS boot image. For my 16GB flash disk, I chose the one for a drive larger than 2GB:
  - http://chtaube.eu/computers/freedos/bootable-usb/
2. `bunzip` your downloaded image file.
3. Write it to your drive. On Mac OS X systems:
    1. `diskutil list` to get the /dev/disk*n* path to the drive
    2. `diskutil unmountDisk /dev/disk*n*` to unmount it
    3. `dd if=path/to/Freedos.img of=/dev/rdisk*n* bs=512` to write it to the disk
    4. `fdisk -e /dev/disk*n*` to fix the boot partition
    5. `f 1` to flag the first partition "bootable"
    6. `w` to write the partition table
    7. `q` to quit fdisk

I believe at this point the disk re-mounts on a Mac automatically.

Now you'll want to copy the .bin, .rom, and sas2flsh.exe files and executable from LSI to your USB drive:

1. Download the firmware from LSI. [TIL](/lsi-firmware-v20-unsupported/) v19 is the one you want to use. Here it is for the 9211-4i card:
  - Downloads page: http://www.lsi.com/products/host-bus-adapters/pages/lsi-sas-9211-4i.aspx#tab/tab4
  - Firmware: `9211-4i_Package_P19_IR_IT_Firmware_BIOS_for_MSDOS_Windows`
2. Unzip the archive, copy the key files to a new folder on your USB drive, and eject it:
```cd ~/Downloads
unzip 9211-4i_Package_P19_IR_IT_Firmware_BIOS_for_MSDOS_Windows.zip
cd 9211-4i_Package_P19_IR_IT_Firmware_BIOS_for_MSDOS_Windows
mkdir -p /Volumes/FreeDOS/lsi/19
cp sas2flash_dos_rel/sas2flsh.exe /Volumes/FreeDOS/lsi/19/
cp Firmware/HBA_9211_4i_IT/2114it.bin /Volumes/FreeDOS/lsi/19/
cp sasbios_rel/mptsas2.rom /Volumes/FreeDOS/lsi/19/
diskutil eject /Volumes/FreeDOS
```

Now you're ready to boot up the machine from the USB drive.

#Flash the card
For the actual flashing, these instructions proved helpful and concise:
http://lime-technology.com/forum/index.php?topic=34081.5;wap2

Everything you'll read on this should warn you not to reboot the machine between erasing the old flash and flashing the new flash. To do so will apparently brick your card.

1. `sas2flsh -list` to identify the card you want to flash. If you have only one controller installed, it will be "controller 0". Otherwise, it'll be "controller *something else*".
2. `sas2flsh -listall` to verify your choice.
3. `sas2flsh -o -e 6 -c 0` to erase the flash area on controller 0.
4. `sas2flsh -o -f 2114it.bin -c 0` to write the IT flash image to controller 0.
5. `sas2flsh -o -b mptsas2.rom -c 0` to write the BIOS ROM to controller 0.
6. `sas2flsh -list` to verify the new version.
7. `sas2flsh -listall` to verify again.

Now you can reboot. When the machine comes back up, the card will be factory-fresh with version 19 of the initiator-target flavored firmware. 

On my SuperMicro board, this meant I needed to enter the LSI configuration BIOS tool (ctrl-C when prompted) to disable boot support. LSI boot support blocks booting from a USB flash drive.

Once that was done, I was booting back up into SmartOS from my normal USB drive. 

