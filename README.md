# My Arch Installation Instructions #

This manual is gibrid of official installation guide - https://wiki.archlinux.org/index.php/Installation_guide</br>
Arch users manuals like - https://gist.github.com/tz4678/bd33f94ab96c96bc6719035fcac2b807</br>
And my own improvements and configurations.</br>
This is instruction for UEFI, but anyway you can find there some interesing commands and tricks.</br>
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
For example:
```bash
sudo pacman-key -v archlinux-2020.11.01-x86_64.iso.sig
```
Output for correct signature:
>==> Checking archlinux-2020.11.01-x86_64.iso.sig... (detached)</br>
>gpg: Signature made Sun 01 Nov 2020 09:42:16 AM MSK</br>
>gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC</br>
>gpg: ___Good___ signature from "Pierre Schmitz <pierre@archlinux.de>" [full]

This is wrong output(i'am replaced only 1 letter in .sig file):
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
Then we should download public key to verify sing(one command for all distros):
```bash
gpg --recv-keys --keyserver=hkp://keys.gnupg.net 7F2D434B9741E8AC
```
And verify the sing:
```bash
gpg --verify archlinux-version-x86_64.iso.sig
```
For example:
```bash
gpg --verify archlinux-2020.11.01-x86_64.iso.sig
```
## 2. Writing an image on flash ##
Download official iso from https://www.archlinux.org/download/  , and write it on your flash:
### Using Linux dd (terminal) ###
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

## 4. Network configuration ##
Lets set up an internet connection.
### 4.1 Ethernet ###
If you use ethernet internet working already without any configurations. But you should check your Internet connection.
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
And connecting to AP using ___iwctl___
```bash
iwctl
iwctl <wlan> connect <"network name">
```
Replace wlan with your wlan interface and <"network name"> with AP name. Enter pass to your AP and quit form iwctl.
Then check your connection with:
```bash
ping archlinux.org
```

## 5. Partition the disks ##
 
I have two disks ssd and hdd. Root(/) and efi(/boot/efi) partitions will be on ssd, /home on hdd. To partition the disks i will use fdisk
```bash
fdisk /dev/XXX # in my case nvme0n1
```
Creating partitions using "n" command, 512M to efi all other space to root part.
Than using "t" command changing type of 512 MB partition to EFI System
And write changes with "w"

## 6. Format the partitions ##

```bash
mkfs.fat -F32 /dev/efi_partition # nvme0n1p1
mkfs.ext4 /dev/root_partition # in my case nvme0n1p2
mkfs.ext4 /dev/sda1 # home partition
```
## 7. Mount the file systems ##

```bash
mount /dev/root_partition /mnt
mkdir -p /mnt/boot/efi
mount /dev/efi_partition /mnt/boot/efi
mkdir /mnt/home
mount /dev/sda1 /mnt/home
```

## 8. Install essential packages and generating fstab ##

```bash
pacstrap /mnt base base-devel #!!!!!!!!!!!!!!!!!! linux Забыл
genfstab -U /mnt >> /mnt/etc/fstab
```

## 9. Chroot ##

arch-chroot /mnt

ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
Далее:
nano /etc/locale.gen
Раскоментируем:
en_US.UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo archlinux > /etc/hostname
nano /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 archlinux.localdomain archlinux
passwd # root pass
pacman -S sudo grub efibootmgr os-prober alsa-utils
useradd -m -g users -G wheel -s /bin/bash snekyy
visudo и расскоментировать:
%wheel ALL=(ALL:ALL) ALL
grub-install --target=x86_64-efi --efi-directory=/boot/efi 
grub-mkconfig -o /boot/grub/grub.cfg
systemctl enable NetworkManager
exit
umount -R /mnt
reboot
