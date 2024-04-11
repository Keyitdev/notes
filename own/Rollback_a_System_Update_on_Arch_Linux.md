>Author: [Keyitdev](https://github.com/keyitdev)\
Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
Last edited on: 11 Apr 2024

# Rollback a system update on Arch Linux

### Back up mirrorlist

```sh
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
```

### Edit mirrorlist (see available [archives](https://archive.archlinux.org/repos/))

```sh
echo 'Server=https://archive.archlinux.org/repos/2024/04/11/$repo/os/$arch' | sudo tee /etc/pacman.d/mirrorlist
```

### Do a full system upgrade

Double `y` refreshes your package database, double `u` allows pacman to downgrade packages.

```sh
sudo pacman -Syyuu
```

### Issues with Revoked Package Keys

You may encounter issues if you are rolling back any packages to versions which are signed using a signature that has since been removed from the pacman-key gpg database.

To get around this, you can remove affected packages.

```sh
sudo pacman -R package-name
```

Or  set `SigLevel` to `TrustAll` in `/etc/pacman.conf`.

```sh
[custom]
SigLevel = TrustAll
```

**Warning:** The `SigLevel = TrustAll` option exists for debugging purposes and makes it very easy to trust keys that have not been verified. [Package and database signature checking](https://man.archlinux.org/man/pacman.conf.5#PACKAGE_AND_DATABASE_SIGNATURE_CHECKING).

**Change it back when you are done.**

### Sources

https://wiki.archlinux.org/title/Arch_Linux_Archive

https://www.rdeeson.com/weblog/176/how-to-rollback-a-system-update-on-arch-linux

