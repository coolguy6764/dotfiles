Always check Arch Wiki for latest changes. This guide might not work in a newer release 
https://wiki.archlinux.org/index.php/Installation_guide

- [ ] Burn iso into USB
``` bash
dd bs=4M if=path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```

- [ ] Connect ethernet cable to use active internet connection
- [ ] Check EFI boot mode
``` bash
ls /sys/firmware/efi/efivars
```
- [ ] Check internet
``` bash
ping www.google.com
```

- [ ] Update the system clock
``` bash
timedatectl set-ntp true
```

- [ ] Partition the disk (Single disk that is wiped, EFI and EXT4, no swap). Labels are added to simplify bootloader configuration
``` bash
lsblk

parted
(parted) select /dev/sda
(parted) mklabel gpt (!!!! WIPES THE DISK !!!)
(parted) mkpart primary fat32 1MiB 500MiB
(parted) set 1 esp on
(parted) mkpart primary ext4 500MiB 100%
(parted) quit

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2

e2label /dev/sda2 SYSTEM
pacman -S dosfstools
fatlabel /dev/sda1 BOOT

mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

- [ ] Install the base packages
``` bash 
pacstrap /mnt base linux linux-firmware
```

- [ ] Generate fstab with UUIDs and visually check everything is ok
``` bash
genfstab -U /mnt >> /mnt/etc/fstab
nano /mnt/etc/fstab
```

- [ ] Change root into the new system
``` bash 
arch-chroot /mnt
```

- [ ] Set time zone (you can type up to /zoneinfo/ and use tab to see contents instead of using ls in a separate command
``` bash 
ln -sf /usr/share/zoneinfo/Europe/Athens /etc/localtime
```

- [ ] Uncomment and generate locales, create the locale.conf
``` bash
pacman -S nano
nano /etc/locale.gen
locale-gen
echo LANG=en_US.UTF8 > /etc/locale.conf
```

- [ ] Network identifiers for dhcp network connection
``` bash
echo dimitris-t450 > /etc/hostname
echo 127.0.0.1 localhost >> /etc/hosts
echo ::1 localhost >> /etc/hosts
echo 127.0.1.1 dimitris-t450 >> /etc/hosts
```
- [ ] Set root password
``` bash 
passwd
```

- [ ] Install systemd-boot bootloader. Uses partition labels. This might stop working if you change those or change disks
``` bash
pacman -S efivar
efivar --list
bootctl --path=/boot install
rm /boot/loader/loader.conf
echo default arch-* > /boot/loader/loader.conf

pacman -S intel-ucode
echo -e title'\t'Arch Linux > /boot/loader/entries/arch.conf
echo -e linux'\t'/initramfs-linux.img >> /boot/loader/entries/arch.conf
echo -e linux'\t'/intel-ucode.img >> /boot/loader/entries/arch.conf
echo -e linux'\t'options=LABEL=SYSTEM rw >> /boot/loader/entries/arch.conf

reboot
```
- [ ] Login and enable dhcpd
``` bash 
echo -e [Match]'\n'Name=enp0s25'\n\n'[Network]'\n'DHCP=ipv4'\n\n'UseDNS=true'\n'UseDomains=true  > /etc/systemd/network/20-wired.network
systemctl enable systemd-networkd
systemctl enable systemd-resolved
```
- [ ] Create a user, add to groups and uncomment %wheel on visudo
``` bash 
useradd -m dimitris 
passwd dimitris
pacman -S sudo
usermod -aG wheel,audio,video,optical,storage,tty,users dimitris
groups dimitris
pacman -S vi
visudo
```

- [ ] Driver Installation as su
``` bash
pacman -S mesa
```

- [ ] Install desktop
``` bash
pacman -S xorg xorg-server
pacman -S plasma-meta
```

- [ ] Enable dhcp
``` bash
pacman -S dhcpcd
systemctl enable dhcpcd
systemctl start dhcpcd
```
