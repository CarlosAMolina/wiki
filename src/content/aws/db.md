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

Cada instancia RDS tiene su almacenamiento dedicado, utiliza EBS.

RDS gestiona el hardware, sistema operativo, instalación y mantenimiento.

DB engines disponibles: MySQL, MariaDb, PostgreSQL, Oracle, Microsoft SQL Server. Como se ve, algunos casos requieren de licencia.

No tenemos acceso al sistema operativo o acceso con SSH; aunque hay casos en que se puede tener cierto nivel de acceso.

### Instancia standby

De utilizar high availability, se crea además de la instancia principal, una de standby en otra RDS subnet group (explicado qué es más adelante) y cada instancia tiene su almacenamiento dedicado.

Se realiza una replicación síncrona de la instancia primaria a la standby, es decir, se replica la información tan pronto llega a la primaria.

### Read replica

Utiliza replicación asíncrona.

La instancia read replica puede estar en otra región.

### Backups

Se almacenan en S3.

De utilizar modo multi AZ, el backup se realiza desde la instancia standby, por lo que no hay un impacto en el rendimiento.

### Arquitectura

Es un servicio que funciona en una VPC, por lo que no es público como S3 a menos que lo configuremos para ello asignándole una IP pública.

Ejemplo de cuántas crear, una por cada RDS deployment.

Accedemos a las instancias por VPN o con AWS Direct Connect.

#### RDS subnet group

La creamos nosotros e indica qué subredes pueden utilizar las instancias RDS.

### Coste

Cobran por:

- Tipo de instancia y tamaño.
- Si se utiliza multi AZ o no, utilizarlo implica que hay más de una instancia.
- Tipo de almacenamiento y cantidad.
- Coste por giga transmitido desde o hacia la instancia.
- Backups y snapshots. El snaphsot es gratis hasta que ocupa más del tamaño del almacenamiento de la instancia; por ejemplo si el almacenamiento es 1 TB, tenemos 1 TB gratis de snapshot. Cobra por giga al mes, por ejemplo, 1 TB almacenado un mes cuesta lo mismo que 0.5 TB dos meses.
- Costes por posibles licencias.

## Amazon Aurora

No está dentro de RDS, se trata de un producto diferente y hay diferencias entre ellos.

Es un engine creado por AWS; tiene compatibilidad con otros engines.
