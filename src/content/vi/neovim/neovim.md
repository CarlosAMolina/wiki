# Neovim

## Contenidos

- [Introducción Neovim](#introducción-neovim)
- [Instalación](#instalación)
  - [Arch Linux](#arch-linux)
- [Configuración](#configuración)
  - [Copiar entre instancias](#copiar-entre-instancias)
    - [Copiar en Wayland](#copiar-en-wayland)
  - [Corrección ortográfica](#corrección-ortográfica)
    - [Errores corrección ortográfica](#errores-corrección-ortográfica)
      - [No encuentra archivo](#no-encuentra-archivo)
- [Comandos útiles](#comandos-útiles)
- [Plugins](#plugins)
- [Regex](#regex)
  - [Reemplazar](#reemplazar)
- [Referencias](#referencias)

## Introducción Neovim

Neovim es un fork de Vim con mejoras.

## Instalación

### Arch Linux

```bash
sudo pacman -S neovim
```

## Configuración

Primero, debemos saber qué archivo de configuración se utiliza. Para ello, en Neovim lanzamos el siguiente comando:

```bash
:echo stdpath("config")
```

Ya con esto, conocemos el archivo de configuración con el que trabajar:

```bash
vi ~/.config/nvim/init.vim
```

### Copiar entre instancias

Para poder copiar entre diferentes instancias de Neovim utilizando `yy` y `pp`, añadir la siguiente línea al archivo de configuración:

```bash
set clipboard+=unnamedplus
```

#### Copiar en Wayland

En Wayland puede ser necesario instalar `wl-clipboard`. Dicho [en este link](https://stackoverflow.com/questions/61379318/how-to-copy-from-vim-to-system-clipboard-using-wayland-and-without-compiled-vim).

Ejemplo Ubuntu:

```bash
sudo apt install wl-clipboard
```

### Corrección ortográfica

Habilitamos la corrección ortográfica en español para solo archivos de tipo Markdown con la siguiente configuración:

```bash
augroup markdownSpell
    autocmd!
    autocmd FileType markdown setlocal spell
    autocmd BufRead,BufNewFile *.md setlocal spell
augroup END
set spelllang=es
```

#### Errores corrección ortográfica

##### No encuentra archivo

De tener este error al iniciar Neovim:

```bash
Warning: Cannot find word list "en.utf-8.spl" or "en.ascii.spl"
Warning: Cannot find word list "en.utf-8.spl" or "en.ascii.spl"
```

La solución es:

```bash
ln -s .vimrc .nvimrc
ln -s .vim .nvim
```

## Comandos útiles

- Control + 6: cambiar entre 2 archivos (el actual y el anterior).
- Control + o: volver al anterior archivo que se abrió.
- Control + i: volver al siguiente archivo que se abrió.
- `:jumps`: histórico de archivos abiertos. Se utilizará este orden al pulsar `control o` y `control + i`.
- Control + d: mientras escribes un comando, esta opción muestra lista de comandos disponibles.
- `%`: ejecutado sobre un elemento de apertura o cierre (ejemplo, `{`, tags html como `div`, etc) lleva a su correspondiente cierre o apertura, respectivamente.
- `c`: como `d` pero tras cortar accedes a modo `insert` respectando la indentación.
- `dap`: eliminar párrafo.

## Plugins

Ver [plugins](plugins.html).

## Regex

### Reemplazar

Eliminar letras y números entre `0x` y `>`:

```bash
# Input: <sqlalchemy.sql.elements.BooleanClauseList object at 0x78c624cfccc0>
# Output: <sqlalchemy.sql.elements.BooleanClauseList object at >
:%s/0x[0-9a-zA-Z]\+>//g
```

## Referencias

Arch Linux wiki

<https://wiki.archlinux.org/title/Neovim>

Clipboard

<https://neovim.io/doc/user/provider.html#provider-clipboard>

Corrección ortográfica

  - Habilitar para ciertos archivos

    <https://vi.stackexchange.com/questions/6950/how-to-enable-spell-check-for-certain-file-types>

  - Error no encontrar archivo

    <https://github.com/neovim/neovim/issues/1551#issuecomment-64467321>

