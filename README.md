#HOWTO create a bootable usb of Kali Linux

## Automatically

Using *Mac Linux USB Loader*

https://sevenbits.github.io/Mac-Linux-USB-Loader/

##Manually

- download an iso of kali linux
https://www.kali.org/downloads/

- check *checksum* :
```bash
		$ openssl sha1 <path to iso>
```
-  **Optional** convert iso to img

*It seems that this step is optional. The transferred .iso is fully functional*
```bash
		$ hdiutil convert -format UDRW -o <path to target.img> <path to image.iso>
```
**Note : OS X tends to put the .dmg ending on the output file automatically**

- Use Disk Utility to erase the USB drive to *Mac OS journalised* (GUID Table of partition)

- Get the current list of devices and note the node for the USB Drive (*probably /dev/disk1*)
```bash
		$ diskutil list
```
- unmount the drive
```bash
		$ diskutil unmountDisk /dev/diskn
```
- copy the iso to the drive
```bash
		$ sudo dd if=< path to downloaded.img> of=</dev/diskN>
```
**Note : you can specify "bs=1m" for the blocksize used in transfer. However, it seems that it gets the iso corrupted. Need some testing with smaller values : 512k etc...**

**Note : you can check the progress of dd by pressing ctrl-t in the shell**

- change the EFI on it from the broken default one to Grub.
To do so, mount the disk and replace the EFI folder by the one in the folder (*EFI.zip*). Make sure that the folder is deleted by emptying the trash og your computer (Otherwise, error : "Not enough space")

- unmount the disk :
```bash
		$ diskutil eject /dev/diskN
```
### Try to boot on the disk by rebooting with the USB stick and pressing alt

## Add persistence

To do so, we have to add a new partition to our USB drive and add a *persistence.conf* file in it. We need a tool to format in *ext4*. I created a virtual machine of Ubuntu on VirtualBox on my Mac OS X, since it is not able to format in *ext4* by default. Then :

- Create and format new partitions with gparted. Execute :
```bash
		$ sudo gparted
```

then format all the space available to *ext4*, with label *persistence*, *primary partition*. NOTE that those two parameters are crucial.

- Mount the usb
```bash
	$ mkdir /mnt/usb
	$ mount /dev/sdb2 /mnt/usb
```
- Create the *persistence.conf* file : 
```bash
	$ echo "/ union" >> /mnt/usb/persistence.conf
```

- Unmount:
```bash
	$ umount /mnt/usb
```
Then, when booting on the USB, choose **Live USB persistence**

## Virtual Box : Boot from USB Drive

- Unmount USB Drive
```bash
	$ diskutil list
```
find the usb drive, then unmount it :
```bash
	$ diskutil unmountDisk /dev/disk2
```

The tricky part is that the VirtualBox process can only read/write files owned by the current user you are logged with. However Mac OS X, had put root as owner. With this default, you won’t be able to import the disk file that we are going to create. So the solution is too change the permission of the device.

```bash
	$ sudo chown bo /dev/disk2
```

- Create a Disk File
```bash
$ sudo VBoxManage internalcommands createrawvmdk -filename ~/Documents/myusbdrive.vmdk
-rawdisk /dev/disk2
```
- Change permission of the disk .vmdk :
```bash
$ sudo chmod 777 ~/Documents/myusbdrive.vmdk
```
- Change permission of the USB Drive :
```bash
$ sudo chmod 777 /dev/disk2
```

*NB* you will have to change permission each time you plug the USB drive in.

- Create a new virtual machine in VirtualBox and select *Use an existing virtual hard disk file*, to pick *~/Documents/myusbdrive.vmdk*

<small>This file is converted to pdf from markdown using *pandoc*

```bash
	$ pandoc HOWTO.md -o HOWTO.pdf
```
</small>
