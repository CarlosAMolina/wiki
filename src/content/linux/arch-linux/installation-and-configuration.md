## Contents

- [Introduction](#introduction)
- [Installation](#installation)
  - [MacBook](#macbook)
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
# base: minimal Arch system.
# linux: kernel.
# linux-firmware: firmware for devices.
# intel-ucode: CPU microcode updates for your Intel CPU.
# base-devel: useful build tools.
# networkmanager: easy network management.
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

(We are in arch-chroot). If these command does not show these 3 files:

```bash
# ls -lh /boot
total 158M
drwxr-xr-x 5 root root  512 Jan  1  1970 efi
drwxr-xr-x 6 root root 4.0K Aug  4 00:08 grub
-rw------- 1 root root 128M Aug  4 00:18 initramfs-linux.img
-rw-r--r-- 1 root root  15M May 12 19:27 intel-ucode.img
-rw-r--r-- 1 root root  17M Aug  4 00:08 vmlinuz-linux
```

To get intel-ucode.img (image with microcode updates for Intel CPUs):

```bash
pacman -S intel-ucode
```

Continue:

```bash
grub-install \
  --target=x86_64-efi \
  --efi-directory=/boot/efi \
  --bootloader-id=GRUB \
  --recheck
grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

Enable listen ssh:

```bash
sudo pacman -S openssh
sudo systemctl start sshd
vim /etc/ssh/sshd_config
# Set:
# PasswordAuthentication yes
# PermitRootLogin yes  # If you didn't create a non root user previously.
sudo systemctl restart sshd
```

Configure the system:

```bash
# Update the system.
sudo pacman -Syu
# Install graphics and utilities.
sudo pacman -S mesa mesa-utils intel-ucode linux-firmware
```

Check if the pc is using NVIDIA GPU:

```bash
[root@macbook ~]# sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS:+:Pwr:0000:01:00.0  # DIS:+:Pwr -> NVIDIA is driving the display
1:IGD: :Pwr:0000:00:02.0  # IGD:Pwr -> Intel GPU is powered, but not the display GPU.
2:DIS-Audio: :DynOff:0000:01:00.1
# 0 and 1 tell us that NVIDIA GPU is still the active display GPU.
# See available displays.
lspci -k | grep -A3 -E "VGA|3D"
# 00:02.0 VGA compatible controller: Intel Corporation Ivy Bridge mobile GT2 [HD Graphics 4000] (rev 09)
#         Subsystem: Apple Inc. Device 00fb
#         Kernel driver in use: i915
#         Kernel modules: i915
# --
# 01:00.0 VGA compatible controller: NVIDIA Corporation GK107M [GeForce GT 650M Mac Edition] (rev a1)
#         Subsystem: Apple Inc. Device 00fc
#         Kernel driver in use: nouveau
#         Kernel modules: nouveau
# Intal XFCE
sudo pacman -S \
    xorg \
    xfce4 \
    xfce4-goodies \
    lightdm \
    lightdm-gtk-greeter \
    mesa \
    mesa-utils
# Press enter if asked something like: :: There are ... members in group xorg: ... Enter a selection (default=all):
# Enable the display manager. On the next reboot, LightDM will present a graphical login, and after logging in you'll be in XFCE.
sudo systemctl enable lightdm
reboot
# glxinfo -B # If shows `OpenGL renderer: NVE7` -> uses NVIDIA.
# Determine whether MacBook is using:
# - hardware gmux switching, or
# - muxless Optimus.
cat /sys/class/drm/card*/device/power_state
# D0
# D0
# D0 -> both GPUs are in DO (powered on).
lspci -nn | grep -E "VGA|3D"
ls /sys/class/backlight
# gmux_backlight -> I am using gmux graphics multiplexer, the gmux chip controls the backlight on this Mac This hardware multiplexer selects which GPU drives the internal display.
echo IGD | sudo tee /sys/kernel/debug/vgaswitcheroo/switch
# If no error -> we changed the GPU, the firmware does not lock the GPU selection, good news. Lets see if the changes was accepted.
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
# It should say:
# IGD:+:Pwr
# DIS: :DynOff  # DynOff = Dynamic power management turned it off
# If not, lets continue investigating.
# Logs
sudo dmesg | tail -50 | grep -i -E "gmux|vga|switch|nouveau|i915"
# IGD should switch the display to the integrated GPU only if no userspace process is currently using the GPU
sudo lsof /dev/dri/*
sudo fuser -v /dev/dri/*
sudo fuser -v /dev/snd/*
# Delayed switch mode. DIGD means "switch to the integrated GPU the next time the graphics stack restarts."
# Stop the graphical session.
sudo systemctl isolate multi-user.target
# Check again
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
# 0:IGD:+:Pwr  # Integrated Graphics Device, the Intel HD 4000. + -> is driving the display. Pwr =  powered on.
# 1:DIS: :Off  # Discrete Graphics, your NVIDIA GT 650M.
# Solved! We switched to the Intel GPU.
# Recover the GUI:
sudo systemctl start lightdm  # or: sudo systemctl isolate graphical.target. If not works, reboot.
```

Note. I press the XFCE power off button and it fails, the screen was black but the computer didn't turn off, after debugging, the error was that NVIDIA didn't ends a process, a nouveau issue. Lets fix this by creating a service that changes to Intel.

First, lets verify if swith before LightDM solvers this.

```bash
# Boot to multi-user.target
sudo systemctl set-default multi-user.target
sudo reboot
# IMPORTANT revert this later:
# sudo systemctl set-default graphical.target
# sudo reboot

echo IGD | sudo tee /sys/kernel/debug/vgaswitcheroo/switch
# Verify.
cat /sys/kernel/debug/vgaswitcheroo/switch
# Should show:
# IGD:+:Pwr
# DIS: :Off
# Then start LightDM manually.
sudo systemctl start lightdm
# If XFCE starts and glxinfo -B reports OpenGL renderer string: Mesa Intel HD Graphics 4000, the proven is ok.
glxinfo -B | grep "OpenGL renderer"
# Lets automate it.
sudo vim /etc/systemd/system/gpu-switch-intel.service
```

Paste:

```bash
[Unit]
Description=Switch Apple gmux to Intel GPU before graphical login
After=systemd-modules-load.service
Before=display-manager.service

[Service]
Type=oneshot
ExecStart=/usr/bin/sh -c 'for i in $(seq 1 20); do [ -e /sys/kernel/debug/vgaswitcheroo/switch ] && break; sleep 0.2; done; echo IGD > /sys/kernel/debug/vgaswitcheroo/switch'
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable gpu-switch-intel.service
# Check
sudo systemctl is-enabled gpu-switch-intel.service  # Should show: enabled
sudo systemctl show gpu-switch-intel.service -p Before -p WantedBy
sudo systemctl cat gpu-switch-intel.service
sudo systemctl list-dependencies --before lightdm.service | grep gpu-switch
sudo systemctl show lightdm.service -p Requires -p After  # We should see gpu-switch-intel.service
```

If no output in the last command:

```bash
sudo systemctl edit lightdm.service
```

````bash
[Unit]
Requires=gpu-switch-intel.service
After=gpu-switch-intel.service
````

```bash
sudo systemctl daemon-reload
sudo systemctl show lightdm.service -p Requires -p After  # now we should see gpu-switch-intel.service
```

Lets reboot not shudown to test the new systemd works.

```bash
# Boot to multi-user.target
sudo systemctl set-default multi-user.target
sudo reboot
# IMPORTANT revert this later:
# sudo systemctl set-default graphical.target
# sudo reboot
# The new service won't run in multi user mode, run it manually
sudo systemctl start gpu-switch-intel.service
sudo systemctl status gpu-switch-intel.service
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
# should show
# IGD:+:Pwr
# DIS: :Off
# If ok:
sudo systemctl start lightdm
glxinfo -B | grep "OpenGL renderer"  # should report Intel HD Graphics 4000
```

Power off with the XFCE button.

To avoid errors when shutting down (sometimes nouveau can freeze the shut down), we will disable it. Steps:

- Ensure i915 is loaded.
- Wait for vgaswitcheroo.
- Switch to Intel.
- Wait until Intel is active.
- Unload nouveau.
- Allow LightDM to start. The kernel requires the switch to happen before processes such as Xorg or audio services open the GPU devices, which is why placing this before LightDM is appropriate.

```bash
sudo vim /etc/systemd/system/gpu-switch-intel.service
```

Set:

```
[Unit]
Description=Switch Apple gmux to Intel and unload nouveau
After=systemd-modules-load.service
Before=display-manager.service

[Service]
Type=oneshot
ExecStartPre=/usr/bin/modprobe i915
ExecStart=/usr/bin/bash -c '\
    for i in {1..50}; do \
        test -e /sys/kernel/debug/vgaswitcheroo/switch && break; \
        sleep 0.2; \
    done; \
    test -e /sys/kernel/debug/vgaswitcheroo/switch; \
    echo IGD > /sys/kernel/debug/vgaswitcheroo/switch; \
    for i in {1..50}; do \
        grep -q "IGD:+:Pwr" /sys/kernel/debug/vgaswitcheroo/switch && exit 0; \
        sleep 0.2; \
    done; \
    exit 1'
ExecStartPost=/usr/bin/modprobe -r nouveau
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
```

We need the `lightdm.service` that we created. Without it, Before=display-manager.service in the service only defines ordering. It does not guarantee that your service will actually be started as part of the same boot transaction.

```bash
sudo systemctl daemon-reload
sudo systemctl enable gpu-switch-intel.service
# Deactivate XFCE to avoid black screen
sudo systemctl isolate multi-user.target
sudo systemctl restart gpu-switch-intel.service
sudo systemctl start gpu-switch-intel.service
# Verify
# The following file will dissapear `sudo cat /sys/kernel/debug/vgaswitcheroo/switch` so we run this other command
lspci -k -s 00:02.0  # Intel. Should show: Kernel driver in use: i915
lspci -k -s 01:00.0  # NVIDIA. Should NOT show: Kernel driver in use: i915
lsmod | grep nouveau  # No output should be produced.
sudo systemctl status gpu-switch-intel.service
# If the previous checks are ok:
sudo systemctl start lightdm
# After logging in, verify:
glxinfo -B | grep "OpenGL renderer"  # Should show Intel.
```

Lets configure the Wifi.

Identify the Broadcom chip:

```bash
lspci -nn | grep -i network
# 03:00.0 Network controller [0280]: Broadcom Inc. and subsidiaries BCM4331 802.11a/b/g/n [14e4:4331] (rev 02)
```

We need to install the driver for the PCI ID `14e4:4331`, some options are `b43`and `brcmsmac` which is proprietary, so lets use `b43` and only change to `brcmsmac` if we have stability or performance problems.

Check if `b43` is the driver in use:

```bash
$ lspci -k -s 03:00.0
03:00.0 Network controller: Broadcom Inc. and subsidiaries BCM4331 802.11a/b/g/n (rev 02)
        Subsystem: Apple Inc. AirPort Extreme
        Kernel driver in use: bcma-pci-bridge
        Kernel modules: bcma
```

We have `bcma`, this is not the Wi-Fi driver, is the Broadcom bus driver, that discovers the Broadcom chip and then another driver (like `b43`) should attach to the Wi-Fi core.

Checking the system logs and using Artificial Intelligence to analyze them, I know that everyting is ok in my computer (hardware, PCI, driver and bus) and I only need to install the firmware:

```bash
sudo dmesg | grep -Ei 'b43|bcma|firmware|bcm'
sudo journalctl -k -b | grep -Ei 'b43|bcma|firmware'
```

We find:

```bash
b43-phy0: Broadcom 4331 WLAN found
...
Firmware file "b43/ucode29_mimo.fw" not found
```

Lets see if we have the firmware:

```bash
$ pacman -Qs firmware
...
local/linux-firmware-broadcom 20260622-1
    Firmware files for Linux - Firmware for Broadcom and Cypress network adapters
...
```

The firmware `linux-firmware-broadcom` is installed but this package does not have the proprietary firmware needed by BCM4331, because Broadcom's old firmware wasn't released under a license that allowed redistribution.

Lets install with AUR:

```bash
# base-devel has tools like: make, gcc, patch...
sudo pacman -S --needed base-devel git
# Utility to extract the firmware from the original Broadcom's driver.
sudo pacman -S b43-fwcutter
# Start installation.
git clone https://aur.archlinux.org/b43-firmware.git
makepkg -si
# Verify the firmware exists.
sudo ls /usr/lib/firmware/b43 | head
sudo reboot
ip link  # We should see something like wlp3s0
nmcli device  # We should see something like (disconnected instead of unavailable): wlp3s0   wifi disconnected
nmcli device wifi list  # Scan networks.
# Connect to the SSID: nmcli device wifi connect "YOUR_WIFI_NAME" password "YOUR_PASSWORD"
# The password is stored at sudo cat /etc/NetworkManager/system-connections/{WIFI_NAME}.nmconnection
# Show sotred connection profiles
nmcli connection show
```

If I try to connect to the WiFI using the XFCE WiFi graphical icon, I get the error `Failed to execute command "nm-connection-editor`. Lets fix it:

```bash
which nm-connection-editor  # No output -> no installed.
sudo pacman -S network-manager-applet
```

Install web browser:

```bash
sudo pacman -S firefox
# When asked for:
# - The audio system: 1) jack2  2) pipewire-jack. Select 2) pipewire-jack, is modern and have good compatibility with other software.
# - ttf-font. Select noto-fonts.
```

If we get `The requested URL returned error: 404` errors, usually mean your local package databases reference package versions that the mirrors have already replaced. Refresh repository databases and upgrade the system (the system must be full upgraded to install software) with:

```bash
sudo pacman -Syyu
# Install Firefox again.
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
