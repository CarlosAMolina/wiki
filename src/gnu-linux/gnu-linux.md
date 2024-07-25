# GNU Linux

## Contenidos

- [Arch Linux](arch-linux/arch-linux.html)
- [Debian](debian.html)
- [Directories](directories.html)
- [Network](network.html)
- [Partitions](partitions.html)
- [Streams](streams.html)
- [TouchPad](touchpad.html)
- [Reemplazar texto en archivos](#reemplazar-texto-en-archivos)

## Reemplazar texto en archivos

```bash
grep -rlZe "EXAMPLETEXT==0\.5" --exclude-dir=.git . | xargs -0 sed -i 's/EXAMPLETEXT==0.5/EXAMPLETEXT==0.6/g'
```

