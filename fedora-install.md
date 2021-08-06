# Intallation Guide: Fedora on the Raspberry Pi 4B+

> **_NOTE:_**  Some commands will need to be run as superuser.

---
*** Equipment *** 
- Raspberry Pi 4B+
- SD card (Minimum 1GB)
- SSD or Flash Drive (Minimum 32GB)
- Keyboard and monitor for the RPi (If you don't have this, you may follow a headless installation guide, avaiable soon)
---

We will begin by installing UEFI firmware on an SD card (1 GB) and formatting the SD card.

---
1. Download the latest RPi4 UEFI firmware

[v1.20 here](https://github.com/pftf/RPi4/releases/tag/v1.20)

The UEFI for the Raspberry Pi 4B is still in development, and therefore does not provide
all necessary features. The installation requires an SD card or USB that is formatted with either FAT16 or FAT32.
This guide will use an SD card with FAT16. At boot, the UEFI will be run and will detect and run the OS boot loader.

---
2. Format the SD card to hold the UEFI firmware:

At the shell, run `fdisk` on the SD card device to begin formatting:
```bash
fdisk /dev/sdN
```
> ***_NOTE_*** Replace N with the alphabetical index of the SD card, which can be located by calling `ls` on the `/dev/` repository

Enter the following commands (parentheticals are explanations)
```
> o (create DOS partition table)
> n (new partition)
> p (primary)
> 1 (first)
> (leave blank, start at default of 2048)
> +512M
(if it asks you if you want to remove a vfat signature, just do it. We will be formatting anyway)
> t (select partition 1)
> e (type 0e W95 FAT16 (LBA), use > L to see all if you're curious)
> a (toggle the bootable flag on partition 1)
> w (write the changes to the disk)
```

And back at the shell:
```bash
mkfs.vfat -F 16 /dev/sdN1 (make that VFAT partition)
```
---

3. Mount the new partition and extract the UEFI bootloader onto it

Run the following comamnds:
```bash
mount /dev/sdN1 <mount_dir> # (mount dir can be any dir)
unzip RPi4_UEFI_Firmware_v1.20.zip -d <mount_dir>
```
---

4. Get the latest Fedora image (Fedora 33 at the time of writing).

From [here](https://kojipkgs.fedoraproject.org/compose/33/latest-Fedora-33/compose/Server/aarch64/images/Fedora-Server-33-1.2.aarch64.raw.xz)

---

5. Extract Fedora onto a flash drive that you don't mind wiping

Decompress the file and pipe it to dd, which will copy files to the selected device.

Run the following command (replace 33-1.2 with correct version):
```bash
xzcat Fedora-Server-33-1.2.aarch64.raw.xz | sudo dd of=/dev/sdM status=progress bs=4M
```
(where M is alphabetical index of the flash drive in `/dev/`)


>**_NOTE_*** I'm not sure how ideal the `bs=4M` argument is for USB drives.
>I've seen it recommended for SD cards but I haven't looked into it
>for this particular case.*

6. Extend root partition to use more available space on flash drive

Run `parted` to resize the parition to the maximum size of the flash drive (100G in this example).

```bash
sudo parted
(parted) print devices                                                    
(parted) select /dev/sdN (where N is device name)
(parted) p (check size of partition 3)                                                   
(parted) resizepart 3 100G (or desired size)
(parted) p (confirm that partition 3 has been resized)
sudo partedprobe                  
```

Next, resize the physical volume to the extent of the device, and then extend the logical volume (`/dev/fedora_fedora/root`)
to 100% of the free space of the physical volume.

```bash
sudo pvresize /dev/sdb3
sudo lvextend -l +100%FREE /dev/fedora_fedora/root
```

---

7. Boot Fedora via UEFI

Insert the SD card and flash drive into the RPi4.

Once you see a white raspberry logo, press Esc.

Select boot manager, and from there select the last option
(It should be a USB device or storage device, but this may vary).
If you see Linux boot a few seconds later, then you selected the right device.
If you don't, try a different device in the boot manager.

If nothing explodes, you should be greeted with the anaconda CLI interface.
From here, use Fedora as you normally would. Install a GUI maybe, it's up to you.

---
