<img src="https://github.com/javiorfo/arch-install/blob/master/archlogo.png?raw=true" alt="archlinux" style="width:200px;"/>

# Arch Linux Installation (UEFI/BIOS) and Setup with [XMonarch](https://github.com/javiorfo/xmonarch)

## Preparation
- Download [Arch Linux ISO](https://archlinux.org/download/)
- Make a booteable pendrive using some software. Like [Balena Etcher](https://www.balena.io/etcher/) or **dd** bash program.
- Connect the booteable pendrive to your PC and then boot from it.

## First steps
- load keyword language (example spanish)
```console
root@archiso# loadkeys es
```

## Wifi Configuration
- Get the wifi adapter name (wlan0 in this example)
```console
root@archiso# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether g2:gb:6g:6g:g3:g9 brd ff:ff:ff:ff:ff:ff permaddr f4:fg:gd:d6:97:gg
```
- Set the wifi adapter up
```console
root@archiso# ip link set wlan0 up
```
- Configure wifi (essid: network name, password: network password)
```console
root@archiso# wpa_passphrase essid password > /etc/wifiConfig
root@archiso# wpa_supplicant -B -i wlan0 -D wext -c /etc/wifiConfig
root@archiso# dhclient
```
- Test connection
```console
root@archiso# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=10.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=13.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=11.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=11.9 ms
...
```
# IMPORTANT! 
## Check if your computer has UEFI or BIOS
- This is the UEFI output look like
```console
root@archiso# ls /sys/firmware/efi/efivars
 AcpiGlobalVariable-af9ffd67-ec10-488a-9dfc-6cbf5ee22c2e
 AcpiGlobalVariable-c020489e-6db2-4ef2-9aa5-ca06fc11d36a
 AdvMitAttrib-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 AfterReadyToBoot-7b77fb8b-1e0d-4d7e-953f-3980a261e077
 AmiGopPolicySetupData-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 AMITSESetup-c811fa38-42c8-4579-a9bb-60e94eddfb34
 BiosSetupType-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 ...
```
- This is the BIOS output look like
```console
root@archiso# ls /sys/firmware/efi/efivars
file or directory does not exist
```
### Summing up: UEFI = some output, BIOS = directory not found


## Partitions (UEFI)
- Run to enter the partition manager (if a menu shows up, choose GPT option)
```console
root@archiso# cfdisk
```
- Basic partitions

| Partition | Size              | Id Type          |
| --------- | ----------------- | ---------------- |
| BOOT      | 512M              | EFI System       |
| SWAP      | Double RAM        | Linux swap       | 
| SYSTEM    | Rest of your GB   | Linux filesystem |

## Format (UEFI)
```console
root@archiso# mkfs.vfat -F32 /dev/sda1
root@archiso# mkfs.ext4 /dev/sda2
root@archiso# mkswap /dev/sda3
root@archiso# swapon
```

## Mount (UEFI)
```console
root@archiso# /dev/sda2 /mnt
root@archiso# /mnt/boot
root@archiso# /mnt/boot/efi
root@archiso# /dev/sda1 /mnt/boot/efi
```

## Partitions (BIOS)
- Run to enter the partition manager (if a menu shows up, choose DOS option)
```console
root@archiso# cfdisk
```
- Basic partitions

| Partition | Type     | Size              | Id Type          |
| --------- | -------- | ----------------- | ---------------- |
| BOOTEABLE | Primary  | 512M              | 83 Linux         |
| SWAP      | Primary  | Double RAM        | 82 Linux swap    | 
| SYSTEM    | Primary  | Rest of your GB   | 83 Linux         |

## Format (BIOS)
```console
root@archiso# mkfs.ext2 /dev/sda1
root@archiso# mkfs.ext4 /dev/sda2
root@archiso# mkswap /dev/sda3
root@archiso# swapon
```

## Mount (BIOS)
```console
root@archiso# /dev/sda2 /mnt
root@archiso# /mnt/boot
root@archiso# /dev/sda1 /mnt/boot
```

## Install basic packages (UEFI)
```console
root@archiso# pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd netctl wpa_supplicant dialog efibootmgr
```

## Install basic packages (BIOS)
```console
root@archiso# pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd netctl wpa_supplicant dialog
```

## Generate fstab (UEFI & BIOS)
```console
root@archiso# genfstab /mnt >> /mnt/etc/fstab
```
- Check 
```console
root@archiso# cat /mnt/etc/fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# UUID=XXXXX-XXXX-XXXX-XXXX-XXXX
/dev/sda2           	/         	ext4      	rw,relatime	0 1
...
```

## Host, Clock and Locale configurations (UEFI & BIOS)
- Mount the filesystem
```console
root@archiso# arch-chroot /mnt
```
- Set Host name (whatever name you choose)
```console
root@archiso# echo your_host_name > /etc/hostname
```
- Set localtime (choose yours, this is an example with Argentina)
```console
root@archiso# ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime
```
- Set keyword distribution
```console
root@archiso# nano /etc/locale.gen
```
- Uncomment the distribution you want (in this example: es_AR.UTF8 UTF8)
<img src="https://github.com/javiorfo/arch-install/blob/master/arch-locale.png?raw=true" alt="archlinux" style="width:600px;"/>

- Save file pressing `Ctrl+O` and `Ctrl+X` to exit
- Execute to generate locale:
```console
root@archiso# locale-gen
Generating locales...
    es_AR.UTF8... done
Generation complete.
```
- Set Clock
```console
root@archiso# hwclock -w
```
- Set language (again, this is an example with Argentina)
```console
root@archiso# echo KEYMAP=es > /etc/wconsole.conf
root@archiso# echo LANG=es_AR.UTF8 > /etc/locale.conf
```

## Installing GRUB (UEFI)
- Run `grub-install` with this params
```console
root@archiso# grub-install --efi-directory=/boot/efi --bootloader -id='Arch Linux' --target=x86_64-efi
Installing for x86_64-efi platform
Installation finished. No error reported.
```
- Configure grub
```console
root@archiso# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub installation file...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other booteable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

## Installing GRUB (BIOS)
- Run `grub-install` with this params
```console
root@archiso# grub-install /dev/sda
Installing for i386-pc platform
Installation finished. No error reported.
```
- Configure grub
```console
root@archiso# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub installation file...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other booteable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for ...
done
```

## Root and User configuration (UEFI & BIOS)
- Set root password
```console
root@archiso# passwd
New password:
Retype new password:
passwd: password updated successfully
```
- Create user and set password
```console
root@archiso# useradd -m your_user_name
root@archiso# passwd your_user_name
New password:
Retype new password:
passwd: password updated successfully
```
- Umount and reboot system
```console
root@archiso# exit
root@archiso# umount -R /mnt
root@archiso# reboot
```

## Wifi Configuration (UEFI & BIOS)
- Enter your user name to login
```console
Arch Linux 5.17.1-arch1-1 (tty1)

your_host_name login:
Password:
```
- Change to root login
```console
[your_user_name@your_host_name ~ ]$ su
Password: 
```
- Enable Network Manager
```console
[root@your_host_name your_user_name]# systemctl start NetworkManager
[root@your_host_name your_user_name]# systemctl enable NetworkManager
Created symlink /etc/systemd/system/multi-user.target.wants/NetworkManager.service -> /usr/lib/systemd/system/NetworkManager.service
Created symlink /etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service -> /usr/lib/systemd/system/NetworkManager-dispatcher.service
Created symlink /etc/systemd/system/network-online.target.wants/NetworkManager-wait-online.service -> /usr/lib/systemd/system/NetworkManager-wait-online.service
```
- Get the wifi adapter name (wlp0s9j1 in this example)
```console
[root@your_host_name your_user_name]#ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp0s9j1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether g2:gb:6g:6g:g3:g9 brd ff:ff:ff:ff:ff:ff permaddr f4:fg:gd:d6:97:gg
```
- Set the wifi adapter up (wlp0s9j1 in this example)
```console
[root@your_host_name your_user_name]# ip link set wlp0s9j1 up
```
- Connet to wifi (essid: network name, your_password: network password)
```console
[root@your_host_name your_user_name]# nmcli dev wifi connect essid password your_password
```

## Drivers (UEFI & BIOS)
- Check video controllers (in this example: Intel Graphics)
```console
[root@your_host_name your_user_name]# lspci | grep VGA
00:02.0 VGA compatible controller: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor Integrated Graphics Controller (rev 06)
```
#### According to the output of the previous step, install one of these video cotrollers:
- `NVIDIA`
```console
[root@your_host_name your_user_name]# pacman -S xf86-video-nouveau
```
- `INTEL`
```console
[root@your_host_name your_user_name]# pacman -S xf86-video-intel intel-ucode
```
- `AMD/ATI` [IMPORTANT for AMD troubleshooting](https://wiki.archlinux.org/title/AMDGPU)
```console
[root@your_host_name your_user_name]# pacman -S xf86-video-amdgpu
```

## AUR repositories
- Install base-devel
```console
[your_user_name@your_host_name ~]$ sudo pacman -S base-devel
```
- Download and install paru
```console
[your_user_name@your_host_name ~]$ git clone https://aur.archlinux.org/paru.git
[your_user_name@your_host_name ~]$ cd paru
[your_user_name@your_host_name ~]$ makepkg -si
```

## Environment (UEFI & BIOS)
- Lightdm
```console
[root@your_host_name your_user_name]# pacman -S --needed xorg lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
```

#### If you want [XMonarch](https://github.com/javiorfo/xmonarch) (Xmonad + Xmobar).
```console
[root@your_host_name your_user_name]# pacman -S git wget
[root@your_host_name your_user_name]# git clone https://github.com/javiorfo/xmonarch
[root@your_host_name your_user_name]# cd xmonarch
[root@your_host_name your_user_name]# ./xmonarch.sh
```

- Enable DM
```console
[root@your_host_name your_user_name]# systemctl enable lightdm
```
- Reboot 
```console
[root@your_host_name your_user_name]# reboot
```

# More configurations after your first login
## Bluetooth - [Official Doc](https://wiki.archlinux.org/title/Bluetooth)
- Install bluetooth
```console
[your_user_name@your_host_name ~]$ sudo pacman -S bluez bluez-utils blueman
```
- Edit bluetooth configuration
```console
[your_user_name@your_host_name ~]$ sudo nano /etc/bluetooth/main.conf
```
- Set AutoEnable to true

<img src="https://github.com/javiorfo/arch-install/blob/master/arch-bluetooth.jpg?raw=true" alt="archlinux" style="width:600px;"/>

- Enable bluetooth
```console
[your_user_name@your_host_name ~]$ sudo systemctl start bluetooth
[your_user_name@your_host_name ~]$ sudo systemctl enable bluetooth
```

## Audio - [Official Doc](https://wiki.archlinux.org/title/PulseAudio)
- Install audio
```console
[your_user_name@your_host_name ~]$ sudo pacman -S pulseaudio pulseaudio-bluetooth pavucontrol
```
- Start Pulse Audio
```console
[your_user_name@your_host_name ~]$ sudo systemctl start pulseaudio
```

## Extras
```console
[your_user_name@your_host_name ~]$ sudo pacman -S gvfs libreoffice-still vlc neovim vim transmission-gtk gimp
```

### Support
- [Paypal](https://www.paypal.com/donate/?hosted_button_id=9BFAD3RVEZNQ2)
