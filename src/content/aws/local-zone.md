# Local Zone

Es infraestructura para ofrecer servicio se AWS (como compute, storage, db, etc.) más cerca de quien lo necesita (industrias, personas, etc).

Se emplean cuando requerimos de la mínima latencia.

Su nombre esta compuesto por el de la región de AWS de la misma zona geográfica, que es el parent region, mas un ID. Por ejemplo, para us-west-2 podemos crear el lozal zone us-west-2-last-1.

Los local zone:

- Puede haber varias local zone en la misma ciudad.
- Tienen su conexión independiente a internet.
- Podemos conectarnos a ellas con direct connect.
- No tienen resilencia; son como las AZ.

Una VPC en un parent region puede extenderse a las local zones creando subredes en ellas.

No todos los servicios pueden utilizarse en los local zones, y harán uso del parent region, también habrá algunos que estén limitados. Por ejemplo, de crear EBS snapshots, estos se guardan en S3 del parent region.

## Recursos

[Documentación](https://aws.amazon.com/es/about-aws/global-infrastructure/localzones/features/).
