# EC2

## Introducción

EC2 = Elastic Compute Cloud.

Son máquinas virtuales, se les llama instancias.

## Características

Por defecto no usan redes de AWS públicas.

Tipo de resilencia: AZ.

## Almacenamiento

Ejemplo de algunos tipos de almacenamiento disponible:

- Local on-host storage: se utiliza el propio almacenamiento de la instancia.
- EBS (Elastic Block Store): es almacenamiento que se accede por la red.

## Estados

Una instancia puede encontrarse en los siguientes estados:

- Running: tiene este estado una vez se ha lanzado la instancia y queda aprovisionada.
- Stopped: al apagar la instancia, pasa de running a stopped. Se puede volver de este estado a running.
- Terminated: desde running o stopped, la instancia puede pasar a terminated. Pero terminated es un estado irreversible, no puede volver a running o stopped. En este estado se elimina la instancia y su almacenamiento.
- Otros estados intermedios: stopping, pending, etc.

## Cobro

El cobro es por:

- Tiempo de uso.
- Recursos:
  - CPU.
  - Memoria.
  - Almacenamiento (sistema operativo, aplicaciones, etc.).
  - Networking.
- Cobro extra por ejemplo por otro software que pueda emplear.

Según el estado, puede que haya cobro o no:

- Stopped: aunque no se consumen recursos como CPU, memoria o red, el almacenamiento continúa utilizándose y cuesta dinero.
- Terminated: es el único estado donde no te cobran.

## AMI

AMI = Amazon Machine Image.

Es una imagen de una instancia EC2.

La AMI Puede crearse desde una instancia EC2 y una instancia EC2 puede crearse a partir de una AMI.

Posee permisos que indican qué cuentas de AWS pueden usar o no la AMI:

- Publica: todo el mundo puede usarla.
- Privada: solo el owner de la AMI puede trabajar con ella.
- El owner puede dar permisos a otras cuentas.

La AMI determina el tipo de root volume que es donde se inicia el sistema operativo.

El block device mapping es la configuración que indica a la máquina EC2 qué volumen es el root, cual del almacenamiento, etc.
