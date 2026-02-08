# Network

## Contenidos

- [Configurar interfaces](#configurar-interfaces)
  - [Introducción](#introducción)
  - [Configuración manual](#configuración-manual)
  - [Configuración automática con DHCP](#configuración-automática-con-dhcp)
- [Reiniciar interfaz](#reiniciar-interfaz)
- [Recursos](#recursos)

## Configurar interfaces

Este apartado describe la configuración utilizada en Arch Linux

### Introducción

Los pasos son:

- Activar interfaz.
- Configurar dirección IP en la interfaz.
- Configurar ruta en la interfaz.
- Iniciar servicio systemd-resolved: para DNS.

En el caso de la interfaz wireless, se debe introducir las credenciales del punto de acceso deseado.

### Configuración manual

Pueden verse los pasos en el siguiente script:

```bash
# run script as sudo

if [[ "$1" != "enp2s0" ]] && [[ "$1" != "wlp3s0" ]]; then
    echo "[ERROR] Specify a valid interface"
    echo
    ip address show
    exit 1
fi

echo "[DEBUG] Init ${1}"
ip link set $1 up

if [[ "$1" == "wlp3s0" ]]; then
    wpa_supplicant -B -i $1 -c /etc/wpa_supplicant/wpa_supplicant.conf
fi

echo "[DEBUG] Config static IP"
ip address add 192.168.1.38/24 broadcast + dev $1
ip route add default via 192.168.1.1 dev $1

echo "[DEBUG] Config done"
ip address show
ip route show

echo "[DEBUG] Start DNS resolution"
systemctl start systemd-resolved
watch -n 1 ping -c 1 archlinux.org
```

### Configuración automática con DHCP

Para configurar en la interfaz la dirección IP y la ruta mediante DHCP, se utiliza el servicio `systemd-networkd`.

Con el servicio `wpa_supplicant@wlp3s0` activaremos la interfaz inalámbrica y nos identificaremos en la red Wifi, siendo wlp3s0 el nombre de la interfaz.

La configuración de las interfaces se realiza mediante archivos, los cuales los guardaremos en la ruta `/etc/systemd/network` con nombres `20-wired.network` y `25-wireless.network`:

Configuración de la interfaz cableada:

```bash
[Match]
Name=enp2s0

[Network]
DHCP=yes

[DHCPv4]
RouteMetric=10
```

Configuración de la interfaz wireless:

```bash
[Match]
Name=wlp3s0

[Network]
DHCP=yes
IgnoreCarrierLoss=3s

[DHCPv4]
RouteMetric=20
```

A menor RotureMetric, mayor prioridad de uso en caso de poder usarse ambas interfaces.

Activar servicios para que se inicien automáticamente al inicio de la sesión:

```bash
sudo systemctl enable systemd-resolved
sudo systemctl enable wpa_supplicant@wlp3s0
sudo systemctl enable systemd-networkd
```

Pueden verse los servicios iniciados con `sudo systemctl --type=service`.

## Reiniciar interfaz

```bash
sudo ifconfig wlp3s0 down && sudo ifconfig wlp3s0 up
```

## Recursos

<https://wiki.archlinux.org/title/Network_configuration>

<https://wiki.archlinux.org/title/Systemd-networkd>
