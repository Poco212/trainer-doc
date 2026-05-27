
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
mount --mkdir -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/paritition /mnt/boot
```


## var
```
lvcreate -L size (G | M) proc -n vars
```
```
mkfs.ext4 /dev/proc/vars
```
```
mount --mkdir -o rw,nodev,nosuid,relatime /dev/proc/vars /mnt/var
```


## vtmp
```
lvcreate -L size (G | M) proc -n vtmp
```
```
mkfs.ext4 /dev/proc/vtmp
```
```
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/proc/vtmp /mnt/var/tmp
```

## vlog
```
lvcreate -L size (G | M) proc -n vlog
```
```
mkfs.ext4 /dev/proc/vlog
```
```
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/proc/vlog /mnt/var/log
```

## vaud
```
lvcreate -L size (G | M) proc -n vaud
```
```
mkfs.ext4 /dev/proc/vaud
```
```
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/proc/vaud /mnt/var/log/audit
```

## home public
```
lvcreate -L size (G | M) proc -n home
```
```
mkfs.ext4 /dev/proc/home
```
```
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/proc/home /mnt/home
```

## home internal
```
lvcreate -l50%FREE proc -n priv
```
```
cryptsetup luksFormat /dev/proc/priv
```

# packages
```
pacstrap /mnt intel-ucode linux-lts linux-lts-headers linux-firmware lvm2 base base-devel neovim openssh superfile podman podman-desktop iptables mpd mpc mpv keepassxc secrets booster networkmanager pam_mount
```
# fstab
```
genfstab -U /mnt > /mnt/etc/fstab
```

# tmpfs

```
echo "tmpfs /tmp tmpfs defaults,rw,nosuid,nodev,noexec,relatime,size=1G 0 0" >> /mnt/etc/fstab
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

## pam_mount
```
cryptsetup luksOpen /dev/proc/priv internal
```

```
mkdir -p /home/user
```

```
useradd -d /home/user user_name
```

```
chown -R user_name:user_name /home/user
```

```
passwd user
```
> password must same like luks for this partition

```
echo 'nama_user ALL=(ALL:ALL) ALL' > /etc/sudoers.d/none
```
                                                                                 
### Configure the Volume

```
nvim /etc/security/pam_mount.conf.xml
```
> adjust like the lines below
```/etc/security/pam_mount.conf.xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE pam_mount SYSTEM "pam_mount.conf.xml.dtd">
<!--
	See pam_mount.conf(5) for a description.
-->

<pam_mount>

		<!-- debug should come before everything else,
		since this file is still processed in a single pass
		from top-to-bottom -->

<debug enable="0" />

		<!-- Volume definitions -->


		<!-- pam_mount parameters: General tunables -->

<!--
<luserconf name=".pam_mount.conf.xml" />
-->

<!-- Note that commenting out mntoptions will give you the defaults.
     You will need to explicitly initialize it with the empty string
     to reset the defaults to nothing. -->
<mntoptions allow="nosuid,nodev,loop,encryption,fsck,nonempty,allow_root,allow_other" />
<!--
<mntoptions deny="suid,dev" />
<mntoptions allow="*" />
<mntoptions deny="*" />
-->
<mntoptions require="nosuid,nodev" />

<!-- requires ofl from hxtools to be present -->
<logout wait="0" hup="no" term="no" kill="no" />

<!-- Example entry for a LUKS partition -->
<volume 
    user="[user name]" 
    fstype="crypt" 
    path="/dev/proc/priv" 
    mountpoint="/home/user" 
/>
		<!-- pam_mount parameters: Volume-related -->

<mkmountpoint enable="1" remove="true" />


</pam_mount>
```

Edit the `pam_mount` configuration file at `/etc/security/pam_mount.conf.xml`. You need to add a `<volume>` entry for your encrypted device.

```xml
<!-- Example entry for a LUKS partition -->
<volume 
    user="[user name]" 
    fstype="crypt" 
    path="/dev/proc/priv" 
    mountpoint="/home/user" 
/>
```
*   **path:** The identifier for your encrypted partition (e.g., `/dev/sdb1` or a UUID).
*   **mountpoint:** Where the partition should be accessible after unlocking.
*   **options:** `allow-discard` is useful for SSD performance.

### Update PAM Configuration

```
nvim /etc/pam.d/system-login
```
> adjust like the lines below
```/etc/pam.d/system-login
#%PAM-1.0

auth       required   pam_shells.so
auth       requisite  pam_nologin.so
auth       include    system-auth
auth       required   pam_mount.so

account    required   pam_access.so
account    required   pam_nologin.so
account    include    system-auth

password   include    system-auth

session    optional   pam_loginuid.so
session    optional   pam_keyinit.so       force revoke
session    include    system-auth
session    optional   pam_lastlog2.so      silent
session    optional   pam_motd.so
session    optional   pam_mail.so          dir=/var/spool/mail standard quiet
session    optional   pam_umask.so
session    optional  pam_mount.so
-session   optional   pam_systemd.so
session    required   pam_env.so
```

You must tell the system to use `pam_mount` during the login process. Edit `/etc/pam.d/system-login` to include the following lines in the correct sections:

```/etc/pam.d/system-login
# Add to the 'auth' section
auth        required    pam_mount.so

# Add to the 'session' section
session     optional    pam_mount.so
```
*Note: If you use a Display Manager (like GDM or SDDM), ensure its specific PAM file also includes these or inherits from `system-login`.*


## booster
```
nvim /etc/booster.yaml
```
add value
```
network:
  dhcp: on
universal: false
modules: -*,ext4,(tambahain nvme jika laptop menggunakan nvme)
extra_files: fsck,fsck.ext4
strip: true
enable_lvm: true
```
```
cd /boot
```
for cek kernel version
```
ls /usr/lib/modules
```
```
booster build --kernel-version <version> /boot/booster-linux-lts-new.img
```
```
rm -fr booster-linux-lts.img
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
initrd   /booster-linux-lts-new.img
options  root=/dev/proc/root rw
```
```
nvim /boot/loader/loader.conf
```
tambahkan paling bawah

```
default  booster.conf
```
```
bootctl --graceful update 
```
## desktop
```
pacman -S xfce4 sddm pipewire pipewire-pulse pipewire-alsa pipewire-jack network-manager-applet
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

## after booting
```
sudo chown -R nama_user:nama_user /home/[nama user]
```
```
sudo systemctl enable sddm
```
```
reboot
```
