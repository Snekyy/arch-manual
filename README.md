# My Arch Installation Manual #

This manual is gibrid of official installation guide - https://wiki.archlinux.org/index.php/Installation_guide</br>
Arch users manuals like - https://gist.github.com/tz4678/bd33f94ab96c96bc6719035fcac2b807</br>
And my own improvements and configurations.</br>
Hope it will help you with own arch installation, but i create this instruction only to save my own time)))

## 1. Check iso file ##
It is recommended to verify the image signature before use, especially when downloading from an HTTP mirror, where downloads are generally prone to be intercepted to serve malicious images. First, we should find sha1 or md5 sums of iso file on official web-site - https://www.archlinux.org/download/ and copy one of it for verify

### 1.1 sha1sum ###
```bash
echo "archlinux_sha1sum_from_site archlinux-version-x86_64.iso" | sha1sum --check -
```
In my time this command will be:</br>
```bash
echo "739fab8d23430a01629a131ae02713a09af86968 archlinux-2020.11.01-x86_64.iso" | sha1sum --check -
```
If you will see smt like this:</br>
>archlinux-2020.11.01-x86_64.iso: OK</br>

Than your iso is OK and you can write it on your flash.

Otherwise, you should know that your image is damaged and needs to be reinstalled. This is looks like this:</br>
>archlinux-2020.11.01-x86_64.iso: FAILED</br>
>sha1sum: WARNING: 1 computed checksum did NOT match

### 1.2 md5sum ###
Everything is the same but with MD5:
```bash
echo "md5sum_of_archlinux_iso archlinus-version-x86_64.iso" | md5sum --check -
```

### 1.3 Signature verify ###\
If you already use arch you can use:
```bash

## 2. Writing an image on flash ##

Download official iso from https://www.archlinux.org/download/  , and write it on your flash:
### Using terminal dd (terminal) ###
```bash
sudo dd of=/path/to/arch_iso if=/dev/your_flash bs=8M status=progress
```
### Using gui image writer ###
Everything is the same but with gui application. I prefer use Balena Etcher, it is simple, minimalistic and beautiful app. Etcher is available for Windows, Linux and MacOS. You can download it from offical web-site - https://www.balena.io/etcher/
## **3. Network configuration (real installation)** ##
### 3.1 Ethernet ###
If you use ethernet, you should skip this point, because ethernet internet working already. But you should check your Internet connection.
```bash
ping archlinux.org
```
### 3.2 Wifi ###
Wifi configuration requires couple of commands.
Make sure the wireless card is not blocked with rfkill: 
```bash
rfkill list
```
If this command will has output enter next command to unblock your wifi:
```bash
rfkill unblock wifi
```
And check your connection with:
```bash
ping archlinux.org
```
Great!
## 4. Time ##
 
