>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 30 Mar 2024

# Enabling Hibernation in Arch Linux (using swap partition, hibernation to disk)

### Find swap UUID

```sh
sudo lsblk -o FSTYPE,UUID | awk '/swap/ {print $2}'
```

### Edit `/etc/default/grub` and generate grub config

```sh
GRUB_CMDLINE_LINUX_DEFAULT='quiet resume=UUID=YourSwapPartitionUUID'
```

```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Edit `/etc/mkinitcpio.conf` and generate initramfs

Add `resume` **AFTER** `udev`. 

```sh
HOOKS=(base udev resume ...)
```

```sh
mkinitcpio -p linux
```

### Hibernate

```sh
systemctl hibernate
```

#### Other useful commands

```sh
cat /etc/fstab | grep swap
```

```sh
sudo blkid | awk '/swap/ {print $2}'
```

```sh
swapon
```

### Sources

https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation

https://www.baeldung.com/linux/swap-uuid