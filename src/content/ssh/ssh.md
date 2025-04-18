# SSH

## Contenidos

- [Conexión a servidor con SSH sin credenciales](#conexión-a-servidor-con-ssh-sin-credenciales)
  - [Generar clave de identificación](#generar-clave-de-identificación)
  - [Configurar cliente para acceder al servidor](#configurar-cliente-para-acceder-al-servidor)
  - [Eliminar clave de identificación](#eliminar-clave-de-identificación)
  - [scp](#scp)
  - [Referencias](#referencias)

## Conexión a servidor con SSH sin credenciales

### Generar clave de identificación

Comenzamos generando la clave en nuestro equipo que se conectará al servidor:

```bash
ssh-keygen -t rsa
```

Tras ejecutar el anterior comando, aparecerán una serie de preguntas, podemos dejar la opción por defecto en todas ellas. Cuando nos pregunta por el nombre del archivo, puede cambiarse a otro, por ejemplo a `id_rsa_vps`. Esto creará los archivos necesarios en `~/.ssh/`.

En el servidor al que conectarnos, creamos el siguiente archivo si no existe: `~/.ssh/authorized_keys`.

Copiamos todo el contenido del archivo que creamos inicialmente `~/.ssh/id_rsa_vps.pub` y lo pegaremos en una nueva línea en el archivo `~/.ssh/authorized_keys` del servidor.

Después de pegarlo, podemos cambiar en el archivo del servidor la parte final tras el símbolo igual, para indicar con ese nombre más datos sobre el equipo al que pertenece la clave; por ejemplo, si la clave termina en `...asdf= foo@bar`, mostramos que es de un equipo de sobremesa dejándolo como `...asdf= foo@bar_desktop`.

Ahora, nos conectamos al servidor sin introducir contraseña mediante este comando:

```bash
ssh -p 22 -i ~/.ssh/id_rsa_vps foo@1.2.3.4
```

### Configurar cliente para acceder al servidor 

Con el fin de facilitar la conexión, evitando tener que indicar el puerto, archivo de claves y dirección IP, creamos en nuestro equipo el archivo `~/.ssh/config` con este contenido:

```bash
Host foo
    Hostname 1.2.3.4
    IdentityFile ~/.ssh/id_rsa_vps
    IdentitiesOnly yes
    User bar
    Port 22
```

Ahora, la conexión se realiza simplemente con:

```bash
ssh foo
```

### Eliminar clave de identificación

Para evitar que un equipo se conecte al servidor con el método explicado en este apartado, solamente debemos eliminar en el archivo `~/.ssh/authorized_keys` del servidor la clave que corresponda al equipo.

### scp

De tener configurado el host en `~/.ssh/config`:

```bash
scp HOST_IN_THE_CONFIG:VPS_FILE_PATH LOCAL_FILE_PATH
```

De no tener configurado el host:

```bash
scp -P VPS_PORT VPS_USER@VPS_IP:VPS_FILE_PATH LOCAL_FILE_PATH
```

### Referencias

Generar claves:

<https://atareao.es/ubuntu/sincronizacion-sin-contrasena-en-linux/>

Configurar cliente:

<https://atareao.es/ubuntu/configuracion-de-ssh/>
