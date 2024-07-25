# Nginx HTTP2

## Contenidos

- [Introducción](#introducción)
- [Configuración](#configuración)
  - [Instalación y activación](#instalación-y-activación)
  - [Certificado SSL](#certificado-ssl)
    - [Certificado SSL autofirmado](#certificado-ssl-autofirmado)
    - [Certificado SSL con Let s Encrypt y Certbot](#certificado-ssl-con-let-s-encrypt-y-certbot)
      - [Renovación del certificado](#renovación-del-certificado)
        - [Renovación del certificado si fue creado manualmente](#renovación-del-certificado-si-fue-creado-manualmente)
        - [Renovación del certificado manualmente](#renovación-del-certificado-manualmente)
        - [Renovación del certificado automáticamente](#renovación-del-certificado-automáticamente)
    - [Configurar SSL en Nginx](#configurar-ssl-en-nginx)
  - [Evitar escuchar HTTP](#evitar-escuchar-http)
  - [Mejorar la seguridad](#mejorar-la-seguridad)
    - [Deshabilitar SSL](#deshabilitar-ssl)

## Introducción

Diferencias con HTTP1:

- HTTP1 es un protocolo que envía la información como texto mientras que HTTP2 lo hace en formato binario, lo que reduce los errores.
- HTTP2 comprime las cabeceras de respuesta, lo que reduce los tiempos de transferencia.
- HTTP2 utiliza conexiones persistentes y multiplex streaming. Con HTTP1 cada recurso (archivo html y las imágenes, estilos css, código js que utiliza, etc.) solicitado necesita una petición distinta (simple streaming), lo que implica tiempo para iniciar y terminar la petición. Por el contrario, HTTP2 crea una conexión y en ella se envían varios recursos, ya que en un string de datos binario puede comprimirse; por ejemplo, la conexión pide el html y luego en la misma conexión se piden los recursos css, js, etc. que necesita y estos se envían juntos.
- HTTP2 puede utilizar server push, es decir, que en la respuesta en que se envía el archivo html, también se envíen los archivos css, js, etc. asociados.

## Configuración

### Instalación y activación

HTTP2 necesita:

- Instalar el módulo `http_v2_module` de Nginx.
- SSL.

Ejemplo de configuración:

```bash
http {
    server {
        listen 443 ssl http2;
        ....
    }
}
```

Con esto, al ver las peticiones en nuestro navegador web, podemos ver la versión HTTP/2.

La mayoría de navegadores soporta HTTP2; en caso contrario, la página se devolverá como HTTP1.

Ejemplo:

```bash
$ curl -Ik https://localhost:8080/
HTTP/2 200
...
```

### Certificado SSL

Podemos crear un certificado autofirmado, o utilizando servicios externos como Let's Encrypt.

No es recomendable utilizar un certificado autofirmado ya que en el navegador web se tendrá un mensaje de aviso de seguridad.

#### Certificado SSL autofirmado

Es un certificado creado y firmado por nosotros.

Podemos generarlo con el siguiente comando; cuando lo ejecutemos, deberemos responder unas preguntas, podemos dejar las opciones por defecto.

```bash
mkdir /tmp/ssl
openssl req -x509 -nodes -days 1 -newkey rsa:2048 -keyout /tmp/ssl/nginx-selfsigned.key -out /tmp/ssl/nginx-selfsigned.crt
```

Del comando anterior:

- req: pedir un certificado.
- x509: el estándar solicitado.
- days: número de días para los que el certificado será válido.
- nodes: evitar passphrase o password para el key file.
- newkey rsa:2048: generar una nueva private key, de tipo rsa de 2048 bytes.
- keyout: dónde guardar la clave privada.
- out: dónde guardar el certificado.

Documentación del comando en este [link](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04).

#### Certificado SSL con Let s Encrypt y Certbot

Gracias a [Let's Encrypt](https://letsencrypt.org/) podemos tener un certificado para nuestro sitio web; más información sobre Let's Encrypt en [este enlace](https://www.digitalocean.com/community/tutorials/an-introduction-to-let-s-encrypt).

Instalamos Cerbot (documentación en [este link](https://eff-certbot.readthedocs.io/en/stable/)) siguiendo las instrucciones de [su sitio web](https://certbot.eff.org/). Por ejemplo, para [Debian 10](https://certbot.eff.org/instructions?ws=nginx&os=debianbuster):

```bash
sudo apt update
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Debido a que al instalar Nginx utilizamos un path personalizado, no podemos utilizar el plugin Nginx de Certbot que configura Nginx automáticamente, por lo que tendremos que hacer varios pasos manualmente.

En este [enlace](https://community.letsencrypt.org/t/how-to-change-nginx-config-path/110180) recomiendan no utilizar el plugin de Nginx pese a que indiquemos manualmente los paths personalizados.

Para crear los archivos a utilizar con las directivas `ssl_certificate` y `ssl_certificate_key`, seguimos las instrucciones indicadas al ejecutar:

```bash
sudo certbot certonly --manual
```

Cuando el anterior comando nos pregunte por los dominios para los que crear el certificado, si queremos crearlo también para el subdominio `www`, debemos especificarlo. Ejemplo `cmoli.es,www.cmoli.es`.

De haber utilizado el plugin de Certbot para Nginx, habría configurado las siguientes directivas (la mayoría de ellas creando el archivo de configuración `/etc/letsencrypt/options-ssl-nginx.conf`):

- `ssl_session_cache`
- `ssl_session_timeout`
- `ssl_protocols`
- `ssl_prefer_server_ciphers`
- `ssl_ciphers`
- `ssl_dhparam`

Podemos configurarlas utilizando esta web <https://ssl-config.mozilla.org/>, como recomiendan en este [link](https://community.letsencrypt.org/t/generating-options-ssl-nginx-conf-and-ssl-dhparams-in-certonly-mode/136272). Solamente debemos indicar la versión de Nginx y de OpenSSL (se obtiene con el comando `openssl version`).

Información sobre los archivos creados puede verse en el siguiente archivo:

```bash
less /etc/letsencrypt/renewal/cmoli.es.conf
```

##### Renovación del certificado

El certificado caduca cada 90 días, el motivo está explicado en [su sitio web](https://letsencrypt.org/docs/faq/#what-is-the-lifetime-for-let-s-encrypt-certificates-for-how-long-are-they-valid); por lo que deberemos renovarlo manual o automáticamente.

###### Renovación del certificado si fue creado manualmente

De haber creado el certificado manualmente (con la opción `--manual`), no podrá ser renovado con los métodos indicados a continuación, hay que repetir el mismo proceso que se realizó para crear el certificado. Esto viene indicado en el siguiente [link](https://github.com/certbot/certbot/issues/7489#issuecomment-548971511).

Para [ver los dominios que tenemos configurados](https://stackoverflow.com/questions/59206631/how-can-i-see-all-domains-in-my-ssl-certificate-made-by-certbot), leemos la línea `Domains` que muestra el siguiente comando:

```bash
sudo certbot certificates
```

Procedemos con su renovación [de este modo](https://github.com/certbot/certbot/issues/7489#issuecomment-548971511), nos guiará sobre unas acciones a realizar, donde deberemos especificar los dominios a renovar:

```bash
sudo certbot certonly --manual
```

Tras esto, debemos reiniciar el servidor Nginx.

###### Renovación del certificado manualmente

Esta opción puede utilizarse si no hemos creado el certificado manualmente; de haberlo creado manualmente, leer el apartado correspondiente para proceder con la renovación.

Simplemente ejecutamos:

```bash
sudo certbot renew
```

Aquellos certificados próximos a expirar, serán renovados y, el resto serán omitidos.

Si queremos hacer una prueba para verificar que el certificado se renovaría correctamente, podemos hacer una simulación con:

```bash
sudo certbot renew --dry-run
```

El anterior comando ejecutará todos los pasos pero sin renovar el certificado.

###### Renovación del certificado automáticamente

Esta es una muy buena opción para no tener que renovar el certificado cada cierto tiempo.

Como hemos visto, con el comando `sudo certbot renew` solo se renuevan los certificados próximos a caducar, por lo que podemos hacer que ese ejecute diariamente en un cron:

```bash
@daily certbot renew
```

#### Configurar SSL en Nginx

Para utilizar el certificado SSL, indicamos el puerto y el módulo a utilizar por la directiva listen:

En el siguiente ejemplo, utilizamos el certificado autofirmado.

```bash
...
http {
    ...
    server {
        listen 443 ssl;
        ...
        ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
    }

}
...
```

Para hacer petición con curl, es necesario permitir el certificado autofirmado mediante la opción `k`:

```bash
curl -Ik https://localhost:8080/
```

### Evitar escuchar HTTP

Aunque tengamos HTTP2 funcionando, si al acceder a la URL con HTTP en lugar de HTTP2 esta devuelve respuesta, podemos evitarlo redirigiendo todas las peticiones HTTP a HTTP2, una manera de conseguirlo es añadir un nuevo server context:

```bash
# Redirect all trafic to HTTPS.
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}
```

De la anterior configuración:

- `server_name`: se le da el mismo valor que el que tengamos para el context server con puerto 443.
- Puede utilizarse `$host`, `$server_name` o el mismo valor que en `server_name` (IP o dominio).

### Mejorar la seguridad

#### Deshabilitar SSL

En este apartado se explica cómo mejorar el cifrado de las conexiones.

Respecto el protocolo que cifra las conexiones, el protocolo SSL (Secure Sockets Layer) ha sido reemplazado por TLS (Transport Layer Security).

Configuración:

```bash
server {
    ...
    # Disable SSL.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimise cipher suits.
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enable DH Params.
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS.
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;


    # SSL sessions.
    ssl_session_cache shared:SSL:40m;
    ssl_session_timeout 4h;
    ssl_session_tickets off;
    ...
}
```

De la anterior configuración:

- `ssl_protocols`: de no especificarlos, Nginx tiene configurados valores por defecto ([link](https://nginx.org/en/docs/http/configuring_https_servers.html#compatibility)).
- `ssl_prefer_server_ciphers`: dar prioridad a los ciphers del servidor sobre los del cliente al utilizar los protocolos SSLv3 y TLS. En este [link](https://serverfault.com/questions/997614/setting-ssl-prefer-server-ciphers-directive-in-nginx-config) se explica si utilizar `on` o `off`.
- `ssl_ciphers`: cada uno es separado con `:`, para no usar un cipher, utilizamos `:!`. Esta configuración va cambiando por el tiempo (pueden encontrarse bugs, etc) por lo que hay que buscar una fuente fiable y utilizar los valores que mejor resultados den en la actualidad. De no especificarlos, Nginx tiene configurados valores por defecto ([link](https://nginx.org/en/docs/http/configuring_https_servers.html#compatibility)).
- `ssl_dhparam`: mejora la seguridad en el intercambio de keys entre el cliente y el servidor. Artículos sobre esto en [link](https://hackernoon.com/algorithms-explained-diffie-hellman-1034210d5100) y [link](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Para generar el archivo necesario, utilizamos `openssl` (documentación en este [link](https://wiki.archlinux.org/title/OpenSSL)):

    ```bash
    openssl dhparam -out /tmp/dhparam.pem 2048
    ```

    Del anterior comando:
    - Tamaño 2048: debe coincidir con el valor que utilizamos para crear la clave privada en el certificado SSL autofirmado.

    Como se explicó anteriormente, también se puede obtener el archivo como se explica en esta web <https://ssl-config.mozilla.org/>.

- `add_header Strict-Transport-Security`: cabecera que indica al navegador no cargar nada por HTTP. Esto minimiza las redirecciones del puerto 80 al 443.

    El valor de `max-age` son segundos y especifica el tiempo que le navegador debe recordar que el sitio web funciona solo con HTTPs. Documentación en este [link](https://developer.mozilla.org/es/docs/Web/HTTP/Headers/Strict-Transport-Security).

    Con `includeSubDomains`, la política HSTS aplicará a todos los subdominios.

    La opción `always`, asegura añadir la cabecera a todas las respuesta, incluso a las internas de error. Explicado en este [link](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/).

- `ssl_session_cache`. Documentación en este [link](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache). Guarda en caché los handshakes SSL durante el tiempo especificado, lo que hace las conexiones más rápidas. Sus opciones son:
  - Cómo realizar el caché:
      - `builtin`: específico para un worker process, no muy útil
      - `shared`: la sesión cacheada puede ser utilizada por cualquier worker process.
  - `SSL`: nombre dado al caché (para tipo `shared`).
  - 40m: tamaño del caché de tipo `shared` en bytes (la configuración de la caché `builtin` cambia un poco).
- `ssl_session_timeout`: tiempo que mantener una sesión cacheada.
- `ssl_session_tickets`: ofrece mejor rendimiento ya que es un modo de identificar la sesión SSL del navegador web y evitar leer las sesiones cacheadas del servidor para identificarla. Se recomienda deshabilitarlo ([link](https://github.com/mozilla/server-side-tls/issues/135)).
