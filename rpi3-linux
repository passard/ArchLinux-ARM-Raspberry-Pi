# Raspberry Pi 3 Installation from a Linux system (In this case, Arch Linux)

## 1) Micro SD Card Partition Table and File System Creation
  Open a terminal and *Type 'def -h'* to identify your micro SD card. 

Replace sdX in the following instructions with the device name for the micro SD card as it appears on your computer.

##### Start fdisk (as root or with sudo) to partition the virtual drive:
	fdisk /dev/sdX
	
##### At the fdisk prompt, delete old partitions (if there is any) and create a new one:  
 *Type 'o'*. This will clear out any partitions on the drive.  
 *Type 'p'* to list partitions. There should be no partitions left.

### 1.1 Boot partition
 *Type 'n'*, then *'p'* for primary, *'1'* (Default) for the first partition on the drive, press *ENTER* to accept the default first sector, then *type '+100M'* for the last sector.  
 *Type 't'*, then *'c'* to set the first partition to type W95 FAT32 (LBA).  
 *Type 'a'*, then *'1'* to toggle bootable flag on Boot partition. 

### 1.2 Extended partition
 *Type 'n'*, then *'e'* for extended, *'2'* (Default) for the first partition on the drive, press *ENTER* to accept the default first sector, then *ENTER* for the last sector (Extended partition will take all the available space left).

### 1.3 Root Partition
 *Type 'n'* : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 5  
 Press *ENTER* to accept default first sector, then *'+49G'* (eg: this will create a 49gb partition) for root partition size.  
 *Type 't'*, partition number should be 5, press *ENTER*, then *'8e'* to set the first logical partition to type Linux LVM.
 
### 1.4 Swap partition
 *Type 'n'* : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 6  
 Press *ENTER* to accept default first sector, then ENTER again for default last sector.  
 *Type 't'*, partition number should be 6, press *ENTER*, then *'82'* to set the second logical partition to type Linux Swap.  
  
 *Type 'p'* to check how partition table is looking, if everything looks good, write the partition table and exit by typing *'w'*.

### 1.5 Create and mount the FAT file system (-n is for label option):
	mkfs.vfat /dev/sdX1 -n b00t
 	mkdir /mnt/boot
 	mount /dev/sdX1 /mnt/boot

### 1.6 Create and mount the ext4 file system (-L is for label option):
 	mkfs.ext4 /dev/sdX5 -L r00t
 	mount /dev/sdX5 /mnt
 
### 1.7 Create and mount Swap file system (-L is for label option):
 	mkswap /dev/sdX6 -L sw4p
 	swapon /dev/sdX6
 
## 2) Now we can start Installation

### 2.1 Install the base and development packages
 	pacstrap /mnt base base-devel
 
### 2.2 Configure the system  
##### Generate an fstab file (use -U or -L to define by UUID or labels):  
	genfstab -pL /mnt  /mnt/etc/fstab

### 2.3 Change root into the new system:
	arch-chroot /mnt

### 2.4 Create the /etc/hostname file.
 	echo myhostname  /etc/hostname
 
### 2.5 Set the time zone:
 	ln -s /usr/share/zoneinfo/Europe/Brussels /etc/localtime

### 2.6 Uncomment the needed locales in /etc/locale.gen, then generate them with:
 	locale-gen
##### Add at least LANG=your_locale in /etc/locale.conf  
	nano /etc/locale.conf
	LANG=en_GB.UTF-8
	LC_COLLATE=C
	LC_TIME=en_GB.UTF-8
	
### 2.7 Add console keymap and font preferences in /etc/vconsole.conf:
 	echo /etc/vconsole.conf  KEYMAP=be
 
### 2.8 Configure kernel options with /etc/mkinitcpio.conf:  
##### Add virtual machine support:  
	MODULES="virtio virtio_blk virtio_pci virtio_net virtio_ring"
##### Create a new initial RAM disk with:  
	mkinitcpio -p linux
##### If you get these warnings:  
	== WARNING: Possibly missing firmware for module: wd719x
	== WARNING: Possibly missing firmware for module: aic94xx
 
[You can safely ignore them !](https://wiki.archlinux.org/index.php/mkinitcpio### Possibly_missing_firmware_for_module_XXXX)
 
### 2.9 Set the root password:
 	passwd
 
## 3) Install a boot loader (GRUB):
 	pacman -S grub
 	grub-install --target=i386-pc /dev/sdX
 	grub-mkconfig -o /boot/grub/grub.cfg
 
## 4) Create a user with administrator rights:
 	useradd -m -g users -G storage,power,wheel -s /bin/bash "username"
 	nano /etc/sudoers and uncomment the %wheel line
 	passwd "username" to define a password and su "username" to login as newly created user

## 5) Install fancy packages:
	pacman -S xfce4 xfce4-goodies xorg alsa-utils slim wget baobab mlocate binutils synapse firefox p7zip xarchiver
	optionnally : sh gcc make autoconf m4 qt5 pygtk mono networkmanager webkitgtk gvfs python-setuptools python-pip tinc pcmanfm ffmpeg
	
##### Now we can assume our Installation and basic setup is finished:
	exit && umount -R /mnt
	reboot
 
## 6) Configure Xfce  
##### Create .xinitrc:
	nano ~/.xinitrc and "starxfce4" then save
	Edit slim.conf default username and autologin yes for autologin.
	starxfce4'

## 7) Configure Network  -linux
	cd /etc/netctl
	install -m640 examples/ethernet-dhcp internet
	sudo nano internet
	Inteface=ens3

 Getting started with AUR:
 pacman -S --needed base-devel git 
