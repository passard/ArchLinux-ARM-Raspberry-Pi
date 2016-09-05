# Raspberry Pi 1, 2 and 3 installation from a Linux system (In this case, Arch Linux)

## 1) Micro SD Card Partition Table and File System Creation
Open a terminal and _type_ **'def -h'** to identify your micro SD card. 

Replace sdX in the following instructions with the device name for the micro SD card as it appears on your computer.

##### Start fdisk (as root or with sudo) to start partitioning :
	fdisk /dev/sdX
	
##### At the fdisk prompt, delete old partitions (if there is any) and create a new one:  
 _Type_ **'o'**. This will clear out any partitions on the drive.  
 _Type_ **'p'** to list partitions. There should be no partitions left.

### 1.1 Boot partition
 _Type_ **'n'**, then **'p'** for primary, **'1'** (Default) for the first partition on the drive, press **ENTER** to accept the default first sector, then _type_ **'+100M'** for the last sector.  
 _Type_ **'t'**, then **'c'** to set the first partition to type W95 FAT32 (LBA).  
 _Type_ **'a'**, then **'1'** to toggle bootable flag on Boot partition. 

### 1.2 Extended partition
 _Type_ **'n'**, then **'e'** for extended, **'2'** (Default) for the first partition on the drive, press **ENTER** to accept the default first sector, then **ENTER** for the last sector (Extended partition will take all the available space left).

### 1.3 Root Partition
 _Type_ **'n'** : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 5
 Press **ENTER** to accept default first sector, then **'+49G'** (eg: this will create a 49gb partition) for root partition size.
 _Type_ **'t'**, partition number should be 5, press **ENTER**, then **'8e'** to set the first logical partition to type Linux LVM.
 
### 1.4 Swap partition
 _Type_ **'n'** : All space for primary partition is in use, so fdisk will automatically add a logical partition.  
 Adding logical partition 6  
 Press **ENTER** to accept default first sector, then **ENTER** again for default last sector.  
 _Type_ **'t'**, partition number should be 6, press **ENTER**, then **'82'** to set the second logical partition to type Linux Swap.  
  
 _Type_ **'p'** to check how partition table is looking, if everything looks good, write the partition table and exit by typing **'w'**.

### 1.5 Create and mount the FAT file system (-n is for label option):
	mkfs.vfat /dev/sdX1 -n boot
 	mkdir /mnt/boot
 	mount /dev/sdX1 /mnt/boot

### 1.6 Create and mount the extfile system (-L is for label option):
 	mkfs.ext4 /dev/sdX5 -L root
	mkdir /mnt/root
	mount /dev/sdX5 /mnt/root

 
### 1.7 Create and mount Swap file system (-L is for label option):
 	mkswap /dev/sdX6 -L swap
 	swapon /dev/sdX6
 	
### 1.8 Download with 'wget' and extract the appropriate root filesystem :
##### For Raspberry Pi B+
	wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
##### For Raspberry Pi 2 & 3
	wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz

##### To extract, run as root (not via sudo):
	bsdtar -xpf ArchLinuxARM*.tar.gz -C /mnt/root && sync

