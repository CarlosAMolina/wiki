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
