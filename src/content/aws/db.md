# DB AWS

## Db en ec2

Normalmente es una mala idea, pero hay situaciones en que puede ser necesario:

- Para poder acceder al sistema operativo de la base de datos ya que las soluciones de DB que ofrece AWS no lo permiten.
- Querer hace configuraciones que no lo permitan bases de datos gestionadas por AWS.
- AWS no ofrece la versión deseada de la base de datos.
- AWS no ofrece la arquitectura que necesitamos (por ejemplo para conseguir una replicación o resilencia específicas).

Los puntos negativos de ejecutarlo en una instancia EC2 es:

- Mayor gestión por nuestra parte, realizar backups, etc.
- El EC2 funciona en una AZ por lo que la db también y si la AZ falla, fallará la db.
- EC2 no es serverless, es más complicado de escalar.
- Perdemos características y mejoras de las db que ofrece AWS.

## RDS

RDS = Relational Database Service

No se trata de DBaaS (Database as a Service) porque en DBaaS pagas por una base de datos; en el caso de RDS pagamos por un servidor de base de datos, por lo que podemos tener varias bases de datos en una instancia RDS, es decir, en un servidor DB.

Por defecto, al crear una instancia de RDS, no se crea una base de datos en ella, pero puede elegirse que sí.

RDS gestiona el hardware, sistema operativo, instalación y mantenimiento.

DB engines disponibles: MySQL, MariaDb, PostgreSQL, Oracle, Microsoft SQL Server. Como se ve, algunos casos requieren de licencia.

Accedemos a las instancias por VPN o con AWS Direct Connect. No tenemos acceso al sistema operativo o acceso con SSH; aunque hay casos en que se puede tener cierto nivel de acceso.

Es un servicio que funciona en una VPC, por lo que no es público como S3 a menos que lo configuremos para ello asignándole una IP pública.

Pueden activarse algunas opciones de autoscaling, por ejemplo:

- Almacenamiento.

### Endpoint and port

Al crear una instancia RDS, AWS le asinga:

- Edpointl Ejemplo: mywordpress.asdf.us-east-1.rds.amazonaws.com. Se utiliza este valor y el del puerto para conectar con la instancia RDS; por ejemplo, es lo que especificar en el archivo `wp-config.php` de configuración de WordPress para DB_HOST.
- Port. Ejemplo: 3306.

### VPC securit group

Es cómo controlamos el acceso a las instancias RDS.

Configura las conexiones permitidas en las network interfaces.

### Instancias RDS

Cada instancia RDS tiene su almacenamiento dedicado, el almacenamiento utiliza EBS. Es decir, cada tipo de instancias que veremos a continuación tiene un almacenamiento separado al resto.

Las instancias pueden estar en distintas AZ, y como veremos, algunas en otra región. No estoy seguro de si puede haber más de una instancia en una AZ.

Ejemplo de cuántas instancias RDS crear, una por cada RDS deployment, por ejemplo una por AZ.

#### Instancia primaria

Es la principal, la que utilizamos de manera habitual.

El database CNAME apunta a las instancias Primary y Standby en caso de haberla.

#### Instancia standby

Ver sección high availability.

El database CNAME apunta a las instancias Primary y Standby en caso de haberla.

#### Instancia read replica

Utilizadas solo para lectura. A menos que se modifiquen para reemplazar a la instancia primaria, tendrán las mismas funcionalidades que la antigua primaria.

Utiliza replicación asíncrona, primero se escriben los datos en la instancia primaria y tras esto se replica en las read replica, por lo que puede que no sea una operación inmediata, puede haber un poco de lag.

La instancia read replica puede estar en otra región; es decir, permite cross region failover. De utilizar multi región, AWS gestiona automáticamente el cifrado en las comunicaciones.

Tienen su propio endpoint address por lo que las aplicaciones deben cambiar cuál utilizan.

Características:

- Mejora el rendimiento y escalado en la lectura.
- Máximo 5 read replicas asociadas de manera directa a cada instancia. A su vez pueden asociarse read replicas a las read replicas pero debido a la sincronización asíncrona, el lag puede acabar siendo un problema.
- Da tiempos de recuperación ante errores muy rápidos porque su provisión requiree poco tiempo. Es decir, son una buena opción cuando hubo un error en la instancia primaria, podemos crear una réplica para que la utilice la aplicación; en caso de que el error no sea por datos corruptos.

#### Backups

Se almacenan en S3, por lo que se replica en las AZ de la región.

Pueden replicarse en otra región; no es la opción por defecto y tiene coste adicional.

Utiliza AWS Managed S3 Buckets lo que implica que desde la consola de AWS no pueden verse en S3, aunque sí pueden verse desde la opción RDS de la consola de AWS.

Puede impactar el rendimiento, en la sección multi AZ se explica que no se ve el rendimiento afectado al realizarse el backup sobre la instancia standby.

El primer snaphsot almacena todos los datos y los posteriores solo los cambios. Por lo que cuantos menos cambios haya, más rápido se creará.

Al restaurar una base de datos, se crea una nueva instancia RDS, por lo que las aplicaciones deben apuntar a su nueva dirección. Es una acción no inmediata, lleva tiempo.

Hay 2 tipos de backups: snapshots y automated backups.

