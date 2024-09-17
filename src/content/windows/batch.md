## Contenidos

- [Comandos iniciales](#comandos-iniciales)
- [Formato](#formato)
- [Procesos](#procesos)
- [Variables](#variables)
- [Recursos](#recursos)

## Comandos iniciales

- `>>`: añadir en la siguiente linea.
- `>`: escribir/sobreescribir en un archivo.
- `@`: ejecutar comando sin mostrar el comando por pantalla.
- `cls`: limpiar pantalla.
- `COMANDO /?`: información de un comando.
- `echo off`: no mostrar los comando por pantalla (los resultados si se muestran).
- `echo.`: es igual que un salto de línea (\n).
- `exit`: cerrar símbolo de sistema.
- `pause>nul`: para que no aparezca mensaje por defecto de pausa (para usar otro mensaje, usar `echo` antes).
- `rem:` comentario.
- `type` o `more`: como cat.

## Formato

- `color AB`: A color fondo y B color letras. `color/?` para ver opciones.
- `Title TITULO`: cambiar título del símbolo de sistema.

## Procesos
- `start PROCESO`.
- `start ...exe`.
- `start www.duckduckgo.com`: abre página en el navegador predeterminado.
- `start “” “C:\Program Files (x86)\Mozilla Firefox\firefox.exe” duckduckgo.com`: iniciar web con el navegador deseado.
- `start "" "C:\Program Files\Sublime Text 3\sublime_text.exe"`: iniciar aplicación.
- `taskkill /f /im proceso.exe`: `/f` fuerza el cierre, para saber si `taskkill` funciona en el ordenador, utilizar `taskkill/?`.

## Variables

- `set NOMBREVARIABLE=VALOR`: declarar variable o modificar su valor.
- `set /p NOMBREVARIABLE=Introduce valor`: para que el usuario introduzca el valor.
- `set /a suma = %numero1% + %numero2%`: hacer operaciones y guardarlas en variables.
- `echo %NOMBREVARIABLE%`: llamar variable con %%.

Importante. Para `set` no usar espacios entre `NOMBREVARIABLE` y el `=`. Si hay espacio antes del `=`, el espacio se añade al nombre de la variable y si hay espacio después del `=`, se guarda en la variable.

## Recursos

Comandos para Batch

<http://www.cristalab.com/tutoriales/programacion-batch-con-archivos-.bat-c48410l/>

Espacio antes y después del símbolo `=` al declarar variables

<https://stackoverflow.com/questions/1884071/windows-echo-command-cant-echo-a-user-set-variable>
