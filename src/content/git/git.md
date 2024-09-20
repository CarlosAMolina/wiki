## Contenidos

- [Configuración](#configuración)
  - [Configuración inicial](#configuración-inicial)
  - [Mostrar configuración de un repositorio](#mostrar-configuración-de-un-repositorio)
  - [SSH](#ssh)
    - [Consideraciones](#consideraciones)
    - [Errores SSH](#errores-ssh)
      - [Error authentication failed](#error-authentication-failed)
      - [Error push permission denied](#error-push-permission-denied)
- [Términos](#términos)
  - [Staging area o index](#staging-area-o-index)
  - [commit-ish](#commit-ish)
  - [HEAD](#head)
  - [Commit](#commit)
    - [Parent commit](#parent-commit)
  - [git log](#git-log)
    - [Argumentos útiles](#argumentos-útiles)
    - [Buscar en el histórico de git](#buscar-en-el-histórico-de-git)
  - [git diff](#git-diff)
  - [bisect](#bisect)
  - [git reflog](#git-reflog)
  - [Branches](#branches)
    - [Common ancestor](#common-ancestor)
  - [Conflictos](#conflictos)
    - [Qué se muestra como HEAD en un conflicto](#qué-se-muestra-como-head-en-un-conflicto)
    - [Resolver un conflicto](#resolver-un-conflicto)
      - [vimdiff](#vimdiff)
    - [Obtener información tras resolver un conflicto](#obtener-información-tras-resolver-un-conflicto)
  - [Merge](#merge)
    - [Funcionamiento merge](#funcionamiento-merge)
    - [Target vs source brach](#target-vs-source-brach)
    - [Por qué a veces aparece mensaje de merge y otras no](#por-qué-a-veces-aparece-mensaje-de-merge-y-otras-no)
      - [Mensaje de merge constante](#mensaje-de-merge-constante)
  - [rebase](#rebase)
    - [Comandos para ejecutar rebase](#comandos-para-ejecutar-rebase)
    - [Introducción rebase](#introducción-rebase)
    - [Pasos que ejecuta rebase](#pasos-que-ejecuta-rebase)
    - [Ventajas rebase](#ventajas-rebase)
    - [Inconvenientes rebase](#inconvenientes-rebase)
    - [Implicaciones de rebase](#implicaciones-de-rebase)
    - [rebase interactivo](#rebase-interactivo)
  - [rerere](#rerere)
  - [Cherry pick](#cherry-pick)
  - [Remote](#remote)
    - [Configurar remote](#configurar-remote)
    - [Convención nombres para remote](#convención-nombres-para-remote)
  - [Fetch](#fetch)
  - [Pull](#pull)
    - [Añadir tracking information](#añadir-tracking-information)
  - [Push](#push)
  - [Stash](#stash)
  - [squash o squashing](#squash-o-squashing)
  - [revert](#revert)
    - [revert vs restore](#revert-vs-restore)
  - [reset](#reset)
    - [Deshacer el último commit manteniendo los archivos modificados](#deshacer-el-último-commit-manteniendo-los-archivos-modificados)
    - [Volver a un commit anterior](#volver-a-un-commit-anterior)
  - [worktree](#worktree)
    - [Comandos worktree](#comandos-worktree)
  - [tags](#tags)
  - [cat-file](#cat-file)
  - [gitignore](#gitignore)
- [Resolución casos comunes en git](#resolución-casos-comunes-en-git)
  - [Crear repositorio remoto desde un repositorio local](#crear-repositorio-remoto-desde-un-repositorio-local)
  - [Cambiar nombre de un repositorio](#cambiar-nombre-de-un-repositorio)
    - [GitLab](#gitlab)
  - [Recuperar un archivo de una rama eliminada](#recuperar-un-archivo-de-una-rama-eliminada)
  - [Continous integration](#continous-integration)
- [OSINT](#osint)
  - [Email de quién realizó el commit](#email-de-quién-realizó-el-commit)
- [Referencias](#referencias)

## Configuración

### Configuración inicial

```bash
git config --global user.name "$USER_NAME"
git config --global user.email $USER_EMAIL
git config --global http.sslVerify false
```

Nota. Indicar que no se verifique el certificado SSL es un error de seguridad, en lugar de añadirlo a la configuración global, puede indicarse cada vez que se utilice el comando git añadiendo alguna de las siguientes opciones:

```bash
git -c http.sslVerify false
```

```bash
git -c http.sslVerify=False
```

### Mostrar configuración de un repositorio

Hay varias opciones para ver la configuración del repositorio actual:

```bash
git config --list --local
cat .git/config
```

### SSH

El protocolo SSH nos permite ejecutar los comandos de git en el servidor remoto sin tener que indicar nuestro usuario y contraseña.

Los pasos para realizar la configuración pueden consultarse en los siguientes enlaces:

<https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh>

<https://docs.gitlab.com/ee/ssh/>

#### Consideraciones

Para poder utilizar el protocolo SSH y no introducir nuestras credenciales, el repositorio con el que estemos trabajando debe haberse descargado del siguiente modo:

```bash
git clone git@$SERVIDOR:$USUARO/$REPOSITORIO
# Ejm: git clone git@github.com:CarlosAMolina/cmoli.es
```

#### Errores SSH

##### Error authentication failed

En caso de tener error de autenticación al utilizar los comandos de git, puede ser debido a tener activado el doble factor de autenticación; en este caso, hay que utilizar como credenciales un token.

Los pasos para tener este token están explicado en el siguiente enlace:

<https://mycyberuniverse.com/how-fix-fatal-authentication-failed-for-https-github-com.html>

##### Error push permission denied

Si al hacer `push` recibimos el siguiente error:

```bash
ERROR: Permission to URL_REPO.git denied to USER.
fatal: No se pudo leer del repositorio remoto.

Por favor asegúrate de que tengas los permisos de acceso correctos
y que el repositorio exista.
```

Puede ser que nuestro equipo tenga distintos usuarios de git y no estemos utilizando la configuración del usuario con permisos para escribir en el repositorio.

Para solucionarlo, importamos al ssh-agent la clave SSH privada correspondiente al repositorio, por ejemplo:

```bash
ssh-add ~/.ssh/id_ed255190
```

[Referencia](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Términos

### Staging area o index

<https://git-scm.com/about/staging-area>

Subir archivo(s) a la zona de index (staging area):

```bash
# Un archivo.
git add NOMBRE_ARCHIVO

# Todos los archivos.
git add .
```

Subir de la zona de index al repositorio local.

```bash
git commit -m "$MENSAJE_PARA_EL_COMMIT"
```

<img src="https://git-scm.com/images/about/index1@2x.png" alt="" width="300">

### commit-ish

`commit-ish` es algo parecido a un commit como por ejemplo el ID de un commit, un commit como referencia a otro usando HEAD~<número-de-commits>.

### HEAD

HEAD apunta a la rama a la que hemos hecho `checkout`.

Si vemos el contenido del archivo HEAD, este es una referencia a la rama. Esa referencia es simplemente un commit. Ejemplo:

```bash
$ cat .git/HEAD
ref: refs/heads/main
$ cat .git/refs/heads/main
ccadc23a11e42a1877c883d59180304fa17ba506
```

### Commit

#### Parent commit

Como se indica [en este link](https://stackoverflow.com/questions/38239521/what-is-the-parent-of-a-git-commit-how-can-there-be-more-than-one-parent-to-a-g#38239664), el commit padre es el commit (o los commits) en que se basan el commit actual:

- Cuando generas un commit, el commit actual es el padre del nuevo que se genera.
- Cuando mergeas dos commits (mergeo no de tipo fast forward), se genera un nuevo commit cuyos padres son los dos anteriores. Pueden verse con `git log --oneline --graph --parents`.

### git log

Muestra el historial de los commits de la rama actual del repositorio de git.

#### Argumentos útiles

- `-p`: añade el diff del commit.

#### Buscar en el histórico de git

Para buscar una expresión regular en los mensajes de los commits:

```bash
git log --grep "<term>"
```

Mostrar los commits que han modificado un archivo o varios archivos:

```bash
git log -p <file_name_1> <file_name_2>
```

### git diff

Al usar `git diff` podemos trabajar con `..` o `...`. Las diferencias se explican en [este enlace](https://stackoverflow.com/questions/5817579/how-can-i-preview-a-merge-in-git).

### bisect

Se utiliza para buscar el commit donde un error actual funcionaba bien.

Para ello, marcaremos un commit ID como en el que existe el bug y otro commit ID como el commit donde no existe el bug; no es necesario que sean exactamente los commits done el bug apareció inicialmente y donde dejó de aparecer, puesto que esto es lo que vamos a buscar gracias a `bisect`.

Tras marcar estos dos puntos, git nos llevará sobre los commits entre estos dos puntos haciendo una búsqueda binaria; de modo que marcaremos si en cada commmit existe el bug o no.

Para utilizarlo:

```bash
# Iniciamos bisect.
git bisect start
# Marcamos el commit actual como referencia de commit donde existe el bug.
git bisect bad
# Marcamos otro commit como referencia de un commit donde el código funcionaba bien, no había bug.
git bisect good <commit>
# git comienza la búsqueda binaria.
# Tras encontrar el commit erróneo, terminamos bisect con el siguiente comando.
git bisect reset
```

Hasta ahora hemos ejecutado bisect manualmente, se puede automatiza así:

```bash
git bisect start
git bisect bad
git bisect good <commit>
git bisect run <cmd> # Por ejemplo, ejecutar en un test: git bisect run ./node_modules/.bin/vitest --run
```

### git reflog

El comando `git reflog` muestra los cambios de HEAD (commits, cambios de rama, etc).

Lo que muestra este comando, está en `cat .git/logs/HEAD`.

### Branches

#### Common ancestor

El `common ancestor` de dos ramas es el commit más reciente que tienen en común ([link](https://www.freecodecamp.org/news/the-definitive-guide-to-git-merge/)).

Un `common ancestor` `B` es mejor que otro `A` si `B` es más reciente, es decir, si `A` es un `ancestor` de `B`. El `best common ancestor` es el `common ancestor` que no tenga otros `ancestor` mejores que él. Al `best common ancestor` también se le llama `merge base`. [Link documentación](https://git-scm.com/docs/git-merge-base).

Hay situaciones en que haya varios `merge base` (ver ejemplos en la [documentación](https://git-scm.com/docs/git-merge-base#_discussion)).

### Conflictos

Ocurren cuando distintas ramas han editado la misma línea de código e intentamos unir una rama en otra.

#### Qué se muestra como HEAD en un conflicto

En un conflicto, se muestra entre `<<<<<<< HEAD` y `=======` los cambios de una rama y desde `=======` hasta `>>>>>>> <commit_id_de_la_otra_rama>` los cambios de la otra rama.

Qué rama es HEAD depende del si hacemos merge o rebase:

- En un merge: HEAD es nuestra rama y la otra rama es la que estamos mergeando en la nuestra.
- En un rebase: HEAD es la otra rama que juntamos con la nuestra, y lo que se muestra como la otra rama es en realidad nuestra rama. Esto es por cómo funciona rebase, como se explicó en su sección, al ahcer rebase, primero se hace checkout a la otra rama.

#### Resolver un conflicto

Para quedarme en un conflicto con el archivo de una rama o de otra, ejecutamos el siguiente comando utilizando `--ours`o `--theirs`, hay que tener en cuenta que el significado cambia:

- ours: en merge es mi rama y en rebase es la otra rama.
- theirs: en merge es la otra rama y en rebase es mi rama inicial.

```bash
git checkout --ours <file_name>
git checkout --theirs <file_name>
```

Puede abortarse el merge con:

```bash
git merge --abort
```

##### vimdiff

Es una herramienta de terminal que muestra una ventana para resolver conflictos similar a cómo lo hace por ejemplo Pycharm.

Explicación de configuración y uso [en este link](https://stackoverflow.com/questions/161813/how-do-i-resolve-merge-conflicts-in-a-git-repository).

#### Obtener información tras resolver un conflicto

Tras resolver el conflicto, puedes ver de qué rama viene cada archivo con:

```bash
git log --graph --oneline
```

### Merge

#### Funcionamiento merge

Merge trabaja a partir del best common ancestor.

#### Target vs source brach

- Target: la rama en la que te encuentras.
- Source: la rama que indicas en el comando `git merge <branch>`.

#### Por qué a veces aparece mensaje de merge y otras no

Cuando hacemos merge, en caso de que el último commit de una rama sea el best common ancestor de la otra, no habrá un commit message de `Mergeo branch X into Y` porque git únicamente tuvo que hacer un fast forward, no combinó archivos. Esta es la diferencia entre `fast forward merge` y `3 way merge`.

##### Mensaje de merge constante

Imagina que ocurre esta situación:

1. Haces `git pull` y hay conflicto.
1. En tu rama local, resuelves el conflicto pero no haces push.
1. En remoto se hacen commits nuevos (que no causan conflicto con nuestra rama actual).
1. Cada vez que hagamos pull, habrá un commit extra de merge porque los históricos de git son distintos en local y en remoto (no hicimos push).

### rebase

#### Comandos para ejecutar rebase

Podemos indicarlo cada vez que hagamos pull:

```bash
git pull --rebase
```

Otra opción es guardarlo en la configuración y de este modo se hará rebase automáticamente cada vez que hagamos `git pull`:

```bash
git config --global pull.rebase true.
```

#### Introducción rebase

Este comando actualiza el commit origen del que sale la rama.

Por ejemplo, tenemos dos ramas, `foo` y `bar`, `foo` tiene los commits B y C, y `bar` tiene los commits D y E; ambas ramas salen del commit A:

```bash
A - B - C
 \- D - E
```

Si estamos en la rama `foo` y aplicamos `rebase`, su primer commit B saldrá de E en lugar de A:

```bash
         /- B - C
A - D - E
```

Es decir, desplaza los commits de una rama al final de los de otra rama.

#### Pasos que ejecuta rebase

Para explicarlo, `current_branch` es nuestra rama y `target_branch` la rama que traemos a la nuestra.

Cuanto ejecutamos `git rebase $target_brach`:

1 - Hace checkout del último commit de target_branch. Es decir, no estamos en nuestra rama, sino en la otra.
2 - Aplica cada uno de los commits de current_branch.
3 - Vuelve a current_branch y actualiza current_branch al actual commit SHA.

#### Ventajas rebase

Permite mergear haciendo merge de tipo fast forward, por lo que no aparecerá un commit de merge y tendremos un git log lineal, lo que es más limpio a la hora de ver el histórico y es mas fácil comparar nuestros cambios con respecto a la versión actual de la otra rama.

#### Inconvenientes rebase

- Inconveniente al hacer push. Rebase altera el historial de la rama de git, por lo que para hacer push a origin hay que utilizar la opción `force`. Por esto rebase debe usarse en ramas que solo utilicemos nosotros.
- Inconveniente perder historial de git al resolver conflictos. Cuando resolvemos conflictos con rebase, como con rebase modificamos el commit conflictivo y hacemos commit de la modificación, en `git log` puede que perdamos el commit en el que se introdujo el conflicto que hemos resuelto (creo que esto ocurre si el commit conflictivo es el más actual), por lo que la única manera de saber qué código daba conflicto es con `git reflog`.
- Inconveniente de tener que solucionar el mismo conflicto varias veces. Cuando solucionamos un conflicto, si nuestros cambios no están en remote, el conflicto puede aparecer repetidamente aunque lo hallamos solucionado. Esto se evita con `rerere`.

#### Implicaciones de rebase

El hecho de que rebase primero haga checkout a la otra rama afecta a:

- Imagina que hay un conflicto, si escribes `git status` puede que no se muestren los archivos que has modificado en tu rama con respecto a la otra, ya que ahora la referencia es la otra rama.
- Ver apartado de conflictos para más implicaciones.

#### rebase interactivo

Permite diversas opciones como editar mensajes de git, squasing, etc.

 Comando:

```bash
git rebase -i <commit-ish>
```

Con `HEAD~<número>` indicamos los commits desde HEAD sobre los que actuar. Ejemplo: HEAD~1 actúa sobre el último commit.

Ejemplo de rebase interactivo:

```bash
# Vamos a hacer squash sobre los 3 últimos commits.
git rebase -i HEAD~3
# Se abrirá un apantalla donde indicar cómo configurar el rebase.
# Para hacer squash de los 3 últimos commits ponemos `pick` en el más viejo y `s` en el resto.
pick 576b5fd <message de HEAD~1>
s 9398088 <message de HEAD~2>
# Guardamos; en vim es con `:x`.
# Tras esto, aparecerá una ventana donde escribir el mensaje a usar en el commit.
```

### rerere

Significa REuse REcorded REsolution.

Gracias a el, git recuerda cómo solucionamos un conflicto específico y aplica automáticamente la solución.

Hay que tener en cuenta que, si hemos guardado con rerere una solución incorrecta, habrá que eliminarla.

Por defecto está deshabilitado.

Para habilitarlo en el repositorio actual:

```bash
git config --local rerere.enabled true
```

### Cherry pick

Permite traer un commit específico a nuestra rama.

### Remote

Remote es otro repositorio de git con el mismo proyecto que el que estamos desarrollando. Es como una copia de nuestro repo en otro lugar; este lugar puede ser el servidor de GitHub, una carpeta en nuestro PC, etc.

#### Configurar remote

Comando:

```bash
git remote add <name_remote> <uri>
```

En la siguiente sección veremos las convenciones para el nombre a dar a `name_remote`.

Ejemplo de crear y configurar un remote en otra carpeta de nuestro ordenador.

```bash
# Crear repositorio remoto.
$ cd /tmp/
$ mkdir remote-git
$ cd remote-git
$ git init
Inicializado repositorio Git vacío en /tmp/remote-git/.git/
# Crear repositorio que se conectará al remoto.
$ cd /tmp/
$ mkdir local-git
$ cd local-git/
$ git init
Inicializado repositorio Git vacío en /tmp/local-git/.git/
$ git remote add origin ../remote-git
# Comprobar repositorio remoto
[x@arch local-git]$ git remote -v
origin  ../remote-git (fetch)
origin  ../remote-git (push)
```

#### Convención nombres para remote

El `source fo truth` es el repositorio padre, las convenciones al darle un nombre son:

- Si remote es el repositorio del proyecto, el repositorio remoto se le llama `origin`.
- Si remote es mi fork de otro poryecto, `origin` es el fork que hemos hecho y `upstream` es el repositorio original.

### Fetch

El comando `git fecth` actualiza los cambios de los repositorios remotos.

Es decir, actualiza lo que tenemos en las ramas que empiezan por `origin/`, pueden verse con el comando `git branch --all` o en la carepta `.git/refs/remotes/origin/`; pero no actualiza las ramas en las que hemos hecho checkout. Por tanto, después de hacer `git fetch` hay que hacer merge para tener los últimos cambios en nuestras ramas.

```bash
git fetch
# Si origin tiene commits que no hemos mergeado en nuestra rama, no aparecerán al hacer:
git log
# En cambio podemos ver que origin sí tiene más commits.
git log origin/main
# Se soluciona mergeando estos commits.
git merge origin/main
```

En lugar de hacer `git fecth` y `git merge`, puede usarse `git pull`.

Si al escribir `git branch -a` vemos ramas en local que empiezan por `remotes/origin/` pero que no existen en origin, podemos eliminarlas con el siguiente comando:

```bash
git fetch origin --prune
```

### Pull

Para hacer `git fecth` y `git merge` de una rama.

```bash
git pull <remote> <branch>
```

#### Añadir tracking information

Imagina que en remote tenemos una rama llamada `foo` y en local también hay una rama llamada `foo`. Al ejecutar `git pull`, git no tiene relacionadas ambas ramas; para poder ejecutarla, hay que indicar esta relación con:

```bash
git branch --set-upstream-to=origin/<origin_branch_name> <local_branch_name>
```

Hay varias opciones para la configuración del tracking info:

```bash
git branch -vv
git status -sb
```

### Push

Sirve para añadir nuestros commits en el repositorio remoto.

Al igual que `pull`, hay que añadir tracking info (explicado en la sección sobre `pull`); puede que git haga esto automáticamente.

### Stash

Stash es una pila, una zona especial de git, para guardar archivos temporales.

Para que todos los cambios desde HEAD se guarden en stash, ejecutamos:

```bash
git stash
```

Podemos dar un mensaje a stash:

```bash
git stash -m "<message>"
```

Mostrar cuántos índices en stash tenemos actualmente en la pila:

```bash
git stash list
```

Mostrar información del stash:

```bash
git stash show [--index <index>]
```

Para recuperar lo que hay ne stash:

```bash
# Recuperar el más actual.
git stash pop
# Recuperar uno específico.
git stash pop --index <index>
```

### squash o squashing

Permite unir varios commits en uno.

### revert

```bash
git revert <commit-ish>
```

#### revert vs restore

Con `restore` modificamos un archivo, obteniendo su versión de un commit anterior.

Con `revert` hacemos un commit en el que deshacemos los cambios de otro commit.

### reset

Soft vs hard:

- soft: ir hacia atrás a un commit y mantener los cambios en el working tree y el index.
- hard: elimina totalmente los cambios en el working tree y el index.

#### Deshacer el último commit manteniendo los archivos modificados

```bash
git reset --soft HEAD~1
```

#### Volver a un commit anterior

<https://stackoverflow.com/questions/4114095/how-to-revert-a-git-repository-to-a-previous-commit/4114122>

```bash
# Al último commit.
git reset --hard HEAD

# A un commit específico.
git reset --hard $COMMIT_ID
```

### worktree

Cuando ejecutamos `git init`, creamos el `main working tree`.

Al hablar de `worktree`, nos referimos a un `linked working tree`; el cual es un tree como el `main working tree` pero simplificado donde `.git` en lugar de ser una copia de la carpeta `.git` del repositorio original, `.git` es un archivo, una referencia a la carpeta `.git` original.

Si estamos trabajando en una rama y tenemos que cambiar a otra, lo normal es que tengamos que utilizar `stash`, o hacer commit y `push` para luego continuar con nuestro trabajo; es decir, debemos realizar alguna acción. Gracias a worktree podemos trabajar con otra rama dejando la actual tal como está, se crea un directorio como una copia de nuestro proyecto.

Cambiar a un worktree es algo rápido, lo que puede llevar más tiempo es que tengamos que llevar a cabo acciones de configuración como por ejemplo crear un entorno virtual de python e instalar dependencias, instalar dependencias de npm, o compilar un proyecto de rust.

#### Comandos worktree

Añadir un worktree:

```bash
git worktree add <path_donde_crearlo>
```

Se usará el `basename` del path (lo que hay tras el último `/`) para crear una rama.

Ejemplo:

```bash
git worktree add ../foo-bar
# En la ruta ../foo-bar se crea el work-tree y la rama `foo-bar`
```

Mostrar worktrees:

```bash
git worktree list
```

Eliminar worktree:

- Opción 1:

```bash
git worktree remove ../<basename>
```

- Opción 2:

```bash
rm -rf <carpeta_del_worktree>
git worktree prune
```

### tags

[Referencia](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

Un tag es un punto inmutable en el histórico de git.

Hay dos tipos:

- lightweight tag: es un puntero a un commit específico, es como una rama que no cambia, que no puede editarse, solo borrarse (en un branch puedes realizar acciones como por ejemplo commits).
- annotated tag: se almacenan como objetos completos en la base de datos de git. Son checksummed, contienen el nombre de quien a creado la tag, el email, fecha y un mensaje. También ofrecen otras opciones como firmarlos gon GPG.

Crear un tag:

```bash
# annotated. Utilizar `-a`. Puede usarse la opción `-m` para dar tambień un mensaje:
git tag -a <tagname>
# lightweight. No utilizar las opciones `-a`, `-s`, o `-m`, solo el tag name:
git tag <tagname>
```

Borrar un tag:

```bash
# Tag local.
git tag -d <tagname>
# Tag remoto.
git push --delete origin <tagname>
```

Listar tags:

```bash
git tag
```

Cambiar a un tag:

```bash
git checkout <tagname>
```

Fetch:

```bash
git fetch --tags
```

Pull:

```bash
git pull --tags
```

Push:

```bash
# Un tag específico:
git push origin <tagname>
# Todas las tags.
git push --tags
```

### cat-file

Permite obtener información de un commit hash. Algunos ejemplos de qué permite:

- Obtener la rama a la que pertenece el commit.
- Analizar los archivos actuales en ese commit y ver su contenido. Por ejemplo con esto podríamos ver el contenido de archivos eliminados.

A continuación, vamos a ver un ejemplo con explicaciones teóricas.

Si ejecutamos el comandos sobre un commit, tenemos:

```bash
$ git cat-file -p b1a7bea
tree 30c0e0197d84a0772d1cb5c0aeb881799bdbd928
parent 6be153f69922aaa141b64880304186f9f3b98b5a
author CarlosAMolina <15368012+CarlosAMolina@users.noreply.github.com> 1715834028 +0200
committer CarlosAMolina <15368012+CarlosAMolina@users.noreply.github.com> 1715834028 +0200

Update wiki git
```

La información mostrada es:

- tree: `tree` indica que se trata de una carpeta.
- parent: id del commit padre al analizado.

Si analizamos `tree` tenemos los archivos y carpetas en ese commit:

```bash
$ git cat-file -p 30c0e01
100644 blob 777be3e2e2b6e8aabd6e817a7e1588b54fe0380a    .gitignore
100644 blob a347037c1839ae4f34a9e3399748fbcdcb2d3de4    CHANGELOG.md
100644 blob b3fbee3f989b71ea85a1d5922c4bebf31fc439d1    LICENSE
100644 blob de6d9dea4245ee0e6ec5365011f7b3d69d945b9f    README.md
040000 tree 8437ea087b6b9727c278e3814e346c1f5737022d    deploy
040000 tree 87ea3d4f0df06354806cb126b8dbf39d87e10aaa    src
```

El término `blob` indica que se trata de un archivo.

Cada commit de git contiene todos los archivos y carpetas en ese momento.

Los 2 primeros caracteres mostrados en el hash de los archivos indica donde se encuentran estos archivo en `.git`. Por ejemplo, para `f7c5a4e5545b761005adc08da8b1efd1499de336` tenemos el archivo:

```bash
.git/objects/f7/c5a4e5545b761005adc08da8b1efd1499de336
```

Si vemos su contenido, será incomprensible, pero podemos verlo con:

```bash
git cat-file -p f7c5a4e5545b761005adc08da8b1efd1499de336
```

Sobre las carpetas con el nombre de los 2 primeros caracteres del hash, he visto que en proyectos con pocos commits, en `.git/object` hay carpetas por cada hash, pero en más grandes no, tal vez se vayan eliminando con el tiempo.

## gitignore

Es posible omitir archivos que ya fueron añadidos en un commit antiguo, explicación [en este enlace](https://stackoverflow.com/a/11366713).

## Resolución casos comunes en git

### Crear repositorio remoto desde un repositorio local

<https://docs.gitlab.com/ee/gitlab-basics/create-project.html>

<https://georgik.rocks/common-mistake-when-creating-new-git-repo>

<https://docs.gitlab.com/ee/api/namespaces.html>

Paso 1. Hacer un commit local para crear el branch master. Este aparece en .git/config como [branch "master"].

Paso 2. Hacer push. Ejm:

```bash
git push --set-upstream https://$DOMINIO/$USER/$REPOSITORY_NAME.git master
```

También, puede crearse utilizando [GitHub CLI, como se explica en este enlace](https://dev.to/techturnip/github-cli-how-to-create-a-repo-from-your-terminal-1aeg).

### Cambiar nombre de un repositorio

#### GitLab

<https://docs.gitlab.com/ee/user/project/settings/>

Ir a configuración del repositorio: https://domain/user/project/edit

Advanced settings > Rename repository: cambiar project name y path.

### Recuperar un archivo de una rama eliminada

Supongamos que hemos creado una rama nueva `bar`, comiteamos un archivo en ella y tras cambiar de rama hemos borrado la rama `bar` sin haber hecho push a origin:

```bash
$ git branch --show-current
foo
$ git checkout -b bar
$ echo hi > hello.md
$ ga .
$ gc "Add file"
$ git checkout foo
$ git branch -D bar
```

El primero paso para recuperar el archivo es utilizar `reflog`:

```bash
$ git reflog
e06f35d (HEAD -> foo) HEAD@{0}: checkout: moving from bar to foo
8ce060e HEAD@{1}: commit: Add file
e06f35d (HEAD -> foo) HEAD@{2}: checkout: moving from foo to bar
e06f35d (HEAD -> foo) HEAD@{3}: commit (initial): Initial commit
```

Gracias a que tenemos el commit en el que se añadió el archivo, podemos reconstruir todo el repositorio en ese punto y recuperar el archivo.

Podemos recuperar el archivo de varias formas.

Una manera es acceder al contenido del archivo:

```bash
$ git cat-file -p 8ce060e
tree fae6fd945e413c7b0d7d8129db4749b3bfa1362a
parent e06f35ddf158dbc6acdf313f27dc11358605b212
author CarlosAMolina <15368012+CarlosAMolina@users.noreply.github.com> 1720981727 +0200
committer CarlosAMolina <15368012+CarlosAMolina@users.noreply.github.com> 1720981727 +0200

Add file
$ git cat-file -p fae6fd945e413c7b0d7d8129db4749b3bfa1362a
100644 blob 9daeafb9864cf43055ae93beb0afd6c7d144bfa4    README.md
100644 blob 45b983be36b73c0788dc9cbcb76cbb80fc7bb057    hello.md
$ git cat-file -p 45b983be36b73c0788dc9cbcb76cbb80fc7bb057
hi
$ git cat-file -p 45b983be36b73c0788dc9cbcb76cbb80fc7bb057 > hello.md
```

Pero una opción mas sencilla es mergear el commit donde se creó el archivo:

```bash
$ git merge 8ce060e
Actualizando e06f35d..8ce060e
Fast-forward
 hello.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 hello.md
```

Esto puede tener un inconveniente; como mergeamos todo el historial de git, podemos traer más cambios de los deseados en caso de que haya distintos commits entre las ramas. Por tanto, la opción más sencilla y segura es usar cherry-pick:

```bash
$ git cherry-pick 8ce060e
[foo 7f22d82] Add file
 Date: Sun Jul 14 20:28:47 2024 +0200
 1 file changed, 1 insertion(+)
 create mode 100644 hello.md
```

### Continous integration

Ver runner en una máquina:

```bash
ps -ef | grep runner
```

## OSINT

### Email de quién realizó el commit

Opción 1.

Añadir `.patch` al final del ID del commit. Ejm:

```bash
https://github.com/$USER/$REPOSITORY_NAME/commit/$COMMIT_ID.patch
```

Opción 2.

Buscar el ID del commit:

```bash
https://api.github.com/repos/$USER/$REPOSITORY_NAME/commits
```

Ver info del commit

```bash
https://api.github.com/repos/$USER/$REPOSITORY_NAME/git/commits/$COMMIT_ID
```

Nota. Es posible configurar que se oculte el mail de quien realizó el commit, pero los commits realizados antes de esta configuración seguirán mostrando el email (<https://help.github.com/en/github/setting-up-and-managing-your-github-user-account/setting-your-commit-email-address>).

## Referencias

Frontend Masters. Everything You'll Need to Know About Git:

- [Curso](https://frontendmasters.com/courses/everything-git)
- [Diapositivas](https://theprimeagen.github.io/fem-git/)
