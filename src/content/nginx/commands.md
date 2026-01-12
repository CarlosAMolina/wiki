# Comandos Nginx

## Contenidos

- [Introducción](#introducción)
- [Listar comandos disponibles](#listar-comandos-disponibles)
- [Iniciar servidor](#iniciar-servidor)
- [Parar servidor](#parar-servidor)
- [Verificar configuración es correcta](#verificar-configuración-es-correcta)
- [Reiniciar servicio nginx](#reiniciar-servicio-nginx)

## Introducción

Los comandos que paran, inician, etc. el servidor pueden realizarse con el propio ejecutable de Nginx o con `systemctl` (ver apartado instalación para añadir el servicio `nginx.service`).

## Listar comandos disponibles

```bash
nginx -h
```

## Iniciar servidor

Utilizando el ejecutable:

```bash
sudo nginx
```

Mediante `systemctl`:

```bash
sudo systemctl start nginx.service
```

Puede verificarse su funcionamiento con cualquiera de estos dos comandos:

```bash
ps aux | grep nignx
```

```bash
sudo systemctl status nginx.service
```

## Parar servidor

Utilizando el ejecutable:

```bash
sudo nginx -s quit
# Option `stop` executes a fast shutdown and option `quit` a graceful one.
# Resources: https://nginx.org/en/docs/beginners_guide.html
```

Mediante `systemctl`:

```bash
sudo systemctl stop nginx.service
```

## Verificar configuración es correcta

```bash
sudo nginx -t
```

Como se explica en el apartado de reiniciar el servidor, esta comprobación puede hacerse con los comandos `systemctl`.

## Reiniciar servicio nginx

Tras modificar archivos de configuración, para recargarlos y comprobar que no contienen errores, ejecutamos el siguiente comando:

```bash
sudo systemctl reload nginx.service
```

Otra opción es lanzar el comando utilizando el ejecutable:

```bash
sudo nginx -s reload
```

La ventaja de emplear `systemctl` es que es inmediato (no deja Nginx sin servicio) y de haber errores en la configuración, el comando fallará.

Si en lugar de utilizar `reload` empleamos `restart`, primero el servidor se para y no se iniciará de haber errores en la configuración.

