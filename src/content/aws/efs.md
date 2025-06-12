# EFS

EFS = Elastic File System

Es una implementación de NFSv4 (network file system).

Proporciona un file system sobre la red, se le otorga una IP, puede montarse en diversas instancias linux EC2.

No es compatible con Windows.

Por defecto, está aislado en la VPC donde se monte; para acceder desde fuera de la VPC utizamos VPN o AWS Direct Connect.

Para tener high availability, conviene montarlo en cada AZ.

Modos de funcionamiento:

- General purpose: da la menor latencia.
- Max I/O: da mejor rendimiento pero mayor latencia, utilizado en procesos en paralelo.

Modos throughput:

- Bursting: mejora el rendimiento cuantos más datos haya. Es la opción defecto.
- Provisioned: puedes configurar el rendimiento independientemente de la cantidad de datos.

Clases de almacenamiento:

- Standard: para datos que se acceden habitualmente. Es la opción por defecto.
- Infrequent Access (IA): tiene menor coste, para datos que no se acceden normalmente.

A estas clases de almacenamiento se les puede aplicar políticas lifecycle para mover los datos entre los tipos de almacenamiento.
