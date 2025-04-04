# EC2

## Introducción

EC2 = Elastic Compute Cloud.

Son máquinas virtuales; es decir, un sistema operativo (OS) y recursos (cpu, ram, network, etc.).

Se les llama instancias.

## Cuándo usar EC2

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
- La instancia se para y vuelve a iniciarse (parar, `stop instance`, es diferente a reiniciar, `reboot`).
- Cambiar el tipo de instancia.

De cambiar a otro host por alguno de los motivos anteriores, estará en la misma AZ.

Una manera de saber que una instancia EC2 cambia de EC2 Host es viendo si su dirección IP pública ha cambiado.

## Almacenamiento

### Términos

- Local on-host storage o instance storage: es un almacenamiento conectado de manera directa, conectado físicamente, al Host EC2. Se utiliza el propio almacenamiento de la instancia. Se pierde si una instancia EC2 cambia de EC2 host, o en caso de error en el dispositivo de almacenamiento.
- Almacenamiento conectado por red: un volumen de almacenamiento se asocia a la instancia mediante el producto EBS.
- Almacenamiento efímero: es almacenamiento temporal. Ejemplo, el conectado de manera directa.
- Almacenamiento persistente: es un almacenamiento permanente que dura más que el tiempo de vida de la instancia. Ejemplo, el almacenamiento conectado por red.
- Volúmenes: son partes de un almacenamiento persistente. Pueden usarse en EC2 de la misma AZ. Como veremos, también hay un tipo llamado instance storage volume.

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

### Tipos de almacenamiento

#### EBS

EBS = Elastic Block Store

Está dentro de la categoría block storage.

Es almacenamiento que se accede por la red.

El almacenamiento que ofrece se llama volúmenes. Estos volúmenes pueden tener distintas características de tamaño, performance, etc.

Puede cifrarse con KMS.

Tiene resilencia AZ y no puede utilizarse en otras AZ. La información de un volumen se replica en distintos dispositivos físicos dentro de la misma AZ.

Al ser independiente del servicio al que se conecta, por ejemplo del EC2 Host, no se ve afectado por problemas de hardware en este.

Un servicio puede tener asociados varios EBS. También, puede asociarse el mismo EBS a más de un servicio, pero es mejor no hacerlo para evitar problemas por múltiples escrituras simultáneas.

Puede estar attach a un servicio, dejar de estarlo y asociarlo con otro servicio. El almacenamiento es persistente y no se ve afectado por estos cambios; los datos se mantienen hasta que los borramos.

Puede guardarse un snapshot (backup) en S3 (ver sección sobre ello).

Se cobra por GB usado en un mes, también puede haber cobros por rendimiento.

##### EBS Snapshots

Son backups que se guardan en S3.

Como S3 tiene resilencia regional, se copia en el resto de AZ y es una manera de poder migrar el EBS a otra AZ. También permite realizar migración entre regiones llevando el snapshot a otra región.

En S3 se realiza una copia incremental, lo que significa que la primera copia es de todo el volumen, pero las siguientes solo de los cambios. En AWS, aunque se borre una copia intermedia, los siguientes backups siguen sirviendo para recuperar los datos (hay sistemas incrementales que no lo permiten).

El snapshot no tiene tamaño igual al del volumen, sino el tamaño de los datos almacenados en el volumen.

Sobre el performance:

- Al crear un volumen desde un snapshot, el performance del volumen es máximo.
- Si restauramos un volumen de un snapshot, hasta que no se restaure de manera completa las peticiones no tendrán un performance óptimo.
- Para obtener un performance máximo podemos leer todo el volumen, de manera manual con comandos como `dd` de Linux o activando la opción FSR (Fast Snapshot Restore). FSR realiza la restauración de manera inmediata, hay que indicar las AZ además de la región donde activarlo, puede haber 5 por región (cada combinación de 1 región y 1 AZ cuenta como 1 de estas 50 máximas); tiene coste.

El coste se calcula con los gigas almacenados por mes (10GB durante un mes cuesta igual que 20GB en medio mes). Al ser incremental solo se cobran por los datos que han cambiado; si por ejemplo el 1º snapshot es de 10GB y el 2º de 4GB porque se han añadido nuevos, el cobro del segundo es solo por los nuevos 4GB.

##### Cifrado en EBS

Por defecto los datos en EBS no están cifrados.

KMS y DEK:

- Se utiliza KMS para generar el DEK y con el DEK se cifran los datos.
- El DEK se almacena en el volumen y queda cargado en la memoria del EC2 Host. Cuando la instancia cambia de EC2 Host, se pierde la key, debe volver a obtenrla del volumen.
- El DEK es única por cada volumen, pero si se genera un snapshot y a partir de este otros volúmenes, el snapshot y los volúmenes futuros sí que tienen el mismo DEK que el volumen original.

Una vez activado el cifrado de un volumen, no puede desactivarse, los snapshots y otros volúmenes creados a partir de él también están cifrados.

