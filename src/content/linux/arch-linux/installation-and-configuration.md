## Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Keyboard layout](#keyboard-layout)
- [Check boot mode is efi](#check-boot-mode-is-efi)
- [Partitions to use](#partitions-to-use)
- [Install packages](#install-packages)
- [Configure the system](#configure-the-system)
- [Start session](#start-session)
- [Configure network](#configure-network)
- [Create non root user](#create-non-root-user)
- [Add non root user to the sudoers file](#add-non-root-user-to-the-sudoers-file)
- [Configure GUI](#configure-gui)
  - [Language packages](#language-packages)
- [Audio](#audio)
- [Autocompletion](#autocompletion)
  - [Autocomplete make command](#autocomplete-make-command)
  - [Autocomplete git command](#autocomplete-git-command)

## Introduction

Installation steps: <https://wiki.archlinux.org/title/Installation_guide>

The following sections show a summary of the required commands.

## Installation

In the [main installation web page](https://archlinux.org/download/), select a mirror, for example [Spain](https://mirror.es.cdn-perfprod.com/archlinux/iso/2026.07.01/) and download the `.iso` file, for example `archlinux-2026.07.01-x86_64.iso`.

Verify the signature matches the one indicated in the [main installation web page](https://archlinux.org/download/):

```bash
sha256sum ~/Downloads/archlinux-2026.07.01-x86_64.iso
```

Lets [configure the USB](https://wiki.archlinux.org/title/Netboot#Boot_from_a_USB_flash_drive)

- Before plug the USB run `lsblk`, plug the USB and run `lsblk` again, the new name that appears is the USB, for example `sda`.
- Unmount the USB: `sudo umount /dev/sda*`.
- Write the ISO sector by sector, this does not require format the USB: `sudo dd if=~/Downloads/archlinux-2026.07.01-x86_64.iso of=/dev/sda bs=4M status=progress oflag=sync`.
- When finished, run `sync` to ensure all writes to the USB have ended.
- Eject the USB: `sudo eject /dev/sda`.

### MacBook

I am using a MacBook with an Ubuntu partition:

```bash
# Command executed in the Ubuntu partition.
sudo dmidecode -s system-product-name
# MacBookPro9,1 -> Mid 2012 15"
```

With the Mac off:

- Insert Arch USB.
- Hold Option (⌥) while powering on. Key at the left of the space bar.
- Select EFI Boot. Two EFI options appear, i select the one with the USB icon.
- Select Arch Linux install medium.

I want to replace the Ubuntu partition with Arch and maintain the macOS partition.

When we reach the prompt:

```bash
root@archiso ~#
```

If we want to connect via ssh:

```bash
# In Arch
passwd # Write a password.
systemctl start sshd
ip a | grep 192  # Get the ip to connect to.

# In another pc
ssh root@192.168.1.40
```

I it's using an english layout, you can set it to spanish see the `Keyboard layout` section, if not, the `/` key in english is the key `-` and the `-` in english is the key `?` (don't press shift).

Let's see files/directories we're booted in UEFI mode, we're in UEFI mode if the next command shows files/dirs:

```bash
ls /sys/firmware/efi
```

Verify the internet connection (I plugged the ethernet cable):

```bash
ip link
ping 8.8.8.8
```

See current partition layout:

```bash
lsblk -f
fdisk -l
```

The previous commands tell me:

```bash
/dev/sda1    EFI System Partition
/dev/sda2    macOS
/dev/sda3    macOS Recovery
/dev/sda4    Ubuntu (we'll replace it with Arch)
```

Delete Ubuntu and mount a new Arch partition:

```bash
cfdisk /dev/sda
# Select /dev/sda4, Delete and New.
mkfs.ext4 /dev/sda4
mount /dev/sda4 /mnt
```

We won't format the EFI partition, we will use the existing one:

```bash
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
```

Lets start these packages:

```bash
# pacstrap: installs packages into a new Arch system located somewhere else (e.g. /mnt).
pacstrap -K /mnt base linux linux-firmware intel-ucode base-devel networkmanager vim git
```

Generate /etc/fstab (file systems table) to tell Linux the fileystems to mount at boot:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
# We see / mounted on /dev/sda4 and /boot/ on existing /dev/sda1 EFI partition.
cat /mnt/etc/fstab
```

Enter the new Arch system and we are not longer configuring the live USB:

```bash
arch-chroot /mnt  # `root@archiso ~ #` changes to [root@archiso /]#`
```

Configure the Arch system:

```bash
# I'm in Spain. Create a symbolic link that tells Linux your time zone.
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
# Copy the current system time into the hardware clock.
hwclock --systohc
# Language.
vim /etc/locale.gen
# Ucomment these two lines by removing the leading #:
# - en_US.UTF-8 because most documentation, logs, and error messages are in English.
# - es_ES.UTF-8 because it's useful if you want Spanish formatting or applications.
# Generate the locales.
locale-gen
# Create the default locale file. We keep the system language in English to make troubleshooting easier because almost all Linux documentation and forum posts assume English messages.
echo "LANG=en_US.UTF-8" > /etc/locale.conf
# Keyboard layot.
echo "KEYMAP=es" > /etc/vconsole.conf
# Hostname.
echo "macbook" > /etc/hostname
# /etc/hosts
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   macbook.localdomain macbook
EOF
# root password.
passwd
# Create user.
useradd -m -G wheel -s /bin/bash x
passwd x
# Install sudo and configure.
pacman -S sudo
# Remove the `#` in `# %wheel ALL=(ALL:ALL) ALL`, to allow users in the wheel group to use sudo.
# Enable networking at boot.
systemctl enable NetworkManager
```

Make the Mac boot cleanly, we will use GRUB over systemd-boot because detects macOS automatically and is more flexible than systemd-boot for dual-booting.

```bash
pacman -S grub efibootmgr os-prober
# grub: the bootloader.
# efibootmgr: creates UEFI boot entries.
# os-prober: finds macOS automatically.

# Install GRUB into the EFI partition without touching macOS.
grub-install \
    --target=x86_64-efi \
    --efi-directory=/boot \
    --bootloader-id=GRUB
```

If we get this error: `cannot copy `/usr/share/locale/ca/LC_MESSAGES/grub.mo' to `/boot/grub/locale/ca.mo': No space left on device.` is because we mounted the EFI System Partition directly as /boot, but the EFI partition is only 200 MB; GRUB is trying to copy all its modules and translations into the EFI partition, and it runs out of space. Instead of /boot as the EFI partition, it should be:

- /boot: directory on the Arch root filesystem (ext4). The Linux kernel and initramfs live on your large ext4 partition.
- /boot/efi EFI System Partition (FAT32). Only the EFI boot files live on the 200 MB EFI partition.

To fix it:

```bash
umount /boot
mkdir -p /boot/efi
mount /dev/sda1 /boot/efi
# Check with:
df -h /boot  # Mounted on the large Arch partition.
df -h /boot/efi  # On the 200 MB partition.
```

Important, later we will must regenerate fstab: to mount the EFI partition at /boot/efi, not /boot (I didn't verify this step):

```bash
systemctl daemon-reload
```

Reinstall GRUB:

```bash
grub-install \
  --target=x86_64-efi \
  --efi-directory=/boot/efi \
  --bootloader-id=GRUB
```

Enable macOS detection:

```bash
vim /etc/default/grub
# Uncomment `#GRUB_DISABLE_OS_PROBER=false`
```

Generate the configuration

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

If we don't see line similar to `Found Mac OS X` or `Found Darwin`, maybe we need to hold the option key (⌥) at startup to select Mac when booting.

The command `efibootmgr` must show Mac OS X.

Lets finish the installation:

```bash
# Exit chroot.
exit  # `[root@archiso /]#` should change to `root@archiso ~ #`
# Reboot and remove the USB.
umount -R /mnt/
```

If we enter in the GNU GRUB screen with iminimal bash-lie line editing support, is because GRUB started but couldn't find its configuration file, the UEFI is finding grubx64.efi but grubx64.efi this can't locate its modules or grub.cfg.

The fastest way to recover is repeat the first steps:

- Reboot.
- Hold Option (⌥).
- Boot from the Arch USB again.
- Enter the Arch installation like we did in the first steps. Acess the arch-chroot.

```bash
mount /dev/sda4 /mnt
mount /dev/sda1 /mnt/boot/efi
arch-chroot /mnt
grub-install --target=x86_64-efi \
  --efi-directory=/boot/efi \
  --bootloader-id=GRUB \
  --recheck
grub-mkconfig -o /boot/grub/grub.cfg
vim /etc/fstab  # Change `... /boot vfat ...` to `... /boot/efi/ vfat ...`
# Verify.
grep -E '/boot| / ' /etc/fstab
grub-probe /  # Should report ext2
grub-probe /boot  # Should report ext2
exit
umount -R /mnt
reboot
```


## Keyboard layout

<https://wiki.archlinux.org/title/Installation_guide#Set_the_keyboard_layout>

```bash
loadkeys es
```

## Check boot mode is efi

```bash
ls /sys/firmware/efi/efivars
```

If the directory is showed, the boot mode is efi.

## Partitions to use

```bash
# Check partitions to use
fdisk -l
# Example, I will use /dev/sda2 which already has an EFI System and /dev/sda6 to install Linux.

# Format the partitions
mkfs.ext4 /dev/sda6

# Mount file systems
mount /dev/sda6 /mnt/
mount --mkdir /dev/sda2 /mnt/boot
```

## Install packages

```bash
pacstrap -K /mnt base linux linux-firmware
```

## Configure the system

See [system configuration](system-configuration.html).

## Start session

Turn on the pc and write `root` as the user, then write your password.

## Configure network

<https://cmoli.es/wiki/gnu-linux/network.html>

## Create non root user

<https://wiki.archlinux.org/title/Users_and_groups#Example_adding_a_user>

```bash
useradd -m x
passwd x
```

## Add non root user to the sudoers file

<https://wiki.archlinux.org/title/Sudo#Using_visudo>

```bash
pacman -S vi
# Add: x   ALL=(ALL:ALL) ALL
```

## Configure GUI

<https://wiki.archlinux.org/title/Xorg>

```bash
pacman -S xorg-server
# Find driver to install
lspci -v | grep -A1 -e VGA -e 3D
# 01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Caicos XT [Radeon HD 7470/8470 / R5 235/310 OEM] (prog-if 00 [VGA controller])
# 	Subsystem: Micro-Star International Co., Ltd. [MSI] Radeon R5 235 OEM
# Radeon HD 7470/8470 -> TeraScale -> ATI (<https://wiki.archlinux.org/title/Xorg#AMD>):
pacman -S xf86-video-ati
# Install display manager
# https://wiki.archlinux.org/title/LightDM
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm
# Configure Xorg keyboard
#<https://wiki.archlinux.org/title/Xorg/Keyboard_configuration#Setting_keyboard_layout>
#<https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration>
localectl --no-convert set-x11-keymap es
localectl status # check x11 is configured
# Install windows manager
# https://wiki.archlinux.org/title/I3
pacman -S i3-wm
pacman -S xfce4-terminal
reboot
# i3lock
pacman -S i3lock
# Configure i3lock in i3 config file adding:
# ```
# bindsym Control+Mod1+l exec i3lock
# ```
# i3 status bar
pacman -S i3status
# Reload i3 with: shift + alt + r
```

### Language packages

In order to be able to write the `~` character, install:

```bash
# This package was installed while installing Firefox.
pacman -S ttf-dejavu
```

## Audio

<https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture>

Note. During the installation of Firefox, the audio package `jack2` was installed.

```bash
sudo pacman -S alsa-utils
```

If the sound is muted, you can unmute the Master with:

```bash
alsamixer
# Set `Master` volume for example to 50 by pressing the up arrow key and unmute it by pressing the `m` key.
```

Configure keyboard volume control:

```bash
# Comment lines in ~/.config/i3/config `# Use pactl to adjust volume in PulseAudio.` section and use:
bindsym XF86AudioRaiseVolume exec --no-startup-id amixer set Master 5%+ && $refresh_i3status
bindsym XF86AudioLowerVolume exec --no-startup-id amixer set Master 5%- && $refresh_i3status
bindsym XF86AudioMute exec --no-startup-id amixer set Master toggle && $refresh_i3status
```

## Autocompletion

### Autocomplete make command

For example, when using the `make` command, in order to complete options when pressing the tab key, we must install ([link](https://bbs.archlinux.org/viewtopic.php?id=143180)):

```bash
sudo pacman -S bash-completion
```

### Autocomplete git command

For example, to complete git branches, we can install `bash-completion` as we see before. Other option is to source the following script ([link](https://wiki.archlinux.org/title/Git)):

```bash
source ~/.git-completion.bash
```
