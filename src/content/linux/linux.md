# GNU Linux

## Contenidos

- [Arch Linux](arch-linux/arch-linux.html)
- [Comandos](#comandos)
- [Debian](debian.html)
- [Directories](directories.html)
- [Network](network.html)
- [Partitions](partitions.html)
- [Streams](streams.html)
- [TouchPad](touchpad.html)
- [gnome-terminal](#gnome-terminal)
- [Libros](#libros)
- [Reemplazar texto en archivos](#reemplazar-texto-en-archivos)

## comandos

### dd

Crear un archivo de 10 MiB (datos aleatorios):

```bash
dd if=/dev/urandom of=test1.file bs=1M count=10
```

## gnome-terminal

Abrir en pantalla completa (muestra barra de menú):

```bash
gnome-terminal --window --maximize
```

Abrir en full screeen (no muestra barra de menú):

```bash
gnome-terminal --full-screen
```

## Libros

- [Linux Programming Interface](https://nostarch.com/tlpi#content).

## Reemplazar texto en archivos

```bash
grep -rlZe "EXAMPLETEXT==0\.5" --exclude-dir=.git . | xargs -0 sed -i 's/EXAMPLETEXT==0.5/EXAMPLETEXT==0.6/g'
```
