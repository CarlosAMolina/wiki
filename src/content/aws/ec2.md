# EC2

## Introducción

EC2 = Elastic Compute Cloud.

Son máquinas virtuales; es decir, un sistema operativo (OS) y recursos (cpu, ram, network, etc.).

Se les llama instancias.

## Cuándo usar Ec2

Si la aplicación requiere un OS.

Cuando la aplicación es monolítica.

En procesos de larga duración.

## Características

Por defecto no usan redes de AWS públicas.

Tipo de resilencia: AZ.

## EC2 Hosts

Los EC2 funcionan en EC2 Hosts, que son servidores físicos gestionados por AWS.

Tipos:

- Shared hosts:
  - Son compartidos por diferentes usuarios de AWS. Las instancias de cada usuario son independientes de las de otros usuarios.
  - Pagas por la instancia que corre en el host.
  - Como los recursos se comparten, lo habitual es que instancias del mismo tipo se creen en el mismo host e instancias diferentes en distintos hosts.
- Dedicated host: pagas por un host individual entero, no lo compartes con más usuarios.

Al reiniciar una instancia EC2, continúa en el mismo EC2 Host. Cambia de Host si:

- El host se cae o hay mantenimiento por parte de AWS.
- La instancia se para y vuelve a iniciarse (parar es diferente a reiniciar).

De cambiar a otro host por alguno de los motivos anteriores, estará en la misma AZ.

## Almacenamiento

### Términos

- Local on-host storage o instance storage: es un almacenamiento conectado de manera directa, está conectado físicamente al Host EC2. Se utiliza el propio almacenamiento de la instancia. Se pierde si una instancia EC2 cambia de EC2 host, o en caso de error en el dispositivo de almacenamiento.
- Almacenamiento conectado por red: un volumen de almacenamiento se asocia a la instancia mediante el producto EBS.
- Almacenamiento efímero: es almacenamiento temporal. Ejemplo, el conectado de manera directa.
- Almacenamiento persistente: es un almacenamiento permanente que dura más que el tiempo de vida de la instancia. Ejemplo, el almacenamiento conectado por red.
- Volúmenes: son partes de un almacenamiento persistente. Pueden usarse en EC2 de la misma AZ.

#### EBS

EBS = Elastic Block Store

Está dentro de la categoría block storage.

El almacenamiento que ofrece se llama volúmenes. Estos volúmenes pueden tener distintas características de tamaño, performance, etc.

Puede cifrarse con KMS.

Tiene resilencia AZ y no puede utilizarse en otras AZ. La información de un volumen se replica en distintos dispositivos físicos dentro de la misma AZ.

Al ser independiente del servicio al que se conecta, por ejemplo del EC2 Host, no se ve afectado por problemas de hardware en este.

Un servicio puede tener asociados varios EBS. También, puede asociarse el mismo EBS a más de un servicio, pero es mejor no hacerlo para evitar problemas por múltiples escrituras simultáneas.

Puede estar attach a un servicio, dejar de estarlo y asociarlo con otro servicio. El almacenamiento es persistente y no se ve afectado por estos cambios; los datos se mantienen hasta que los borramos.

Puede guardarse un snapshot (backup) en S3, como S3 tiene resilencia regional, se copia en otras AZ y es una manera de poder migrar el EBS a otra AZ.

Se cobra por GB usado en un mes, también puede haber cobros por rendimiento.

##### Tipos de volúmenes EBS

##### General Purpose SSD

###### GP2

Los volúmenes van de 1GB a 16TB.

Utilizado para boot volumes, interacciones de baja latencia o dev & test.

Utiliza un sistema de créditos según su uso. Escala según su uso.

###### GP3

Los volúmenes van de 1GB a 16TB.

Más barato que GP2 y también ofrece mayor throughput.

Mismos usos que GP2.

A diferencia de GP2, no utiliza un sistema de créditos por uso ni escala según uso.

##### Provisioned IOPS SSD

Hay 3 tipos: io1, io2, io2 Block Express, se diferencian en el performance (io1 e io2 Block Express tienen el mismo e io2 menos) y el precio.

En comparación con GP, io1 e io2 Block Express tienen más performance, io2 menos.

Utilizados al necesitar alto performance y baja latencia o continua. Ejemplo, uso en base de datos.

Son configurables independiente del tamaño del volumen. Esto por ejemplo permite alto performance en volúmenes pequeños, lo cual no puede conseguirse con el resto de volúmenes.

En estos volúmenes se paga por el tamaño y la cantidad de IOPS.

Hay un máximo de performance por instancia, para evitar esto hay que usar varios volúmenes en la misma instancia.

