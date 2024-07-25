# Neovim plugins

## Contenidos

- [Introducción Neovim plugins](#introducción-neovim-plugins)
- [Vim-plug](#vim-plug)
  - [Instalar Vim-plug](#instalar-vim-plug)
  - [Instalar plugins con Vim-plug](#instalar-plugins-con-vim-plug)
  - [Mostrar plugins instalados con Vim-plug](#mostrar-plugins-instalados-con-vim-plug)
  - [Desinstalar plugins con Vim-plug](#desinstalar-plugins-con-vim-plug)
- [Coc](#coc)
  - [Coc instalación](#coc-instalación)
    - [Coc comandos gestión extensiones](#coc-comandos-gestión-extensiones)
      - [Instalar extensiones con Coc](#instalar-extensiones-con-coc)
      - [Mostrar extensiones instaladas](#mostrar-extensiones-instaladas)
      - [Actualizar extensiones instaladas](#actualizar-extensiones-instaladas)
      - [Desinstalar extensiones con Coc](#desinstalar-extensiones-con-coc)
  - [Configuración](#configuración)
    - [Reducir updatetime](#reducir-updatetime)
    - [Navegar entre las opciones de autocompletado con el tabulador](#navegar-entre-las-opciones-de-autocompletado-con-el-tabulador)
    - [Seleccionar una opción de autocompletado con la tecla enter](#seleccionar-una-opción-de-autocompletado-con-la-tecla-enter)
    - [Navegar por el código](#navegar-por-el-código)
    - [Resaltar código seleccionado](#resaltar-código-seleccionado)
    - [Organizar automáticamente los imports](#organizar-automáticamente-los-imports)
    - [Mostrar comandos disponibles](#mostrar-comandos-disponibles)
- [pyright](#pyright)
  - [Configurar Pyright](#configurar-pyright)
    - [Pyright configurations](#pyright-configurations)
    - [Realizar imports desde otros proyectos](#realizar-imports-desde-otros-proyectos)
- [rust-analyzer](#rust-analyzer)
- [nvim-tree](#nvim-tree)
  - [Instalar nvim-tree](#instalar-nvim-tree)
  - [Configurar nvim-tree](#configurar-nvim-tree)
  - [Utilizar nvim-tree](#utilizar-nvim-tree)
- [Referencias](#referencias)


## Introducción Neovim plugins

En este apartado veremos cómo instalar varios plugins que suelo utilizar en Neovim.

## Vim-plug

Se trata de un gestor de plugins.

### Instalar Vim-plug

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

### Instalar plugins con Vim-plug

Para instalar plugins con `vim-plug`, en `~/.config/nvim/init.vim` creamos la sección `call` e indicamos los plugins deseados:

```bash
call plug#begin()
Plug 'neoclide/coc.nvim', {'branch': 'release'}
call plug#end()
```

Tras esto, reiniciamos Neovim y ejecutamos la orden:

```bash
:PlugInstall
```

### Mostrar plugins instalados con Vim-plug

```bash
:PlugStatus
```

### Desinstalar plugins con Vim-plug

Primero, los eliminamos en el archivo `init.vim` de la sección `call plug`; tras esto ejecutamos:

```bash
:PlugClean
```

## Coc

Coc (Conquer of Completion) nos permitirá autocompletar texto.

### Coc instalación

Es necesario tener instado `nodejs`:

```bash
sudo pacman -S nodejs
```

Instalaremos Coc utilizando el gestor de plugins `vim-plug` (ver sección donde se explica su funcionamiento), añadiendo el plugin `Plug 'neoclide/coc.nvim', {'branch': 'release'}`.

Para poder instalar plugins mediante `CocInstall`, necesitamos instalar `npm`:

```bash
sudo pacman -S npm
```

#### Coc comandos gestión extensiones

##### Instalar extensiones con Coc

El siguiente ejemplo instala `coc-pyright`:

```bash
:CocInstall coc-pyright
```

##### Mostrar extensiones instaladas

```bash
:CocList extensions
```

##### Actualizar extensiones instaladas

```bash
:CocUpdate
```

Con el comando anterior pueden solucionarse errores de tipo `Unhandled exception` que ocurran con coc.

##### Desinstalar extensiones con Coc

El siguiente ejemplo desinstala `coc-pyright`:

```bash
:CocUninstall coc-pyright
```

### Configuración

Una vez instalado, al escribir nos aparecerán opciones de autocompletado, pero no podremos seleccionarlas, para ello hemos editar el archivo de configuración.

```bash
vi ~/.config/nvim/init.vim
```

Para utilizar funcionalidades que nos ayuden al editar archivos, es necesario realizar otras acciones; una opción es instalar extensiones. Por ejemplo, al trabajar con un archivo Python, si queremos ordenar los imports, aunque la opción esté configurada en Coc, debemos instalar la extensión `coc-pyright`.

#### Reducir updatetime

Con esto conseguimos que las acciones se realicen antes, por ejemplo, resaltar dónde se utiliza la función sobre la que tengamos el cursor (este resaltado de ejemplo también hay que configurarlo).

```bash
" Having longer updatetime (default is 4000 ms = 4s) leads to noticeable
" delays and poor user experience
set updatetime=300
```

#### Navegar entre las opciones de autocompletado con el tabulador

```bash
" Use tab for trigger completion with characters ahead and navigate
" NOTE: There's always complete item selected by default, you may want to enable
" no select by `"suggest.noselect": true` in your configuration file
" NOTE: Use command ':verbose imap <tab>' to make sure tab is not mapped by
" other plugin before putting this into your config
inoremap <silent><expr> <TAB>
      \ coc#pum#visible() ? coc#pum#next(1) :
      \ CheckBackspace() ? "\<Tab>" :
      \ coc#refresh()
inoremap <expr><S-TAB> coc#pum#visible() ? coc#pum#prev(1) : "\<C-h>"
```

#### Seleccionar una opción de autocompletado con la tecla enter

```bash
" Make <CR> to accept selected completion item or notify coc.nvim to format
" <C-g>u breaks current undo, please make your own choice
inoremap <silent><expr> <CR> coc#pum#visible() ? coc#pum#confirm()
                              \: "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"
```

#### Navegar por el código

Debemos estar en el modo comando de Neovim (pulsar escape).

```bash
" GoTo code navigation
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)
```

Por ejemplo, para ir a la definición del método sobre el que tengamos situado el cursor, estando Neovim en modo comando pulsaremos `gd`.

#### Resaltar código seleccionado

Esta opción resalta dónde se definió o dónde se utiliza la función, clase, etc sobre la que tengamos el cursor.

```bash
" Highlight the symbol and its references when holding the cursor
autocmd CursorHold * silent call CocActionAsync('highlight')
```

#### Organizar automáticamente los imports

Desde el modo comando, escribiendo `:OR` se organizarán los imports.

```bash
" Add `:OR` command for organize imports of the current buffer
command! -nargs=0 OR   :call     CocActionAsync('runCommand', 'editor.action.organizeImport')
```

#### Mostrar comandos disponibles

En modo comando, pulsando la barra espaciadora y la tecla `c`, podremos ver las opciones que permite Coc.

```bash
" Show commands
nnoremap <silent><nowait> <space>c  :<C-u>CocList commands<cr>
```

Podemos movernos por la lista con los cursores de navegación del teclado, seleccionar una opción al pulsar enter y salir de la lista con escape.

Por ejemplo, de tener configurada la opción de organizar los imports, esta saldrá en la lista mostrada por el comando anterior como `editor.action.organizeImport`

## pyright

Para programar en el lenguaje Python, utilizo el plugin `pyright`, puede instalarse a través de `Coc` (explicado en este apartado de plugins):

```bash
:CocInstall coc-pyright
```

Ahora, al editar un archivo de Python, recibiremos ayuda de `pyright`.

### Configurar Pyright

#### Pyright configurations

Las configuraciones disponibles están describas [en la página del proyecto](https://github.com/fannheyward/coc-pyright?tab=readme-ov-file#configurations).

Pueden añadirse ejecutando en neovim `:CocConfig`, se abrirá el archivo de configuración `.config/nvim/coc-settings.json` donde podemos escribir por ejemplo:

```json
{
    "pyright.inlayHints.functionReturnTypes": false,
    "pyright.inlayHints.variableTypes": false,
    "pyright.inlayHints.parameterTypes": false
}
```

#### Realizar imports desde otros proyectos

Con esta configuración evitaremos el error `Pyright reportMissingImports`.

En el root de nuestro proyecto (donde está la carpeta `.git`), creamos el archivo `pyrightconfig.json`:

```bash
{
    "executionEnvironments": [
        {
            "root": "folder_a",
            "extraPaths": [
                "/home/user/project_b/folder_1",
                "/home/user/project_b/folder_2"
            ]
        }
    ]
}
```

En el ejemplo anterior, suponemos que estamos trabajando en el proyecto con path `/home/user/project_a` y sus archivos python importan contenido de la carpeta `folder_1` y `folder_2` de un proyecto diferente llamado `project_b`.

Importante, debemos editar los archivos desde el root de nuestro proyecto, ejemplo `cd /home/user/project_a` y luego `vi foo/bar.py`.

La documentación puede verse en este [enlace](https://github.com/microsoft/pyright/blob/main/docs/configuration.md).

## rust-analyzer

Esta extensión nos ayudará al editar archivos del lenguaje Rust.

Podemos instalarla mediante Coc:

```bash
:CocInstall coc-rust-analyzer
```

## nvim-tree

Este plugin mostrará un árbol con los archivos del proyecto en el que nos encontremos.

### Instalar nvim-tree

Añadiremos a `vim-plug` el siguiente plugin:

```bash
Plug 'nvim-tree/nvim-tree.lua'
```

### Configurar nvim-tree

Para poder utilizar esta extensión, por ejemplo, abrir y cerrar el árbol de archivos con el comando `:NvimTreeToggle`, añadiremos la siguiente línea al archivo `init.vim`:

```bash
lua require'nvim-tree'.setup {}
```

De querer utilizar más opciones de configuración, lo haremos también en el archivo `init.vim` con lua heredoc, por ejemplo, para añadir la opción `vim.opt.termguicolors = true`, escribiremos en `init.vim`:

```bash
lua << EOF
vim.opt.termguicolors = true
EOF
```

### Utilizar nvim-tree

Los comandos se introducen en el modo comando de Neovim.

Abrir y cerrar árbol de archivos:

```bash
:NvimTreeToggle
```

Para expandir el contenido de una carpeta, utilizamos la tecla enter.

## Referencias

Coc

- Documentación:

  <https://github.com/neoclide/coc.nvim>

- Comandos gestión extensiones:

  <https://github.com/neoclide/coc.nvim/issues/1311#issuecomment-547225815>

nvim-tree

- Documentación:

  <https://github.com/nvim-tree/nvim-tree.lua>

- Poder utilizar sus comandos:

  <https://github.com/nvim-tree/nvim-tree.lua/issues/767#issuecomment-962637481>

Pyright

<https://github.com/microsoft/pyright>

rust-analyzer

<https://rust-analyzer.github.io/>

Vim-plug

<https://github.com/junegunn/vim-plug#neovim>
