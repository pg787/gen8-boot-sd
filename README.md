# HP Microserver Gen8 Universal Boot SD

This project provides a universal GRUB bootloader for HP Microserver Gen8 to enable booting from the otherwise non-bootable internal SSD (ODD SATA port) or any of the front drive bays.  

It includes:  
- A GRUB configuration (`grub.cfg`) that automatically cycles through available drives starting from the internal SSD (`hd5`) down to the front bays (`hd4` → `hd1`) and finally the SD card (`hd0`).  
- Optional manual entries for each drive for direct selection.  
- A small bootable SD card image (~30MB) for chainloading the OS from your SSD or other drives.  
- Tested with TrueNAS, but should work for other OSes installed on Gen8 drives.  

## Features
- Auto-cycle boot from hd5 → hd4 → hd3 → hd2 → hd1 → hd0 until a bootable drive is found.  
- Manual menu entries for every disk (`hd0`–`hd5`).  
- Fully free and open-source under GPLv3.  
- Compatible with AHCI mode, no RAID required.  
- Minimal SD card footprint (128 MiB partition).  

## Usage
HP Gen8 GRUB Boot SD Card – Manual Install & Imaging Guide

This guide shows how to create a bootable GRUB SD/USB for HP Microserver Gen8, 
and how to make a compact image of it for reuse.

------------------------------------------------------------

BEFORE YOU START

- All data on the SD/USB will be erased.
- Identify your SD card or USB stick carefully, e.g., /dev/sdX.
- Use one of these commands to confirm:

  sudo fdisk -l
  lsblk

- Make sure you are using the correct device, or you risk losing data.

------------------------------------------------------------
MANUAL INSTALL
------------------------------------------------------------

You need to perform the following steps:

1. Format your SD card / USB stick with a new MBR.
2. Create a FAT32 W95 (LBA) partition (128 MiB recommended, smaller possible, e.g., 32 MiB).
3. Mark the partition as bootable.
4. Format the partition.
5. Install GRUB.
6. Copy the grub.cfg file.
7. Sync and unmount.

You can do this any way you like. Linux Mint users can use GNOME Disks, or you can use parted and fdisk.

------------------------------------------------------------
Step 1: Wipe the disk and create a fresh partition table

sudo parted /dev/sdX --script mklabel msdos

------------------------------------------------------------
Step 2: Create a primary partition (128 MiB)

sudo parted /dev/sdX --script mkpart primary 1MiB 128MiB

- Adjust 128MiB if you want a smaller partition (e.g., 32MiB).

------------------------------------------------------------
Step 3: Set the boot flag

sudo parted /dev/sdX --script set 1 boot on

------------------------------------------------------------
Step 4: Set the partition type to FAT32 (W95 LBA)

sudo fdisk /dev/sdX
# inside fdisk:
t   # change partition type
c   # code for W95 FAT32 (LBA)
w   # write changes and exit

- t → change partition type
- c → FAT32 (LBA)
- w → write table and exit

------------------------------------------------------------
Step 5: Format the partition as FAT32

sudo mkfs.vfat -F32 -n GRUB /dev/sdX1

- /dev/sdX1 is the first partition, not the entire disk.

------------------------------------------------------------
Step 6: Mount the SD card

sudo mkdir -p /mnt/sd
sudo mount /dev/sdX1 /mnt/sd

------------------------------------------------------------
Step 7: Install GRUB

sudo grub-install --target=i386-pc --boot-directory=/mnt/sd/boot/ --force /dev/sdX

- --force may be needed if GRUB warns about BIOS vs UEFI detection.

------------------------------------------------------------
Step 8: Copy the grub.cfg file

sudo cp ./grub.cfg /mnt/sd/boot/grub/grub.cfg

- Make sure your grub.cfg is prepared in your current directory.

------------------------------------------------------------
Step 9: Sync and unmount

sync
sudo umount /mnt/sd

------------------------------------------------------------
MAKE AN IMAGE OF THE SD CARD
------------------------------------------------------------

This creates a compact image of MBR + first partition without copying the whole SD card.

Step 1: Identify the SD card

sudo fdisk -l
lsblk

- Example for first partition:

Device     Boot Start    End Sectors  Size Id Type
/dev/sdX1  *     2048 264191  262144  128M  c W95 FAT32 (LBA)

- Start = 2048
- End = 264191 → add 1 to include the last sector → 264192 total sectors

------------------------------------------------------------
Step 2: Create the image

sudo dd if=/dev/sdX of=./gen8-grub-sd.img bs=512 count=264192 status=progress conv=sync

- bs=512 → matches sector size
- count=264192 → MBR + first partition
- conv=sync → ensures all blocks are written

This produces a ~128 MiB image ready to restore.

------------------------------------------------------------
Step 2: Verify image size

ls -lh ./gen8-grub-sd.img

- Should be approximately 128 MiB.

------------------------------------------------------------
Step 3: Restore the image

sudo dd if=./gen8-grub-sd.img of=/dev/sdX bs=512 status=progress conv=sync

- /dev/sdX → target SD/USB card
- Delete all existing partitions on the target before restoring.

------------------------------------------------------------
Optional Verification

- Check MBR:

sudo hexdump -C -n 512 /dev/sdX

- Check GRUB files:

ls /mnt/sd/boot/grub

------------------------------------------------------------
Notes

- Partition size can be smaller than 128 MiB; 32 MiB is usually sufficient for GRUB.
- Always double-check the device path (/dev/sdX) to prevent data loss.
- The image includes MBR + first partition only, which is portable and compact.

## License
This project is licensed under GPLv3.  
It includes the unmodified GRUB bootloader (GPLv3). All scripts, configurations, and SD images are also under GPLv3.  
