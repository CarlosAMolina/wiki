# Streams

## ¿Qué son?

Son canales de comunicación de entrada y salida, entre un programa y su entorno.

Los streams se representan como `file descriptors`.

Los 3 streams estándar son:

Standard stream | Nombre          | File descriptor
----------------|-----------------|----------------
stdin           | Standard Input  | 0
stdout          | Standard Output | 1
stderr          | Standard Error  | 2

Respecto `stderr`, hay que tener en cuenta que no se usa solo para mensajes de error. Por ejemplo, en el lenguaje de programación Rust, las librerías de logging utilizar `stderr` como salida por defecto; Rust por ejemplo usa `stdout` con `println!`.

## Redirección

Ejemplo redirigir stderr a un archivo:

```bash
./program 2> logs.txt
```

## Recursos

Introducción:

<https://www.putorius.net/linux-io-file-descriptors-and-redirection.html>

