# Raspberry Pi installation from a Linux system (In this case, Arch Linux)
##### Setup a Raspberry Pi (1, 2, 3 and 4, model B and B+) with Arch Linux, from scratch.
##### All steps described below should be done in a terminal (command line interface), either from Mac OS or your favorite Linux distribution.

## 1) Micro SD Card Partition Table and File System Creation
Open a terminal and _type_ **'df -h'** to identify your micro SD card. Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.

#### Replace sdX in the following instructions with the device identifier for the micro SD card as it appears on your computer (eg. sda, sdb, sdc, sdd...).

##### Start fdisk (__***as root or with sudo***__) to start partitioning :
	fdisk /dev/sdX
	
##### At the fdisk prompt, delete old partitions (if there is any) and create a new one:  
 _Type_ **'o'**. This will clear out any partitions on the drive.  
 _Type_ **'p'** to list partitions. There should be no partitions left.

### 1.1 Boot partition
 _Type_ **'n'**, then **'p'** for primary, **'1'** (Default) for the first partition on the drive, press **ENTER** to accept the default first sector, then _type_ **'+100M'** for the last sector.  
 _Type_ **'t'**, press **ENTER**, then _type_ **'c'** to set the first partition as W95 FAT32 (LBA).
_Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'._
 _Type_ **'a'**, default partition number should be **'1'**, press **ENTER** to toggle bootable flag on boot partition. 
_The bootable flag on partition 1 is enabled now._

### 1.2 Root partition
 _Type_ **'n'**, then **'p'** for primary, **'2'** (Default) should be the next partition on the drive, press **ENTER** to accept the default first sector, then **'+49G'** (eg: this will create a 49gb partition) for root partition size.
 
### 1.3 Swap partition
 _Type_ **'n'** : then **'p'** for primary, **'3'** (Default) should be the next partition on the drive, press **ENTER** to accept default first sector, then **ENTER** again for default last sector (it will take all the available space left).  
 _Type_ **'t'**, default partition number should be **'3'**, press **ENTER**, then **'82'** to set the second logical partition to type Linux Swap.  

### 1.4 Check partition table
 _Type_ **'p'** to check how partition table is looking, if everything looks good, write the partition table and exit by typing **'w'**.

## Creating file system operations (__***must be done as a root user***__)

### 1.5 Create and mount the FAT file system (-n is for FAT label option):
	mkfs.vfat /dev/sdX1 -n boot
 	mkdir /mnt/boot
 	mount /dev/sdX1 /mnt/boot

### 1.6 Create and mount the extfile system (-L is for label option):
 	mkfs.ext4 /dev/sdX2 -L root
	mkdir /mnt/root
	mount /dev/sdX2 /mnt/root
 
### 1.7 Create and mount Swap file system (-L is for label option):
 	mkswap /dev/sdX3 -L swap
 	swapon /dev/sdX3
 	
### 1.8 Download with 'wget' and extract the appropriate root filesystem :
##### For Raspberry Pi B+
	wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
##### For Raspberry Pi 2
	wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
##### For Raspberry Pi 3
	wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-3-latest.tar.gz
##### For Raspberry Pi 4
	wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-4-latest.tar.gz

##### To extract, run as root (not via sudo):
	bsdtar -xpf ArchLinuxARM-rpi*.tar.gz -C /mnt/root && sync

