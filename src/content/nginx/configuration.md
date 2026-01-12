# Nginx configuración

## Contenidos

- [Términos utilizados en archivos de configuración](#términos-utilizados-en-archivos-de-configuración)
  - [Context](#context)
    - [Contexts más importantes](#contexts-más-importantes)
  - [Directive](#directive)
    - [Tipos y herencia](#tipos-y-herencia)
- [Editar archivo nginx.conf](#editar-archivo-nginx.conf)
- [Context](#context)
  - [events](#events)
  - [Context types o directive include](#context-types-o-directive-include)
  - [Context server o virtual host](#context-server-o-virtual-host)
  - [Location blocks](#location-blocks)
    - [Exact match](#exact-match)
    - [Preferencial prefix match](#preferencial-prefix-match)
    - [Regex match](#regex-match)
    - [Prefix match](#prefix-match)
- [Directives](#directives)
  - [include](#include)
  - [listen](#listen)
  - [server_name](#server_name)
  - [root](#root)
  - [index](#index)
  - [user](#user)
  - [return](#return)
  - [rewrite](#rewrite)
    - [Last flag](#last-flag)
  - [try_files](#try_files)
    - [Named locations](#named-locations)
    - [Buffers y timeouts](#buffers-y-timeouts)
- [Variables](#variables)
  - [Variables propias de Nginx](#variables-propias-de-nginx)
  - [Variables que podemos definir](#variables-que-podemos-definir)
- [Condicionales o if statements](#condicionales-o-if-statements)
- [Procesos](#procesos)
- [Respuestas comprimidas](#respuestas-comprimidas)
- [HTTP2](#http2)
- [Server Push](#server-push)
- [Headers](#headers)
- [Limitar peticiones](#limitar-peticiones)
- [Autenticación](#autenticación)
- [Ocultar versión Nginx en las cabeceras de respuesta](#ocultar-versión-nginx-en-las-cabeceras-de-respuesta)
- [Denegar usar página en un frame](#denegar-usar-página-en-un-frame)
- [Logs](#logs)
- [Subdominio](#subdominio)

## Introducción

Tras modificar la configuración de Nginx, es necesario verificar que es correcta (ver apartado con los comandos) y reiniciar el servicio `nginx`.

## Términos utilizados en archivos de configuración

Estos términos se utilizan en archivos de configuración, como por ejemplo en el `nginx.conf`.

Hay dos términos a diferenciar en la configuración de Nginx: `context` y `directive`.

[Recursos](https://udemy.com/course/nginx-fundamentals).

### Context

Son secciones en la configuración. Ejemplo:

```bash
http {
   ...
}
```

En estas secciones se indican los `directive`.

Cada context puede contener otros context. Los context heredan la configuración del context padre.

El context superior es el propio archivo de configuración, llamado `main context`.

#### Contexts más importantes

- http: configura lo relacionado con HTTP.
- server: donde definimos un host virtual.
- location: para gestionar/buscar términos en las peticiones que recibe el servidor.

### Directive

Son opciones de configuración específicas utilizadas en los archivos, formados por un nombre y un valor. Por ejemplo:

```bash
server_name foo.com;
```

En el `main context` configuramos `directives` globales que aplican a todos los procesos.

La lista de directives puede verse en este [link](https://nginx.org/en/docs/dirindex.html).

#### Tipos y herencia

Hay tres tipos de directives.

- Standard Directive
- Array Directive
- Action Directive

En el siguiente ejemplo, se muestran las características de cada tipo (ejemplo obtenido de [este curso](https://udemy.com/course/nginx-fundamentals/).

```bash
events {}

######################
# (1) Array Directive
######################
# Puede especificarse varias veces sin sobreescribir el anterior, por ejemplo `access_log` aplica a cada archivo configurado.
# Lo heredan todos los context hijos.
# Los contexts hijos pueden sobreescribirlo para dentro de dicho context, redeclarando el directive.
access_log /var/log/nginx/access.log;
access_log /var/log/nginx/test.log.gz custom_format;

http {

  # Esto es un Include statement, no un directive.
  include mime.types;

  server {
    listen 80;
    server_name site1.com;

    # Hereda access_log del context padre (1).
  }

  server {
    listen 80;
    server_name site2.com;

    #########################
    # (2) Standard Directive
    #########################
    # Solo puede declararse una vez por contexto. Otra declaración lo sobreescribe.
    # Lo heredan todos los context hijos.
    # Los contexts hijos pueden sobreescribirlo para dentro de dicho context, redeclarando la directiva.
    root /sites/site2;

    # Los context dentro de este context server tienen deshabilitados los logs, hemos sobreescrito la herencia (1).
    access_log off;

    location /images {
      # Utiliza la directive root heredada de (2).
      try_files $uri /stock.png;
    }

    location /secret {
      #######################
      # (3) Action Directive
      #######################
      # Realiza una acción como rewrite or redirect.
      # No se les aplica la herencia ya que modifican el curso normal de la configuración puesto que que la petición se para (redirect/response) or reevalúa (rewrite).
      return 403 "No permission";
    }
  }
}
```

## Editar archivo nginx.conf

En esta sección veremos cómo editar el archivo `nginx.conf`, podemos borrar todo su contenido e ir añadiendo lo que veremos a continuación, donde se describe cómo configurar cada context.

## Context

En este apartado se explican los `context` que pueden configurarse.

### events

Aunque esté vacío, es necesario dejarlo en el archivo para tener una configuración válida:

```bash
event {}
```

### Context types o directive include

Utilizado en el context `http`.

De no tener este directive, Nginx no enviará los archivos `.css` con la cabecera con el MIME type correcto, sino como `Content-Type: text/plain`, puede verificarse haciendo una petición a las cabeceras del archivo:

```bash
curl -I http://1.2.3.4./style.css
```

Se soluciona definiendo el content type para cada extensión de archivos mediante:

```bash
types {
    text/html html
    text/css css;
}
```

En lugar de escribir todos los casos manualmente, puede cargarse el archivo `mime.types` con la directive `include`:

```bash
include mime.types
```

Este archivo posee los content type para diferentes extensiones de archivos y se define utilizando el path relativo a `nginx.conf`, en este casos ambos archivos se encuentran en la misma ruta.

### Context server o virtual host

Utilizado en el context `http`.

En los archivos de configuración, los context `server` dentro del context `http` se conocen como `virtual host`.

Los `virtual host` se utilizan para ofrecer contenido que se encuentra en una ruta de nuestro servidor. Se encargan de escuchar en un puerto.

En el siguiente ejemplo vemos directivas que explicaremos en el apartado de directives.

```bash
server {
    listen 80;
    server_name 1.2.3.4;

    root /home/foo/bar/public_html;
}
```

### Location blocks

El context `location` sirve para interceptar una petición y ofrece alguna respuesta, por ejemplo una redirección, devolver un string, etc.

Tras `location` se indica el prefix match de ser necesario (lo veremos a continuación) y la URI a interceptar.

Hay diferentes modos, cada cual tiene mayor prioridad que el resto, es decir, se ejecutará aunque los otros estén escritos antes en la configuración. De mayor a menor orden de prioridad son:

- Exact match.
- Preferencial prefix match.
- Regex match.
- Prefix match.

#### Exact match

Utiliza el símbolo `=` como el match modifier. Ofrece la respuesta de solicitar específicamente esa URI.

```bash
location = /greet {
    return 200 'Hi from "/greet" location.';
}
```

La URL que obtendrá la respuesta es:

- http://1.2.3.4/greet

#### Preferencial prefix match

Es igual que `prefix match` pero se configura utilizando `^~` y tiene mayor prioridad que el `regex math`.

```bash
location ^~ /greet {
    return 200 'Hi from "/greet" location.';
}
```

#### Regex match

Añadiendo el símbolo `~` como el match modifier, ofreceremos la respuesta de solicitar lo que concuerde con la expresión regular especificada.

Importante, es sensible a mayúsculas y minúsculas, para hacerlo case insensitive, utilizamos `~*`.

Por ejemplo, para responder a `/greet`, case insensitive, seguido de cualquier número del 0 al 9:

```bash
location ~* /greet[0-9] {
    return 200 'Hi from "/greet" location.';
}
```

#### Prefix match

En el siguiente ejemplo, todo lo que empiece por `/greet` devolverá la respuesta indicada.

```bash
location /greet {
    return 200 'Hi from "/greet" location.';
}
```

Ejemplo de URLs que devolverán esa respuesta:

- http://1.2.3.4/greet
- http://1.2.3.4/greeting/foo

## Directives

Explicamos los directives.

### include

Ver apartado de `context types`.

### listen

Especifica el puerto que escucha.

### server_name

Configura el dominio, subdominio o IP para el que aplica el context `server`.

Puede aceptar wildcards como el asterisco, por ejemplo `*.foo.com` aceptará conexiones de cualquier subdominio, como `www.foo.com`, `images.foo.com`, etc.

### root

Es el `path` principal desde el que Nginx gestionará las peticiones.

Por ejemplo, si tenemos configurado `root /home/foo/bar/public_html;`, cuando un cliente web acceda a `http://1.2.3.4./images/dog.png`, el servidor Nginx recibe la petición `/images/dog.png` y buscará el archivo `/home/foo/bar/public_html/images/dog.png`.

[Recursos](https://www.nginx.com/blog/setting-up-nginx/)

### index

Indica el archivo que ofrecer si la petición apunta a un directorio.

Por defecto es `index.html` por lo que, al solicitar nuestro dominio, se nos ofrece directamente este archivo.

Se le pasan varios argumentos, en orden de prioridad con que ofrecerlos. Ejemplo:

```bash
...
server {
    ...
    root /foo/bar/
    index index.php index.html;
    ...
}
```

### user

Para configurar el usuario con el que Nginx ejecuta el proceso de servidor web.

Se especifica en el main context, por ejemplo, en la primera línea de la configuración, fuera de cualquier directive.

Por ejemplo, para ejecutar el proceso como usuario `wwww-data`:

```bash
user www-data;
...
```

Tras recargar el servicio, con `ps aux | grep nginx` puede verificarse el usuario empleado.

### return

Toma un `status code` y el string a devolver, pero si el `status code` es un código de redirección (los del rango 300), en lugar de indicarle un string, hay que pasarle la URL (absoluta o relativa) a la que el cliente será redirigido.

Por ejemplo, para devolver una imagen al visitar el path `/logo`:

```bash
location /logo {
    return 307 /image.png;
}
```

El usuario que visite `http://1.2.3.4/logo` será redirigido a `http://1.2.3.4/image.png`.

### rewrite

Tras `rewrite` se especifica la expresión regular para matchear el path con el que trabajar y el segundo argumento en la URL a utilizar en la respuesta.

En lugar de redirigir el cliente a otra URL, la URL no cambia, pero el contenido mostrado sí corresponde a otra.

Ejemplo:

```bash
server {
    ....
    rewrite ^/user/\w+ /greet;

    location /greet {
        return 200 "Hi user";
    ...
}
```

El directive rewrite consume más recursos de Nginx porque modifica la URL enviada por el usuario con la nueva URL especificada y el servidor la trata como una nueva petición, volviéndola a procesar de nuevo (internamente se cambia la petición y se vuelve a analizar toda la configuración desde el inicio del context `server`). En el ejemplo de configuración anterior, cuando el usuario visite `http://1.2.3.4/user/john`, la parte `/user/john` se convierte en `greet` y matcheará el location especificado después; el usuario seguirá viendo en el navegador `http://1.2.3.4/user/john`.

También, se puede trabajar con partes específicas del resultado de la expresión regular, por ejemplo, para tomar el nombre del usuario:

```bash
rewrite ^/user/(\w+) /greet/$1;

# All users except `john` (defined later) will match this.
location /greet {
    return 200 "Hi user";
}

# Only the user `jonh` will match this.
location /greet/john {
    return 200 "Hi John";
}
```

El resultado del primer grupo matcheado se accede con `$1`, el segundo con `$2` (ejemplo `rewrite ^/user/(\w+)/(something) /greet/$1 $2;`), etc.

#### Last flag

Para evitar que una URL sea modificada por `rewrite` más de una vez, en el siguiente ejemplo, la petición a `/user/john` sería modificada a `/greet/john` y luego esta a `/thumb.png`

```bash
server {
    ....
    rewrite ^/user/(\w+) /greet/$1;
    rewrite ^/greet/john /thumb.png;
    ....
}
```

Añadiendo `flag` al final de la línea con `rewrite` evitamos que se apliquen más `rewrite` en la petición. En el ejemplo anterior, la petición quedaría como `/greet/john`.

### try_files

Puede utilizarse en `server` y `location` contexts.

```bash
server {
    ....
    root /files
    ....
    try_files /image.png /image2.png /page_404;

    location /page_404 {
        return 404 "Not found";
    }
}
```

La configuración anterior revisa si `/files/image.png` existe, de ser así lo devuelve pero en caso contrario, intenta devolver `/files/image2.png`; de nuevo, lo devolverá de existir o pasará a lo siguiente especificado. Lo último configurado, en este caso `/page_404`, se gestiona como una directiva `rewrite`, como vimos, se modificará la petición internamente y volverá a procesarse en el context `server`.

Suele utilizarse con variables, por ejemplo, con la `$uri` devolveremos cualquier path solicitado si existe, en caso contrario se pasa al resto de opciones.

```bash
server {
    ....
    root /files
    ....
    try_files $uri /image.png /page_404;
    ....
}
```

Hay que tener en cuenta que `try_files`, excepto para la última opción especificada que es un `rewrite`, el resto se gestionan relativas a  lo definido en `root`. En el siguiente ejemplo, si el usuario accede a `http://1.2.3.4/greet`, no se ejecutará el `location` `/greet` sino que se buscará en el servidor `/files/greet` y si no existe se pasa a lo siguiente configurado en `try_files`.

```bash
server {
    ....
    server 1.2.3.4
    root /files
    ....
    try_files /image.png /greet /page_404;

    location /greet {
        return 200 "Hi";
    }

    location /page_404 {
        return 404 "Not found";
    }
}
```

#### Named locations

Como lo último especificado en `try_files` se gestiona como un rewrite y la petición volverá a evaluarse, esto lo evitamos con los named locations.

Asignamos un nombre a un context `location`, de ese modo el directive `try_files` lo llama directamente.

Para utilizarlo, se añade `@` en `try_files` y en `location`. Ejemplo:

```bash
server {
    ....
    server 1.2.3.4
    root /files
    ....
    try_files /image.png /greet @page_404;

    location /greet {
        return 200 "Hi";
    }

    location @page_404 {
        return 404 "Not found";
    }
}
```

#### Buffers y timeouts

Buffering significa utilizar la memoria RAM, o el disco duro de no haber suficiente RAM, para almacenar peticiones y respuestas; así pueden utilizarse desde memoria.

Los timeouts es el tiempo máximo permitido para un evento; por ejemplo, al recibir una petición de un cliente, terminar después de unos segundos. Esto protege al servidor de peticiones muy largas que puedan estropear su funcionamiento.

Al contrario que al configurar los procesos, la configuración de los buffers y los timeouts no depende de las características del servidor, sino de las peticiones recibidas y respuestas dadas.

En caso de no estar seguro de qué configuración utilizar, es mejor dejar los valores por defecto.

Los directives pueden escribirse en el context `http`.

En el siguiente ejemplo (obtenido de [este curso](https://udemy.com/course/nginx-fundamentals))se muestran directives para buffer y timeouts:

```bash
http {
  ...
  # Buffer size for POST submissions.
  client_body_buffer_size 10K;
  client_max_body_size 8m;
  # Dar un mayor valor de client_max_body_buffer_size que el necesario implica perder memoria en el servidor. De ser pequeño, se esribirán los datos en el disco, lo cual es muy lento.
  # De recibir una petición POST con un body mayor al client_max_body_size, el servidor responderá con error 413 `Request Entity too Large`. Sirve de protección contra peticiones maliciosas que por ejemplo afecten al rendimiento del servidor.

  # Buffer size for Headers.
  client_header_buffer_size 1k;
  # 1k es suficiente para la mayoría de peticiones.

  # Max time to receive client body/headers.
  client_body_timeout 12;
  client_header_timeout 12;
  # El client_body_timeout se refiere al tiempo ente peticiones consecutivas de lectura.

  # Max time to keep a connection open.
  keepalive_timeout 15;
  # Útil si un cliente solicita muchos archivos ya que ahorra tener que crear nuevas conexiones.
  # Tener en cuenta que no sea muy largo para no sobrepasar el número de máximas peticiones (`worker_processes` x `worker_connections`).

  # Max time for the client accept/receive a response
  send_timeout 10;

  # Skip buffering for static files
  sendfile on;
  # Esto da un mejor rendimiento.

  # Optimise sendfile packets.
  tcp_nopush on;

  server {

    listen 80;
    ...
  }
}
```

Las unidades pueden verse en este [link](https://nginx.org/en/docs/syntax.html). De no especificar unidades, por defecto el tamaño son bytes y el tiempo milisegundos.

## Variables

Nginx tiene 2 tipos de variables:

### Variables propias de Nginx

Ejemplo: `$http`, `$uri`, `$args`.

La lista de variables de Nginx puede verse en este [link](https://nginx.org/en/docs/varindex.html); entre paréntesis se indica el módulo que permite utilizarlas.

Ejemplo uso en un string:

```bash
location /whoami {
    return 200 "$host\n$uri\n$args\nName: $arg_name";
}
```

Visitar la URL "http://1.2.3.4/whoami?name=foo" devolverá:

```bash
2.8.2.1
/whoami
name=foo
Name: foo
```

Se observa cómo con `arg_...` podemos acceder al valor de los argumentos.

### Variables que podemos definir

Se especifican indicando su nombre (iniciado con el símbolo dolar) y luego el valor.

Pueden ser de tipo:

- Booleano.
- String.
- Números enteros.

Ejemplo:

```bash
set $is_holiday true;
set $user_name 'foo';
set $min_age 18;
```

## Condicionales o if statements

No deben usarse dentro de los context `location` ya que provocan comportamientos inesperados, mas información en el [link](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/).

Ejemplo para devolver pantalla de error de no incorporar el API Key correcto en la petición:

```bash
if ( $arg_apikey != 123 ) {
    return 401 "Incorrect API Key";
}

location ...
```

Pueden utilizarse expresiones regulares con `~`. Ejemplo:

```bash
set $weekend 'No';

if ( $date_local ~ 'Saturday|Sunday' ) {
    set $weekend 'Yes';
}

location /is_weekend {
    return 200 $weekend
}
```

## Procesos

Ver [procesos](processes.html).

## Respuestas comprimidas

Sirven para reducir el tamaño de las respuestas al solicitar recursos estáticos, lo que también reduce el tiempo para recibirlas.

Si el cliente es capaz de gestionar respuestas comprimidas (prácticamente todos los navegadores modernos pueden), el servidor comprime el recurso antes de enviarlo y, una vez en el cliente, este descomprime lo recibido.

Uso:

```bash
http {
    ...
    gzip on;
    gzip_comp_level 3;
    gzip_types text/css;
    gzip_types text/javascript;
    ...

    location = file-to-compress.txt {
        add_header Vary Accept-encoding;
    }

}
```

Del ejemplo anterior:

- `gzip on`: activar el directive del módulo `gzip`.
- `gzip_comp_level`: nivel de compresión. A mayor nivel de compresión, el archivo tendrá menor tamaño pero se necesitan más recursos del servidor para crearlo. El valor mínimo es 0 (tamaño original) y a partir del valor 5 no se nota una gran mejora en el tamaño, por lo que son buenas opciones el nivel de compresión 3 y 4.
- `gzip_types`: a quién aplicar la compresión.
- `add_header Vary Accept-encoding`: esta cabecera es necesaria ya que el cliente debe indicar en la petición si quiere recibir respuestas comprimidas, para ello envía la cabecera `Accpet-Encoding`.

Por ejemplo, para que un cliente acepte archivos comprimidos en `gzip`:

```bash
curl -I -H "Accept-Encoding: gzip" http://localhost:8080/file-to-compress.js
```

A las cabeceras recibidas se les ha añadido la cabecera `Content-Encoding: gzip`.

Podemos ver la diferencia de tamaño transmitido, sin comprimir son 892 Bytes y 293 con compresión:

```bash
$ curl http://localhost:8080/file-to-compress.js > /tmp/no-compressed.js
% Received
892

$ curl -H "Accept-Encoding: gzip" http://localhost:8080/file-to-compress.js > compressed.js
% Received
293
```
Documentación del módulo `gzip` en el siguiente [link](https://nginx.org/en/docs/http/ngx_http_gzip_module.html).

## HTTP2

Ver [http2](http2.html).

## Server Push

Ver [server push](server-push.html).

## Headers

Ver apartado [headers](headers.html).

## Limitar peticiones

Ver [rate-limiting](rate-limiting.html).

## Autenticación

Ver [Autenticación](authentication.html).

## Ocultar versión Nginx en las cabeceras de respuesta

Por defecto, Ningx devuelve en las cabeceras de respuesta su versión:

```bash
$ curl -Ik https://localhost:8080
....
server: nginx/1.23.3
....
```

Para evitarlo, añadimos a la configuración:

```bash
http {
    ...
    # Avoid send the Nginx version in the reponse headers.
    server_tokens off;
    ...
}
```

De este modo obtenemos:

```bash
$ curl -Ik https://localhost:8080
....
server: nginx
....
```

## Denegar usar página en un frame

Es bueno evitar esto en ciertas páginas, como por ejemplo las de login.

Ejemplo de configuración para todas las páginas del servidor:

```bash
server {
    ...
    # Frame configuration.
    # SAMEORIGIN: the page can be in a frame if the frame is in the same site as the page.
    add_header X-Frame-Options "SAMEORIGIN";
    ...
}
```

Con el inspector del navegador web, en la pestaña `Console` veremos el siguiente error al incluir páginas en un frame no permitido:

```bash
The loading of “https://localhost:8080/no-allowed-in-frame.html” in a frame is denied by “X-Frame-Options“ directive set to “SAMEORIGIN“.
```

## Logs

Ver apartado [logs](logs.html).

## Subdominio

Primero, en la web del proveedor donde compramos el dominio (por ejemplo [OVH](https://www.ovh.com/)), en la configuración de la zona DNS añadimos un registro de tipo A, indicando el nombre del subdominio y el destino (la dirección IP).

En los archivos de configuración de Nginx, donde tenemos la configuración de nuestro dominio, por ejemplo `conf.d/cmoli.conf`, creamos un nuevo archivo, ejemplo `conf.d/wiki.cmoli.conf` y añadimos la información necesaria.

Finalmente con Certbot, creamos un certificado para que el subdomino funcione por HTTPS.

Verificamos que la configuración es correcta y reiniciamos Nginx.
