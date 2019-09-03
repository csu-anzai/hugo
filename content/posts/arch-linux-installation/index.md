+++
title = "Arch Linux Installation"
date = 2019-09-02T14:44:31+07:00
author = "ashpex"
tags = ["linux","guide"]
draft = false
description = "Why? As you can notice, there are various tutorials in the net for the keyword “Arch installation”. As an Arch user, I will recommmend you to take a look at the Arch wiki for such an installation progess instead. So what is the purpose of this post? You may ask."
showFullContent = false
cover = "posts/arch-linux-installation/archlinux.jpg"
+++

## Why?

As you can notice, there are various tutorials in the net for the keyword "Arch installation". As an Arch user, I will recommmend you to take a look at the Arch wiki for such an installation progess instead.  So what is the purpose of this post? You may ask. 

First of all, this post serves as a snippset for my arch installation. I don't want to forget anything esstensial for my daily workflow incase I have to make a complete reinstall. Secondly, as personalized as this installation guide may seems, it may help new users in some ways.

Now let's get started:
 
_`/dev/nvme0n1` can be replaced with `/dev/sda` depending on different hardware._

### Setting up


#### Setting up network

`$ ip link`

`$ wifi-menu`

#### Disks partition 


`$ lsbk`

`$ cgdisk /dev/nvme0n1`

| 	Partitions      | Space     | Type | Lable  |
| :----------------:|:---------:|:----:|:------:|
|	/dev/nvme0n1p1  | 512M      | ef00 | boot   |
|	/dev/nvme0n1p2  | 4G        | 8200 | swap   |
|	/dev/nvme0n1p3  | remaining | 8300 | system | 

#### Format partitions

**1. EFI partition**

`$ mkfs.fat -F32 /dev/nvme0n1p1`

**2. Activate swap**

`$ mkswap /dev/nvme0n1p2`

`$ swapon /dev/nvme0n1p2`

**3. System partition**

`$ mkfs.ext4 /dev/nvme0n1p3`

#### Mount and setting up 

```
$ mount /dev/nvme0n1p3 /mnt
$ mkdir /mnt/boot
$ mount /dev/nvme0n1p1 /mnt/boot
$ df
```

#### Installation

**1. Select mirror**
- `$ nano /etc/pacman.d/mirrorlist`
- Place this on top:  `Server = http://f.archlinuxvn.org/arclinux/$repo/os/$arch`

**2. Install base system**

```
$ pacstrap /mnt base base-devel
$ genfstab -U /mnt
$ genfstab -U /mnt >> /mnt/etc/fstab
$ cd /mnt/etc
$ cat fstab
```

**3. Chroot into system: setting up timezone, passwd,...**

```
$ arch-chroot /mnt
$ ln -sf /usr/share/zoneinfo/ /  /etc/localtime
$ hwclock --systohc --utc
$ nano /etc/locale.gen` then uncomment `en_US.UFT-8`
$ locale-gen
$ echo "LANG=en_US.UFT-8" > /etc/locale.conf
$ nano /etc/hostname
$ passwd
$ useradd -g users -G wheel,storage,power -m ashpex
```
or `$ localectl set-locale LANG=en_US.UTF-8`

**4. Setting up Bootloader**

```
$ pacman -S grub efibootmgr
$ grub-install --target-x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
$ pacman -S os-prober
$ grub-mkconfig -o /boot/grub/grub.cfg
```

**5. Setting up wifi**

```
$ pacman -S dialog wpa_supplicant
$ exit
$ reboot
```

### Install DEs or WM.

#### Gnome

```
$ sudo pacman -Syu
$ sudo pacman -S xorg xorg-server
$ sudo pacman -S gnome
$ sudo systemctl start gdm.service
$ sudo systemctl enable gdm.service
$ sudo pacman -S pulseaudio pulseaudio-alsa
```

#### i3



```
$ sudo pacman -S i3-gaps polybar dunst dmenu feh mpd mpv ranger rofi vim xorg xorg-server pulseaudio pulseaudio-alsa alsa-utils lightdm nautilus termite ibus-unikey firefox compton pamac yay yaourt gnome-screenshot git
```
_Notes: install `light` package to control brightness_

#### XFCE

```
$ sudo pacman -S xfce4 xfce4-goodies
```

_Notes:_

- Setting time
- Set hostfile
- Setting up sudoers
- Install yaourt in arch repo then replace it with yay.
- Setup wifi_

### Troubleshooting

#### 1. Wifi icon

**Initial Requirements**

1. Hosts

Check the configuration of your /etc/hosts file, a valid configuration looks like this:

#<ip-address> <hostname.domain.org> <hostname> 127.0.0.1 localhost.localdomain yourHostname ::1 localhost.localdomain yourHostname 

2. Devices

- You can identify your networking-devices like this:

- `$ lspci | grep -i net `

- If your device is not listed, it is maybe an usb-device, so try this command:

- `$ lsusb` 

- With the following command you can check the current state of all your network-devices:

- `$ ip link `

**Installation of Required tools**

1. Install the wpa_supplicant tools

- `$ sudo pacman -S wpa_supplicant` 

2. the wireless tools

- `$ sudo pacman -S wireless_tools` 

3. Install the networkmanager

- `$ sudo pacman -S networkmanager` 

4. Install the network-manager-applet aka nm-applet

- `$ sudo pacman -S network-manager-applet `

5. Install gnome-keyring

- `$ sudo pacman -S gnome-keyring `

6. Configuration

- Make the networkmanager start on boot:

- `$ sudo systemctl enable NetworkManager.service `

7. Disable dhcpcd

- Since networkmanager wants to be the one who handles the dhcpcd related stuff, you have to disable and stop dhcpcd:

```
$ sudo systemctl disable dhcpcd.service 
$ sudo systemctl disable dhcpcd@.service 
$ sudo systemctl stop dhcpcd.service 
$ sudo systemctl stop dhcpcd@.service
```

8. Enable wpa_supplicant, if you want to use your wireless connection:

- `$ sudo systemctl enable wpa_supplicant.service `

9. Add your user to the network group:

- `$ gpasswd -a <USERNAME> network `

 Turn off network interface controllers:

10. Turn off your network interface controllers, in my case eth0 and wlan0:

```
$ ip link set down eth0
$ ip link set down wlan0
```

- Now start wpa_supplicant:

- `$ sudo systemctl start wpa_supplicant.service` 

- Now Start the networkmanager:

- `$ sudo systemctl start NetworkManager.service `

- Now you should See the tray-icon on the top bar

 _Source: https://unix.stackexchange.com/questions/292195/install-network-manager-applet-tray-icon-on-arch-linux-gnome-3-20_

#### 2. Sudoers

_Source: https://www.garron.me/en/linux/visudo-command-sudoers-file-sudo-default-editor.html_

Logging as root

`$ visudo`

Add another line after this one 

`root ALL=(ALL) ALL`

With: (by pressing O, then :X to save)

`username ALL=(ALL) ALL`