# My Arch Installation Manual #

This manual is gibrid of official installation guide - https://wiki.archlinux.org/index.php/Installation_guide</br>
Arch users manuals like - https://gist.github.com/tz4678/bd33f94ab96c96bc6719035fcac2b807</br>
And my own improvements and configurations.</br>
Hope it will help you with own arch installation, but i create this instruction only to save my own time)))

## 1. Check iso sum(md5, sha1) and signature verify ##
### 1.1 Signature verify ###
It is recommended to verify the image signature before use, especially when downloading from an HTTP mirror, where downloads are generally prone to be intercepted to serve malicious images. We can do this with:

## 2. Writing an image on flash ##

Download official iso from https://www.archlinux.org/download/  , and write it on your flash:
```bash
sudo dd of=/path/to/arch_iso if=/dev/your_flash bs=8M status=progress
```

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
 