### Categorías

Indican cómo el almacenamiento se presenta y cómo puede utilizarse.

- Block storage: el volumen se presenta al sistema operativo como un conjunto de bloques sin estructura, la información se solicita apuntando al ID del bloque; el OS es el encargado de crear un file system sobre el block storage y montarlo. Suele usarse en las instancias EC2 como boot volume o cuando queremos un alto performance.
- File storage: presenta una estructura (un file system) a la que solicitar un archivo, puede montarse pero no utilizarse como boot volume. Suele emplearse para compartir un file sytem entre servidores.
- Object storage: no tiene estructura, es una colección de objetos y sus metadatos; puede ser cualquier cosa como binarios, imágenes, etc. No tiene estructura, es como un contenedor de objetos como S3. Es muy escalable (pueden utilizarlo muchas instancias y almacenar gran cantidad de objetos) pero no puede montarse ni usarse como boot volume.

### Performance

Se define con tres términos:

- IO o block size: tamaño del bloque de datos que se escribe en el disco.
- IOPs (input/output operations per second): número de lecturas o escrituras por segundo.
- Throughput: cantidad de datos que pueden transmitirse por segundo. Se muestra en MB per second. Se calcula multiplicando IO x IOPs.

Estos valores no pueden incrementarse como queramos, hay limitaciones o incluso a veces al aumentar el IO disminuyen los IOPs.

## Networking

Tipos:

- Storage networking.
- Data networking.

Cuando las instancias se provisionan en una subred, un `elastic netowrk interface` se crea en la subred y la usa el EC2 Host.

Las instancias EC2 pueden tener varias network interfaces y en distintas subredes, dentro de la misma AZ.

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

En una AMI:

- Sí se almacena:
  - Boot volume.
  - Data volumes.
  - AMI permissions.
  - Block device mapping.

- No se almacena:
  - Configuración de la instancia
  - Configuración de red.

## Eliminar una instancia

En la consola de AWS debemos eliminar:

- La instancia EC2: elegir la opción terminate.
- El security group asociado a esa instancia. Está en el apartado Network & security > Security Groups.

## Tipos de instancias EC2

Links:
- [Página oficial](https://aws.amazon.com/es/ec2/instance-types/).
- [Página no oficial](https://instances.vantage.sh/).

Elegir un tipo de instancia depende de las necesidades respecto recursos (cpu, memoria, etc.), ancho de banda necesaria en la red, arquitectura (x86), vendor (amd, intel), etc.

Categorías:

- General purpose: es la elección por defecto, elegimos otro tipo de necesitar características específicas.
- Compute optimized: ofrecen más cpu que memoria, se usa por ejemplo en procesamiento multimedia, gaming, machine learning, etc.
- Memory optimized: parar procesos que necesitan mucha memoria.
- Accelerated computing: al necesitar características específicas de hardware, por ejemplo uso de GPU.
- Storage optimized: ofrecen gran cantidad de storage y muy rápida, permite un gran número de operaciones I/O. Uso por ejemplo en datawarehousing, analytics workloads, etc.

Significado de las partes del tipo de instancia. Ejemplo R5dn.8xlarge:

- La primera letra es la familia de la instancia. En el ejemplo, tiene el valor `R`. Ejemplo de algunos valores:
  - C: compute optimized.
  - A: memory (ram) optimized.
  - I: I/O.
  - D: dense storage.
  - G: gpu.
  - P: parallel processing.
- La segunda es la generación. A cada generación las instancias son más eficientes y de menor coste. Valor `5` en el ejemplo.
- Después de la generación y hasta el punto puede que haya o no valor, muestra características adicionales. En el ejemplo es `dn`. Ejemplo:
  - d: storage.
  - e: extra capacity de ram o almacenamiento.
  - n: network.
- Tras el punto tenemos el tamaño de la instancia, indica la cantidad de cpu y memoria; cada familia y generación tiene diferentes tamaños disponibles, ejemplo: nano, micro, small, medium, large, xlarge (extra large), 2xlarge, 4xlarge 8xlarge, etc.

## Conectarnos a una instancia EC2

Podemos conectarnos desde nuestro ordenador utilizando las claves SSH (creadas en la consola de AWS en EC2 > Key pairs > Create key pair) o desde la consola de AWS con la opción `EC2 Instance Connect`.

De filtrar las direcciones IP que pueden conectarse, hay que tener en cuenta que, para permitir la conexión desde la consola de AWS, hay que habilitar la IP descrita como `EC2_INSTANCE_CONNECT` para la región deseada en [este enlace](https://ip-ranges.amazonaws.com/ip-ranges.json).
