# Basic Auth

## Contenidos

- [Introducción](#introducción)
- [Configuración](#configuración)

## Introducción

Permite requerir usuario y contraseña para dar respuesta a peticiones.

## Configuración

Primero, creamos el archivo `.htpasswd`, para ello:

```bash
htpasswd -c /etc/nginx/.htpasswd user_test
```

Del comando anterior:

- `-c`: generar un nuevo archivo.
- `user_test`: nombre del usuario.

Con esto, podemos configurar Nginx. Ejemplo:

```bash
...
location /secure {
    auth_basic "Secure Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    try_files $uri $uri/ =404;
}
...
```

Las directivas utilizadas han sido:

- `auth_basic` seguida del mensaje a mostrar.
- `auth_basic_user_file` seguida del archivo de contraseñas.

Tras introducir las credenciales para obtener el recurso, si accedemos a él de nuevo, no se solicitarán credenciales; para que vuelvan a ser pedidas habría que abrir una navegación privada.

