# Arch Linux installation (Windows 10 dual boot)


## Before

1. Disable Windows Fast-Startup
2. Disable Secure Boot


## Partitioning
Windows 10 Efi partitioning

| Partition  | Location     | Size       | File system |
| ---------- |:------------:|:----------:|:-----------:|
| ESP        | nvme0n1p1    | 100 MB     | vfat        |
| Reserved   | nvme0n1p2    | 16 MB      | ?           |
| Windows 10 | nvme0n1p3    | 360 GB     | ntfs        |
| Recovery   | nvme0n1p4    | 500 MB     | ntfs        |

## Connect to the internet (Wi-Fi)

```bash
# wifi-menu
# ping  -c 3 www.google.com
```

## Format and mount disks

```bash
mkfs.ext4 /dev/nvme0n1p5
mkswap /dev/nvme0n1p6
mkfs.fat -F32 /dev/nvme0n1p7

mount /dev/nvme0n1p5 /mnt
swapon /dev/nvme0n1p6
mkdir -p /mnt/boot
mount /dev/nvme0n1p7 /mnt/boot

```

## Install

```bash
pacstrap /mnt base base-devel linux linux-firmware vim 
```


## Generate fstab

```bash
genfstab -U /mnt > /mnt/etc/fstab
```
Add noatime to fstab file on root and home partitions


## Chroot and configure base system

```bash
arch-chroot /mnt /bin/bash
```

### Locale

```bash
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
locale-gen
```

### Timezone

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```
### Hardware clock

```bash
hwclock --systohc
timedatectl set-ntp true
```


### Hostname

```bash
echo archie > /etc/hostname
vim /etc/hosts
```

/etc/hosts should look like:

```
127.0.0.1   localhost.localdomain   localhost 
::1         localhost.localdomain   localhost
127.0.1.1   archie.localdomain        archie 
```

### Root password

```bash
passwd
```

### Initial ramdisk environment

```bash
cd /boot/ && mkinitcpio -p linux
```

### GRUB

```bash
pacman -S grub efibootmgr intel-ucode os-prober mtools dosfstools 
mkdir /boot/efi
mount /dev/nvme0n1p1 /boot/efi
```

Edit `/etc/default/grub`, set `DEFAULT_TIMEOUT=30`.

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### User

```bash
pacman -S zsh 
sudo useradd -m -g users -G wheel lee -s /bin/zsh
passwd lee
```

### Misc

```bash
pacman -S wireless_tools wpa_supplicant dhclient
```

## Unmount and reboot

```bash
exit      # If still on arch-chroot mode
umount -R /mnt
reboot
```

## Post-install

Give lee/user root in /etc/sudoers

### Update
```bash
sudo pacman Syu

```


### Update
```bash
sudo pacman Syu

```


### Desktop Environment
```bash
sudo pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils 
sudo pacman -S xf86-input-synaptics
sudo pacman -S cinnamon nemo-fileroller gmd net-tools network-manager-applet
sudo systemctl start NetworkManager gdm
sudo systemctl enable NetworkManager gdm
```

### Programs
```bash
sudo pacman -S pulseaudio pulseaudio-alsa pavucontrol gnome-terminal firefox flashplugin vlc chromium unzip unrar p7zip smplayer audacious qmmp gimp xfburn thunderbird gedit gnome-system-monitor
sudo pacman -S a52dec faac faad2 flac jasper lame libdca libdv libmad libmpeg2 libtheora libvorbis libxv wavpack x264 xvidcore gstreamer0.10-plugins
sudo pacman -S libreoffice
```

### Dev
Tools:
```bash
sudo pacman -S git python-pip gnuradio hackrf
```
Programs:
```bash
sudo pacman -Sy --noconfirm zsh-theme-powerlevel10k-git
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>! ~/.zshrc

```



- [Arch Linux Wiki: Beginners Guide](https://wiki.archlinux.org/index.php/beginners'_guide#GRUB_2)
- [Arch Linux Wiki: GRUB](https://wiki.archlinux.org/index.php/GRUB)
- [Arch Linux Wiki: Installation](https://wiki.archlinux.org/index.php/Installation_guide)
- [Arch Linux Wiki: Windows and Arch dual boot](https://wiki.archlinux.org/index.php/Windows_and_Arch_Dual_Boot)

