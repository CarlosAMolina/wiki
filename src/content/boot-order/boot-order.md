## Introducción

En esta sección resumo cómo cambiar el orden de arranque de distintos equipos.

## HP Pavilion 500 sobremesa

[Ejemplo imagen](https://support.hp.com/es-es/product/product-specs/hp-pavilion-500-200-desktop-pc-series/model/6846587).

Para acceder al menú donde cambiar el orden de arranque, durante el inicio del PC, pulsar la tecla `F10`.

## HP Pavilion 11-k101ns

[Ejemplo imagen](https://www.fnac.es/Convertible-2-en-1-HP-Pavilion-x360-11-k101ns-Ordenador-portatil-PC-Portatil/a1155195).

Para acceder al menú donde cambiar el orden de arranque, durante el inicio del PC, pulsar la tecla `F10`.

Acceder a System Configuration > Boot Options > UEFI Boot Order: OS boot Manager. Pulsar tecla enter para que aparezca lista donde seleccionar sistema operativo.

## HP ZBook 14u G6

[Ejemplo imagen](https://support.hp.com/mx-es/document/c06337099).

Para acceder al menú donde cambiar el orden de arranque, durante el inicio del PC, pulsar la tecla `esc` (tecla escape, situada arriba a la izquierda del teclado) ([link explicación](https://tecnobits.com/como-iniciar-la-bios-en-un-hp-zbook/)). Importante, no mantener presionada la tecla, sino pulsarla con toques breves y repetidos hasta que aparezca el menú donde cambiar la configuración.

Nota. Si aparece el menú GRUB en lugar de Bios, escribir `fwsetup` ([link](https://askubuntu.com/questions/318796/when-trying-to-enter-bios-gnu-grub-screen-appears)), pulsar enter y esperar a que aparezca el Startup Menú con varias opciones.

Puede configurarse el orden de arranque en:

- Opción 1. Boot Menu (F9)
- Opción 2. BIOS Setup (F10). Advanzed > Boot Options > UEFI Boot Order.

### Instalar sistema operativo

El Secure Boot debe estar desactivado, se hace desde el menú de la BIOS.

## MacBook Pro (15 pulgadas, mediados de 2012)

[Ejemplo imagen](https://support.apple.com/es-es/HT201300). Buscar, Identificador del modelo: MacBookPro9,1.

Mantener presionada la tecla opción (tecla `alt` abajo a la izquierda) y encender el portátil ([enlace explicación](https://support.apple.com/es-es/guide/mac-help/mchlp1034/14.0/mac/14.0)).

### GRUB

Si en lugar de la pantalla de configuración de la BIOS accedemos a la pantalla de GRUB, salimos de ella escribiendo `exit` y pulsando enter y durante el arranque, accedemos a la configuración de la BIOS pulsando la tecla F10 (no sé si es necesario que esté presionada/activa la tecla `fn`).

## Secure boot request

Si tras cambiar las opciones de arranque, al iniciar el equipo aparece un mensaje indicando:

> A request has been made to change this system's secure boot configuration wich may affect the secure boot keys and/or may disable secure boot.
> Please type in and enter the below number fo authorization

Lo que debemos hacer es escribir el número que aparece y pulsar enter.
