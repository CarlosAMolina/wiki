# Directorios

## /dev

Contiene los archivos llamados `raw devies` o `device filename`.

Estos archivos son un puente a los discos conectados al host. Es decir, para leer y escribir en los discos, se leen y escriben los raw devices.

Tipos:

- Para los PATA: hdxy
- Para los SATA: sdxy

Donde la `x` tiene los valores `a`, `b`, `c`, etc. y la `y` el valor del número de partición.

Hay un `raw device` por cada device y partición.

Los discos y particiones se detectan durante el proceso de boot. Para detectar hardware (usb, cd...), que se conecte tras el boot, se utiliza el programa `udev`, este puede crear `device files` persistentes, los cuales están en `/dev/disks`.

### /dev/disks

Son links a los archivos en `/dev/` y de este modo identifican de manera única a un dispositivo.

## Recursos

- /dev: obtenido de la teoría de LPCI.
