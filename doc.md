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
mount /dev/paritition /mnt/boot
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
```
cryptsetup luksFormat /dev/proc/home
```
```
cryptsetup luksOpen /dev/proc/home system
```
```
mkfs.ext4 /dev/mapper/system
```
```
mkdir /mnt/home
```
```
mount -o rw,nodev,nosuid,relatime /dev/mapper/system /mnt/home
```
# packages
```
pacstrap /mnt intel linux-lts linux-lts-headers iwd lvm2 base base-devel neovim openssh superfile podman podman-desktop iptables mpd mpc mpv keepassxc secrets booster efibootmgr networkmanager network-manager-applet
```
# fstab
```
genfstab -U /mnt > /mnt/etc/fstab
```
# chroot
```
arch-chroot /mnt
```

## jika 1 kata tidak perlu pake `""` kalo lebih menggunakan petik `""`
```
echo [nama komputer] > /etc/hostname
```

## LOCALTIME
```
ln -fs /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```
```
hwclock --systohc
```
****
## LOCALE

```
nvim /etc/locale.gen
```

## lalu pencarian di nvim menggunakan `/`

```
lalu uncommenting kedua en_US
```

### generate bahasa yg di uncommenting 
```
locale-gen
```

```
locale > /etc/locale.conf
```

### config locale
```
nvim /etc/locale.conf
```
### config file locale 
```
isi lang=C menjadi lang=en_US.UTF-8
dan isi ALL=en_US.UTF-8
```
****
## USERADD
```
useradd -m [user]
passwd [user]
```
```
echo 'nama_user ALL=(ALL:ALL) ALL' >> /etc/sudoers.d/none
```
```
usermod -aG wheel [user]
```
****

## KERNEL PARAMETER
```
touch /etc/kernel/cmdline
```

```
echo "rd.luks.name=$(blkid -o UUID -s value /dev/patition_root)=proc root=/dev/proc/root" > /etc/kernel/cmdline
```

## secureboot

## BOOSTER
```
nvim /etc/booster.yaml
```
add value
```
network: true
modules: vfat, ext4, nvme
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
booctl --path=/boot install
```
```
echo "title   Arch Linux Minimal" > /boot/loader/entries/arch.conf
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
echo "options $(cat /etc/kernel/cmdline) rw >> /boot/loader/entries/arch.conf
```
```
echo "default  arch.conf" >> /boot/loader/loader.conf
```
```
touch /etc/vconsole.conf
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
