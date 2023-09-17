>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 18 Sep 2023

# Arch Linux Installation Guide EFI

### Connect to network

```bash
iwctl device list
iwctl station <DEVICE> scan
iwctl station <DEVICE> get-networks
iwctl station <DEVICE> connect <NETOWRK NAME>
```

### Create disk partition table

> Example

| Partition | Name | Type             | Size            | File system |
| ----------| ---- | -----------------| --------------- | ------------|
| sda1      | efi  | efi system       | at least 512M   | fat32       |
| sda2      | swap | linux swap       | 2x ram          | swap        |
| sda3      | root | linux filesystem | Remaining space | ext4        |
| sda4      | home | linux filesystem | Remaining space | ext4        |

Choose the `gpt` partition type.

```bash
lsblk
cfdisk /dev/sda
```

### Format partitions

```bash
mkfs.fat -F32 /dev/sda1
```
```bash
mkswap /dev/sda2
swapon /dev/sda2
```
```bash
mkfs.ext4 /dev/sda3
```
> `home` is optional.
```bash
mkfs.ext4 /dev/sda4
```
```bash
lsblk
```

### Mount partitions

```bash
mount /dev/sda3 /mnt
```
> `home` is optional.
```bash
mkdir /mnt/home
mount /dev/sda4 /mnt/home
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
```
Optionally you can change last line to:
```bash
echo "127.0.1.1       <HOSTNAME>.localdomain <HOSTNAME>" >> /etc/hosts
```

### Set root password

```bash
passwd
```

### Install bootloader

```bash
pacman -S grub efibootmgr
```
```bash
mkdir /boot/efi
mount /dev/sda1 /boot/efi
lsblk
```
```bash
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --removable
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

### Exit

```bash
exit
umount -R /mnt
reboot
```

### Sources

https://averagelinuxuser.com/a-step-by-step-arch-linux-installation-guide/
