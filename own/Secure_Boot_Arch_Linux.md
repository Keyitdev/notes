> **Made by [Keyitdev](https://github.com/keyitdev)**\
> Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
>Last edited on 4 May 2024

# Setting up Secure Boot (UEFI, GRUB)

## Pre-requisites

### Boot in setup mode
In your UEFI settings, set your secure boot mode to setup mode. 
If you have enrolled keys, simply delete them.
### Install required packages

```sh
sudo pacman -S grub efibootmgr sbctl
```

## Configure GRUB

### Mount boot partition

Check where your boot partition is.

```sh
lsblk
```

Mount your partition to the EFI directory.

```sh
sudo mount /dev/sda1 /boot/efi
```

### Install (or reinstall) GRUB

Replace `--efi-directory=/boot/efi` with your EFI system partition.

```sh
sudo grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --removable --modules="tpm" --disable-shim-lock
```

`--disable-shim-lock`  disables shim_lock verifier\
`--modules="tpm"` enables tmp

[Grub certification authority keys](https://wiki.archlinux.org/title/GRUB#CA_Keys)

### Regenerate your grub configuration

```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Umount boot partition

```sh
sudo umount /boot/efi
```

## Configure sbctl

### Verify that you are in setup mode

```sh
sbctl status
```

It should look like this:

```sh
Installed:      ✓ sbctl is installed
Setup Mode:     ✗ Enabled
Secure Boot:	✗ Disabled
Vendor Keys:	none
```

### Create and enroll your custom secure boot keys
```sh
sudo sbctl create-keys
sudo sbctl enroll-keys -m
```

`-m` is required to include Microsoft's CA certificates

### Sign usigned files 

Check which files need to be signed for secure boot to work.
```sh
sudo sbctl verify
```

I recommend to always sing bootloader and kernel.
```sh
sudo sbctl sign -s /efi/EFI/GRUB/grubx64.efi
sudo sbctl sign -s /boot/vmlinuz-linux
```

Verify that everything has been signed correctly.

```sh
sudo sbctl verify
```

It should look like this:

```sh
Verifying file database and EFI images in /efi...
✓ /boot/vmlinuz-linux is signed
✓ /efi/EFI/BOOT/BOOTX64.EFI is signed
```
## Post-requisites

### Enable secure boot

Reboot and in the UEFI settings enable secure boot.

### Check if everything worked

```sh
sbctl status
```

It should look like this:

```sh
Installed:      ✓ sbctl is installed
Setup Mode:     ✓ Disabled
Secure Boot:    ✓ Enabled
Vendor Keys:    microsoft
```

## Troubleshooting

You may get an error because of an issue with certain files being immutable. To make those files mutable, run the following command for each file then re-sign afterwards.

```sh
sudo chattr -i /sys/firmware/efi/efivars/<filename> 
```

## Sources

https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Implementing_Secure_Boot

https://www.reddit.com/r/archlinux/comments/10pq74e/my_easy_method_for_setting_up_secure_boot_with/

https://wiki.archlinux.org/title/GRUB#CA_Keys