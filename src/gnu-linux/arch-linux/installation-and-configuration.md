## Contents

- [Introduction](#introduction)
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