Los datos están cifrados en el volumen, en el sistema operativo de la instancia no. El cifrado se realiza entre el EC2 host y el volumen.

Puede configurarse una cuenta AWS para que cifre por defecto, utilizando siempre el mismo key KMS o uno distinto cada vez.

No supone coste económico extra. Conviene activarlo.

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

##### HDD-based

HDD = Hard Disk Drive.

Debido a su funcionamiento (brazo robótico lee un disco) tiene una menor velocidad.

Tipos:

- st1 (Throughput Optimized HDD): más barato que los SSD. Utilizado para accesos frecuentes e intenso throughtput. Ejemplo: big data, data warehosues y log processing.
- sc1 (Cold HDD): la opción HDD más económica. Para accesos infrecuentes, ejemplo pocas lecturas por día.

El almacenamiento de st1 y sc1 va de 125GB a 16TB.

##### Instance Store Volumes

Son block storage devices.

Están conectados físicamente al EC2 Host, solo conectados a uno y asilados del resto. Una instancia EC2 puede usar varios volúmenes de este tipo.

Ofrece el mayor performance; da más IOPS y throughput.

Está incluido en el coste de la instancia. Puedes utilizarlo o no.

Se conectan a las instancias durante el arranque, después no se puede. Lo que sí hacemos una vez arrancada la instancia es montar el volumen; por defecto no se montan automáticamente.

El almacenamiento es temporal y son instance storage (ver en otro apartado lo que esto implica).

El acceso a los datos se pierde al cambiar de EC2 Host; es decir, tras parar e iniciar una instancia o cambiar su tamaño o tipo, o cuando falla el volumen. También el file system se pierde ya que al cambiar de EC2 Host se usa otro volumen. Esto no ocurre de reiniciar la instancia.

Algunas instancias no pueden usar este tipo de volúmenes y diferentes tipos de instancias tienen distintos tipos de estos volúmenes.

## Networking

Tipos:

- Storage networking.
- Data networking.

### ENI

ENI = Elastic Network Interface.

Cuando las instancias se provisionan en una subred, un ENI se crea en la subred y la usa el EC2 Host.

Las instancias EC2 pueden tener varias network interfaces y en distintas subredes, dentro de la misma AZ.

Además de la ENI principal, los EC2 pueden tener secundarias; las secundarias pueden desasociarse de la instancia e ir a otra. Es una manera de dar de alta una licencia y llevarla a otra instancia.

Las ENI tienen atributos como los Security Groups (SG) que desde la consola de AWS parecen estar asociados a la instancia EC2 pero en realidad lo están al ENI. Por lo que puede que no se vea desde el sistema operativo de la instancia; por ejemplo, el OS no ve la IPv4 pública asociada.

Atributos:

- Dirección MAC.
- Primary private IPv4. Es estática, no cambia durante el tiempo de vida de la instancia EC2. Se asocia un nombre DNS que solo puede resolverse dentro de la VPC.
- 0 o más secondary IPv4 privadas.
- 0 o 1 IPv4 públicas. Es dinámica, cambia cuando se cambia de EC2 Host o se para e inicia la instancia (no cuando se reinicia). Se le asocia un nombre DNS público; dentro de la VPC, este nombre DNS resuelve a la dirección IPv4 primaria y fuera de la VPC a la IPv4 pública. La dirección IPv4 pública no está asociada ni a la instancia EC2 ni a la ENI, sino al Interface Gateway, que tiene los registros para hacer la relación; así distintas instancias de la VPC pueden usar el mismo nombre DNS.
- 1 elastic IP por cada IPv4 privada. De usarla, sustituye a la IPv4 pública; de quitar la elastic IP, se asignaría una IPv4 pública distinta a la que tenía antes. Las elastic IP son estáticas, puede asignarse desde la consola de AWS.
- 0 o más IPv6.
- Security groups. Aplican a todas las direcciones de la interfaz; por lo que si queremos distintos SG, hace falta distintas interfaces.
- Source/Destination check: el tráfico se descarta si no viene o va a una IP de la interfaz. Es necesario desactivarlo para que la instancia EC2 funcione como una instancia NAT.

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

Es una plantilla de la configuración de una instancia EC2.

Ciclo de vida:

- Launch: lanzar una instancia dese una AMI.
- Configuration: se aplican configuraciones a la instancia.
- Crear AMI. A partir de una instancia que hemos configurado. Cuando creamos una AMI, entre otras cosas se crea:
  - Un snapshot de los volúmenes asociados.
  - Un `Block Device Mapping`: es un archivo que asocia cada volumen con el device que lo utiliza; es la configuración que indica a la máquina EC2 qué volumen es el root, cual del almacenamiento, etc.
- Launch: lanzar una instancia con lo creado anteriormente. También se crean nuevos volúmenes a partir de los snapshots.