##### Snapshots

Podemos ejecutarlos manualmente o automatizarlos con un script ya que RDS no los realiza automáticamente.

Guardan los datos de toda la instancia, no solo de una base de datos, de todas las db que haya en la instancia.

Al eliminar la instancia RDS, los snapshots se mantienen, hay que eliminarlos explícitamente.

##### Automated backups

Se ejecutan una vez al día.

Cada 5 minutos los transaction logs se guardan en S3. Estos guardan las operaciones que modifican los datos.

Por tanto con los backups y los transaction logs almacenados, puede recuperarse el estado de una base de datos con un margen de 5 minutos.

Podemos configurar retention period con valor de 0 a 35 días. El 0 deshabilita los backups automáticos y el resto de valores son los días de antigüedad que tiene que tener un backup para eliminarse. Al eliminar la base de datos, el retention period sigue aplicando por lo que hay que crear un snaphsot para que no se acabe eliminando el backup.

### RDS subnet group

La creamos nosotros e indica en qué subredes de la VPC utilizar las instancias RDS.

Al crearla, se configura:

- VPC a la que pertenecerá.
- AZ donde están las subredes que deseamos utilizar.
- Subredes de la VPC en las AZ que utilizar.

Al crear la instancia RDS podemos elegir que utilice una AZ específica o que AWS la sitúe en la más óptima; esto se selecciona al configura el VPC security group.

### Coste

Cobran por:

- Tipo de instancia y tamaño.
- Si se utiliza multi AZ o no, utilizarlo implica que hay más de una instancia.
- Tipo de almacenamiento y cantidad.
- Coste por giga transmitido desde o hacia la instancia.
- Backups y snapshots. El snaphsot es gratis hasta que ocupa más del tamaño del almacenamiento de la instancia; por ejemplo si el almacenamiento es 1 TB, tenemos 1 TB gratis de snapshot. Cobra por giga al mes, por ejemplo, 1 TB almacenado un mes cuesta lo mismo que 0.5 TB dos meses.
- Costes por posibles licencias.

### High availability

#### Multi AZ Instance Deployment

Históricamente, era el único modo de tener high availability en RDS.

De activarlo, se crea una instancia standby en otra AZ, mas específicamente, utiliza otra subnet de las disponibles en el RDS subnet group.

Solo hay una instancia standby.

La replicación es a nivel de storage, lo que es menos eficiente que la opción de multi AZ en modo cluster.

Al acceder a la db, se usa el CNAME de la instancia primaria, la instancia standby solo en caso de error en la primaria; por lo que este modo no permite escalar el rendimiento, solo se enfoca en la disponibilidad. Cuando la primaria deja de estar disponible (fallo en la AZ o en la instancia, o porque se actualiza software o cambiamos el tipo de instancia), RDS modifica el CNAME para apuntar a la instancia standby que pasa a ser la nueva instancia primaria. Este cambio puede tardar entre 1 y 2 minutos, para reducir el tiempo, la aplicación debería eliminar la caché DNS.

Se realiza una replicación síncrona de la instancia primaria a la standby, es decir, se replica la información tan pronto llega a la primaria; puede entenderse como si solo hubiera una operación de escritura, tanto en la instancia primaria como en la standby.

Solo puede utilizarse en la misma región.

El backup se realiza desde la instancia standby, por lo que no hay un impacto en el rendimiento de la primaria. Se guardan los datos en S3 y se replican en las otras AZ de la región.

#### Multi AZ Cluster

En esta arquitectura:

- La instancia primaria se utiliza para escribir, es el writer. Se utiliza tanto para escritura como lectura.
- En otras AZ se crean dos instancias utilizadas solo para lectura (solo puede haber 2), cada una en una AZ. Los datos se copian del writer a los readers con replicación síncrona. Permite un poco de scaling en la lectura.

En el Writer se considera que los datos están committed no solo cuando se guardan en el writer, también por lo menos un reader debe confirmar que se ha escrito el dato en él.

Cada instancia tiene su propio local storage. Los datos primero se escriben en un local storage y luego van a EBS.

Se accede al cluster utilizando unos endpoints:

- Cluster endpoint: similar al CNAME en la arquitectura Multi AZ. Apunta a la instancia writer y se utiliza para lecturas, escrituras y administración.
- Reader endpoint: apunta a cualquier instancia reader disponible, a veces esto incluye al writer.
- Instance endpoint: cada instancia en el cluster tiene uno de estos. Utilizados para testing y encontrar indisponibilidades.

Otras diferencias con respecto a Multi AZ modo instance:

- El hardware empleado es más rápido que en Multi AZ instance deployment.
- En caso de error, el tiempo de recuperar la disponibilidad es de 35 segundos porque utiliza transaction log que es mas rápido.

## Amazon Aurora

No está dentro de RDS, se trata de un producto diferente y hay diferencias entre ellos.

Es un engine creado por AWS; tiene compatibilidad con los engines MySQL y PostgreSQL.

Diferencias con RDS:

- Al configurarlo no es necesario indicar cuánto almacenamiento provisionar, como sí es necesario en algunos casos de RDS.
- Al contrario que high availability con Multi AZ Cluster, con Aurora se puede tener más de 2 instancias Reader.
