>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 9 Sep 2023

# KVM, QEMU and Virt Manager installation on Arch Linux

### Install packages

```bash
sudo pacman -S qemu-full virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat dmidecode
```

```bash
sudo pacman -S ebtables iptables
```

### Enable the libvirtd service

```bash
sudo systemctl start libvirtd.service
sudo systemctl enable libvirtd.service
```

### Configure the libvirtd service

Uncomment `unix_sock_group = "libvirt"` and `unix_sock_rw_perms = "0770"` in `/etc/libvirt/libvirtd.conf`.

```bash
sudo sed -i 's/^#unix_sock_group = "libvirt"/unix_sock_group = "libvirt"/' /etc/libvirt/libvirtd.conf
sudo sed -i 's/^#unix_sock_rw_perms = "0770"/unix_sock_rw_perms = "0770"/' /etc/libvirt/libvirtd.conf
```

### Add user to libvirt group

```bash
sudo usermod -aG libvirt $(whoami)
```

### Restart the libvirtd service

```bash
sudo systemctl restart libvirtd.service
```

### Start virtual network

Run this command every time you use a virtual machine that uses the Internet.

```bash
sudo virsh net-start default
```

### Sources

https://computingforgeeks.com/install-kvm-qemu-virt-manager-arch-manjar/

https://www.makeuseof.com/how-to-install-arch-linux-kvm-configure-virtual-machine/
