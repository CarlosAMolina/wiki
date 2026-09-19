TODO: move this text to installation-and-configuration.md

#### Configure Graphics

Let's configure the graphics of the system.

##### Initial installations

First, install graphics and utilities:

- mesa. Provides graphics drivers.
- mesa-utils. Provides graphics-testing tools.
- intel-ucode. Provides Intel CPU microcode updates.
- linux-firmware. Provides firmware for hardware devices.

```bash
# Update the system.
sudo pacman -Syu
sudo pacman -S mesa mesa-utils intel-ucode linux-firmware
```

Intal XFCE Desktop environment:

```bash
sudo pacman -S \
    xorg \
    xfce4 \
    xfce4-goodies \
    lightdm \
    lightdm-gtk-greeter 
# Press enter if asked something like: There are ... members in group xorg: ... Enter a selection (default=all)
```

Enable the display manager. On the next reboot, LightDM will present a graphical login, and after logging in you'll be in XFCE:

```bash
sudo systemctl enable lightdm
reboot
```

Despite you configure the desktop keyboard to use Spanish, the login screen of the display manager probably uses English keyboard. This is because the XFCE's keyboard configuration applies after login, is different. To configure to Spanish:

```bash
$ localectl status
System Locale: LANG=en_US.UTF-8
    VC Keymap: es
   X11 Layout: (unset)
# Verify we use LightDM:
$ systemctl status display-manager
# For LightDM, to set the system-wide X11 keyboard layout to Spanish:
$ sudo localectl set-x11-keymap es
$ localectl status
System Locale: LANG=en_US.UTF-8
    VC Keymap: es
   X11 Layout: es
$ sudo systemctl restart lightdm
```

##### Configure the Graphics Processing Units (GPUs)

Lets review the PC GPUs:

```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:DIS:+:Pwr:0000:01:00.0
1:IGD: :Pwr:0000:00:02.0
2:DIS-Audio: :DynOff:0000:01:00.1
```

The meaning is:

- `0:DIS:+`. Entry 0 is the NVIDIA GPU. The `+`  marks the GPU currently selected as the active display device.
- `1:IGD: `: Entry 1 is the Intel GPU. The blank space instead of `+` means the device is not the active display device.
- `:Pwr`. The device is powered, so Intel is powered but is not the active display GPU.
- `DynOff`. Runtime power management has dynamically powered that device off.
- `0000:01:00.0`, `0000:00:02.0` and `0000:01:00.1`. The PCI addresses.

PCIs stands for Peripheral Component Interconnect, which is a standard for connecting peripheral devices to a computer's motherboard in Linux and other operating systems. It allows for the integration of various hardware components, such as graphics cards and network cards, into the system.

See available displays:

```bash
$ lspci -k | grep -A3 -E "VGA|3D"
00:02.0 VGA compatible controller: Intel Corporation Ivy Bridge mobile GT2 [HD Graphics 4000] (rev 09)
        Subsystem: Apple Inc. Device 00fb
        Kernel driver in use: i915
        Kernel modules: i915
...
01:00.0 VGA compatible controller: NVIDIA Corporation GK107M [GeForce GT 650M Mac Edition] (rev a1)
        Subsystem: Apple Inc. Device 00fc
        Kernel driver in use: nouveau
        Kernel modules: nouveau
```

If the MacBook uses the NVIDIA GPU, the temperature of the computer will increase a lot and the fans will make noise due to their speed. We can see the temperature and the fans RPM with the `sensors` command.

I need to use Intel instead of NVIDIA.

There are multiple possibilities to configure the compute to work with Intel instead of NVIDIA. I needed to try different options until get one that works because some were not available on my computer and others raised errors. I will show the final solution now and later a section with the different attemps until get to the correct solution; is a long section but i keep it here for future reference.

###### History of attemps to configure Intel GPU instead of NVIDIA

As is said, this is a long section. It contains my failed attemps to make the configuration until it works. As it contains information about the computer, I keep it here for future reference.