Son regionales; en cada región hay distintas AMIS de lo mismo; en cada región tienen un ID único. Las AMIS pueden desplegarse en la AZ que queramos de la región; también pueden copiarse entre regiones (las AMIS y los snapshots de los volúmenes), al copiarse a otras regiones se les asigna otro ID.

Posee permisos que indican qué cuentas de AWS pueden usar o no la AMI:

- Publica: todo el mundo puede usarla.
- Privada: solo el owner de la AMI puede trabajar con ella. Es el permiso por defecto.
- El owner puede dar permisos a otras cuentas.

La AMI determina el tipo de root volume que es donde se inicia el sistema operativo.

En una AMI:

- Sí se almacena:
  - Boot volume.
  - Data volumes.
  - AMI permissions.
  - Block device mapping.

- No se almacena:
  - Configuración de la instancia
  - Configuración de red.
  - No se almacenan datos, sino la configuración y relación con los volúmenes.

Las AMIS son creadas por AWS, terceros o podemos crear AMIS propias.

Significado del término `AMI Banking`: crear una AMI a partir de una instancia configurada con aplicaciones instaladas.

Una AMI no puede ser modificada. Hay que lanzar la instancia, modificarla y crear una nueva AMI.

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

## EC2 Purchase Options

También se les llama Launch Types.

### Tipos

#### On Demand

Es el tipo por defecto.

Recomendado para:

-Aplicaciones que no pueden tener interrupciones. Aunque para aplicaciones críticas, son mejores otras opciones que den preferencia de acceso a los recursos en caso de problemas en el EC2 host.
- Aplicaciones para las que se desconoce la carga.
- Aplicaciones de corta duración.

Pese a que las instancias están aisladas unas de otras, el hardware es compartido entre las instancias de distintos clientes.

Se cobra por los segundos que la instancia funciona (el cobro de los volúmenes, etc. es aparte).

#### Spot

Es la opción más económica.

Consiste en utilizar los recursos que quedan libres en las instancias EC2.

Recomendado para:

- Aplicaciones que pueden interrumpirse

Tiene un precio que varía según los recursos disponibles en cada momento (los que deja libre las instancias On Demand). Indicas el precio máximo que estas dispuesto a pagar, vas pagando por el precio en cada momento y cuando el spot price en ese momento es mayor al que has establecido, la instancia pasa a estado terminated.

De manera independiente al spot price máximo que podamos pagar, habrá momentos en que las instancias pasen a estado terminated.

Es precio es como máximo 90% menos al precio habitual.

#### Dedicated Host

Se reserva un EC2 host para nosotros, no se comparte con otros usuarios.

Este host está diseñado para una familia específica de instancias EC2. Hay diferentes tipos de dedicated Hosts. Para utilizar en él distintos tipos de instancias, creo que hay que emplear el sistema Nitro.

El dedicated host no soporta:

- Algunas AMI. Ejemplo Suse o Windows.
- AWS RDS.
- Placement groups.

Es host tiene un número de sockets y cores, lo que limita el número de instancias que pueden utilizarse.

Nosotros gestionamos el host.

Se paga por el host, las instancias no tienen coste por segundo. Puedes pagar por uso bajo demanda o hay opciones de reserva.

Ejemplo de uso, al tener licencias de software basadas en sockets o cores.

Tienen la característica llamada host affinity por la que una instancia EC2 al parar e iniciarse se mantiene en el mismo EC2 Host.

Puede compartirse con otras cuentas de la organización, gracias al servicio de AWS RAM (Resource Assest Management). Las otras cuentas solo podrán ver las instancias que inician ellos, y yo como administrador del host puedo verlas todas.

