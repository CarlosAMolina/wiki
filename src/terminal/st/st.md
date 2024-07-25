# st

## Contenidos

- [Instalación](#instalación)
- [Patches](#patches)
  - [Scroll](#scroll)
- [Referencias](#referencias)

## Instalación

Descargamos st del siguiente [link](https://st.suckless.org/).

Tras descomprimir la carpeta, ejecutamos:

```bash
sudo make clean install
```

## Patches

### Scroll

```bash
cd ST_PATH
wget https://st.suckless.org/patches/scrollback/st-scrollback-0.8.5.diff
cp config.h config.def.h # The patch will modify config.def.h
patch -i st-scrollback-0.8.5.diff
cp config.def.h config.h
sudo make clean install
```

## Referencias

Página oficial

<https://st.suckless.org>

Documentación

<https://wiki.archlinux.org/title/St>

