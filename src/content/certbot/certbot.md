# Certbot

## Certificado manual

Explicado en la [configuración de Nginx](https://cmoli.es/wiki/content/nginx/http2.html#certificado-ssl-con-let-s-encrypt-y-certbot).

## Certificado renovado automáticamente

Primero, en el VPS creamos el directorio donde se almacenará el challenge y permitimos que el usuario que ejecuta el servidor web pueda acceder a él:

```bash
sudo mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
sudo chown -R www-data:www-data /var/www/letsencrypt
```

El servidor web debe ofrecer el contenido de este path:

```
/var/www/letsencrypt/.well-known/acme-challenge/
```

Es decir, debe funcionar la petición a:

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

Para probar la renovación, el siguiente comando debe funcionar sin error:

```bash
sudo certbot renew --dry-run
```
