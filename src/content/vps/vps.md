# VPS

## Configuración de mi VPS Debian.

Acceder al VPS:

```bash
ssh root@{IP_VPS}
```

```bash
apt update && apt upgrade
# Nota. Utilizar contraseña robusta.
adduser userfoo
# Dar derechos de admin.
gpasswd -a userfoo sudo
```

Para configurar acceso por ssh, ver la sección [ssh](../ssh/ssh.html) de esta wiki.

Mejora la protección del login al VPS:

```bash
sudo vi /etc/ssh/sshd_config
```

Añadir:

```bash
# Disable login using password (you should use public key).
PasswordAuthentication no
# Disable root login.
PermitRootLogin no
# List of allowed users to login.
AllowUsers userfoo
```

Cambiamos el puerto en el que escucha ssh, al inicio del archivo, descomentar `#Port 22` e indicamos el puerto deseado, ejemplo:

```bash
Port 1234
```

Reiniciamos el servicio ssh.

```bash
systemctl restart ssh
```

## Referencias

[Configuración inicial de un VPS](https://atareao.es/tutorial/servidor-virtual/primeros-pasos-con-tu-vps/)
