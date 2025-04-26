# ACM

## Introducción

ACM = AWS Certificate Manager.

Permite la creación, gestión, renovación y despliegue de certificados.

Es un servicio regional. Un certificado no puede cambiar de región.

Hay que tener en cuenta la región de los servicios y de ACM:

- Los servicios deben estar en la misma región que el certificado. Por ejemplo, para usar un certificado en un ALB, el certificado en el ACM debe estar en la misma región que el ALB.
- CloudFront presenta una excepción a esto. Al ser un servicio global, el certificado debe encontrarse en `us-east-1` (Northern Virginia). El Distribution de ClouFront debe estar en `us-east-1`; luego el Distribution despliega el certificado en los edge locations aunque estos se encuentren en otras regiones.

Realiza la función de Certificate Authority (CA). Puede ser:

- Pública. Puede utilizarse desde Internet. Los navegadores confían en los CA que indica su sistema operativo, y estos CA pueden indicar que se confíen en otros CA.
- Privada. Solamente puede utilizarse en nuestra organización. Los equipos clientes deben configurarse para confiar en esta CA.

También permite generar e importar certificados. Los certificados que genera ACM puede renovarlos automáticamente; pero de importar en ACM un certificado emitido por un tercero, es responsabilidad nuestra renovarlo con el tercero y volver a importarlo en ACM.

Los certificados se almacenan cifrados, pueden ser desplegados en servicios compatibles, no todos los servicios lo son:

- Servicios compatibles: CloudFront, ALB, API Gateway.
- Servicios no compatibles:
    - EC2 no es compatible ya que la máquina EC2 es gestionada por nosotros y podemos acceder y modificar los certificados, lo que va en contra de la idea de ACM, de que es ACM quien gestiona el certificado, su almacenamiento y despliegue.
    - S3 no utiliza ACM.
    - Lambda.
    - Servicios que no son de AWS, ejemplo Elastic Certificate Controller.
