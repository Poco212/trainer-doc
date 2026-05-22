# preparation

Ubah mode Secure Boot ke Setup Mode (biasanya dengan memilih opsi "Clear Secure Boot Keys" atau "Reset to Setup Mode")

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
lvcreate -L size (G | M) proc -n home
```

# packages
```
pacstrap /mnt intel-ucode linux-lts linux-lts-headers iwd lvm2 base base-devel neovim openssh superfile podman podman-desktop iptables mpd mpc mpv keepassxc secrets booster efibootmgr networkmanager network-manager-applet
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
echo 'nama_user ALL=(ALL:ALL) ALL' >> /etc/sudoers.d/none
```

## kernel parameter
```
echo "root=/dev/proc/root rw" > /etc/kernel/cmdline
```

## secureboot

## booster
```
nvim /etc/booster.yaml
```
add value
```
network:
  dhcp: on
modules: vfat
compression: zstd
enable_lvm: true
```
```
cd /boot
```
```
/usr/lib/booster/regenerate_images
```
```
bootctl --path=/boot install
```
```
echo "title   Arch Linux" > /boot/loader/entries/arch.conf
```
```
echo "linux   /vmlinuz-linux-lts" >> /boot/loader/entries/arch.conf
```
```
echo "initrd  /intel-ucode.img" >> /boot/loader/entries/arch.conf
```
```
echo "initrd  /booster-linux-lts.img" >> /boot/loader/entries/arch.conf
```
```
echo "options $(cat /etc/kernel/cmdline)" >> /boot/loader/entries/arch.conf
```
```
echo "default  arch.conf" >> /boot/loader/loader.conf
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
