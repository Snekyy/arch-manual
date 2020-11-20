# My Arch Installation Instructions (2020, UEFI) #

This manual is merge of official installation guide - https://wiki.archlinux.org/index.php/Installation_guide</br>
Arch users manuals like - https://gist.github.com/tz4678/bd33f94ab96c96bc6719035fcac2b807</br>
And my own improvements and configurations.</br>
This is instruction for UEFI, but anyway you can find there some interesing for you too.</br>
Hope it will help you with first arch installation, but i create this instruction only to save my own time)))

## 1. Check iso file ##
It is recommended to verify the image signature and hashsum before use, especially when downloading from an HTTP mirror, where downloads are generally prone to be intercepted to serve malicious images. 

First, we should find sha1 or md5 sums of iso file on official web-site - https://www.archlinux.org/download/ and copy one of it for verify

### 1.1 sha1sum ###
```bash
echo "archlinux_sha1sum_from_site archlinux-version-x86_64.iso" | sha1sum --check -
```
For example:</br>
```bash
echo "739fab8d23430a01629a131ae02713a09af86968 archlinux-2020.11.01-x86_64.iso" | sha1sum --check -
```
If you will see smt like this:</br>
>archlinux-2020.11.01-x86_64.iso: OK

Than your iso is OK and you can write it on your flash.

Otherwise, you should know that your image is damaged and needs to be reinstalled. For example:</br>
>archlinux-2020.11.01-x86_64.iso: FAILED</br>
>sha1sum: WARNING: 1 computed checksum did NOT match
### 1.2 md5sum ###
Everything is the same but with MD5:
```bash
echo "md5sum_of_archlinux_iso archlinus-version-x86_64.iso" | md5sum --check -
```
### 1.3 Signature verify ###
You can download pgp signature from official web-site - https://www.archlinux.org/download/
#### 1.3.1 Already have Arch Linux ####
If you already have Arch Linux you can use key in your system to verify signature:
```bash
sudo pacman-key -v archlinux-version-x86_64.iso.sig
```
Output for correct signature:
>==> Checking archlinux-2020.11.01-x86_64.iso.sig... (detached)</br>
>gpg: Signature made Sun 01 Nov 2020 09:42:16 AM MSK</br>
>gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC</br>
>gpg: ___Good___ signature from "Pierre Schmitz <pierre@archlinux.de>" [full]

