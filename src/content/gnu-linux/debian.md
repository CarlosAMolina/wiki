# Debian

## Instalación

Crear usb bootable:

1. Insert usb.
2. Unmount usb.
3. `cp debian.iso /dev/sdX`. Importante, utilizar `sdX`, no `sdX1`.

[Link](https://wiki.debian.org/DebianInstaller/CreateUSBMedia).

## Evitar suspensión al cerrar tapa del portátil

Los primeros pasos, como indican estos links (<https://unix.stackexchange.com/questions/563729/looking-for-the-settings-that-causes-debian-to-suspend-when-laptop-lid-is-closed>, <https://github.com/systemd/systemd/issues/11638>, <https://superuser.com/questions/1605504/etc-systemd-logind-conf-is-being-ignored>) son:

Modificar el siguiente archivo:

```bash
sudo vi /etc/systemd/logind.conf
```

Cambiar:

```bash
#HandleLidSwitch=suspend
#HandleLidSwitchExternalPower=suspend
#HandleLidSwitchDocked=suspend
```

por:

```bash
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Tras esto, ejecutar:

```bash
sudo service systemd-logind restart
```

Tras esto, seguimos lo explicado en [este enlace](https://docs.xfce.org/xfce/xfce4-power-manager/faq#how_can_i_make_logind_handle_button_events_instead_of_xfce4-power-manager) obtenido de [este enlace](https://forum.xfce.org/viewtopic.php?id=14016), el siguiente comando debe dar el valor `true`:

```bash
xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/logind-handle-lid-switch
```

Si devuelve `false`, ejecutar:

```bash
xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/logind-handle-lid-switch -n -t bool -s true
```

Si con esto no funciona, configurar `Settings > Power Manager` como se indica en [este enlace](https://forums.debian.net/viewtopic.php?p=797844&sid=665a361d8ec3c3d83177811f881adb08#p797844). Lee el enlace para aplicar todos los cambios, uno de los cambios es:
- System > On battery y Pluggen id: Laptop Lid = Switch off display.
