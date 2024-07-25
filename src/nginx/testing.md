# Testing

## Contenidos

- [Introducción](#introducción)
- [Apache Benchmark](#apache-benchmark)
- [nghttp2](#nghttp2)
  - [ngttp2 introducción](#ngttp2-introducción)
  - [nghttp2 enlaces](#nghttp2-enlaces)
  - [nghttp2 instalación](#nghttp2-instalación)
  - [nghttp2 uso](#nghttp2-uso)
- [Siege](#siege)
  - [Siege introducción](#siege-introducción)
  - [Siege instalación](#siege-instalación)
  - [Siege uso](#siege-uso)

## Introducción

En este apartado veremos distintas herramientas para probar el servidor Nginx.

## Apache Benchmark

Sirve para conocer el comportamiento del servidor al responder a las peticiones.

Documentación en el siguiente [link](https://httpd.apache.org/docs/2.4/programs/ab.html).

Ejemplo, para enviar 100 peticiones siendo 10 concurrentes:

```bash
ab -n 100 -c 10 http://localhost
```

De los resultados del anterior comando, destacar los `Request per second` y `Time per request`, este último es el tiempo medio en recibir respuesta para una petición.

## nghttp2

### ngttp2 introducción

Es una implementación del protocolo HTTP2, ofrece entre otras opciones un cliente.

### nghttp2 enlaces

Página oficial, [link](https://nghttp2.org/).
Código, [link](https://nghttp2.org/).

### nghttp2 instalación

```bash
wget https://github.com/nghttp2/nghttp2/releases/download/v1.52.0/nghttp2-1.52.0.tar.bz2
tar xfv nghttp2-1.52.0.tar.bz2
cd nghttp2-1.52.0/
./configure
make
```

### nghttp2 uso

```bash
./src/nghttp -nysa http://localhost/index.html
```

Opciones utilizadas:

- n: descartar las respuestas, ya que no queremos guardarlas.
- y: ignorar certificado autofirmado.
- s: mostrar las estadísticas de las respuestas.
- a: solicitar los archivos asociados al archivo html (css, imágenes, etc), de no utilizar esta opción, solo se descargaría el html.

Puede verse un ejemplo de uso y sus resultados en la sección server push de Nginx.

## Siege

### Siege introducción

Permite hacer pruebas de load testing.

Web en el siguiente [link](https://www.joedog.org/siege-home/).

### Siege instalación

```bash
sudo pacman -S siege
```

### Siege uso

Más ejemplos en el apartado de rate limit.

Ejemplo realizar 10 peticiones en 2 tandas de 5 peticiones concurrentes cada una:

```bash
siege -v -r 2 -c 5 https://localhost:8080/nginx.png
```

Del anterior comando:

- `v`: mostrar logs en modo verbose.
- `r`: número de pruebas que realizar.
- `c`: número de peticiones concurrentes.

En color azul se mostrarán las peticiones exitosas y en rojo las que fueron fallidas o rechazadas por el servidor.

