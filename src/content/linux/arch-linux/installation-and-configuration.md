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
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
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

First, lets verify if switch before LightDM solves this.

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

To verify that this works ok, lets investigate the service after a reboot:

```bash
$ systemctl status gpu-switch-intel.service
...
Aug 08 21:39:49 macbook systemd[1]: Starting Switch Apple gmux to Intel and unload nouveau...
Aug 08 21:40:20 macbook systemd[1]: Finished Switch Apple gmux to Intel and unload nouveau.
```

The previous last two lines show that ti takes 31 seconds that is a lot, something is not working correctly.

Reviewing the logs we can see that nouveau tries to disable the GPU but it fails lots of times until is done, so this solution should be improved:

```bash
journalctl -b -k --since "01:39:45" --until "01:40:25" | grep -Ei 'vgaswitcheroo|gmux|nouveau|i915'
```

A solution is to prevent nouveau to be loaded at boot, but this can be dangerous i the system needs it. After an investigation about when nouveau is loaded, I determined that it can be disabled.

See the current mkinitcpio hooks to know if modconf is available to carry a blacklist into the initramfs:

```bash
grep '^HOOKS=' /etc/mkinitcpio.conf
```

It shows:

- kms. This pull modules as i915 and nouveau into the initramfs. We see that mkinitcpio detects them as relevant modules in this machine:
- mdconf. Copies /etc/modprobe.d/*.conf into the initramfs. So will copy a blacklist file that we will create.

```bash
mkinitcpio -M | grep -E '^(i915|nouveau)$'
```

So we can:

- Blacklist nouveau.
- Rebuild initramfs.
- Verify i915 is present and nouveau not.

Process:

```bash
# Create the blacklist file:
echo 'blacklist nouveau' | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
# Rebuild initramfs so that modconf copies the new blacklist into it.
sudo mkinitcpio -P
# Check the blaklist has been embedded in the initramfs.
sudo lsinitcpio /boot/initramfs-linux.img | grep blacklist-nouveau
```

Test first reboot to be safe, reboot in text mode and inspect the GPU state without LightDM to make it simple:

```bash
sudo systemctl set-default multi-user.target
sudo reboot
lsmod | grep nouveau # Should not have output, so we check that the blacklist has been applied.
lsmod | grep i915  # Should show output, so Intel has been initialized correctly without Nouveau.
```

Without Nouveau, the file `sudo cat /sys/kernel/debug/vgaswitcheroo/switch` may disappear, this file is created by vgaswitcheroo which coordinates GPU switching, and gmux delas with Apple's hardware multiplexer to change between the GPUs (Intel and NVIDIA).

```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
cat: /sys/kernel/debug/vgaswitcheroo/switch: No such file or directory
# So vgaswitcheroo/switch is abset and our gpu-switch-intel.service cannot work.
```

As Nouveau is not present, lets see if the Intel GPU is driving the console and not changes are required:

```bash
$ cat /sys/class/graphics/fb0/name
simpledrmdrmfb  # Linux text console is currently drawing into a framebuffer that the firmware prepared during boot. Linux's simpledrm driver can use that already-created framebuffer without needing to use i915 or nouveau as the console framebuffer. We don't know if Intel is driving the physical display, despite we see that it is loaded in the kernel.
$ lspci -k -s 00:02.0
00:02.0 VGA compatible controller: Intel Corporation Ivy Bridge mobile GT2 [HD Graphics 4000] (rev 09)
        Subsystem: Apple Inc. Device 00fb
        Kernel driver in use: i915
        Kernel modules: i915
# That proves the Intel GPU is detected and i915 kernel driver is bound to it.
```

But i915 controlling the Intel GPU does not necessarily mean Apple gmux has routed the physical internal display to Intel. Check what state gmux selected when we booted without Nouveau.

```bash
$ journalctl -b -k | grep -i gmux
Aug 08 22:40:33 macbook kernel: apple_gmux: Found gmux version 1.9.35 [classic]
```

The previous output ony shay that apple_gmux detected the hardware and initialized tits driver, not what GPU is routed to the display. Lets if X can start on Intel without our switch service, as we are in multi-user.target, run:

```bash
$ sudo systemctl start lightdm
A dependency job for lightdm.service failed. See 'journalctl -xe' for details.
```

If the graphical login appears, the firmware/gmux path already leaves the internal panel usable with Intel when Nouveau never loads. In this case i had an error, the reason is the missing file that does not allow the gpu-switch-intel.service to run and is required by LightDM:

```bash
$ systemctl status gpu-switch-intel.service
× gpu-switch-intel.service - Switch Apple gmux to Intel and unload nouveau
     Loaded: loaded (/etc/systemd/system/gpu-switch-intel.service; enabled; preset: disabled)
     Active: failed (Result: exit-code) since Sat 2026-08-08 03:08:03 CEST; 2min 38s ago
 Invocation: 57f7d766f2154a5fa4b84e0bb89016a5
    Process: 931 ExecStartPre=/usr/bin/modprobe i915 (code=exited, status=0/SUCCESS)
    Process: 932 ExecStart=/usr/bin/bash -c      for i in {1..50}; do          test -e /sys/kernel/debug/vgaswitcheroo/switch && break;          sleep 0.2;      done;      test -e>
   Main PID: 932 (code=exited, status=1/FAILURE)
   Mem peak: 2.5M
        CPU: 319ms

Aug 08 23:08:02 macbook bash[1080]: grep: /sys/kernel/debug/vgaswitcheroo/switch: No such file or directory
Aug 08 23:08:02 macbook bash[1082]: grep: /sys/kernel/debug/vgaswitcheroo/switch: No such file or directory
Aug 08 23:08:03 macbook systemd[1]: gpu-switch-intel.service: Main process exited, code=exited, status=1/FAILURE
Aug 08 23:08:03 macbook systemd[1]: gpu-switch-intel.service: Failed with result 'exit-code'.
Aug 08 23:08:03 macbook systemd[1]: Failed to start Switch Apple gmux to Intel and unload nouveau.
```

Lets see if LightDM can start on Intel without the gpu-switch service:

```bash
sudo systemctl disable gpu-switch-intel.service
# Modify LightDM to not need the gpu-switch service.
sudo rm /etc/systemd/system/lightdm.service.d/override.conf
sudo systemctl daemon-reload
# Start LightDM manually.
sudo systemctl start lightdm
```

Once it works, let's verify that the graphical session really is rendering through Intel rather than merely appearing successfully. Run this in the graphical session:

```bash
$ glxinfo -B | grep "OpenGL renderer"
OpenGL renderer string: llvmpipe (LLVM 22.1.8, 256 bits)
```

We have graphical desktop, but without Intel hardware acceleration. Because llvmpipe means Mesa is rendering everything on the CPU in software.

We can see why Xorg did not use i915:

```bash
grep -Ei 'i915|modeset|glamor|dri|drm|\(EE\)|failed' /var/log/Xorg.0.log
```

The logs show that simpledrm is being the primary Xorg device instead of Intel. Lets see the DRM devices:

```bash
$ ls -l /dev/dri/by-path/
total 0
lrwxrwxrwx 1 root root  8 Aug  8 02:40 pci-0000:00:02.0-card -> ../card1
lrwxrwxrwx 1 root root 13 Aug  8 02:40 pci-0000:00:02.0-render -> ../renderD128
lrwxrwxrwx 1 root root  8 Aug  8 02:40 pci-0000:01:00.0-platform-simple-framebuffer.0-card -> ../card0
```

We had:

- PCI 00:02.0 -> Intel HD 4000 -> /dev/dri/card1
- PCI 01:00.0 -> simple framebuffer -> /dev/dri/card0

Lets configure Xorg to use card1:

```bash
sudo mkdir -p /etc/X11/xorg.conf.d

sudo tee /etc/X11/xorg.conf.d/20-intel.conf >/dev/null <<'EOF'
Section "Device"
    Identifier "Intel Graphics"
    Driver "modesetting"
    BusID "PCI:0:2:0"
    Option "PrimaryGPU" "yes"
EndSection
EOF
```

Restart LightDM and re-check glxinfo before reboot:

```bash
sudo systemctl restart lightdm
```

Ups, black screen, lets investigate:

```bash
$ sudo rm /etc/X11/xorg.conf.d/20-intel.conf
$ sudo systemctl restart lightdm
$ grep -Ei 'Intel Graphics|modeset|LVDS|connected|no screens|failed|\(EE\)' /var/log/Xorg.0.log.old  # .old should contain our black-screen attempt.
```

```bash
glxinfo -B | grep "OpenGL renderer"  # It should show something like Mesa Intel(R) HD Graphics 4000 (IVB GT2)
```

It seems the problem is outside Xorg. Let's see if we can switch gmux directly to INtel.

```bash
ls /sys/firmware/efi/efivars/ | grep -i gpu-power-prefs
```

The kernel documentation explains that on these dual-GPU MacBook Pros, apple_gmux can choose the initial GPU from an EFI variable named gpu-power-prefs-fa4ce28d-b62f-4c99-9cc3-6815686e30f9, its 5th byte selects the initial GPU: 1 = IGD (Intel), 0 = DIS (NVIDIA). The firmware then switches gmux and allocates the framebuffer for that GPU before Linux starts.

```bash
ls /sys/firmware/efi/efivars/ | grep -i gpu-power-prefs  # Should have no output. This means the EFI variable is not currently set, so the firmware is falling back to its default GPU choice.
 mount | grep efivarfs  # Should see efivarfs and rw. So Linux has access to EFI variable storage.
journalctl -b -k | grep -Ei 'efi.*(error|fail|warn)|efivar.*(error|fail|warn)'  # To check no EFI problems before write to NVRAM.
sudo journalctl -b -k | grep -iE '\bEFI\b|efivar|efifb' | head -n 30  # Kernel EFI architecture/environment. Check Mac booted in native Apple EFI mode: 'efi: EFI v1.1 by Apple', 'efivars: Registered efivars operations'
sudo ls -l /sys/firmware/efi/efivars > ~/efivars-before.txt  # back up the existing EFI variables directory metadata/list.
command -v efivar  # If shows something like /usr/bin/efivar, we can use efivar to write the value.
# Create a 4-byte Intel payload. Until 4º byte: EFI attributes. 5º byte is 01 -> Use Intel.
printf '\x01\x00\x00\x00' > /tmp/gpu-power-prefs-data.bin
# Verify
od -An -tx1 /tmp/gpu-power-prefs-data.bin  # Must be: 01 00 00 00
# Write the payload.
# fa4ce28d-b62f-4c99-9cc3-6815686e30f9: obtained from the documented Apple gmux interface in the kernel.
sudo efivar --write \
  --name 'fa4ce28d-b62f-4c99-9cc3-6815686e30f9-gpu-power-prefs' \
  --datafile /tmp/gpu-power-prefs-data.bin \
  --attributes 7
# The following command shows 4 bytes, should be 8, so this solution is not correct.
sudo ls -l /sys/firmware/efi/efivars/gpu-power-prefs-*
# Undo the changes
sudo chattr -i /sys/firmware/efi/efivars/gpu-power-prefs-fa4ce28d-b62f-4c99-9cc3-6815686e30f9 2>/dev/null; sudo rm -f /sys/firmware/efi/efivars/gpu-power-prefs-fa4ce28d-b62f-4c99-9cc3-6815686e30f9
```

So forgot about modify the EFI NVRAM and lets try with improve the vgaswitcheroo service, lets check if vgaswitcheroo can switch/power down NVIDIA without immediately unloading Nouveau.

The part that takes 30 seconds is `sudo modprobe nouveau`, lets see if we can omit this part. First, enable again nouveau:

```bash
sudo modprobe nouveau
```

But this don't create the missing file:


```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
cat: /sys/kernel/debug/vgaswitcheroo/switch: No such file or directory
```

The modification should be at boot time commen the line `ExecStartPost=/usr/bin/modprobe -r nouveau` (use #):

```bash
sudo systemctl edit --full gpu-switch-intel.service
reboot
```

With that change we confirm that nouveau can be active but the NVIDIA GPU wont be used and the pc won't be hot:

```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
[sudo] password for x:
0:DIS: :Off:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1
```

If we run `sensors` command, we check that the fans are ok (near 2.000 RPM), and the temperature is not high.

So this is our final config:

```bash
BOOT
 │
 ├─ i915 initializes Intel
 ├─ nouveau initializes NVIDIA
 ├─ apple_gmux registers
 │
 └─ vgaswitcheroo becomes available
          │
          ▼
 gpu-switch-intel.service
          │
          ├─ select IGD
          ▼
 Apple gmux -> Intel
          │
          ├─ Intel -> Pwr + selected
          ├─ NVIDIA -> Off
          └─ NVIDIA Audio -> DynOff
          │
          ▼
       LightDM
          │
          ▼
 XFCE + Intel/crocus acceleration
```

Lets create a cleaner final service:

```bash
sudo systemctl edit --full gpu-switch-intel.service
```

```bash
[Unit]
Description=Switch Apple gmux to Intel graphics
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
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
```

```bash
sudo systemctl daemon-reload
```

Lets improve the service

```bash
sudo cp /etc/systemd/system/gpu-switch-intel.service \
        /etc/systemd/system/gpu-switch-intel.service.working
```

```bash
sudo tee /usr/local/sbin/gpu-switch-intel >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

# This is NOT a normal file stored on disk.
# It is a virtual control/status interface exposed by the Linux kernel
# through debugfs for the vgaswitcheroo subsystem.
# Reading from it asks the kernel for the current GPU state:
#   cat "$SWITCH"
# Writing a command to it asks the kernel to perform an operation:
#   echo IGD > "$SWITCH"
# Therefore ">" here does NOT mean that we are replacing some persistent
# file containing the GPU status. The kernel receives "IGD" as a command.
SWITCH=/sys/kernel/debug/vgaswitcheroo/switch

# Wait up to approximately 10 seconds for vgaswitcheroo to become available.
# During boot, i915, nouveau and apple_gmux need some time to initialize
# and register with vgaswitcheroo.
for _ in {1..50}; do
    [[ -e "$SWITCH" ]] && break
    sleep 0.2
done

# If the kernel interface still does not exist after waiting, fail the
# service instead of continuing with an invalid GPU configuration.
[[ -e "$SWITCH" ]]

# Ask the kernel's vgaswitcheroo subsystem to switch the graphics mux
# to IGD (Integrated Graphics Device), which is our Intel HD 4000.
# Again: "$SWITCH" is a kernel control interface, not an ordinary file.
# The shell sends the characters "IGD\n" to the kernel through that
# interface. The kernel interprets IGD as the GPU-switching command.
# Conceptually:
#   echo IGD > "$SWITCH"
# means:
#   "vgaswitcheroo: switch the display to the integrated GPU"
# It does NOT mean:
#   "replace the GPU status file with the text IGD"
echo IGD > "$SWITCH"

# Wait until the kernel reports that Intel is both:
#   +    selected/active
#   Pwr  powered
for _ in {1..50}; do
    grep -q 'IGD:+:Pwr' "$SWITCH" && exit 0
    sleep 0.2
done

# Intel never reached the expected state, so report failure to systemd.
exit 1
EOF
```

```bash
sudo chmod 755 /usr/local/sbin/gpu-switch-intel
```

```bash
sudo systemctl edit --full gpu-switch-intel.service
```

Set:

```bash
[Unit]
Description=Switch Apple gmux to Intel graphics
After=systemd-modules-load.service
Before=display-manager.service

[Service]
Type=oneshot
ExecStartPre=/usr/bin/modprobe i915
ExecStart=/usr/local/sbin/gpu-switch-intel
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
```

```bash
sudo systemctl daemon-reload
```

Validate the syntax:

```bash
# No output should be shown.
sudo systemd-analyze verify /etc/systemd/system/gpu-switch-intel.service
```

So, we have:

```bash
BOOT
 │
 ├─ nouveau initializes NVIDIA
 ├─ apple_gmux registers
 ├─ i915 becomes available
 │
 └─ vgaswitcheroo becomes available
          │
          ▼
 systemd starts gpu-switch-intel.service
          │
          ├─ ExecStartPre:
          │      modprobe i915
          │
          └─ ExecStart:
                 /usr/local/sbin/gpu-switch-intel
                         │
                         ├─ wait for vgaswitcheroo
                         ├─ write IGD to switch interface
                         └─ verify IGD:+:Pwr
                                  │
                                  ▼
                          Apple gmux -> Intel
                                  │
                                  ├─ Intel -> selected + Pwr
                                  ├─ NVIDIA -> Off
                                  └─ NVIDIA Audio -> DynOff
                                  │
                                  ▼
                               LightDM
                                  │
                                  ▼
                       XFCE + Intel/crocus acceleration
```

Reboot does not work :(, lets investigate, force power off by pressing the power button, after that:

```bash
journalctl -b -1
```

Nouveau causes a problem when shutting the pc down, shutdown makes fbcon interact with Nouveau while DIS is already Off.

Verify that we can turn on and off the GPU while using Intel:

```bash
$ sudo sh -c 'echo ON > /sys/kernel/debug/vgaswitcheroo/switch'
# Verify it turns On.
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
DIS       : Pwr     -> NVIDIA GPU powered on
IGD     + : Pwr     -> Intel still selected and powered
DIS-Audio : DynPwr  -> NVIDIA audio powered dynamically
Check nouveau can see NVIDIA after power it on, it should show realistic info instead of N/A:
$ sensors | sed -n '/nouveau-pci-0100/,+8p'
$ echo OFF > /sys/kernel/debug/vgaswitcheroo/switch
$ sudo sh -c 'echo OFF > /sys/kernel/debug/vgaswitcheroo/switch'
# Verify it turns Off.
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS: :Off:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1
```

Modify the service:

```bash
sudo systemctl edit --full gpu-switch-intel.service
# This service is at  /etc/systemd/system/gpu-switch-intel.service
```

```bash
[Unit]
Description=Switch Apple gmux to Intel graphics
After=systemd-modules-load.service
Before=display-manager.service

[Service]
Type=oneshot
ExecStartPre=/usr/bin/modprobe i915
ExecStart=/usr/local/sbin/gpu-switch-intel
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
```

```bash
sudo systemctl daemon-reload
sudo systemd-analyze verify /etc/systemd/system/gpu-switch-intel.service
```

Before restart, lets test the ExecStop behavior manually while we can still inspect the resulting GPU state:

```bash
$ sudo systemctl stop gpu-switch-intel.service
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS: :Pwr:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
`2:DIS-Audio: :DynPwr:0000:01:00.1` or `2:DIS-Audio: :DynOff:0000:01:00.1`

$ sudo systemctl start gpu-switch-intel.service
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS: :Pwr:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1
```

As we see, our service does not turn off DIS, lets fix this, as we can turn it off with `sudo sh -c 'echo OFF > /sys/kernel/debug/vgaswitcheroo/switch'`, lets add it:

```bash
sudo vim /usr/local/sbin/gpu-switch-intel
```

```bash
#!/usr/bin/env bash
set -euo pipefail

# This is NOT a normal file stored on disk.
# It is a virtual control/status interface exposed by the Linux kernel
# through debugfs for the vgaswitcheroo subsystem.
# Reading from it asks the kernel for the current GPU state:
#   cat "$SWITCH"
# Writing a command to it asks the kernel to perform an operation:
#   echo IGD > "$SWITCH"
# Therefore ">" here does NOT mean that we are replacing some persistent
# file containing the GPU status. The kernel receives "IGD" as a command.
SWITCH=/sys/kernel/debug/vgaswitcheroo/switch

# Wait up to approximately 10 seconds for vgaswitcheroo to become available.
# During boot, i915, nouveau and apple_gmux need some time to initialize
# and register with vgaswitcheroo.
for _ in {1..50}; do
    [[ -e "$SWITCH" ]] && break
    sleep 0.2
done

# If the kernel interface still does not exist after waiting, fail the
# service instead of continuing with an invalid GPU configuration.
[[ -e "$SWITCH" ]]

# Ask the kernel's vgaswitcheroo subsystem to switch the graphics mux
# to IGD (Integrated Graphics Device), which is our Intel HD 4000.
# Again: "$SWITCH" is a kernel control interface, not an ordinary file.
# The shell sends the characters "IGD\n" to the kernel through that
# interface. The kernel interprets IGD as the GPU-switching command.
# Conceptually:
#   echo IGD > "$SWITCH"
# means:
#   "vgaswitcheroo: switch the display to the integrated GPU"
# It does NOT mean:
#   "replace the GPU status file with the text IGD"
echo IGD > "$SWITCH"

# Wait until the kernel reports that Intel is both:
#   +    selected/active
#   Pwr  powered
for _ in {1..50}; do
    grep -q 'IGD:+:Pwr' "$SWITCH" && break
    sleep 0.2
done

# Exit if Intel is not ready (a failed grep will exit thanks
# to `set -euo pipefail` at the top of the script).
grep -q 'IGD:+:Pwr' "$SWITCH"

# Power off the unused discrete GPU.
# This keeps Intel selected, but powers down the NVIDIA GPU.
# On this MacBook, the expected state afterwards is:
#   DIS: :Off
#   IGD:+:Pwr
echo OFF > "$SWITCH"

# Verify that the NVIDIA GPU actually reached the Off state.
for _ in {1..50}; do
    grep -q 'DIS: :Off' "$SWITCH" && exit 0
    sleep 0.2
done

# NVIDIA did not reach the expected Off state, so report failure to systemd.
exit 1
```

```bash
$ sudo systemctl restart gpu-switch-intel.service
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS: :Off:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1
```

This solution was not correct, after reboot, it works but with kernel warnings. Our next solution should be designed around never asking Nouveau to wake the GPU again, rather than trying to repair the shutdown by turning NVIDIA back on.

My boot log says Nouveau creates nouveaudrmfb and makes it the primary fbcon device. If we detach the console from that framebuffer after Intel/Xorg is established, then during reboot there should be no fbcon ->nouveaudrmfb -> dead NVIDIA path to trigger the failure we saw.

Lets force fbcon to use Intel’s fb1 instead of Nouveau’s fb0. The numbers can be checked with:

```bash
$ cat /proc/fb
0 nouveaudrmfb
1 i915drmfb
```

```bash
BOOT
 │
 ├─ nouveau -> fb0
 ├─ i915    -> fb1
 │
 ├─ fbcon configured -> map to fb1 (Intel)
 │
 └─ gpu-switch-intel.service
          │
          ├─ select IGD
          ├─ power NVIDIA OFF
          └─ verify state
                   │
                   ▼
               LightDM/XFCE
                   │
                   ▼
                 Intel

SHUTDOWN
 │
 ├─ LightDM stops
 │
 ├─ fbcon needs to take over
 │
 └─ fbcon -> fb1/i915 ✅
             │
             └─ does NOT touch dead nouveau fb0
```

Verify no `fbcon=` config in :

```bash
cat /proc/cmdline
```

Before editing anything, let’s confirm which bootloader generated that command line. Run:

```bash
if [ -f /boot/grub/grub.cfg ]; then
    echo "GRUB detected"
fi

bootctl status 2>/dev/null | head -n 12
```

If it says `GRUB detected`, we’ll add `fbcon=map:1` to GRUB_CMDLINE_LINUX_DEFAULT, regenerate grub.cfg, and then reboot for the real test.

```bash
$ sudo vim /etc/default/grub
# Replace `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"` with `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet fbcon=map:1"`.
# Regenerate GRUB.
$ sudo grub-mkconfig -o /boot/grub/grub.cfg
# Verify before reboot:
$ sudo grep -n 'fbcon=map:1' /boot/grub/grub.cfg
```

Verify:

```bash
$ sudo reboot
$ cat /proc/cmdline  # Should show ...fbcon=map:1
$ systemctl status gpu-switch-intel.service  # Must show shor process (low CPU ms value) and correct (active (exited)).
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS: :Off:0000:01:00.0
1:IGD:+:Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1

$ sudo journalctl -b -1 -k | grep -iE 'fbcon|nouveau.*(timeout|stalled|inaccessible)|g84_bar_flush|gf119_disp|VGA switcheroo'
# We see: 19:45:58 VGA switcheroo: switched nouveau off.
# If we see `fbcon: nouveaudrmfb (fb0) is primary device` doesn't by itself mean fbcon=map:1 failed. That's reporting Nouveau's framebuffer as the primary framebuffer during initialization; what matters for our shutdown problem is that we no longer see the late fbcon: Taking over console followed by Nouveau failures.
```

Idea: Nouveau is loaded and NVIDIA GPU is off.

#### MacBook. Wifi

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

#### Audio

We will avoid use NVIDIA GPU, currently:

```bash
$ cat /sys/kernel/debug/vgaswitcheroo/switch
DIS:       Off      ← NVIDIA GPU off
IGD:       +:Pwr    ← Intel active
DIS-Audio: DynOff   ← NVIDIA audio runtime-suspended
```

The DIS-Audio suspended, not consuming power while waiting, so is not necessary to turn it off. We will configure audio to not use it.

Concepts:

- PipeWire. The audio engine/server. Moves audio between applications and hardware.
- WirePlumber. The manager for PipeWire. Decides which speakers/microphones to use, routing, etc.
- pipewire-pulse. A PulseAudio compatibility layer. Lets programs designed for PulseAudio (pactl, older applications, etc.) talk to PipeWire.
- PulseAudio. An older audio server. It allowed multiple applications to share audio, control volumes independently, switch outputs, route audio, etc. PipeWire has largely replaced it on modern Linux desktops.
- ALSA (Advanced Linux Sound Architecture). The low-level Linux audio system. Provides kernel drivers and interfaces for communicating with sound hardware.
- pactl. Command to issue control commands to PulseAudio.
- wpctl - WirePlumber Control CLI.
- RTKit (RealtimeKit). A system service that safely grants real-time CPU scheduling priority to applications such as PipeWire, helping prevent audio glitches/dropouts. It does not process or route audio.
- Cirrus codec. An audio chip made by Cirrus Logic that converts audio between digital and analog signals.

Audio flow in

- New apps: App -> PipeWire (managed/configured by WirePlumber) -> ALSA -> hardware
- Old apps: PulseAudio compatible App -> pipewire-pulse -> PipeWire (managed/configured by WirePlumber) -> ALSA -> hardware

Some applications can use the old path and other the new, so we will configure both. With pipewire-pulse the PipeWire system is compatible with apps expecting a PulseAudio server.

Lets check the computer hardware and software to configure the audio.

Hardware (the output contains only a summary of the desired info):

```bash
$ lspci -nnk | grep -A4 -i audio

00:1b.0 Intel HDA. Kernel driver: snd_hda_intel
01:00.1 NVIDIA Corporation GK107 HDMI Audio Controller. Kernel driver: snd_hda_intel
```

We see PulseAudio is not working because pactl speaks the PulseAudio protocol and it fails:

```bash
$ pactl info 2>&1 | head -n 20

Connection failure: Connection refused
pa_context_connect() failed: Connection refused
```

This error is because pipewire-pulse is not running:

```bash
$ systemctl --user --no-pager status pipewire pipewire-pulse wireplumber

- pipewire: running.
- pipewire-pulse: not found.
- wireplumber: running.
- interesing messages:
 - ALSA/WirePlumber discovers PCI 01:00.1, but the NVIDIA side is powered down:
     - ... macbook wireplumber[1530]: spa.alsa: Card can't get card\_name from c…ex 1
     - ... macbook wireplumber[1530]: pa.alsa: Error opening low-level control…tory
  - PipeWire cannot obtain the preferred realtime scheduling privileges through RTKit. Audio may still work, but I'd clean that up as part of a proper Arch audio setup:
    - RTKit error: org.freedesktop.DBus.Error.ServiceUnknown
```

With the following command, we can see the:

- Default sink and source (marked with *) used by WirePlumber.
- wpctl connects to PipeWire (does not depend on PulseAudio) and discovers audio hardware, that means that these pieces are communicating: wpctl -> PipeWire (WirePlumber) -> ALSA devices discovered -> Built-in Audio.

```bash
$ wpctl status
...
W 21:40:22.440607             mod.rt ../pipewire/src/modules/module-rt.c:331:translate_error: RTKit error: org.freedesktop.DBus.Error.ServiceUnknown
...
PipeWire 'pipewire-0' [1.6.8, x@macbook, cookie:3355530220]
 └─ Clients:
        33. WirePlumber
        41. WirePlumber [export]
        62. wpctl
...
Audio
├─ Devices:
│    42. GK107 HDMI Audio Controller
│    43. Built-in Audio
│
├─ Sinks:
│  * 50. Built-in Audio Analog Stereo
│
└─ Sources:
   * 51. Built-in Audio Analog Stereo
```

To verify that them belongs to the PCI 00:1c.0 (Intel), look for properties such as `alsa.card_name`:

```bash
$ wpctl inspect 50
$ wpctl inspect 51
```

Check installed packages and we see that pipewire-pulse and rtkit are missing:

```bash
pacman -Q pipewire wireplumber pipewire-audio pipewire-pulse rtkit 2>&1
```

Install them:

```bash
sudo pacman -S pipewire-pulse rtkit
```

Restart audio stack to allow it to request the new installed software:

```bash
systemctl --user restart pipewire wireplumber
```

Check it works:

```bash
pactl info
```

If it doesn't work, check the socket is `inactive`:

```bash
systemctl --user --no-pager status pipewire-pulse.socket
```

Activate it (run previous command again to verify that it has been activated):

```bash
systemctl --user start pipewire-pulse.socket
```

Not it should work (check this after a reboot to ensure that is activated automatically):

```bash
$ pactl info
...
Server Name: PulseAudio (on PipeWire 1.6.8)
...
```

To check RTKit:

```bash
# See that is working.
systemctl --no-pager status rtkit-daemon
# See no ServiceUnknown warnings after our systemctl restart.
journalctl --user -b -u pipewire -u wireplumber --no-pager | grep -i rtkit
```

Check the speakers work:

```bash
# Verifty Intel is used.
wpctl status | grep "Built-in Audio Analog Stereo"
# Show volume.
wpctl get-volume @DEFAULT_AUDIO_SINK@
# Test.
sudo pacman -S alsa-utils
speaker-test -D pipewire -c 2 -t wav
```

Check the laptop microphone works:

```bash
# See what alsa exposes (for me shows Intel PCH + Cirrus CS4206).
arecord -l
# Test the microphone through PipeWire. Speak to the macbook during 5 seconds after running:
# 48,000 samples/sec × 5 sec = 240,000 samples
pw-record --sample-count=240000 /tmp/mic-test.wav
pw-play /tmp/mic-test.wav
```

Connect headphone with jack an repeat the tests:

```bash
speaker-test -D pipewire -c 2 -t wav
pw-record --sample-count=240000 /tmp/mic-test.wav
pw-play /tmp/mic-test.wav
```

Lets see if `RTKit error: org.freedesktop.DBus.Error.ServiceUnknown` because we installed rtkit:

```bash
# Should be 'active (running)'. If inactive, don't manually enable it, is better to inspect its D-Bus activation because normally it should be started on demand:
systemctl --no-pager status rtkit-daemon
# No new RTKit ServiceUnknown warnings:
# Loaded: ... disabled -> Is ok, because RTKit is designed to be activated on demand through D-Bus, so we don't need to enable it manually.
journalctl --user -b -u pipewire -u wireplumber --since "10 minutes ago" --no-pager | grep -i rtkit
```

We have `DIS-Audio: DynOff`, which means that it is runtime suspended, if we connect something that required NVIDIA HDMI audio, Linux could attempt to wake it, but as I had problems trying to power NVIDIA up (and when it was up the computer temperature increased and the fans were too loud), lets tell WirePlumber to ignore NVIDIA device entirely. After disabling, we won't be able to send audio through the NVIDIA GPU.

First, verify that nothing depends on it.

Its number is `01:00.1`:

```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
...
2:DIS-Audio: :DynOff:0000:01:00.1
```

Check its ID and that is not the default used now, the ID change change since reboot/login:

```bash
$ wpctl status | grep GK107
 │      49. GK107 HDMI Audio Controller         [alsa]
```

See what  PipeWire exposes to verify ID 49 is NVIDIA `01:00.1`:

```bash
$ wpctl inspect 49
# Some output values can confirm it, for example:
# device.name = "alsa_card.pci-0000_01_00.1"
```

See how ALSA has registered the cards as 0 for Intel and 1 for NVIDIA:

```bash
$ cat /proc/asound/cards
 0 [PCH            ]: HDA-Intel - HDA Intel PCH
 ...
 1 [NVidia         ]: HDA-Intel - HDA NVidia
 ...
```

Let's make sure nothing currently has the NVIDIA audio device open:

```bash
$ sudo fuser -v /dev/snd/*
                     USER        PID ACCESS COMMAND
/dev/snd/controlC0:  x           870 F.... wireplumber
/dev/snd/seq:        x           869 F.... pipewire
```

The previous output tell that 2 devices are open:

- `controlC0`. This is the ALSA control interface for sound card 0, that is Intel. The NVIDIA interface is controlC1.
- `seq`. Not related with Intel or NVIDIA, is the global ALSA sequencer interface, opened by Pipewire.

So nothing is using NVIDIA.

Check if the NVIDIA audio PCI function itself is currently runtime-suspended:
```bash
$ cat /sys/bus/pci/devices/0000:01:00.1/power/runtime_status
unsupported
# So it isn't providing a normal active/suspended state on this machine and we cannot use runtime_status to prove that function is suspended.
```

At the end, we won't create extra configuration to ignore NVIDIA because now we see that WirePlumber discovers NVIDIA but that doesn't produce any sink or source:

```bash
$ wpctl status
...
Audio
 ├─ Devices:
 │      49. GK107 HDMI Audio Controller         [alsa]
 │      50. Built-in Audio                      [alsa]
 ├─ Sinks:
 │  *   58. Built-in Audio Analog Stereo        [vol: 0.55]
 │
 ├─ Sources:
 │  *   59. Built-in Audio Analog Stereo        [vol: 1.00]
 │
 ├─ Filters:
 │
 └─ Streams:
...
```

#### Power management

As our GPU setup is special, we'll inspect the current configuration before installing or changing anything.

Which CPU frequency driver Linux is actually using:

```bash
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver

intel_cpufreq
```

This output means that Linux exposes the intel_cpufreq scaling driver and a governor makes the frequency decisions.

To see the governor:

```bash
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
schedutil
```

Schedutil dynamically reacts to actual CPU workload.

This is a correct configuration.

Check the frequency range Linux allows the CPU to use:

```bash
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_{min,max}_freq
1200000
3600000
```

Previous values are in kHz so we have 1.2 GHz minimum and 3.6 GHz maximum. But the 1.2 GHz minimum does not mean the CPU continuously consumes power as though it were actively running at 1.2 GHz. Modern CPUs can enter deep idle C-states where large parts of the core are effectively sleeping. We'll investigate that separately.

What frequencies the CPU is actually reporting right now while the machine is mostly idle:

```bash
$ grep -H . /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq

/sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq:3375105
/sys/devices/system/cpu/cpu1/cpufreq/scaling_cur_freq:3355970
/sys/devices/system/cpu/cpu2/cpufreq/scaling_cur_freq:1200000
/sys/devices/system/cpu/cpu3/cpufreq/scaling_cur_freq:1200000
/sys/devices/system/cpu/cpu4/cpufreq/scaling_cur_freq:1200000
/sys/devices/system/cpu/cpu5/cpufreq/scaling_cur_freq:1730484
/sys/devices/system/cpu/cpu6/cpufreq/scaling_cur_freq:2471035
/sys/devices/system/cpu/cpu7/cpufreq/scaling_cur_freq:2974727
```

A few cores are at the minimum 1.2 GHz, but several are currently much higher, up to ~3.3 GHz. That does not automatically mean there's a power problem. scaling_cur_freq can jump around very quickly, and simply running commands over SSH can wake cores and boost them briefly.

What matters is whether the machine is actually idle and whether the CPU spends most of its time in deep sleep states. Let's check whether anything is actually using CPU right now:

```bash
$ top -b -n1 | head -n 15

$ top -b -n1 | head -n 15
top - 20:58:56 up 30 min,  1 user,  load average: 0.00, 0.02, 0.03
Tasks: 194 total, 2 running, 192 sleep, 0 d-sleep, 0 stopped, 0 zombie
%Cpu(s):  0.5 us,  0.5 sy,  0.0 ni, 96.7 id,  2.3 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7870.3 total,   6895.3 free,    675.6 used,    704.2 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   7194.8 avail Mem
```

In summary we see:

- Load average:  0.00, 0.02, 0.03
- CPU idle:      96.7%
- user CPU:       0.5%
- system CPU:     0.5%

This is ok. Now we will investigate something more important for battery life than instantaneous MHz: CPU idle C-states.

```bash
$ cat /sys/devices/system/cpu/cpu0/cpuidle/state*/name
POLL
C1
C1E
C3
C6
C7
```

The CPU exposes all the useful deep idle states:

- POLL: CPU essentially keeps checking for work (highest idle power)
- C1: light sleep
- C1E: enhanced light sleep
- C3: deeper sleep
- C6: very deep sleep
- C7: deepest available here (lowest idle power)

Difference between:

- Frequency scaling: schedutil chooses frequency from 1.2 GHz to 3.6 GHz
- C-states: cpuidle is the Linux kernel subsystem that chooses C-state from C1 to C7

So seeing a core briefly report 3.3 GHz isn't necessarily bad for battery life. If it completes the work quickly and then spends a long time in C6/C7, power consumption can still be excellent. We need to know now whether the CPU is reaching C6/C7, rather than merely supporting them. To see each state's name and how many times CPU0 has entered it:

```bash
$ grep -H . /sys/devices/system/cpu/cpu0/cpuidle/state*/{name,usage}
/sys/devices/system/cpu/cpu0/cpuidle/state0/name:POLL
/sys/devices/system/cpu/cpu0/cpuidle/state1/name:C1
/sys/devices/system/cpu/cpu0/cpuidle/state2/name:C1E
/sys/devices/system/cpu/cpu0/cpuidle/state3/name:C3
/sys/devices/system/cpu/cpu0/cpuidle/state4/name:C6
/sys/devices/system/cpu/cpu0/cpuidle/state5/name:C7
/sys/devices/system/cpu/cpu0/cpuidle/state0/usage:291
/sys/devices/system/cpu/cpu0/cpuidle/state1/usage:10233
/sys/devices/system/cpu/cpu0/cpuidle/state2/usage:1420
/sys/devices/system/cpu/cpu0/cpuidle/state3/usage:2069
/sys/devices/system/cpu/cpu0/cpuidle/state4/usage:0
/sys/devices/system/cpu/cpu0/cpuidle/state5/usage:67345
```

We have a hight value for C7 comparing to the others, so although we previously saw occasional frequencies around 3 GHz, when the CPU becomes idle it is frequently entering the deepest available C7 state. That's much more relevant to idle power consumption. C6 = 0 means that the CPU uses C7 directly, no problem.

The previous command shows entries, so next let's examine accumulated time in each C-state:

```bash
$ grep -H . /sys/devices/system/cpu/cpu0/cpuidle/state*/{name,time}
/sys/devices/system/cpu/cpu0/cpuidle/state0/name:POLL
/sys/devices/system/cpu/cpu0/cpuidle/state1/name:C1
/sys/devices/system/cpu/cpu0/cpuidle/state2/name:C1E
/sys/devices/system/cpu/cpu0/cpuidle/state3/name:C3
/sys/devices/system/cpu/cpu0/cpuidle/state4/name:C6
/sys/devices/system/cpu/cpu0/cpuidle/state5/name:C7
/sys/devices/system/cpu/cpu0/cpuidle/state0/time:5987
/sys/devices/system/cpu/cpu0/cpuidle/state1/time:221707
/sys/devices/system/cpu/cpu0/cpuidle/state2/time:158176
/sys/devices/system/cpu/cpu0/cpuidle/state3/time:598376
/sys/devices/system/cpu/cpu0/cpuidle/state4/time:0
/sys/devices/system/cpu/cpu0/cpuidle/state5/time:2965784901
```

CPU0 is spending essentially all of its recorded idle time in C7, the deepest available idle state, 2,965,784,901 microseconds (near 2,966 seconds or 49.4 minutes). The other states' duration is low so the GHz consumption was made by work that ends quickly.

We don't need to tune CPU frequency or C-states at all.

Next, we'll check whether some other power-management daemon is already installed/configuring the machine, because we don't want two tools fighting each other:

```bash
systemctl --no-pager --type=service --state=running | grep -Ei 'tlp|power-profiles|thermald|auto-cpufreq|tuned'
```

TODO continue

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
