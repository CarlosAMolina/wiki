# Rate limiting

## Contenidos

- [Introducción](#introducción)
- [Configuración](#configuración)
  - [Aumentar peticiones permitidas](#aumentar-peticiones-permitidas)
  - [Opción nodelay](#opción-nodelay)
- [Enlaces](#enlaces)

## Introducción

Controlar el número de peticiones que el servidor puede responder y cómo hacerlo. Esto ofrece:

- Seguridad. Por ejemplo contra ataques de fuerza bruta.
- Reliability. Protege el servidor contra picos de peticiones.
- Permite controlar la prioridad de las peticiones.

## Configuración

El limit zone se define en el context `http` y luego se utiliza en otros context de manera mas específica (`server`, `location`, etc).

Se define el limit zone con la directiva `limit_req_zone` y las siguientes configuraciones:

- En qué fijarse para aplicar el rate limiting:

    - `$server_name`: aplicarlo a peticiones en base al nombre del servidor, es decir sobre todo lo que haya en la directiva `server`.
    - `$binary_remote_addr`: aplicarlo según la IP del cliente que se conecta al servidor. Es útil en páginas de login para evitar ataques de fuerza bruta desde una misma IP.
    - `$request_uri`: sin importar el cliente, se fija en la URI solicitada. De recibir esta más peticiones de las configuradas, aplicará las limitaciones.

- Nombre del limit zone.
- Tamaño del limit zone en memoria. Es definido tras el nombre y dos puntos.
- Frecuencia de peticiones por unidad de tiempo que no pueden excederse. Por ejemplo, `60r/m` indica 60 peticiones por minuto, una por segundo, por lo que es lo mismo a `1r/s`. Indica la frecuencia de peticiones por la unidad de tiempo, no el número de peticiones; es decir, continuando con el ejemplo anterior, no significa que pueda aceptar máximo 60 peticiones en 1 minuto y tener que esperar al siguiente para seguir aceptando, sino que solo puede aceptar 1 petición por segundo.

Tras definir la zona, se utiliza con la directiva `limit_req`.

Ejemplo de configuración en la que solo permitir 1 petición por segundo:

```bash
http {
    ...
    # Define limit zone.
    limit_req_zone $request_uri zone=MYZONE:10m rate=1r/s;

    server {
        ...
        location = /rate-limiting.html {
            limit_req zone=MYZONE;
            ...
        }
    }
}
```

Para probarlo, con el siguiente test enviaremos 10 peticiones en menos de 1 segundo (2 tandas de 5 peticiones a la vez en cada una) por lo que solo tendrá respuesta la primera de las 10 peticiones.

```bash
siege -v -r 2 -c 5 https://localhost:8080/rate-limiting.html
```

También, podemos verificando al pedir las cabeceras, si lo hacemos en menos de un segundo, obtendremos un error 503 Service Unavailable:

```bash
curl -Ik https://localhost:8080/rate-limiting.html
```

### Aumentar peticiones permitidas

El parámetro `burst` añadirá un número de peticiones extra que pueden tener respuesta.

Puede indicarse al definir la zona (en la directiva `limit_req_zone`), en este caso aplicará siempre que esta se use, o también se puede configurar en el lugar donde sea utilizada la zona (directiva `limit_req`).

Por ejemplo, de permitir la zona 1 petición por segundo y nosotros añadimos 5 burst, se podrá dar respuesta a 6 peticiones; aunque no se envíe la respuesta inmediatamente. Es decir, no modifica el límite de 1 petición por segundo pero no descartará las peticiones extras que reciba.

Es como si se creara una especie de buffer, y puede reducir la velocidad de las conexiones.

Siguiendo con el ejemplo de tener configurado un límite de 1 petición por segundo y añadimos 5 burst, de enviar 2 tandas de 5 peticiones como hicimos en el apartado anterior, el servidor responderá a la primera petición inmediatamente y luego dará respuesta a las otras; en este caso sí habrá respuesta para las 10 peticiones.

```bash
$ siege -v -r 2 -c 5 https://localhost:8080/rate-limiting-with-burst.html
HTTP/1.1 200     0.01 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     1.01 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     2.01 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     3.01 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     4.01 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.00 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.00 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.00 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.00 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.00 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
```

Nota. En el comando anterior, aunque aparezca `5.00 secs` tras la petición 6, en realidad cada respuesta llegaba a cada segundo.

Si solicitamos 10 peticiones en una sola tanda, tendremos 6 peticiones con respuesta (1 petición por segundo mas 5 de burst) y 4 con error 503:

```bash
$ siege -v -r 1 -c 10 https://localhost:8080/rate-limiting-with-burst.html
HTTP/1.1 200     0.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 503     0.02 secs:     497 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 503     0.02 secs:     497 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 503     0.02 secs:     497 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 503     0.02 secs:     497 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     1.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     2.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     3.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     4.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
HTTP/1.1 200     5.02 secs:      51 bytes ==> GET  /rate-limiting-with-burst.html
```

### Opción nodelay

Solo puede utilizarse junto con la opción `burst`.

Indica que las peticiones dentro de `burst` se envíen lo más rápido posible, es decir, sin que les afecte el límite de X peticiones por segundo que haya configurado.

Pero el límite se seguirá aplicando por lo que, por ejemplo, si con una configuración de 1 petición por segundo y un burst de 5, enviamos 6 peticiones concurrentes, todas tendrán respuesta de manera inmediata, pero si antes de que pase un segundo volvemos a enviar 6 peticiones concurrentes, solo algunas tendrán respuesta ya que no ha pasado el límite.

Ejemplo de configuración:

```bash
...
      limit_req zone=MYZONE burst=5 nodelay;
...
```

## Enlaces

- Artículo Nginx

<https://www.nginx.com/blog/rate-limiting-nginx/>

- Artículo freeCodeCamp

<https://www.freecodecamp.org/news/nginx-rate-limiting-in-a-nutshell-128fe9e0126c>