[Enlace configuración y precio](https://aws.amazon.com/ec2/dedicated-hosts/pricing/).

#### Dedicated Instances

Es un punto intermedio entre dedicated host y cuando compartimos un host (por ejemplo en On Demand).

El hardware no se comparte con otros usuarios, pero no gestionamos el EC2 Host nosotros.

No pagamos por el host EC2 pero las instancias tienen un coste extra, también hay un coste extra por cada hora que tenemos este tipo de purchase.

#### Reserva

Recomendado para:

- Aplicaciones de larga duración.
- Aplicaciones para las que se conoce la carga.
- Aplicaciones que no pueden tolerar interrupciones.

Cuando una instancia On Demand es de larga duración, podemos hacer una reserva y si esta coincide con la instancia On Demand, el coste se verá reducido o incluso ser nulo.

La idea es que reservamos recursos que sepamos vamos a utilizar. Por los recursos reservados siempre hay coste, se usen o no.

La reserva puede aplicar a:

- Una AZ: los recursos quedan reservados solo para nosotros, se aplica a las instancias de la AZ.
- Una región: no se reservan recursos pero puede veneficiar a las instancias de la región.

Si tenemos configurada una reserva pero utilizamos una instancia EC2 mayor, los beneficios aplicarán parcialmente.

La reserva puede ser de 1 o 3 años, si es 3 años tenemos descuentos y también hay descuento según el modo de pago, por ejemplo el descuento es mayor si se realiza un pago inicial completo.

##### Tipos de reserved instances

###### Scheduled Reserved Instances

Utilizado cuando el funcionamiento va a ser durante un largo periodo de tiempo pero no es continuo. Ejemplo, funciona diariamente pero solo 5 horas.

Reservas los recursos, especificando la frecuencia, la duración y cuándo se utilizará.

Tiene limitaciones:

- No soporta todos los tipos de instancias o regiones.
- Mínimo debe usarse 1.200 horas / año.
- Mínimo periodo contratado es de 1 año.

##### Reservar recursos

AWS los recursos los da con esta prioridad: primer a los que hayan contratado reserva, luego a los on demand y luego a los spot.

Reservar recursos es diferente a reserva de instancias.

Tipos:

- Reservar recursos en una región y utilizar cualquier AZ. Los recursos no quedan reservados hasta el momento de usarlos por lo que de haber pocos disponibles puede dar lugar a problemas. Debe hacerse para periodos de 1 a 3 años. Ofrece un descuento.
- Reservar recursos en una AZ. La capacidad queda reservada para nosotros. Debe hacerse para periodos de 1 a 3 años. Ofrece un descuento.
- On demand capacity reservation: no es necesario indicar el periodo. Los recursos quedan reservados y pagas por ellos los uses o no.

#### EC2 Savings Plan

No te enfocas en un tipo de instancia en una AZ o región, sino que contratas para un periodo de 1 o 3 años indicando lo que pagarás por hora; y se va aplicando el descuento hasta que lleguemos a la cantidad indicada.

Actualmente está disponible para los servicios EC2, Fargate y Lambda. Puede reservar de manera general para todos estos tipos o solo para uno de ellos.

## Instance Status Checks y Auto Recovery

Al ver los detalles de la instancia, en la columna `Status check` tenemos las revisiones que se han llevado a cabo, una relativa al system (power, network, host software y host hardware) y otra sobre la instancia (file system, instance network, kernel).

Pueden activarse opciones de recuperación al detectar fallo:

- Reiniciar la instancia.
- Recover: la llevará a otro EC2 host (en la misma AZ) y tendrá la misma configuración, software, etc. No funciona con instancias con Instance Store Volumes, debe usar EBS volume.
- Stop: sirve para luego revisar qué ha ocurrido.
- Terminate: útil porque puede configurarse que tras terminar la instancia se provisione otra.

Esto se activa en la pestaña `Status checks`, botón `Actions` > `Create status check alarm`.

La opción auto recovery no reiniciará todos los problemas, solo los más sencillos, y en caso de ser el error en toda la AZ, no funcionará (EC2 es un servicio en una AZ).

## Terminate protection

Puede activarse esta opción para que no se pueda terminar una instancia.

## Horizontal y Vertical Scaling

Son técnicas para gestionar la carga cuando esta aumenta o disminuye; para ello se asignan más o reducen recursos de un sistema.

### Vertical scaling

Se consigue por ejemplo, al reemplazar la instancia actual por una con mayores recursos.

 Inconvenientes:

 - Se para el servicio porque es necesario un reinicio de la instancia. Puede evitarse haciendo el cambio a determinadas horas que no afecte a los usuarios, pero puede que para entonces el cambio ya no sea necesario.
- Las instancias más grandes suelen tener un coste extra.
- Hay un máximo en la capacidad que puede conseguirse porque las instancias EC2 tienen un límite.

Ventajas:

- No requiere cambios en la aplicación.
- Funciona para todo tipo de aplicaciones, incluso las monolíticas.

### Horizontal scaling

En lugar de incrementar o reducir los recursos de la instancia, se incrementa o reduce el número de instancias.

Esto implica que habrá múltiples copias de la aplicación y hay que distribuir las peticiones con un load balancer.

En este tipo de scaling las sesiones son fundamentales para no perder el estado de lo que está haciendo el usuario cuando las peticiones vayan a otra instancia.

Para ello los servidores deben ser stateless; es decir, la sesión se almacena en un lugar externo (a esto se llama `off-host sessions`) y la aplicación en los servidores funciona sin tener que preocuparse por ella.

Ventajas:

- No hay interrupción al escalar.
- No hay límites en el escalado.
- Más barato que el escalado vertical (se evita el sobre coste de las instancias grandes).

## Instance Metadata

Es un servicio distinto a las instancias EC2; guarda información a la que pueden acceder todas las instancias.

Se acceder mediante la dirección `http://169.254.169.254/latest/meta-data/`.

La información que guarda se divide en categorías:

- Host name, eventos, security groups, etc
- Environment de la instancia.
- Network. Por ejemplo, el sistema operativo de una instancia no conoce su IPv4 pública, pero se pueden usar el metadata para guardar esta información.
- Para autenticación; se guardan credenciales temporales utilizadas por la instancia para asumir un rol. También se utiliza para credenciales SSH, por ejemplo las empleadas en EC2 Instance Connect.
- Datos de usuario, por ejemplo un script a ejecutar al provisionar la instancia (ver apartado bootstrapping).

No tiene ni autenticación ni cifrado; habría que hacer una configuración extra para impedir acceder a la IP en caso de que queramos evitar el acceso a ella. Pese a esto, según la versión que utilicemos del servicio metadata, puede que haya que pedir un token que añadir en las peticiones; este token se solicita al servicio metadata haciendo peticiones a su URL.

Ejemplos:

- Solicitar IP pública: `curl http://169.254.169.254/latest/meta-data/public-ipv4`.
- Solicitar public hostname: `curl http://169.254.169.254/latest/meta-data/public-hostname`.
- Puede descargarse un ejecutable que tiene estas y otras opciones: `wget http://s3.amazonaws.com/ec2metadata/ec2-metadata`
- Solicitar user data (en este ejemplo se utiliza la versión del servicio metadata que requiere un token):

    ```bash
    TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
    curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
    # Puede pedirse específicamente el user data:
    curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/user-data/
    ```

## ECS

ECS = Elastic Container Service

Servicio de AWS que permite usar contenedores pero con la infraestructura gestionada por ellos.

ECS es a los contenedores lo que EC2 es a las máquinas virtuales.

Términos:

- Container Definition: define dónde se encuentra la imagen a emplear y los puertos a utilizar.
- Task Definition: representa la aplicación, puede tener uno o más contenedores, por ejemplo uno para el servidor web y otro para la db. Guarda:
  - Los recursos a utilizar: cpu y memoria.
  - Modo de red empleado.
  - Si funciona en modo EC2 o fargate.
  - El task role. Es el rol que la task puede asumir para que el contenedor acceda a los servicios de AWS.
  - Por defecto no es escalable ni tiene alta disponibilidad; para ello se utiliza el service definition.
  - Creo que a un conjunto de tasks se le conoce como servicio.
- Service definition: sirve para configurar cómo el task escala y tenga alta disponibilidad. Para escalar genera copias de la task y puede desplegar un load balancer para distribuir la carga sobre los tasks. La alta disponibilidad la consigue reemplazando las tasks que fallen.
- ECS cluster: donde se despliegan las tasks. Hay que crearlo en la consola de AWS.

### ECR

ECR = Elastic Container Registry.

Es un servicio de AWS para almacenar las imágenes de los contenedores. Como Docker Hub.

Cada cuenta de AWS tiene un registro público y privado:

- Público: todo el mundo tiene permisos de lectura pero para escribir hace falta solicitarlos.
- Privado: es necesario tener permisos para lectura y escritura.

Cada registro puede almacenar varios repositorios (son como repositorios de GitHub) y en cada repositorio se pueden tener varias imágenes de contenedores. Cada imagen puede tener varias tags; cada tag debe ser única en el repositorio.

Características:

- Utiliza el servicio IAM para los permisos.
- Escaneo de imágenes para buscar vulnerabilidades.
- Tiene métricas sobre autenticación, push, pull, etc.
- Replicación de las imágenes, cross region y cross account.

### Modos de funcionamiento

Los ECS clusters que pueden funcionar de 2 maneras:

- Modo EC2:
  - Los EC2 son los hosts para desplegar el contenedor, son instancias EC2 que ejecutan software ECS.
  - Se crea un ECS Cluster en una VPC, por lo que se beneficia de las AZs.
  - Se define el ASG (auto scaling group) que controla el escalado horizontal.
  - No es una solución serverless, el usuario gestiona la capacidad y disponibilidad del cluster.
  - Se paga por la instancia EC2 aunque no se esté utilizando la aplicación.

- Modo fargate:
  - Ejecuta los contenedores de manera serverless; AWS gestiona los contenedores, nosotros solo los definimos. No nos preocupamos de las instancias EC2 necesarias.
  - Se utiliza un Fargate Shared Infrasturcture, el hardware se comparte por todos los usuarios pero sin tener visibilidad de los otros clientes.
  - Como en el modo EC2, Fargate continúa usando cluster que se despliega en una VPC y emplea sus AZs. Se usa el hardware del Fargate Shared Infraestructure pero se conecta a la VPC con una Elastic Network Interface y le asigna una IP dentro de la VPC. También tiene un security group que debe de estar configurado con los puertos a usar por el contenedor.
  - Pagamos por los recursos utilizados en los contenedores empleados. No hay coste por el host.

Por tanto, la mayor diferencia entre estos modos es si la gestión la hace el usuario o AWS.

Ambos modos utilizan el ECS management component encargado de:

- Scheluding.
- En qué container host ejecutar los contenedores.
- Gestión del cluster.

### Cuándo usar EC2 o ECS

EC2 cuando:

- Necesitamos controlar mucho el precio, para poder seleccionar las opciones más económicas. Ejemplo: spot.

ECS si:

- Empleamos contenedores.
- El objetivo es minimizar la gestión. Ya que con ECS, no tenemos que preocuparnos del contenedor host.
- La carga no es constante. Ya que pagamos solo por los recursos consumidos.

## EKS

EKS = Elastic Kubernetes Service 101.

Es Kubernetes pero ofrecido como servicio de AWS. Interactúa con los otros servicios de AWS (IAM, VPC, load balancers, etc).

Funciona en múltiple AZs. El etcd de Kubernetes (utilizado para configuración, almacena clave valor); está distribuido en múltiples AZ.

Un cluster EKS está compuesto por un EKS control plane y nodos EKS (son instancias EC2); pueden estar en distintas VPC.

Las instancias EC2 donde están los nodos pueden ser gestionadas por nosotros o por AWS automáticamente utilizando Fargate. Pueden gestionarse individualmente o en grupos.

Kubernetes por defecto tiene almacenamiento efímero; con KES pueden usarse diferentes tipos de almacenamiento (EBS, EFS, ...).

## Bootstrapping

Permite que el sistema operativo de la instancia ejecute scripts cuando la instancia se inicia por primera vez (cuando se provisiona).

Sobre este script:

- Puede ser modificado si la instancia se para, podrá acceder al nuevo contenido pero no se ejecutará, solo es ejecutado la primera vez que la instancia se provisionó.
- Llega por la parte `User Data` del servicio Meta-data. URL petición: <http://169.254.169.254/latest/user-data>. El user data debe ser proporcionado en Base64, por ejemplo en la plantilla CloudFormation podemos utilizar la función `Fn::Base64: !Sub`, aunque la consola de AWS permite darlo en texto plano.
- Lo pasa la instancia EC2 al sistema operativo; el contenido del script es opaco para la instancia EC2.
- si provoca algún error, la instancia funcionará pero sin haber ejecutado el script correctamente.
- No debe contener información confidencial porque puede consultarlo cualquier usuario con acceso a la instancia.
- Tamaño máximo de 16 KB.

En la instancia EC2, pueden verse los logs de este proceso en estos archivos:

- /var/log/cloud-init-output.log
- /var/log/cloud-init.log

### CloudFormation Init

Permite pasar a una instancia EC2 instrucciones de bootstrapping más complejas que con User Data.

Se trata de un script instalado en el sistema operativo de la instancia.

Además de ejecutar comandos, permite:

- Indicar el estado deseado; por ejemplo, de querer una versión determinada de Apache, la instalará de no estar Apache instalado o lo actualizará de ya estar instalado.
- Gestionar grupos, usuarios, servicios, paquetes, ejecutar comandos y comprobar que el resultado es el esperado, etc.

En la plantilla de CloudFormation se especifica en las secciones `Metadata` > `AWS::CloudFormation::Init` y `UserData`.

A diferencia de User Data que solo se ejecuta una vez, con CloudFormation Init pueden aplicarse acciones al detectar cambios en la instancia, por ejemplo aplicar actualizaciones.

También ofrece las Creation Policies, que permiten verificar que los procesos configurados de Bootstrapping se ejecutaron correctamente; para ello envía información mediante los signals que indicarán al stack de CloudFormation si puede marcarse la instancia como creada correctamente.

Creará estos archivos de logs (además de los cloud-init explicados en la sección anterior):

- /var/log/cfn-init-cmd.log
- /var/log/cfn-init.log

## Boot-Time-To-Service-Time

Esta métrica indica el tiempo que la instancia tarda en ofrecer servicio desde que se lanza.

Tiene en cuenta el tiempo que se tarda en:

- Que AWS provisione la instancia.
- Se instale/actualice/configure el software en el OS.

Instalaciones realizadas después de que la instancia esté activa se considera `Post Launch Time`, se incluye lo realizado en Bootstrapping.

Para evitar el `Post Launch Time` puede usarse AMI Bake, que consiste en tener la AMI ya preparada. Puede combinarse AMI Bake y Bootstrapping.

## EC2 Placement Groups

Permite controlar qué instancias EC2 se ejecuten o no físicamente cerca.

Hay 3 tipos: cluster, spread y partition.

### Cluster

Las instancias están cerca. Utilizan el mismo rack y a veces el mismo EC2 host.

Las instancias tienen conexión directa entre ellas. Ofrece la menor latencia y máximo número de paquetes por segundo (PPS).

Cuando inicias una instancia del grupo, se ejecuta en una AZ y todas las demás instancias del grupo utilizarán esa AZ. No puede expandirse la AZ. Por lo que no tiene gran resilencia ya que las instancias fallarán de haber problemas con la AZ.

Es recomendable que el grupo lo formen instancias iguales y lanzarlas al mismo tiempo para que AWS asegure que habrá los recursos necesarios.

De emplear diferentes VPC puede penalizar el rendimiento.

Solo pueden utilizarlo determinados tipos de instancias.

## Spread

Ofrece la máxima disponibilidad para una aplicación.

Las instancias tienen infraestructura independiente; están en racks diferentes, lo que implica distintas fuentes de alimentación y red. Si un rack falla no afecta al resto.

Los racks pueden estar en la misma o diferentes AZ, habiendo el límite de 7 instancias por AZ.

No es compatible con instancias dedicadas o hosts.

Se utiliza cuando hay un pequeño número de instancias críticas que deben estar separadas unas de otras.

## Partition

Las instancias están en grupos, los grupos están separados unos de otros.

Utilizado cuando necesitamos la separación ofrecida por spread pero también más de 7 instancias por AZ.

Tiene una estructura similiar al tipo spread. Puede utilizar distintas AZ en una región, cada una dividida en particiones, con un máximo de 7 particiones por AZ. Cada partición tiene su fuente de alimentación y de red individual.

Se diferencia con el tipo spread en que puede iniciar todas las instancias que quieras. La partición a utilizar puedes elegirla tu o AWS.

Utilizado en procesos donde es necesario controlar la topología, permite controlar la arquitectura de resilencia. Por ejemplo, estando las instancias en particiones diferentes sabes que al replicar datos van a estar más protegidos al no estar todos en el mismo hardware.

## Enhanced Networking

Es una técnica para mejora el rendimiento de la red.

Utiliza la técnica SR-IOV (Single Route IO Virtualization).

De no utilizar esta técnica, las instancias usan la tarjeta de red del EC2 Host y este decide cómo enviar el tráfico, lo que consume CPU. Al utilizar enhanced networking, la tarjeta de red tiene tarjetas lógicas y cada una es utilizada por una instancia, lo que conlleva mayor I/O, ancho de banda y paquetes por segundo (PPS) y menor uso de CPU y latencia.

Esta opción está activada por defecto o puede activarse sin coste en la mayoría de instancias.

## EBS Optimized

El almacenamiento EBS utiliza tráfico de red y la red es compartida con la red de datos. Gracias a EBS Optimized tenemos una capacidad dedicada para EBS lo que da mayor velocidad.

La mayoría de instancias son compatibles con esto, por defecto está activado. No tiene coste excepto para algunas instancias antiguas.

## Auto Scaling Groups

ASG = Auto Scaling Groups

Permiten que las instancias EC2 escalen según la demanda.

Suelen utilizarse junto a ELB.

Utilizan launch configurations o launch templates para saber qué provisionar; solo pueden utilizar uno de ellos a la vez.

Indican dónde se ejecutarán las instancias. Definen el VPC y las subredes a utilizar; e intentan que las instancias a provisionar estén repartidas de igual modo entre las subredes (haya el mismo número).

Valores a configurar:

- Tamaño mínimo: como mínimo siempre habrá este número de instancias funcionando.
- Tamaño deseado: el auto scaling group provisionará o terminará instancias para conseguir este número.
- Tamaño máximo: entre el número indicado en el tamaño deseado y el máximo, es el número de instancias que pueden provisionarse pero no se hará inmediatamente, sino que, cuando el número sea menor al indicado como tamaño deseado, se generarán inmediatamente las necesarias para alcanzarlo.

Ejemplo de configuración: `1:2:4`, cada número corresponde respectivamente a cada punto definido antes.

### Scaling policies

Se trata de reglas para configurar los ASG. Modifican el desired capacity respetando el max. y min. values configurados.

Los Scaling Policies no son necesarios para que el ASG funcione.

Tipos de scaling:

- Manual: el usuario ajusta los valores de desired capacity, max. y min. de instancias. Esto es útil para testing o en casos donde aplicar algo urgente.
- Scheduled: se ajusta el desired capacity para tiempos determinados en que se sabe que habrá más o menos demanda.
- Dynamic: al detectar un cambio en algo especificado se configura el autoscaling group. Hay 3 tipos:
  - Simple scaling. Hay dos reglas, una para iniciar la instancia y otra para terminarla; la regla se define en base a métricas propias o externas de la instancia EC2. Por ejemplo, si el uso de la CPU supera el 50%, se añade una instancia y si está por debajo del 50% se elimina una instancia. Algunas reglas necesitan tener instalado el CloudWatch agent.
  - Step scaling. Similar a `simple` pero las reglas tienen más detalle. Esto permite una mejor adaptación al cambio que la regla simple. AWS recomienda utilizar step sacaling en lugar de simple scaling. Por ejemplo si el uso de CPU se encuentra:
    - De 20 a 29%, eliminar 2. Nota, si el número de instancias es el mínimo configurado, no se eliminarán.
    - De 30 a 39%, eliminar 1.
    - De 40 a 49%, no hacer nada.
    - De 50 a 59%, no hacer nada.
    - De 60 a 69%, añadir 1.
  - Target Tracking. Se indica el valor deseado y las instancias escalan automáticamente para conseguirlo. Por ejemplo, quiero un 40% de uso de CPU. No todas las métricas pueden configurarse, ejemplo: CPU, uso de red, peticiones a cada target en el load balancer, etc.
- También se puede escalar en base a colas SQS según el número de mensajes visibles.
- Cooldown period: cantidad de segundos a esperar entre que una acción de escalado termina y comienza otra, esto ayuda a evitar grandes costes por provisión.

### Health

Si una instancia falla o la terminamos manualmente, Auto Scaling Group provisiona una nueva.

Para tener high availability, configurar el auto scaling group como 1:1:1 y configurarlo para usar múltiples subnets en múltiples AZ, así si una AZ falla tendremos la instancia en otra.

Tipos de health checks:

- Status checks de los EC2, detectan fallos en hardware y software. Es el utilizado por defecto en ASG. Se toma como estado unhealthy cuando la instancia no está en estado running, es decir cuando su estado es:
  - Stopping.
  - Stopped.
  - Terminated.
  - Shutting down.
  - Impaired (no ha pasado 2 de 2 status checks).
- Health check de ELB en lugar de los de EC2, lo que ofrece otras opciones de monitorización basadas en la layer 7 OSI. Por defecto están desactivados. Se toma una instancia como health si está running y pasa el ELB health check.
- Health checks custom realizados por sistemas externos.

Los health check tienen periodo de gracia que son un margen de tiempo antes de marcar una instancia como no healthy, por defecto es de 5 minutos. Esto es útil para dar tiempo a la aplicación a configurarse.

### ASG y ELB

Combinándolos se obtiene mejor estructura porque:

- Conseguimos elasticidad en la aplicación. ASG puede provisionar instancias en los target groups de ELB y se monitoriza la carga para añadir o quitar instancias.
- ASG puede usar los health check de ELB en lugar de los de EC2, lo que ofrece otras opciones de monitorización.

Es decir, conseguimos abstraernos de las instancias EC2.

### Scaling Processes

Son funcionalidades o procesos que se realizan en el ASG y pueden configurarse de este modo:

- Launch: si se configura como SUSPEND, el ASG no provisionará nuevas instancias.
- Terminate: si se configura como SUSPEND, el ASG no terminará instancias.
- AddToLoadBalancer: controla si las instancias se añaden al ELB.
- AlarmNotificacion: para escuchar notificaciones de CloudWatch.
- AZRebalance: para que el ASG redistribuya instancias en las AZ.
- HealhCheck: si los health checks están o no activados.
- ReplaceUnhealthy: para terminar instancias unhealthy y reemplazarlas.
- ScheduledActions: activar o desactivar las acciones programadas.
 Standby: para parar la ASG y así poder hacer mantenimiento en algunas instancias sin que el ASG actúe sobre ellas.

### Coste

Los ASG no tienen coste; pero se cobran los recursos creados por lo que el aprovisionamiento tendrá coste (ver lo explicado para Cooldown en otra sección).

### Lifecycle Hooks

Permiten que definamos acciones a realizar cuando los ASG lanzan (ejemplo, cargar datos) o terminan (ejemplo hacer backup) una instancia.

Las instancias quedan en pausa hasta que:

- El lifecycle action se completa cuando se ejecute el comando `complete-lifecycle-action` o la operación `CompleteLifecycleAction`.
- O cuando pasa un cierto tiempo de timeout; por defecto es de 1 hora, al menos en estado Terminating: Wait.

Estados de la instancia:

Cuando ASG hace | Libecycle hook | Estados de la instancia
----------------|----------------|--------------------------------------------------------------------
Scale Out       | Desactivado    | Pending > InService
Scale Out       | Activado       | Pending > Pending: Wait > Pending: Proceed > InService
Scale In        | Desactivado    | Terminating > Terminated
Scale In        | Activado       | Terminating > Terminating: Wait > Terminating: Proceed > Terminated

Pueden combinarse con EventBridge o SNS para que el sistema realice acciones o enviar notificaciones, respectivamente.

## Launch configurations y launch templates

Ambas:

- Se emplean para configurar EC2 en los Auto Scaling Groups que crearán instancias. Configuran: AMI, tipo instancia, almacenamiento, red, iam role, user-data, etc.

Launch configuration:

- Una vez definidas, no pueden editarse. Para cambiarla hay que crear una nueva.
- No pueden utilizarse para iniciar instancias directamente.

Launch templates:

- Una vez definidas, no pueden editarse. Aunque permiten tener versiones, pero una vez lanzada la instancia no puede usar otra versión.
- Es un producto más nuevo y se recomienda su uso, tiene más características como capacity reservations, elastic graphics, etc.
- Además de usarse en los Auto Scaling Groups, podemos utilizarlas para iniciar instancias directamente.

ASG vs launch configuration/templates:

- ASG configura el WHEN, en qué casos provisionar, y WHERE, en qué subredes.
- Launch configuration/templates indican el WHAT, qué instancias provisionar y con qué configuración.
