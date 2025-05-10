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

## FSX for Lustre

Está pensado para procesos que necesitan alto rendimiento de cómputo como machine learning, big data o financial modelling.

En lugar de ser para equipos Windows, es para Linux; utiliza POSIX.

Modos de funcionamiento:

- Scratch. Para trabajos de corto periodo, es rápido pero no tiene high availability o replicación. Cuanto mayor sea el file system, más componentes requiere y aumenta la probabilidad de fallo.
- Persistente. Para procesos más largos, es high available, pero solo e una AZ, y se recupera en caso de error.

Localizaciones on premise pueden acceder a el mediante VPN o direct connect.

Los datos pueden almacenarse (es opcional) en S3, en el file system solo aparecen cuando son necesarios (lazy loaded). Desde lustre pueden enviarse datos a S3, pero no están sincronizados de maneara automática.

Lustre tiene file servers con caché y storage volumes para gestionar los datos.

La VPC se comunica con lustre mediante una ENI.

Pueden realizarse backups a S3.
