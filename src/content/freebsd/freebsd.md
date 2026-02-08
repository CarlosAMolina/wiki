# FreeBSD

## Instalación

En esta sección comparto información para ayudar con el proceso de instalación; el objetivo es tener una referencia que consultar en el futuro.

### Instalación exitosa

Siguiendo estos pasos pude instalar FreeBSD, seguramente haya opciones que no son necesarias, pero escribo aquí de manera exacta lo que utilicé.

Preparar USB:

```bash
# Download
https://download.freebsd.org/releases/ISO-IMAGES/14.3/FreeBSD-14.3-RELEASE-amd64-memstick.img.xz
# Decompress
xz -dv FreeBSD-14.3-RELEASE-amd64-memstick.img.xz
# Send to usb
sudo dd if=FreeBSD-14.3-RELEASE-amd64-memstick.img.xz of=/dev/sdc bs=1M conv=sync status=progress
```

Tras esto, el USB tiene formato FAT32, antes de ejecutar el comando `dd` tenía otro formato por lo que no lo formateé de manera explícita:

```bash
# lsblk  -f
NAME   FSTYPE FSVER LABEL            UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
...
sdc
├─sdc1 vfat   FAT32 EFISYS           ****
├─sdc2
└─sdc5 ufs    2     FreeBSD_Install  ****
```

Importante. Al arrancar el PC con el USB insertado para instalar FreeBSD, desactivé el secure boot de la BIOS del PC.

### Error en la instalación

#### Mounting failed with error 19

Al probar la versión de FreeBSD-15.0-RELEASE-amd64-dvd1.iso, el USB tenía formato `iso...` en lugar de FAT32, al intentar instalarlo obtuve error de montaje con código 19, algo parecido al siguiente texto pero con otras rutas y opción `ro` en lugar de `rw` (obtenido de este [link](https://forums.freebsd.org/threads/mounting-from-ufs-dev-ad0s1a-failed-with-error-19.57135/)):

```bash
Trying to mount root from ufs: /dev/ad0s1a [rw]
mountroot: waiting for device /dev/ad0s1a
Mounting from ufs: /dev/ad0s1a failed with error 19.

Loader variables:
   vfs.root.mountfrom=ufs:/dev/ad0s1a
   vfs.root.mountfrom.options=rw
```

La solución fue recrear el USB, con lo explicado en la sección de instalación exitosa.
