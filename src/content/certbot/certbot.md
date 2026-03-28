# Certbot

## Certificado manual

Explicado en la [configuración de Nginx](https://cmoli.es/wiki/content/nginx/http2.html#certificado-ssl-con-let-s-encrypt-y-certbot).

## Certificado renovado automáticamente

La idea es:

- Certbot (webroot). Renueva el certificado y configura la task para ejecutarse periódicamente.
- systemd timer. Inicia la renovación periódicamente.
- deploy hook. Copia el certificado a la ruta utilizada por el servidor web y reinicia el servidor.
- Servidor web. Debe ofrecer el contenido web y responder al path del challenge de Let's Encrypt.

### Renovar certificado automáticamente

Primero, en el VPS creamos el directorio donde se almacenará el challenge y permitimos que el usuario que ejecuta el servidor web pueda acceder a él:

```bash
sudo mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
sudo chown -R www-data:www-data /var/www/letsencrypt
```

Nota, el path `/var/www/letsencrypt` lo definimos a continuación al ejecutar `cerbot` con `--webroot`, este valor se guardará como `webroot_path`, más adelante se explica cómo ver este valor una vez configurado.

El servidor web debe ofrecer el contenido de este path:

```
/var/www/letsencrypt/.well-known/acme-challenge/
```

Es decir, se debe ofrece el contenido del anterior path cuando se reciban peticiones a:

```
http://example.com/.well-known/acme-challenge/*
```

Es importante que la petición HTTP reciba respuesta; no importa si la petición HTTP se rediga a una HTTPs, lo importante es que finalmente se tenga respuesta al hacer la petición sobre HTTP, que es lo que se utiliza en el challenge.

Solicitamos el nuevo certificado:

```bash
sudo certbot certonly \
  --webroot \
  -w /var/www/letsencrypt \
  -d example.com \
  -d www.example.com \
  -d wiki.example.com
```

Con este comando:

- Se ha generado un nuevo certificado para todos los dominios indicados.
- Certbot ha configurado una task programada para que se renueve automáticamente. Ahora `certbot renew` funciona automáticamente.

Para confiar que la autorenovación está programada, debe aparece información al ejecutar:

```bash
systemctl list-timers | grep certbot
```

Para comprobar que la renovación funciona, sin llevarla a cabo de manera efectiva, ejecutamos el siguiente comando (a continuación comentamos posibles errores) y debe funcionar sin error:

```bash
sudo certbot renew --dry-run
```

En caso de error, por ejemplo si tenemos este resultado:

```bash
Certbot failed to authenticate some domains (authenticator: webroot). The Certificate Authority reported these problems:
  Identifier: example.com
  Type:   unauthorized
  Detail: 1.2.3.4: Invalid response from http://example.com/.well-known/acme-challenge/Bb8Sf9Y5dvx7dcNlIiHjZFAPOTAA9nuct6LYG3zwfYg: 400

  Identifier: wiki.example.com
  Type:   unauthorized
  Detail: 1.2.3.4: Invalid response from http://wiki.example.com/.well-known/acme-challenge/PRuNyiIpmP_MU1CyLwUfigz13MnTKEOUhgof_qfGOp8: 400

  Identifier: www.example.com
  Type:   unauthorized
  Detail: 1.2.3.4: Invalid response from http://www.example.com/.well-known/acme-challenge/AnKGW2pYvTgAGY6dIXcyPC9qcOcNfBZl3XFHI5BRQIo: 400
```

Una causa puede ser que no estemos sirviendo la ruta que configuramos en Certbot para `webroot_path`:

```bash
sudo cat /etc/letsencrypt/renewal/example.com.conf | grep webroot_path
# webroot_path = /var/www/letsencrypt,
```

Es decir, cuando recibamos las peticiones de renovación a `example.com/.well-known/acme-challenge/*`, nuestro servidor debe ofrecer lo que hay en `webroot_path`, en el caso del ejemplo anterior, lo que hay en `/var/www/letsencrypt`.

Los siguientes comandos ayudan para ver logs:

```bash
# Aumentar número de logs
sudo certbot renew -v --dry-run
# Ver logs
sudo view less /var/log/letsencrypt/letsencrypt.log
```

### Copiar certificados automáticamente

El último paso es que nuestro servidor pueda acceder a los nuevos certificados; para ello empleamos hooks.

```bash
sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy
sudo touch /etc/letsencrypt/renewal-hooks/deploy/web-server.sh
# Make executable
sudo chmod 750 /etc/letsencrypt/renewal-hooks/deploy/web-server.sh
```

El contenido de `web-server.sh` debe ser (actualizar valor de DOMAIN):

```bash
#!/bin/sh
set -eu

DOMAIN="example.com"
SRC="/etc/letsencrypt/live/$DOMAIN"
DST="/etc/web-server/tls"

# Ensure destination exists
install -d -o root -g www-data -m 750 "$DST"

# Copy certs with correct permissions
install -o root -g www-data -m 640 "$SRC/fullchain.pem" "$DST/fullchain.pem"
install -o root -g www-data -m 640 "$SRC/privkey.pem" "$DST/privkey.pem"

# Restart service
systemctl restart web-server
```

Como se ve, el servidor web tomará los certificados de `/etc/web-server/tls`.

Este script se ejecutará cada vez que se renueva el certificado en `/etc/letsencrypt/live/example.com/`.
