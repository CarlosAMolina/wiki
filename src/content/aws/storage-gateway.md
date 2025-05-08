# Storage Gateway

## Introducción

Es un virtual machine en la parte on premise, normalmente es virtual aunque puede ser un hardware appliance. Utilizado para migraciones, extender el almacenamiento, reemplazar sistemas de backup, etc.

Es un puente entre el almacenamiento on premise y el de AWS.

## Volume stored

Funcionan de dos maneras distintas: stored y cached mode.

### Stored mode

Todos los datos son almacenados en un volumen local on premise.

Los datos se copian a otro volumen local que sincronizará de manera asíncrona los datos a AWS enviándolos al Storage Gateway Endpoint en AWS y quedan guardados en S3 como EBS snapshots, se guardan como bloques.

Es útil para full disk backups.

No sirve para incrementar la capacidad on premise, lo que hace son copias.

### Cached mode

Presenta la misma arquitectura que el store mode pero el almacenamiento principal no es on premise, se encuentra en S3.

On premise tenemos almacenamiento de caché con el frequently accessed data, y el almacenamiento que se sincronizará con AWS.

Como en stored mode, también se crean EBS snapshots en AWS.

Los datos en AWS solo son visibles mediante la consola del Storage Gateway, en la consola de AWS no podemos verlos ya que se almacenan como bloques, no son archivos.

## Store Gateway Tape o VTL Mode

### VTL

VTL = Virtual Tape Library.

Es una manera de realizar backups para terabytes de datos.

No permite acceso aleatorio; para tomar un dato, primero hay que localizarlo. Tampoco pueden modificarse los datos, hay que sobreescribirlos.

Cuando el almacenamiento deja de utilizarse, es movido otra localización gestionada por un tercero, esto tiene un coste de transporte. En caso de necesitarlos, se transportan de nuevo desde esta localización.

### Storage Gateway Tape

La arquitectura es como la de la funcionalidad de volume store en modo cached. AWS guarda los datos en S3 (realiza la función de VTL) y S3 glacier se comporta como el tercero que en un VTL almacena/recupera en otra localización los datos que ya no son accedidos.

Esta solución puede combinarse con el sistema de backups que tengamos on premise.

## File mode

Sincroniza el almacenamiento on premise con S3, los archivos están disponibles en ambos lugares.

Utiliza NFS en linux y SMB en windows.

En la localización on premise creas FileShares, cada uno de ellos está conectado con un solo S3 Bucket mediante bucket share. Máximo puede haber 10 bucket shares por file gateway.

Se tiene el mismo rendimiento que una solución LAN gracias a la caché de lectura y escritura. En la localización on premise solo tenemos la caché, todos los datos están en S3.

Al añadir un archivo on premise, automáticamente llega a S3.

En la parte on premise, por defecto no está activado el versionado de objetos.

Por defecto, cuando un objeto llega a S3, no se envía al FileShare, o a los FileShares de tener bucket shares en distintos on premises apuntando al mismo bucket. Para que esto ocurra, hay que activar [NotifyWhenUploaded](https://docs.aws.amazon.com/storagegateway/latest/APIReference/API_NotifyWhenUploaded.html).

No soporta object locking por lo que si dos shares modifican el mismo archivo simultáneamente, puede haber pérdidas. Una solución es que solo un share pueda editar y el resto estén en modo read only.

S3 se integra con el resto de productos de AWS.

Hemos de elegir el tipo de almacenamiento: S3 standard, Infrequent Access o Glacier. Podemos configurar ciclos de vida en S3 para moverlo entre cada tipo.
