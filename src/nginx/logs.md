# Logs Nginx

## Contenidos

- [Tipos](#tipos)
  - [Access logs](#access-logs)
  - [Error logs](#error-logs)
- [Configurar qué logs utilizar](#configurar-qué-logs-utilizar)
  - [Archivos por defecto](#archivos-por-defecto)
  - [Archivos específicos para ciertas peticiones](#archivos-específicos-para-ciertas-peticiones)
  - [Omitir logs](#omitir-logs)
- [Documentación](#documentación)

## Tipos 

### Access logs

Almacena todas las peticiones al servidor, en el archivo `access.log`.

También guarda las peticiones a recursos inexistentes.

### Error logs

Guarda aquello que falle, por ejemplo, se solicita un archivo que no exista, el archivo de configuración es incorrecto y no puede recargarse, etc.

Estos logs se guardan en el archivo `error.log`.

## Configurar qué logs utilizar

### Archivos por defecto

Al instalar Nginx especificamos la ruta de los archivos de logs.

### Archivos específicos para ciertas peticiones

También, es posible indicar qué archivo de logs utilizar con las directivas `access_log` y `error_log`.

Con `access_log`, se guardarán en la nueva ruta configurada, no en el archivo habitual `access.log`, habría que especificarlo. Por ejemplo:

```bash
http {
    ...
    server {
        ...
        location /test/ {
	    access_log /var/log/nginx/test.access.log;
	    access_log /var/log/nginx/access.log; # Necesario para utilizar el archivo habitual
	    return 200 "hi";
        }
    }
}
```

### Omitir logs 

Para deshabilitar los logs en ciertas peticiones, utilizamos `access_log off;`. Por ejemplo:

```bash
http {
    ...
    server {
        ...
        location /test/ {
	    access_log off;
	    return 200 "hi";
        }
    }
}
```

## Documentación

- Access log: <https://nginx.org/en/docs/http/ngx_http_log_module.html>
- Error log: <https://nginx.org/en/docs/ngx_core_module.html#error_log>
- Configurar logs: <https://docs.nginx.com/nginx/admin-guide/monitoring/logging/>

