# Procesos

## Contenidos

- [Introducción](#introducción)
- [Worker processes](#worker-processes)
  - [Directive worker_process](#directive-worker_process)
  - [Mostrar número de worker process](#mostrar-número-de-worker-process)
- [Worker connections](#worker-connections)
  - [Directive worker_connections](#directive-worker_connections)
- [Número de peticiones concurrentes posibles](#número-de-peticiones-concurrentes-posibles)
- [Directive pid](#directive-pid)

## Introducción

Los procesos generados por Nginx son:

- Master process: es el servicio Nginx que hemos iniciado
- Worker process: generados por el master process. Escuchan y dan respuesta a las peticiones del cliente.

Esto puede verse con:

```bash
$ systemctl status nginx
...
CGroup: /system.slice/nginx.service
        |- 1211 nginx: master Process /usr/bin/nginx
        |- 25144 nginx: worker process
...
```

En el ejemplo anterior hay 1 worker process.

## Worker processes

Por defecto Nginx tiene configurado 1 worker process.

Aumentar su número no implica un mejor rendimiento; como máximo debe haber 1 worker process por CPU core. El motivo es que los worker process son asíncronos y responderán a las peticiones tan pronto como lo permita el hardware, y cada núcleo de CPU puede gestionar solo un worker process. De configurar 2 worker process en un núcleo, cada proceso funcionarán al 50%, es mejor que solo haya 1 proceso y así trabajará al 100%.

Para comprobar el número de núcleos en nuestro servidor, tenemos los comandos `nproc` o `lscpu | grep CPU\(s\):`.

### Directive worker_process

Para configurar los worker processes.

Se especifica en el main context, ejemplo:

```bash
worker_process auto;
```

Con `auto`, habrá 1 proceso por cada CPU. En lugar de auto puede indicarse el número, por ejemplo `worker_process 2;`.

### Mostrar número de worker process

Puede verse con el comando `systemctl status nginx` o con `ps aux | grep nginx`, aparecerá una línea por cada worker process. 

Ejemplo:

```bash
$ systemctl status nginx
...
CGroup: /system.slice/nginx.service
        |- 1211 nginx: master Process /usr/bin/nginx
        |- 25144 nginx: worker process
...
```

## Worker connections

Es el número de conexiones que cada worker process puede aceptar.

Hay que tener en cuenta el número de archivos que el servidor puede tener abiertos simultáneamente, este límite se muestra con:

```bash
ulimit -n
```

### Directive worker_connections

Se configura en el context `events`, pondremos el valor que obtenemos con `ulimit -n` (comando explicado antes). Ejemplo:

```bash
events {
    worker_connections 1024;
}
```

## Número de peticiones concurrentes posibles

El máximo número de peticiones concurrentes que el servidor puede gestionar se obtiene multiplicando el número de worker_process por el de worker_connections.

## Directive pid

Configura el path del ID del proceso.

Pese a que su valor por defecto fue establecido al instalar el servidor Nginx, puede sobreescribirse en el archivo de configuración con la directive `pid`. Se indica en el main context. Ejemplo:

```bash
pid /var/run/new_nginx.pid;
```