This is bad output(i'am replaced only 1 letter in .sig file):
>==> Checking archlinux-2020.11.01-x86_64.iso.sig... (detached)</br>
>gpg: Signature made Sun 01 Nov 2020 09:42:16 AM MSK</br>
>gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC</br>
>gpg: ___BAD___ signature from "Pierre Schmitz <pierre@archlinux.de>" [full]</br>
>==> ERROR: The signature identified by archlinux-2020.11.01-x86_64.iso.sig could not be verified.
#### 1.3.2 I am in other os ####
There is little bit harder way.</br>
We should download GnuPG ( this package has different names in different distributions ):
##### Ubuntu/Debian #####
```bash
sudo apt install gpg
```
##### CentOS/Fedora/RHEL #####
For __rpm__ or __yum__ based distributions we can install GPG with the following command.
```bash
sudo yum install gnupg
```

Then we should download public key to verify sing:
```bash
gpg --recv-keys --keyserver=hkp://keys.gnupg.net 7F2D434B9741E8AC
```
And verify the sing:
```bash
gpg --verify archlinux-version-x86_64.iso.sig
```

## 2. Writing an image on flash ##
Download official iso from https://www.archlinux.org/download/  , and write it on your flash:
### Using Linux dd (terminal) ###
You can find your flash using command:
```bash
sudo fdisk -l
```
And write image on it:
```bash
sudo dd of=/path/to/arch_iso if=/dev/your_flash bs=8M status=progress
```
### Using gui image writer ###
Everything is the same but with gui application. I prefer use Balena Etcher, it is simple, minimalistic and beautiful app. Etcher is available for Windows, Linux and MacOS. You can download it from offical web-site - https://www.balena.io/etcher/
![There is will be etcher image preview](/img/etcher.png)

## 3. Verify the boot mode (Installation starts!) ##
After we boot with flash first of all we need to verify the boot mode. We are installing arch on UEFI, so lets check the boot mode:
```bash
ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode. If the system did not boot in UEFI refer to your motherboard's manual or your computer doesn't support UEFI...

## 4. Connect to Internet ##
Lets set up an Internet connection.
### 4.1 Ethernet ###
If you use ethernet, everything is already working fine without any configurations. But anywat, you should check your Internet connection.
```bash
ping archlinux.org
```
### 4.2 Wifi ###
Wifi configuration requires couple of commands.
Make sure the wireless card is not blocked with rfkill: 
```bash
rfkill list
```
If it is blocked, do this:
```bash
rfkill unblock wifi
```
And connecting to AP using ___iwctl___:
```bash
iwctl
station list # Here you will find name of your wlan interface
station <wlan> connect <"network name"> 
```
Replace wlan with your wlan interface and <"network name"> with AP name. Enter pass to your AP and quit from iwctl.
Then check your connection with:
```bash
ping archlinux.org
```
## 5. Update the system clock ##
```bash
timedatectl set-timezone Region/City # you can see all timezones with timedatectl list-timezones
timedatectl set-ntp true
```
## 6. Partition the disks, format partitions, mount file systems  ##

### 6.1 Partition the disks ### 
I have two disks ssd and hdd. Root(/) and efi(/boot/efi) partitions will be on ssd, /home on hdd. To partition the disks i will use __fdisk__
```bash
fdisk /dev/XXX # replace XXX with your ssd
```
Creating partitions using "n" command, 512M to efi all other space to root part. 
Than using "t" command changing type of 512 MB partition to EFI System. 
And write changes with "w".</br>

Than we need to create a home partition on hdd:
```bash
fdisk /dev/XXX # replace XXX with your hdd
```
Creating partitions using "n" command, and save changes with "w"</br>

### 6.2 Format partitions ###
Now we need to format they in correct file systems.
```bash
mkfs.fat -F32 /dev/efi_partition # nvme0n1p1
mkfs.ext4 /dev/root_partition # in my case nvme0n1p2
mkfs.ext4 /dev/sda1 # home partition
```

### 6.3 Mount file systems ###
Mount the root volume to /mnt. Efi partition to /mnt/boot/efi. And home to /mnt/home.
```bash
mount /dev/root_partition /mnt
mkdir -p /mnt/boot/efi
mount /dev/efi_partition /mnt/boot/efi
mkdir /mnt/home
mount /dev/sda1 /mnt/home
```
You can check your partitions for correct mount with:
```bash
lsblk
```
## 7. Select an appropriate mirror ##
This is a big problem with installing Arch Linux. If you just go on installing it, you might find that the downloads are way too slow. In some cases, it’s so slow that the download fails. It’s because the mirrorlist (located in /etc/pacman.d/mirrorlist) has a huge number of mirrors but not in a good order. The top mirror is chosen automatically and it may not always be a good choice.</br>
___Reflector___ is a Python script which can retrieve the latest mirror list from the Arch Linux Mirror Status page, filter the most up-to-date mirrors, sort them by speed and overwrite the file. We will use it to sort mirror list by speed</br>

### 7.1 Backup original mirrorlist ###

In the following examples, /etc/pacman.d/mirrorlist will be overwritten. Make a backup before proceeding.
```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```
 
### 7.2 Sync repositories ###

Sync the pacman repository so that you can download and install software:
```bash
pacman -Syy
```

### 7.3 Sorting mirrors ###

Now we can start sorting mirrorlist. Firslty we need to download ___reflector___:
```bash
pacman -S reflector
```

And now we can sort mirrors:
```bash
reflector --verbose --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```
This command will add top 5 fastest mirror to the top of mirrorlist.</br>
Or you can sort at once 200 mirrors:
```bash
reflector --verbose --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
You can also sort mirros by country:
```bash
reflector --country Russia --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
## 8. Install essential packages and generating fstab ##

### 8.1 Pacstrap ###
Pacstrap script installs the base package, Linux kernel, firmware for common hardware, text editors and etc:
```bash
pacstrap /mnt base base-devel linux linux-firmware vim
```

### 8.2 Fstab generating ###
The ___fstab___ file can be used to define how disk partitions, various other block devices, or remote filesystems should be mounted into the filesystem.
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## 9. Chroot ##
A chroot is an operation that changes the apparent root directory for the current running process and their children. A program that is run in such a modified environment cannot access files and commands outside that environmental directory tree. This modified environment is called a chroot jail.</br>
Changing root is commonly done for performing system maintenance on systems where booting and/or logging in is no longer possible. Common examples are:</br>

* Reinstalling the bootloader
* Rebuilding the initramfs image
* Upgrading or downgrading packages
* Resetting a forgotten password
* Building packages in a clean chroot

Lets do this!</br>
```bash
arch-chroot /mnt
```
### 9.1 Set the time zone ###
Set the time zone:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run hwclock to generate /etc/adjtime:
```bash
hwclock --systohc
```

### 9.2 Localization ###
Edit /etc/locale.gen:
```bash
vim /etc/locale.gen
```
Uncomment __en_US.UTF-8 UTF-8__ and other needed locales.
And generate the locales by running
```bash
locale-gen
```
Create the locale.conf file, set the LANG variable accordingly and export it:
```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
export LANG=en_US.UTF-8
```
### 9.3 Network configuration ###

Creating the hostname file:
```bash
echo archlinux > /etc/hostname
```
Add matching entries to hosts:
```bash
vim /etc/hosts
```

>127.0.0.1    localhost</br>
>::1          localhost<br>
>127.0.1.1    archlinux.localdomain  archlinux

Change ___archlinux___ with your hostname, if my hostname different with you</br>
If the system has a permanent IP address, it should be used instead of 127.0.1.1.

### 9.4 Downloading packages ###

```bash
pacman -S sudo grub efibootmgr net-tools wpa_supplicant wireless_tools iw dhcpcd
```

### 9.5 Set the root password ###

```bash
passwd
```

### 9.6 Creating a user ###

```bash
useradd -m -g users -G wheel -s /bin/bash your_username
```
And setting pass to that user:
```bash
passwd username
```
Now, using ___visudo___ in /etc/sudoers we need to uncomment "%wheel ALL=(ALL:ALL) ALL":
```bash
visudo
```

### 9.7 Grub ###

Installing grub:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi
```
Also i am setting GRUB_TIMEOUT=0 in /etc/default/grub, but you can ___SKIP___ this step, because it is dont necessary:
```bash
vim /etc/default/grub
```
And creating cfg file:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### 9.8 Umount /mnt and reboot ###

```bash
systemctl enable dhcpcd
exit
umount -R /mnt
shutdown now
```
Don’t forget to take out the live USB before powering on the system again.

## 10 After installation ##
When we booted in installeted arch, we need configure internet connection, if you are use ethernet skip this step:

### 10.1 Internet connection ### 

Next instruction was taken from http://rus-linux.net/MyLDP/consol/wifi-from-command-line.html

#### 10.1.1 Ethernet ####

SKIP THIS STEP!

#### 10.1.2 WiFi ####

So first, it is assumed that you have the correct drivers loaded for your wireless network card. Without this, nothing will work.</br>
Then you can check which network interfaces wireless connections support with the command:
```bash
iwconfig
```
You will see this output:
PHOTO</br>
And you should find there your wireless network card, in my case its wlp2s0(its has different names, for example: wlan0, wlp2s0....)</br>
Then, just in case, check that the interface is enabled:
```bash
sudo ip link set <wireless network card> up # REPLACE <wireless network card> with your, in my case wlp2s0
```
Once you know that your interface is working, you can search for available wireless networks with the command:</br>
```bash
sudo iw dev <wireless network card> scan | less
```
From the output, you can find out the name of the network (SSID), the signal strength and the type of security used (that is, WEP, WPA / WPA2).<br>

##### 10.1.2.1 Unsecure network #####

If your AP without pass you can immediately connect to it:
```bash
sudo iw dev <wireless network card> connect [network SSID]
```

##### 10.1.2.2 WEP #####

If your network uses WEP encryption, it's also quite simple:
```bash
sudo iw dev wlan0 connect [network SSID] key 0: [WEP key]
```

##### 10.1.2.3 WPA/WPA2 #####

But if your network uses WPA or WPA2, things get more complicated. In this case, you need to use the wpa_supplicant utility, which is not always preinstalled on the system. You need write some lines in this file - /etc/wpa_supplicant/wpa_supplicant.conf. In my case its doesn't exists and i created it, but in other case i recommend adding them to the end of the file and making sure other configurations are commented out. Be careful as both ssid and password are case sensitive. You can enter the access point name instead of ssid, and wpa_supplicant will replace it with the corresponding ssid.
```bash
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf 
```
And write this into file:
>network={</br>
>ssid="[network ssid]"</br>
>psk="[the passphrase]"</br>
>priority=1</br>
>}

After completing the configuration, run this command in the background:
```bash
sudo wpa_supplicant -i <wireless network card> -c /etc/wpa_supplicant/wpa_supplicant.conf
```

Now you need to get the IP address using the command:
```bash
sudo dhcpcd <wireless network card>
```

If done correctly, you should obtain a new IP address via DHCP and the process will run in the background. You can always check for a connection with the command:
```bash
iwconfig
```
### 10.2 Update ###

```bash
sudo pacman -Syu
```

### 10.3 Drivers installation ###

Choose one of them and install:
* xf86-video-amdgpu - new free driver for AMD video cards;
* xf86-video-ati - old free driver for AMD;
* xf86-video-intel - driver for Intel integrated graphics;
* xf86-video-nouveau - free driver for NVIDIA cards;
* xf86-video-vesa - free driver that supports all cards, but with very limited functionality;
* nvidia is a proprietary driver for NVIDIA.
```bash
sudo pacman -S <your-driver>
```

### 10.4 Xorg installation and configuration ###

```bash
sudo pacman -S xorg xorg-xinit bspwm sxhkd dmenu nitrogen picom konsole chromium arandr
sudo Xorg :0 -configure
sudo cp /root/xorg.conf.new /etc/X11/xorg.conf
```

### 10.5 INSTALLING THE GRAPHICAL ENVIRONMENT ###

