## Contenidos

- [Comandos a ejecutar](#comandos-a-ejecutar)
- [Añadir Windows al menú de GRUB](#añadir-windows-al-menú-de-grub)
- [Evitar que Windows modifique el menú GRUB](#evitar-que-windows-modifique-el-menú-grub)

## Comandos a ejecutar

El sistema lo configuramos con los siguientes comandos:

```bash
# fstab
genfstab /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
hwclock --systohc
echo "es_ES.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=es_ES.UTF-8" > /etc/locale.conf
echo "KEYMAP=es" > /etc/vconsole.conf
echo "hpPC" > /etc/hostname
mkinitcpio -P # run only if /etc/mkinitcpio.d/linux.preset does not exist
passwd
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
exit
umount -R /mnt
shutdown -h now
```

Hay que extraer el USB antes de iniciar el sistema de nuevo.

Ahora podemos iniciar Arch Linux, pero no aparecerán en el menú de GRUB otros sistemas operativos que puedan estar instalados.

## Añadir Windows al menú de GRUB

El primer paso es verificar que el ESP (EFI system partition) con el Windows Boot Manager (`bootmgfw.efi`) está montado.

Para encontrar dónde está el ESP, ejecutamos el siguiente comando y buscamos la línea con `Tipo` igual a `Sistema EFI`.

```bash
sudo fdisk -l /dev/sda
```

En mi caso se encuentra en `/dev/sda2`, vemos su punto de montaje con:

```bash
lsblk
```

La columna `MOUNTPOINTS` muestra que está mondado en `/boot`. Verifico que contiene el archivo `bootmgfw.efi`:

```bash
find /boot/ -name bootmgfw.efi
# /boot/EFI/Microsoft/Boot/bootmgfw.efi
```

Con esto, pasamos a modificar el archivo principal de configuración `grub.cfg` para añadir Windows al menú de GRUB

Primero, por seguridad hacemos un backup del archivo que vamos a editar:

```bash
cp /boot/grub/grub.cfg ~/Downloads/grub.cfg.bkp
```

Para detectar otros sistemas operativos, tuve que descomentar la línea `GRUB_DISABLE_OS_PROBER=false` del archivo `/etc/default/grub`:

 ```bash
# sudo vi /etc/default/grub` # Uncomment GRUB_DISABLE_OS_PROBER=false
 ```

Detectamos otros sistemas operativos con:

```bash
sudo os-prober
```

Finalmente, generamos el archivo con la configuración que detecte Windows:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Tras reiniciar el equipo, aparecerá el menú GRUB donde seleccionar el sistema operativo a utilizar.

De elegir el sistema operativo Windows, es posible que el menú GRUB ya no aparezca la próxima vez que iniciemos el equipo; de ocurrir esto, en el siguiente apartado vemos cómo solucionarlo.

Recursos utilizados:

- [Buscar ESP](https://wiki.archlinux.org/title/EFI_system_partition).
- [Añadir Windows a menú inicio de GRUB](https://wiki.archlinux.org/title/GRUB).

## Evitar que Windows modifique el menú GRUB

Si iniciamos Windows, seguramente perderemos el menú GRUB que permite elegir qué sistema operativo iniciar; para evitar esto, primero buscamos el número tras `Boot` asignado a Windows:

```bash
sudo efibootmgr
```

En mi caso es el `0007`, tras el cual habrá un `*` indicando que está activado, pasamos a desactivarlo. Desactivarlo solo evita que perdamos el menú de GRUB tras iniciar Windows, Windows seguirá apareciendo en el menú de GRUB:

```bash
sudo efibootmgr --bootnum 0007 --inactive
```

La explicación puede consultarse en este [link](https://askubuntu.com/questions/1110703/how-can-i-protect-grub-from-being-overwritten-by-windows-in-a-dual-boot).

