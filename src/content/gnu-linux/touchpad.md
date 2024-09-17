# TouchPad

## Contenidos

- [Verificar que TouchPad es detectado por el sistema](#verificar-que-touchpad-es-detectado-por-el-sistema)
- [Ver en logs que el TouchPad es detectado](#ver-en-logs-que-el-touchpad-es-detectado)
- [Ver relación evento con dispositivo](#ver-relación-evento-con-dispositivo)
- [Mostrar pon pantalla los eventos de los dispositivos](#mostrar-pon-pantalla-los-eventos-de-los-dispositivos)
- [Solucionar problemas](#solucionar-problemas)
  - [Dispositivo no es detectado](#dispositivo-no-es-detectado)
  - [TouchPad deja de funcionar](#touchpad-deja-de-funcionar)
- [Recursos](#recursos)

## Verificar que TouchPad es detectado por el sistema

El siguiente comando muestra los dispositivos de entrada:

```bash
$ xinput list

Virtual core pointer                    	id=2	[master pointer  (3)]
  ↳ Virtual core XTEST pointer              id=4	[slave  pointer  (2)]
  ↳ FOO1234:00 1122:4321                   	id=12	[slave  pointer  (2)]
  ↳ SynPS/2 Synaptics TouchPad              id=14	[slave  pointer  (2)]
Virtual core keyboard                   	id=3	[master keyboard (2)]
  ↳ Virtual core XTEST keyboard             id=5	[slave  keyboard (3)]
  ↳ FOO1234:00 1122:4321 Bar               	id=11	[slave  keyboard (3)]
  ↳ AT Translated Set 2 keyboard            id=13	[slave  keyboard (3)]
```

En este ejemplo, el TouchPad corresponde a la línea `SynPS/2 Synaptics TouchPad`.

## Ver en logs que el TouchPad es detectado

Al iniciar el sistema, puede verse que se utiliza `SynPS/2 Synaptics TouchPad`:

```bash
$ less /var/log/Xorg.0.log

[    28.545] (II) config/udev: Adding input device SynPS/2 Synaptics TouchPad (/dev/input/event10)
[    28.545] (**) SynPS/2 Synaptics TouchPad: Applying InputClass "libinput touchpad catchall"
[    28.545] (**) SynPS/2 Synaptics TouchPad: Applying InputClass "touchpad"
[    28.545] (II) Using input driver 'libinput' for 'SynPS/2 Synaptics TouchPad'
[    28.545] (**) SynPS/2 Synaptics TouchPad: always reports core events
[    28.545] (**) Option "Device" "/dev/input/event10"
[    28.548] (II) event10 - SynPS/2 Synaptics TouchPad: is tagged by udev as: Touchpad
[    28.552] (II) event10 - SynPS/2 Synaptics TouchPad: device is a touchpad
[    28.552] (II) event10 - SynPS/2 Synaptics TouchPad: device removed
[    28.595] (**) Option "Tapping" "on"
[    28.596] (**) Option "config_info" "udev:/sys/devices/platform/i8042/serio1/input/input11/event10"
[    28.596] (II) XINPUT: Adding extended input device "SynPS/2 Synaptics TouchPad" (type: TOUCHPAD, id 14)
[    28.600] (**) Option "AccelerationScheme" "none"
[    28.600] (**) SynPS/2 Synaptics TouchPad: (accel) selected scheme none/0
[    28.600] (**) SynPS/2 Synaptics TouchPad: (accel) acceleration factor: 2.000
[    28.600] (**) SynPS/2 Synaptics TouchPad: (accel) acceleration threshold: 4
[    28.602] (II) event10 - SynPS/2 Synaptics TouchPad: is tagged by udev as: Touchpad
[    28.606] (II) event10 - SynPS/2 Synaptics TouchPad: device is a touchpad
[    28.609] (II) config/udev: Adding input device SynPS/2 Synaptics TouchPad (/dev/input/mouse0)
[    28.609] (**) SynPS/2 Synaptics TouchPad: Applying InputClass "touchpad"
[    28.609] (II) Using input driver 'libinput' for 'SynPS/2 Synaptics TouchPad'
[    28.609] (**) SynPS/2 Synaptics TouchPad: always reports core events
[    28.609] (**) Option "Device" "/dev/input/mouse0"
[    28.651] (II) mouse0  - not using input device '/dev/input/mouse0'.
[    28.651] (EE) libinput: SynPS/2 Synaptics TouchPad: Failed to create a device for /dev/input/mouse0
[    28.651] (EE) PreInit returned 2 for "SynPS/2 Synaptics TouchPad"
```

Al deshabilitar y habilitar el TouchPad, los logs son los siguientes:

```bash
$ xinput disable 'SynPS/2 Synaptics TouchPad'

$ less /var/log/Xorg.0.log
[   224.146] (II) event10 - SynPS/2 Synaptics TouchPad: device removed

$ xinput enable 'SynPS/2 Synaptics TouchPad'

$ less /var/log/Xorg.0.log
[   230.953] (II) event10 - SynPS/2 Synaptics TouchPad: is tagged by udev as: Touchpad
[   230.958] (II) event10 - SynPS/2 Synaptics TouchPad: device is a touchpad
```

## Ver relación evento con dispositivo

Cada dispositivo tiene un evento asociado; en los ejemplos de esta entrada al TouchPad le corresponde el evento 10, puede verse con:

```bash
$ sudo libinput list-devices
Device:           SynPS/2 Synaptics TouchPad
Kernel:           /dev/input/event10
...
```

Otra opción para mostrar esta información es en las primeras líneas que da el siguiente comando:

```bash
$ sudo libinput debug-events
...
-event10  DEVICE_ADDED            SynPS/2 Synaptics TouchPad        seat0 default group10 cap:pg  size 93x52mm tap(dl off) left scroll-nat scroll-2fg-edge click-buttonareas-clickfinger dwt-on dwtp-on
...
```

## Mostrar pon pantalla los eventos de los dispositivos

Para ver los eventos generados al pulsar teclas o utilizar el TouchPad y de este modo verificar su correcto funcionamiento, hay que utilizar este comando y se irán mostrando por pantalla los eventos:

```bash
sudo libinput debug-events
```

## Solucionar problemas

### Dispositivo no es detectado

En una ocasión, el archivo `/etc/X11/xorg.conf.d/30-touchpad.conf` impedía que el TouchPad fuera detectado, tuve que eliminar este archivo de configuración.

### TouchPad deja de funcionar

Si por ejemplo, el puntero del ratón queda congelado, la solución puede ser deshabilitar y habilitar el TouchPad, para ello, primero el dispositivo debe ser detectado (explicado cómo verificar esto en anteriores apartados).

Una vez el TouchPad es detectado, puede deshabilitarse y habilitarse:

```bash
$ xinput disable 'SynPS/2 Synaptics TouchPad'
# También puede utilizarse el ID en lugar del nombre (puede verse el ID con el comando `xinput list`):
# $ xinput disable 14

$ xinput enable 'SynPS/2 Synaptics TouchPad'
```

En el apartado que habla de logs generados, se muestran ejemplos de logs creados por estos comandos.

## Recursos

- Documentación Arch Linux

  <https://wiki.archlinux.org/title/Libinput>

- Habilitar y deshabilitar TouchPad

  <https://askubuntu.com/questions/528293/is-there-a-way-to-restart-the-touchpad-driver>

