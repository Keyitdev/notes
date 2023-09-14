>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 14 Sep 2023

# Arch Linux Installation Guide EFI

### Connect to network

```bash
ip a
iwctl station wlan0 scan
iwctl station wlan0 get-networks
iwctl station wlan0 connect <NETOWRK NAME>
```

### Create partitions

| Name | Type             | Size            | File system |
| :--: | :---------------:| :-------------: | :----------:|
| efi  | efi system       | at least 512M   | fat32       |
| swap | linux swap       | 2x ram          | swap        |
| root | linux filesystem | Remaining space | ext4        |

```bash
lsblk
cfdisk /dev/sda
```
```bash
mkfs.fat -F32 /dev/sda1

mkswap /dev/sda2
swapon /dev/sda2

mkfs.ext4 /dev/sda3
mount /dev/sda3 /mnt

lsblk
```

### Install arch system and generate fstab file

```bash
pacstrap -i /mnt base linux linux-firmware sudo nano vim git archlinux-keyring
genfstab -U -p /mnt >> /mnt/etc/fstab
```

### Chroot to the installed system
Since chrooting, you are using the newly installed system as root.
```bash
arch-chroot /mnt /bin/bash
```

### Set locale

```bash
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF8" > /etc/locale.conf
```

```bash
localectl
```

### Set the time zone and local time

```bash
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
hwclock --systohc --utc
date
```

### Set hostname

```bash
echo <HOSTNAME> > /etc/hostname
echo "127.0.0.1       localhost" >> /etc/hosts
echo "::1             localhost" >> /etc/hosts
echo "127.0.1.1       <HOSTNAME>" >> /etc/hosts
# echo "127.0.1.1       <HOSTNAME>.localdomain <HOSTNAME>" >> /etc/hosts
```

### Set root password

```bash
passwd
```

### Install GRUB

```bash
pacman -S grub efibootmgr
```
```bash
mkdir /boot/efi
mount /dev/sda1 /boot/efi
lsblk
```
```bash
grub-install --target=x86_64-efi --bootloader-id=GRUB \
--efi-directory=/boot/efi --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

### Add user

```bash
useradd -m -g users -G wheel,tty,audio,video -s /bin/bash <USERNAME>
sed -i 's/^# %wheel ALL=(ALL:ALL) ALL$/%wheel ALL=(ALL:ALL) ALL/' /etc/sudoers
passwd <USERNAME>
```

### Set up networking

```bash
pacman -S dhcpcd iwd
systemctl enable dhcpcd.service
systemctl enable iwd.service
```

### Install gpu drivers

```bash
pacman -S xf86-video-amdgpu                    # amd
pacman -S nvidia nvidia-settings nvidia-utils  # nvidia
pacman -S xf86-video-intel                     # intel
```

### Exit

```bash
exit
umount -R /mnt
reboot
```

### Sources

https://averagelinuxuser.com/a-step-by-step-arch-linux-installation-guide/