El comando `netsh` ofrece información de redes WiFi como cifrado, BSSID, etc.

## Redes WiFi

```bash
netsh wlan show networks mode=Bssid
```

De solo mostrar datos de la WiFi a la que estoy conectado, se pueden probar las siguientes opciones:

- Desconectarme y ejecutar el comando.
- Dar a conectar a una red WiFi aunque no introduzca contraseña, y ejecutar comando.


## MAC routers

```bash
netsh wlan show all
```

Información con la dirección MAC de los routers en el apartado `MOSTRAR MODE=BSSID DE REDES`.

Sino saca las MAC, desconectarse de la WiFi y ejecutar de nuevo el comando.

Para ver las redes WiFi conocidas (utilizadas alguna vez por el ordenador) buscar Perfiles de usuario.