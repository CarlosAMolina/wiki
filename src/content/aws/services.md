# Servicios

### Tier

Es una agrupación de funcionalidades de una aplicación.

Ejemplo de tiers:

- Web tier. Punto de entrada de los clientes a nuestra aplicación. Ejemplo de servicios que lo forman: load balancers, api gateway, etc.
- Compute tier. Ofrece la funcionalidad a la que acceden los clientes. Ejemplo de servicios: lambda, ec2, ec2.
- Storage. Ejemplo: EBS, EFS, S3.
- DB tier. Ejemplo: RDS, Aurora, DynamoDB, Redshift.
- Caching. Ejemplo: ElastiCache, DynamoDB Accelerator (DAX).
- Application services. Ejemplo: Kinesis, Step Functions, SQS, SNS. Ofrecen servicios como notificaciones por email o modificar la arquitectura de la aplicación desacoplando componente mediante el uso de colas.

## Referencias

- [Terminate active AWS resources that you no longer need](https://aws.amazon.com/es/premiumsupport/knowledge-center/terminate-resources-account-closure/?ref_=pe_3594660_610233740).
