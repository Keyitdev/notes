>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 18 May 2026

# Arch Linux Installation Guide - UEFI, LVM on LUKS, Secure Boot

## Pre-installation requirements

Ensure that you are in **UEFI** mode and that Secure Boot is set to **Setup mode**.

Connect to network.
```sh
iwctl device list
iwctl station <DEVICE> scan
iwctl station <DEVICE> get-networks
iwctl station <DEVICE> connect <NETOWRK NAME>
```

## Partitions

Create disk partition table.

| Partition | Name  | Encryption  | Type                     | Size            | File system |
| ----------| ------|-------------|--------------------------|-----------------|-------------|
| sda1      | efi   | unencrypted | efi system               | 512M            | fat32       |
| sda2      | boot  | unencrypted | linux swap               | 1G              | ext4        |
| sda3      | luks2 | encrypted   | LUKS encrypted partition | Remaining space | ext4        |

Choose the `gpt` partition type.
```sh
lsblk
cfdisk /dev/sda
```

Format partitions.
```sh
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

Set up the encrypted partition.
```sh
cryptsetup --use-random luksFormat /dev/sda3
cryptsetup open /dev/sda3 cryptlvm
```

Create an LVM volume group (`vg0`).
```sh
pvcreate /dev/mapper/cryptlvm
vgcreate vg0 /dev/mapper/cryptlvm
```

Create LVM partitions (logical volumes).
```sh
# lvcreate --size 1000G vg0 -n root
lvcreate -l +100%FREE vg0 -n root
lvreduce --size -256M vg0/root

mkfs.ext4 /dev/vg0/root
```

Check that everything is fine.
```sh
lsblk
```
## System installation

Mount partitions.
```sh
mount /dev/vg0/root /mnt
mount --mkdir /dev/sda1 /mnt/efi
mount --mkdir /dev/sda2 /mnt/boot
```

Install arch system and generate fstab file
```sh
pacstrap -i /mnt base linux linux-firmware archlinux-keyring sudo nano vim git
genfstab -U -p /mnt >> /mnt/etc/fstab
```

Chroot to the installed system.
Since chrooting, you are using the newly installed system as root.
```sh
arch-chroot /mnt /bin/bash
```

### Locales

```sh
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
```
```sh
echo "LANG=en_US.UTF8" > /etc/locale.conf
localectl
```

Set time zone.
```sh
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
hwclock --systohc --utc
date
```

Set hostname.
```sh
echo <HOSTNAME> > /etc/hostname
echo "127.0.1.1       <HOSTNAME>" >> /etc/hosts
```

### Accounts

Set root password.
```sh
passwd
```

Add user.
```sh
useradd -m -g users -G wheel,tty,audio,video -s /bin/bash <USERNAME>
sed -i 's/^# %wheel ALL=(ALL:ALL) ALL$/%wheel ALL=(ALL:ALL) ALL/' /etc/sudoers
passwd <USERNAME>
```

### mkinitcpio

If using the default busybox-based initramfs, add the `keyboard`, `encrypt` and `lvm2` hooks to mkinitcpio.conf **before** 'filesystems'.
If you use a non-US console keymap or a non-default console font, additionally add the `keymap` and `consolefont` hooks, respectively. 

```sh
vim /etc/mkinitcpio.conf
```
```sh
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```
```sh
mkinitcpio -P
```

## Boot

### Bootloader 
Install grub2 with secure boot support.
```sh
pacman -S grub efibootmgr
```
```sh
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/efi --modules="tpm" --disable-shim-lock --removable
```
Get root UUID.
```sh
blkid | grep sda3  
```
Put root UUID in grub config file.
```sh
vim /etc/default/grub

GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID>:cryptlvm"
```
Generate grub config.
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### Secure boot

Install sbctl.
```sh
pacman -S sbctl
```

Verify that you are in setup mode (if not change it in bios).

```sh
sbctl status
```

It should look like this:

```sh
Setup Mode:     ✓ Enabled
```

Create and enroll your custom secure boot keys.
```sh
sudo sbctl create-keys
sudo sbctl enroll-keys -m
```
`-m` is required to include Microsoft's CA certificates

Sign files.
```sh
sbctl sign -s /efi/EFI/GRUB/grubx64.efi
sbctl sign -s /boot/vmlinuz-linux
sbctl sign -s /boot/grub/x86_64-efi/grub.efi
sbctl sign -s /boot/grub/x86_64-efi/core.efi
```

Verify that everything has been signed correctly.

```sh
sudo sbctl verify
```
It should look like this:

```sh
Verifying file database and EFI images in /efi...
✓ /efi/EFI/GRUB/grubx64.efi is signed
✓ /boot/vmlinuz-linux is signed
✓ /boot/grub/x86_64-efi/grub.efi is signed
✓ /boot/grub/x86_64-efi/core.efi is signed
```

## Post installation 

### Set up networking

```sh
pacman -S dhcpcd iwd
systemctl enable dhcpcd.service
systemctl enable iwd.service
```

### Autologin
```sh
mkdir -p /etc/systemd/system/getty@tty1.service.d/
vim /etc/systemd/system/getty@tty1.service.d/override.conf
```
```sh
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin <username> --noclear %I $TERM
```

### Exit

```sh
exit
umount -R /mnt
reboot
```

## Sources

- https://gist.github.com/PHPCore1/c3c99cbbc1989cd4d23ee6a8bc5a3453
- https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS
- https://unix.stackexchange.com/questions/42359/how-can-i-autologin-to-desktop-with-systemd
- https://gist.github.com/mjnaderi/28264ce68f87f52f2cabb823a503e673
- https://averagelinuxuser.com/a-step-by-step-arch-linux-installation-guide/
