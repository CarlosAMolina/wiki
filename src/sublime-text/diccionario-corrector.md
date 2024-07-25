Es importante realizar los siguientes apartados para poder editar el diccionario manualmente.

## Contenidos

- [Descargar e instalar un diccionario](#descargar-e-instalar-un-diccionario)
  - [Instalar package control](#instalar-package-control)
  - [Instalar paquete](#instalar-paquete)
  - [Utilizar diccionario](#utilizar-diccionario)
  - [Desinstalar paquete](#desinstalar-paquete)
- [Modificar diccionario](#modificar-diccionario)
- [Activar diccionario](#activar-diccionario)
- [Posibles errores](#posibles-errores)
- [Referencias](#referencias)

## Descargar e instalar un diccionario

### Instalar package control

Menú Tools > Install Package Control...

### Instalar paquete

Abrir el Command Palette:

```bash
control + shift + p
```

En el cuadro que se abre, escribir `install package` y elegir (hacer click) `Package Control: Install Package`

Esperar a que aparezca otro cuadro, escribir `spanish` y elegir `Language - Spanish` (seleccionarlo con las teclas de dirección del teclado y pulsar enter).

En los siguientes pasos se descomprimirá este paquete para obtener la carpeta `Language - Spanish`, por lo que, una alternativa a obtener la carpeta utilizando el paquete, es descargarla directamente desde GitHub:

```bash
git clone https://github.com/CAERMALO/Language_-_Spanish
```

### Utilizar diccionario

- Acceder a la ruta `~/.config/sublime-text/Installed Packages/`.
- Descomprimir el archivo `Language - Spanish.sublime-package`.
- La carpeta descomprimida se llama `Language - Spanish`, moverla a `~/.config/sublime-text/Packages/User/`

Si las rutas anteriores no existen, se puede acceder a ellas desde Sublime haciendo click menú Preferences > Browser Packages ...

### Desinstalar paquete

Ya no es necesario el paquete.

Abrir el Command Palette:

```bash
control + shift + p
```

Escribir `remove package` y elegir `Package Control: Remove Package`.

Click en el paquete a eliminar (`Language - Spanish`). Al cerrar y abrir de nuevo Sublime, ya no estará disponible.

## Modificar diccionario

En `~/.config/sublime-text/Packages/User/Language - Spanish`, modificar archivo `es_ES.dic`.

Tras guardar las modificaciones, cerrar y abrir Sublime para que apliquen los cambios.

Importante. Las nuevas palabras deben ir a partir de la segunda línea, la primera tiene el valor 71165 y no hay que modificarla.

## Activar diccionario

Click menú View > Seleccionar Spell Check

Para seleccionar el idioma: Menú View > Dictionary > User > Language - Spanish.

## Posibles errores

- Si el diccionario no utiliza las nuevas palabras, hay que realizar pruebas hasta dar con el error, por ejemplo, escribir una palabra que si esté en el diccionario, y si la toma como error ortográfico, comprobar que la primera línea en el diccionario es el numero que se indicó antes, reiniciar Sublime, etc.

## Referencias

Instalar package control

<https://packagecontrol.io/installation>

Instalar paquete

<http://www.storybench.org/install-babel-packages-sublime-text-3/>

Paquetes en Sublime

<https://www.sublimetext.com/docs/3/packages.html>

<https://packagecontrol.io/docs/customizing_packages>