##### Move boot files to the first partition (still as root):
	mv /mnt/root/boot/* /mnt/boot

### 1.9 Unmount the two partitions:
	umount /mnt/boot /mnt/root

## 2 Installation from the Pi
##### Insert the SD card into the Raspberry Pi, connect ethernet and power supply (at least 2.5A power for rpi3, 2A for rpi2).

##### Use the serial console or SSH to the IP address given to the board by your router.
If you don't know your IP yet, you can use **'arp -a'** or **'netstat -r'** commands to retrieve devices connected to your network.

##### Login as the default user alarm with the password **alarm**. The default root password is **root**.

### 2.1 Update the system with pacman
##### First update PGP signatures
	pacman-key --init && pacman-key --populate archlinuxarm
then _type_ **'pacman -Syyu'**

### 2.2 Install desktop UI (I use xfce4) and basic tools (I use slim as greeter, despite being discontinued, it is easy and fast to set up)
	pacman -S xfce4 xfce4-goodies sudo xorg alsa-utils slim wget bluez bluez-utils blueman baobab wireless_tools mlocate binutils synapse firefox p7zip xarchiver networkmanager ffmpeg tinc
##### Optionnally, add some developer tools:
	pacman -S base-devel openssh python2 qt5 pygtk mono libva-mesa-driver python2-dbus gvfs python-setuptools python-pip pcmanfm --needed

### 2.3 Create a new user and enable sudo rights if necessary. Replace **'username'** by a name of your choice. 
	useradd -m -g users -G storage,power,wheel -s /bin/bash "username"
	nano /etc/sudoers and uncomment _%wheel_ line by removing the "#" sign
	passwd "username" to define a password and su "username" to login as newly created user

### 2.4 Configure Xfce
##### Create .xinitrc by typing :
	nano ~/.xinitrc
_Type_ **'starxfce4'** then save (ctrl+o) and exit (ctrl+x)

### 2.5 Edit slim.conf (as root or with sudo)
_Type_ **'sudo nano /etc/slim.conf'** then uncomment _default_user_ line and replace _simone_ with your default username
If you want the session to start automatically, uncomment _autologin_ line and replace _no_ by **'yes'**.

## 3 Raspberry Pi configuration and tweaks
##### Reboot and login with _username_ credentials (if you did not chose autologin)
If xfce4 desktop does not start after a first reboot, _type_ **'startxfce4'** to start it manually.

### 3.1 Optimize Display
Edit /boot/config.txt according to current display. I use the following for HD (1080px). More info on [raspberrypi.org](https://www.raspberrypi.org/documentation/configuration/config-txt/)

##### As root or with sudo, _type_ 'sudo nano /boot/config.txt'
Then uncomment following lines to force a specific HDMI mode (for example, this will force VGA)
	hdmi_group=2
	hdmi_mode=82

##### Uncomment to force a HDMI mode rather than DVI. This can make audio work in some cases.
	hdmi_drive=1

##### Avoid warnings at boot for lower power supply (only if you understand the risks):
	avoid_warnings=1
### OR
##### To avoid even more warnings and allow turbo when low-voltage is present.
	avoid_warnings=2

#### Setup a wireless network
We will use systemd network control (netctl). All commands must be run as root or with sudo.

Create a network profile; /etc/netctl/examples/ contains some examples.

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

If you do not get an error, you should be connected. Let's network by pinging goggle DNS
	$ ping 8.8.8.8

To make this network start on boot:
	# netctl enable wireless-home

Then reload systemd
	# systemctl daemon-reload

## 4 Enable Bluetooth

### Raspberryi 3 Broadcom embed chip
Skip this part if you are running on an older version of the Raspberry Pi

Further setup is require on rpi3.
First, configure access to [AUR packages](https://wiki.archlinux.org/index.php/AUR_helpers)
I used yaourt, pacaur or apacman in the past, feel free to chose your favorite, their setup is quite similar.

Now I will use yay as an AUR helper:
	git clone https://aur.archlinux.org/yay.git && mv yay .yay && cd .yay && makepkg -si

You can now use yay to install packages!
So let's install pi-bluetooth and hciattach-rpi3:
	yay -S pi-bluetooth hciattach-rpi3
	
Then enable and start bcrm43438:
	sudo systemctl enable brcm43438
	sudo systemctl start brcm43438

Then enable and start bluetooth:
	sudo systemctl enable bluetooth
	sudo systemctl start bluetooth

### Download ad install bluetooth utilities

install bluez, bluez-utils by pacman -Sy bluez

You must be root or using sudo for this. 
Load generic drivers: **'modprobe btusb'**
Enable service: **'systemctl enable bluetooth'**
Start service: **'systemctl start bluetooth'**

### 4.1 Configure bluetooth device(s):

Tip: To automate bluetoothctl commands, use echo -e "<command1>\n<command2>\n" | bluetoothctl
	
#### Apple Trackpad
	bluetoothctl
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

#### Keyboard https://wiki.archlinux.org/index.php/Bluetooth_keyboard
	[bluetooth]# agent KeyboardOnly
	[bluetooth]# default-agent
	[bluetooth]# pairable on
	[bluetooth]# scan on
	[bluetooth]# pair ?keyboard mac?
	[bluetooth]# trust ?keyboard mac?
	[bluetooth]# connect ?keyboard mac?
	[bluetooth]# quit

#### Enable bluetooth at startup
	Create new udev rule: nano /etc/udev/rules.d/10-local.rules
		# Set bluetooth power up
		ACTION=="add", KERNEL=="hci0", RUN+="/usr/bin/hciconfig hci0 up"


