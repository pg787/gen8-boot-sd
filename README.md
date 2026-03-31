# HP Microserver Gen8 Universal Boot SD

This project provides a **universal GRUB bootloader for HP Microserver Gen8** to enable booting from the otherwise non-bootable **internal SSD (ODD SATA port)** or any of the front drive bays.  

It includes:  
- A **GRUB configuration (`grub.cfg`)** that automatically cycles through available drives starting from the internal SSD (`hd5`) down to the front bays (`hd4` → `hd1`) and finally the SD card (`hd0`).  
- Optional **manual entries** for each drive for direct selection.  
- A **small bootable SD card image (~30MB)** for chainloading the OS from your SSD or other drives.  
- Tested with **TrueNAS**, but should work for other OSes installed on Gen8 drives.  

## Features
- Auto-cycle boot from **hd5 → hd4 → hd3 → hd2 → hd1 → hd0** until a bootable drive is found.  
- Manual menu entries for every disk (`hd0`–`hd5`).  
- Fully free and open-source under **GPLv3**.  
- Compatible with **AHCI mode**, no RAID required.  
- Minimal SD card footprint (128 MiB partition).  

## Usage
1. Flash the SD image to an internal SD card or USB stick.  
2. Insert the SD card into your Gen8’s internal SD slot or USB stick into the internal USB slot (or external if you like).  
3. Boot the server — GRUB will automatically attempt to boot SATA port 5 (ODD Bay) first, then fallback to other ports one at a time if necessary.  
4. Optionally, select manual entries from the menu.

## License
This project is licensed under **GPLv3**.  
It includes the unmodified **GRUB bootloader** (GPLv3). All scripts, configurations, and SD images are also under **GPLv3**.  
