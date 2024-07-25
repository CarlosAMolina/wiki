# VirtualBox

## Contenidos

- [Botones](#botones)
- [Compartir carpetas entre VBox y host anfitrión](#compartir-carpetas-entre-vbox-y-host-anfitrión)
  - [Compartir archivos](#compartir-archivos)
    - [Máquina virtual con Gnu/Linux](#máquina-virtual-con-gnulinux)
    - [Máquina virtual con Windows](#máquina-virtual-con-windows)
  - [Posibles errores](#posibles-errores)
    - [Paso inicial](#paso-inicial)
    - [Archivos perdidos](#archivos-perdidos)

## Botones

- HOSTS: `control` (el de la derecha) + `c`

## Compartir carpetas entre VBox y host anfitrión

### Compartir archivos

Para pasar archivos entre la máquina virtual y el host anfitrión, los movemos a la carpeta del OS emulado en VBox donde se vea lo compartido con el anfitrión.

#### Máquina virtual con Gnu/Linux

Tutorial: <https://www.youtube.com/watch?v=ddExu55cJOI>

Pasos:

1. Utilizar Gest Additions: menú VirtualBox (desde máquina virtual iniciada) > Dispositivos > insertar la imagen de CD de las "Gest Additions".
2. Crear una carpeta en el OS corriendo en la máquina virtual.
3. Compartir carpeta desde interfaz VBox.
4. En terminal de la máquina virtual:

```bash
sudo mount -t vboxsf NOMBRE_CARPETA_EN_HOST_ANFITRIÓN RUTA_CARPETA_EN_MÁQUINA_VIRTUAL
# Ejemplo: sudo mount -t vboxsf puenteWindows /root/Escritorio/puenteLinux
```

Tras esto, todo lo que haya en la carpeta compartida del host anfitrión aparece en la carpeta utilizada en la máquina virtual, y viceversa.

Notas:

- La carpeta compartida en el host anfitrión, debe estar antes compartida en configuración de VBox. Fuera de la máquina corriendo, en la interfaz de VBox, click en `Configuración > Carpetas compartidas` y seleccionamos la deseada.
- El NOMBRE_CARPETA_EN_HOST_ANFITRIÓN es el nombre en que termine la ruta compartida utilizada en la configuración de compartir carpetas de VirtualBox
- Es importante seleccionar una carpeta vacía que compartir ya que lo que haya en RUTA_CARPETA_EN_MÁQUINA_VIRTUAL será sustituido por el contenido de NOMBRE_CARPETA_EN_HOST_ANFITRIÓN.

#### Máquina virtual con Windows

Tras crear la carpeta compartida, en la VM Windows buscar en la red (está en el menú izquierdo que aparece al abrir una carpeta donde salen listados otros directorios, mi ordenador, etc) la carpeta que se ha compartido.

### Posibles errores

#### Paso inicial

El primer paso para solucionarlos es actualizar VBox, el OS emulado, etc.

#### Archivos perdidos

En caso de no haber compartido una carpeta vacía y hayan desaparecido los archivos, se soluciona apagando VBox y cerrando la aplicación de VBox. Es decir, dejarlo todo como antes de compartir carpetas, no sé si es necesario quitar la carpeta compartida de la configuración de la interfaz de VBox. Al encender todo estará como al principio.
