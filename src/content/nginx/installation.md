# Nginx instalación

## Contenidos

- [Instalación desde repositorios oficiales](#instalación-desde-repositorios-oficiales)
- [Instalación building from sources](#instalación-building-from-sources)
  - [Descargar código](#descargar-código)
  - [Instalar requisitos](#instalar-requisitos)
  - [Configurar instalación](#configurar-instalación)
  - [Compilar configuración](#compilar-configuración)
  - [Instalar el archivo compilado](#instalar-el-archivo-compilado)
  - [Añadir Nginx como un servicio](#añadir-nginx-como-un-servicio)

## Instalación desde repositorios oficiales

Es recomendable realizar la instalación no con este método sino mediante `building from sources` para tener la última versión, más opciones de configuración y añadir módulos.

```bash
sudo apt install nginx
```

## Instalación building from sources

Deben realizarse los siguientes apartados en orden.

Es recomendable parar el servicio nginx antes de la instalación.

[Recursos](https://udemy.com/course/nginx-fundamentals)

### Descargar código

Desde este [link](https://nginx.org/en/download.html) descargamos la rama `mainline`, que es la [que recomiendan](https://www.nginx.com/blog/nginx-1-18-1-19-released/) ya que la rama `stable` se actualiza con menor regularidad:

```bash
wget https://nginx.org/download/nginx-1.23.3.tar.gz
tar -zxvf nginx-1.23.3.tar.gz
cd nginx-1.23.3
```

### Instalar requisitos

Ejecutamos el comando:

```bash
./configure
```

Iremos obteniendo errores de librerías que necesitamos instalar:

```bash
# Error por no tener instalado compilador
sudo apt install build-essential
# Falta librería PCRE
sudo apt install libpcre3 libpcre3-dev
# Falta librería zlib
sudo apt install zlib1g zlib1g-dev
# Para dar soporte a https (esto no se mostrará como error de algo faltante)
sudo apt install libssl-dev
```

### Configurar instalación

Las opciones de configuración disponibles se ven con:

```bash
./configure --help
```

La descripción de cada opción puede consultarse en este [link](https://nginx.org/en/docs/configure.html).

Realizamos la configuración:

```bash
./configure \
    --without-http_autoindex_module \
    --sbin-path=/usr/local/nginx \
    --conf-path=/usr/local/nginx/nginx.conf \
    --error-log-path=/usr/local/nginx/logs/error.log \
    --http-log-path=/usr/local/nginx/logs/access.log \
    --with-pcre \
    --pid-path=/usr/local/nginx/nginx.pid \
    --with-http_ssl_module \
    --with-http_v2_module \
    --modules-path=/usr/local/nginx/modules
```

Las opciones utilizadas han sido:

- `--without-http_autoindex_module`: omitir este módulo en la instalación (ver sección de módulos).
- `--sbin-path`: path del ejecutable que inicia y para el servidor Nginx.
- `--conf-path`: path del archivo de configuración.
- `--error-log-path`: path del log de error.
- `--http-log-path`: path de logs HTTP del servidor.
- `--with-pcre`: utilizar la librería PCRE del sistema para expresiones regulares.
- `--pid-path`: archivo con el ID del proceso principal.
- `--with-http_ssl_module`: módulo que añadiremos durante la instalación.
- `--modules-path`: path para los módulos que instalemos.

### Compilar configuración

```bash
make
```
### Instalar el archivo compilado

```bash
sudo make install
```

Revisamos que los archivos utilizados en la configuración existen:

```bash
ls /usr/local/nginx
```

Para mostrar la versión instalada y la configuración empleada:

```bash
/usr/local/nginx/nginx -V
```

### Añadir Nginx como un servicio

Esto nos permitirá iniciar, parar, etc. el servidor sin tener que llamar al ejecutable usando toda su ruta.

Utilizamos el script del siguiente [link](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/) (ejemplo obtenido de este [enlace](https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/)). Guardamos su contenido en el archivo mostrado en el enlace.

Como explica el enlace, puede que haya que escribir la ruta correcta de PIDFile y del ejecutable de Nginx:

```bash
sudo vi /lib/systemd/system/nginx.service
```

Podemos iniciar el servicio con:

```bash
sudo systemctl start nginx.service
```

De obtener el [error](https://askubuntu.com/questions/710420/why-are-some-systemd-services-in-the-masked-state) `Failed to start nginx.service: Unit nginx.service is masked.`, lo solucionamos con:

```bash
systemctl unmask nginx.service
```

Configuramos que se inicie el servicio cada vez que se inicie el servidor:

```bash
sudo systemctl enable nginx.service
```

Más opciones de `systemctl` pueden consultarse en el apartado de comandos.
