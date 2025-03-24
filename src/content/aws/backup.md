# Backup

Es un servicio de backups y restore gestionado por AWS, ofreciendo información de auditoría.

Puede configurarse para funcionar en varias regiones y cuentas de AWS.

Puedes gestionar y configurar los backups de diferentes productos (rds, ebs, etc.) desde un único lugar.

En otras secciones (ec2, s3, db, etc.) hay más información sobre backups.

## Componentes

- Backup plans: configura aspectos de los backups como:
  - Frecuencia: cada cuanto se ejecuta. También pueden realizarse on-demand backups, es decir, cuando queramos.
  - Activar continous backups para restaurarlo en un punto en el tiempo.
  - Window: cuándo el backup comienza y el tiempo que puede durar.
  - Ciclo de vida: moverlo a la zona de acceso infrecuente, por ejemplo.
  - Copiarlo a otra región.
- Resources: qué recursos van a tener backups.
- Vaults: el lugar donde se almacenarán los datos de los backups. Pueden bloquearse para que no sean borrados (se llama WORM: write once, read many), se activa tras 72 horas de crearse el backup; de activarlo, aún se borrará tras el retention period de haberlo.
- PITR (point in time recovery): algunos servicios permiten recuperar la información de un punto en el tiempo, como S3 o RDS.

[Link servicios de backup](https://aws.amazon.com/es/backup-restore/services/).