(TODO continue here)

Continue with the analysis:

```bash
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
# If no error -> we changed the GPU, the firmware does not lock the GPU selection, good news. Let's see if the changes was accepted.
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
# It should say:
# IGD:+:Pwr
# DIS: :DynOff  # DynOff = Dynamic power management turned it off
# If not, let's continue investigating.
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

Note. I press the XFCE power off button and it fails, the screen was black but the computer didn't turn off, after debugging, the error was that NVIDIA didn't ends a process, a nouveau issue. Let's fix this by creating a service that changes to Intel.

First, let's verify if switch before LightDM solves this.

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
# Let's automate it.
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

Let's reboot not shudown to test the new systemd works.

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

To verify that this works ok, let's investigate the service after a reboot:

```bash
$ systemctl status gpu-switch-intel.service
...
Aug 08 21:39:49 macbook systemd[1]: Starting Switch Apple gmux to Intel and unload nouveau...
Aug 08 21:40:20 macbook systemd[1]: Finished Switch Apple gmux to Intel and unload nouveau.
```

The previous last two lines show that it takes 31 seconds that is a lot, something is not working correctly.

Reviewing the logs we can see that nouveau tries to disable the GPU but it fails lots of times until is done, so this solution should be improved:

```bash
journalctl -b -k --since "01:39:45" --until "01:40:25" | grep -Ei 'vgaswitcheroo|gmux|nouveau|i915'
```

A solution is to prevent nouveau to be loaded at boot, but this can be dangerous if the system needs it. After an investigation about when nouveau is loaded, I determined that it can be disabled.

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

As Nouveau is not present, let's see if the Intel GPU is driving the console and not changes are required:

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

The previous output only say that apple_gmux detected the hardware and initialized its driver, not what GPU is routed to the display. Let's see if X can start on Intel without our switch service, as we are in multi-user.target, run:

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

Let's see if LightDM can start on Intel without the gpu-switch service:

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

The logs show that simpledrm is being the primary Xorg device instead of Intel. Let's see the DRM devices:

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

Let's configure Xorg to use card1:

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

Ups, black screen, let's investigate:

```bash
$ sudo rm /etc/X11/xorg.conf.d/20-intel.conf
$ sudo systemctl restart lightdm
$ grep -Ei 'Intel Graphics|modeset|LVDS|connected|no screens|failed|\(EE\)' /var/log/Xorg.0.log.old  # .old should contain our black-screen attempt.
```

```bash
glxinfo -B | grep "OpenGL renderer"  # It should show something like Mesa Intel(R) HD Graphics 4000 (IVB GT2)
```

It seems the problem is outside Xorg. Let's see if we can switch gmux directly to Intel.

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

So forgot about modify the EFI NVRAM and let's try with improve the vgaswitcheroo service, let's check if vgaswitcheroo can switch/power down NVIDIA without immediately unloading Nouveau.

The part that takes 30 seconds is `sudo modprobe nouveau`, let's see if we can omit this part. First, enable again nouveau:

```bash
sudo modprobe nouveau
```

But this don't create the missing file:


```bash
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
cat: /sys/kernel/debug/vgaswitcheroo/switch: No such file or directory
```

The modification should be at boot time comment the line `ExecStartPost=/usr/bin/modprobe -r nouveau` (use #):

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

Let's create a cleaner final service:

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

Let's improve the service

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
# interface. The kernel interpret's IGD as the GPU-switching command.
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

Reboot does not work :(, let's investigate, force power off by pressing the power button, after that:

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

Before restart, let's test the ExecStop behavior manually while we can still inspect the resulting GPU state:

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

As we see, our service does not turn off DIS, let's fix this, as we can turn it off with `sudo sh -c 'echo OFF > /sys/kernel/debug/vgaswitcheroo/switch'`, let's add it:

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
# interface. The kernel interpret's IGD as the GPU-switching command.
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

Let's force fbcon to use Intel’s fb1 instead of Nouveau’s fb0. The numbers can be checked with:

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
