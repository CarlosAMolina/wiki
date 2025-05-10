# FSx

## Introducción

Es un producto file system que proporciona AWS para que los entornos Windows puedan funcionar en AWS; se trata de un Windows file system nativo.

Es totalmente gestionado por AWS, como RDS para las bases de datos.

Puede integrarse con DS (directory service) o el directory propio del usuario.

Es high available y puede utilizar una o varias AZ en una VPC.

Ofrece:

- Backups bajo demanda o programados, permite versionado por archivo.
- Cifrado KMS.
- Cifrado en la comunicación.
- Compatible con DFS (distributed file system), permite escalado.
- Mediante VSS los usuarios pueden restaurar distintas versiones de los archivos.

Accesible con el protocolo SMB. Puede utilizarse a través de la VPC, con peering, VPN o direct connect.

El active directory puede estar en AWS u on premise; en este último caso AWS se conectará a la solución on premise.

## FSx vs EFS

EFS es para linux y FSx para windows.