##### Move boot files to the first partition (still as root):
	mv /mnt/root/boot/* /mnt/boot

### 1.9 Unmount the two partitions:
	umount /mnt/boot /mnt/root

### 2 Installation from the Pi
##### Insert the SD card into the Raspberry Pi, connect ethernet and power supply (at least 2.5A power for rpi3, 2A for rpi2).

 Use the serial console or SSH to the IP address given to the board by
 your router.

 Login as the default user alarm with the password **alarm**.
 The default root password is **root**.

# ------------------------------- THE FOLLOWING HAS TO BE REVIEWED -----------------------------------

### 3 Update the system : pacman -Syyu

4) pacman -S xfce4 xfce4-goodies sudo xorg alsa-utils slim wget bluez bluez-utils blueman baobab wireless_tools mlocate binutils synapse firefox p7zip xarchiver
	optionnally : sh gcc make autoconf m4 python2 qt5 pygtk mono libva-mesa-driver python2-dbus networkmanager webkitgtk gvfs python-setuptools python-pip tinc pcmanfm ffmpeg

5) Create a new user and enable sudo rights if necessary
	useradd -m -g users -G storage,power,wheel -s /bin/bash "username"
	nano /etc/sudoers and uncomment the %wheel line
	passwd "username" to define a password and su "username" to login as newly created user

6) Configure Xfce
	Create .xinitrc by typing nano ~/.xinitrc and "starxfce4" then save
	Edit slim.conf default username and autologin yes for autologin.

7) Reboot and login with "username"
	if in tty, startxfce4
	edit /boot/config.txt according to current display
	# Uncomment to force a specific HDMI mode (this will force VGA)
	hdmi_group=2
	hdmi_mode=82

	# Uncomment to force a HDMI mode rather than DVI. This can make audio work in
	# DMT (computer monitor) modes
	hdmi_drive=1

 #   override the behaviour in config.txt if you understand the risks:
	# Removes the warning overlay.
	avoid_warnings=1

	# Additionally allows turbo when low-voltage is present.
	avoid_warnings=2

8) Enable Bluetooth
	Load generic drivers : modprobe btusb
	Enable service : systemctl enable bluetooth.service
	Start service : systemctl start bluetooth.service

9) Configure bluetooth device(s) :
	9.1) Trackpad
	# bluetoothctl
	[bluetooth]# list
	Controller <controller mac> BlueZ 5.5 [default]
	[bluetooth]# select <controller mac>
	[bluetooth]# power on
	[bluetooth]# scan on
	[bluetooth]# agent on
	[bluetooth]# devices
	[bluetooth]# pairable on
	Device <mouse mac> Name: Bluetooth Mouse
	[bluetooth]# pair <mouse mac>
	[bluetooth]# trust <mouse mac>
	[bluetooth]# connect <mouse mac>

	9.2) Keyboard https://wiki.archlinux.org/index.php/Bluetooth_keyboard
	[bluetooth]# agent KeyboardOnly
	[bluetooth]# default-agent
	[bluetooth]# pairable on
	[bluetooth]# scan on
	[bluetooth]# pair ?keyboard mac?
	[bluetooth]# trust ?keyboard mac?
	[bluetooth]# connect ?keyboard mac?
	[bluetooth]# quit

	9.3) Enable bluetooth at startup
	Create new udev rule: nano /etc/udev/rules.d/10-local.rules
		# Set bluetooth power up
		ACTION=="add", KERNEL=="hci0", RUN+="/usr/bin/hciconfig hci0 up"

10) Setup a wireless network
	Install netctl if necessary: sudo pacman -S netctl.
	Create a network profile ; /etc/netctl/examples/ contains some examples.
	To setup a WPA2-PSK network, copy over the example file and start editing:

	# cd /etc/netctl
	/etc/netctl# install -m640 examples/wireless-wpa wireless-home
	/etc/netctl# cat wireless-home
	Description='A simple WPA encrypted wireless connection'
	Interface=wlan0
	Connection=wireless
	Security=wpa

	IP=dhcp

	ESSID='MyNetwork'
	# Prepend hexadecimal keys with \"
	# If your key starts with ", write it as '""<key>"'
	# See also: the section on special quoting rules in netctl.profile(5)
	Key='WirelessKey'
	# Uncomment this if your ssid is hidden
	#Hidden=yes

	Edit MyNetwork and WirelessKey as needed. Note the 640 permissions, you do not want to leak your wireless passphrase to the world!

	Proceed with testing:
	netctl start wireless-home

	If you do not get an error, you should be connected. Let's test this:
	$ ping 8.8.8.8

	To make this network start on boot:
	# netctl enable wireless-home

11) systemctl daemon-reload

12) Configure access to AUR packages
	wget https://aur.archlinux.org/cgit/aur.git/snapshot/apacman.tar.gz
	tar xvzf apacman.tar.gz
	cd apacman
	makepkg -s
	sudo pacman -U apacman-2.3-1-any.pkg.tar.xz
 apacman -S apacman-deps apacman-utils customizepkg-git rsync
