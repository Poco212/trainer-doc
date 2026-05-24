
# partition
```
pvcreate /dev/partition
```
```
vgcreate proc /dev/partition
```

## root
```
lvcreate -L size (G | M) proc -n root
```
```
mkfs.ext4 /dev/proc/root
```
```
mount /dev/proc/root /mnt
```

## boot
```
mkfs.vfat -F32 -n BOOT /dev/partition
```
```
mkdir /mnt/boot
```
```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/paritition /mnt/boot
```


## var
```
lvcreate -L size (G | M) proc -n vars
```
```
mkfs.ext4 /dev/proc/vars
```
```
mkdir /mnt/var
```
```
mount -o rw,nodev,nosuid,relatime /dev/proc/vars /mnt/var
```


## vtmp
```
lvcreate -L size (G | M) proc -n vtmp
```
```
mkfs.ext4 /dev/proc/vtmp
```
```
mkdir /mnt/var/tmp
```
```
mount -o rw,nodev,nosuid,noexec,relatime /dev/proc/vtmp /mnt/var/tmp
```

## vlog
```
lvcreate -L size (G | M) proc -n vlog
```
```
mkfs.ext4 /dev/proc/vlog
```
```
mkdir /mnt/var/log
```
```
mount -o rw,nodev,nosuid,noexec,relatime /dev/proc/vlog /mnt/var/log
```

## vaud
```
lvcreate -L size (G | M) proc -n vaud
```
```
mkfs.ext4 /dev/proc/vaud
```
```
mkdir /mnt/var/log/audit
```
```
mount -o rw,nodev,nosuid,noexec,relatime /dev/proc/vaud /mnt/var/log/audit
```

## temp
```
lvcreate -L size (G | M) proc -n temp
```
```
mkfs.ext4 /dev/proc/temp
```
```
mkdir /mnt/tmp
```
```
mount -o rw,nodev,nosuid,noexec,relatime /dev/proc/temp /mnt/tmp
```

## home
```
lvcreate -l100%FREE proc -n home
```

## setup luks partition home

```
cryptsetup luksFormat /dev/proc/[nama user]
```

```
cryptsetup luksOpen /dev/mapper[nama user] [nama device]
```

```
mkfs.ext4 /dev/mapper/cadel
```

# packages
```
pacstrap /mnt intel-ucode linux-lts linux-lts-headers iwd lvm2 base base-devel neovim openssh superfile podman podman-desktop iptables mpd mpc mpv keepassxc secrets booster efibootmgr networkmanager sbctl systemd-ukify
```
# fstab
```
genfstab -U /mnt > /mnt/etc/fstab
```
# chroot
```
arch-chroot /mnt
```

## hostname
```
echo [nama komputer] > /etc/hostname
```

## localtime
```
ln -fs /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```
```
hwclock --systohc
```

## locale

```
nvim /etc/locale.gen
```
uncommentin both `en_US.UTF-8`
```
en_US.UTF-8
en_US.ISO-8859-1
```
```
locale-gen
```

```
locale > /etc/locale.conf
```
```
nvim /etc/locale.conf
```
add value like below
```
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
```

## useradd
```
useradd -m username
```
```
passwd username
```
```
echo 'nama_user ALL=(ALL:ALL) ALL' > /etc/sudoers.d/none
```

## booster
```
nvim /etc/booster.yaml
```
add value
```
network:
  dhcp: on
universal: false
modules: -*, ext4,lvm2
extra_files: fsck,fsck.ext4
strip: true
enable_lvm: true
```
```
cd /boot
```
```

```
```
booster build --kernel-version <version> /boot/booster-linux-lts.img
```
## systemd-boot
```
bootctl --path=/boot install
```
```
nvim /boot/loader/entries/booster.conf
```
```
title    arch with booster
linux    /vmlinuz-linux-lts
initrd   /intel-ucode.img
initrd   /booster-linux-lts.img
options  root=/dev/proc/root rw
```
```
nvim /boot/loader/loader.conf
```
tambahkan paling bawah

```
default  booster.conf
```
## booting
```
exit
```
```
umount -R /mnt
```
```
reboot
```
