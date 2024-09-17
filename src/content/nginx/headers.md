# Nginx Headers

## Contenidos

- [Introducción](#introducción)
- [Expire](#expire)
  - [Expire introducción](#expire-introducción)
  - [Expire configuración](#expire-configuración)

## Introducción

Las cabeceras se especifican con el directive `add_header`. Por ejemplo, con `add_header test_header "foo bar"` obtendremos una cabecera `test_header: foo bar`.

## Expire

### Expire introducción

Indican el tiempo que el cliente puede cachear la respuesta.

Por ejemplo, si una imagen no suele cambiar en nuestro servidor, podemos decir a cliente que la cachee y no volverá a pedirla durante el tiempo configurado, de modo que ahorramos peticiones.

### Expire configuración

```bash
...
http {
    ...
    server {
        ...
    }

    # location = /image.png { # Example specific file.
    location ~* \.(css|js|jpg|png)$ { # Example case insensitive for these extensions.
        add_header Cache-Control public; # Means the resource can be cached.
        add_header Pragma public;
        add_header Vary Accept-encoding; # Means the response can vary based on the request header.
        expires 1h;
    }
}
```

En las cabeceras de respuesta, obtendremos estas adicionales:

```bash
$ curl -I http://localhost:8080/cached-file.html
Expires: Thu, 09 Feb 2023 22:20:26 GMT
Cache-Control: max-age=3600
Cache-Control: public
Pragma: public
Vary: Accept-encoding
```

Sobre estas cabeceras:

- `Expires`: es la fecha de la petición mas el tiempo especificado en el que expirará.
- `Cache-Control: max-age=3600`: el número es el tiempo de expiración en segundos.

