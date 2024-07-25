## Contenidos

- [Instalar add-on](#instalar-add-on)
- [Desarrollar add-on](#desarrollar-add-on)
  - [web-ext](#web-ext)
    - [Ejecutar extensión en navegador y recargar automáticamente con cada cambio](#ejecutar-extensión-en-navegador-y-recargar-automáticamente-con-cada-cambio)
- [Archivos](#archivos)
  - [manifest.json](#manifest.json)
  - [icons](#icons)
  - [file.js](#file.js)
- [Editar add-on en la web de extensiones](#editar-add-on-en-la-web-de-extensiones)
- [Recursos](#recursos)

## Instalar add-on

Acceder en la barra de URLs del navegador Web a `about:debugging` > `Este Firefox` y elegir `Cargar complemento temporal...`, tras esto, seleccionar cualquier archivo de la carpeta del proyecto del add-on.

Aparecerá la extensión en la sección `Extensiones temporales`. Para recargar cambios, click en `Recargar`.

## Desarrollar add-on

Herramientas que ayudan al desarrollo:

### web-ext

[Enlace a documentación](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/).

#### Ejecutar extensión en navegador y recargar automáticamente con cada cambio

```bash
# cd path donde se encuentre el archivo manifest.json
web-ext run
```

## Archivos

```bash
-projectfolder/
  |
  -manifest.jon: info de la extension
  -file.js
  -icons/
    |
    -image.png
```

### manifest.json

Valores:

- manifest_version, name, and version: obligatorios, metadatos básicos de la extension.
- descripcion: opcional, se muestra en el Add-ons Manager.
- icons: optional, icono de la extensión, se indica la resolución. Ejemplo 48 o 96.
- content_scripts: para cargar un script en las webs indicadas.
- applications: a veces es necesario indicar un ID de la extesion, `applications` > `gecko` > `id`.

### icons

Icono que se muestra en el Add-ons Manager en la lista de extensiones.

Resolución indicada en manifest.json.

### file.js

Archivo que puede cargarse en la web visitada.

## Editar add-on en la web de extensiones

Acceder a la siguiente URL (modificar el nombre del usuario): <https://addons.mozilla.org/es/firefox/user/CarlosAMolina/#my-submissions>

En la parte superior derecha, elegir `Register or log in` para iniciar sesión. Volver a acceder a la URL de antes y, en la parte superior derecha, click en `Central del desarrollador`; accederemos a <https://addons.mozilla.org/es/developers/> donde seleccionaremos la opción `Editar página del producto` del complemento deseado.

## Recursos

Instalar add-on, archivos de un add-on

<https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Your_first_WebExtension>

Publicar add-on

<https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Publishing_your_WebExtension>

Más información de los archivos de un add-on

<https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Anatomy_of_a_WebExtension>
