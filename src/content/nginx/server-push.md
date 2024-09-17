# Server Push

## Contenidos

- [Introducción](#introducción)
- [Configuración](#configuración)
- [Análisis](#análisis)

## Introducción

Si al solicitar un archivo .html este utiliza un archivo .css y otro .png, con HTTP2 primero se recibe el .html y luego vendrán juntos el .css y el .png. Gracias a server push, podemos obtener estos tres archivos juntos.

Documentación en el siguiente [link](https://www.nginx.com/blog/nginx-1-13-9-http2-server-push/).

## Configuración

Creamos un location indicando qué queremos devolver; destacar que hay que definir la petición al recurso (empieza por `/`), no el recurso:

```bash
http {
    ...
    server {
        ...
        location = /index.html {
            http2_push /style.css;
            http2_push /nginx.png;
        }
    }
}
```

## Análisis

En nuestro navegador web puede verse en qué momento se envía cada archivo analizando el tráfico de red y viendo la opción Timeline (en Firefox).

Otra opción es utilizar `nghttp2` (explicado en el apartado de testing de Nginx).

Sin estar server push activado en Nginx, de solicitar solo el archivo `index.html`, tenemos:

```bash
$ nghttp -nys https://localhost:8080/index.html
id  responseEnd requestStart  process code size request path
 13     +3.39ms       +242us   3.15ms  200  189 /index.html
```

Y solicitando el `index.html` y los recursos asociados:

```bash
$ nghttp -nysa https://localhost:8080/index.html
id  responseEnd requestStart  process code size request path
 13     +2.40ms       +256us   2.15ms  200  189 /index.html
 15     +4.28ms      +2.59ms   1.69ms  200   23 /style.css
 17     +4.33ms      +2.60ms   1.73ms  200   2K /nginx.png
```

Una vez activado server push, ya no es necesaria la opción `-a`:

```bash
$ nghttp -nys https://localhost:8080/index.html
id  responseEnd requestStart  process code size request path
 13     +3.08ms       +190us   2.89ms  200  189 /index.html
  2     +3.11ms *    +2.88ms    233us  200   23 /style.css
  4     +3.17ms *    +2.94ms    226us  200   2K /nginx.png
```

El `*` en `responseEnd` indica que es una respuesta que utiliza server push.

Como se ven, los archivos llegan al mismo tiempo y el proceso es más corto.

