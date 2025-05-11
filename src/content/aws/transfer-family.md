# Transfer family

## Introducción

Es un servicio que proporciona transferencia de archivos, de forma gestionada por AWS, desde o hacia S3 y EFS; permite interactuar con estos servicios con diferentes protocolos.

Ejemplo de protocolos:

- FTP (File Transfer Protocol).
- FTPS (File Transfer Protocol Secure, emplea cifrado TLS).
- SFTP (Secure Shell (SSH) File Transfer Protocol).
- AS2 (Applicability Statement 2). Utilizado en ámbito empresarial.

FTP y FTPS solo soporta directory service o IDP personalizado.

Características:
  - Multi AZ. Resilente y escalable.
  - Cobro por hora que el servidor está provisionado y por cantidad de datos transferidos.

Puede trabajar con identidades, directorios, o identidades propias (lambda, APIGW), etc. Para que los usuarios se conecten con AWS. En la parte de AWS, los AWS transfer family servers tienen los roles requeridos para interactuar con S3 y EFS.

La característica MFTW (Managed File Transfer Workflow) es una manera de desencadenar trabajos cuando se recibe un archivo, por ejemplo añadir tags.

## Tipos

El endpoint por el que acceder al AWS Transfer Family puede ser de estos modos:

- Público:
  - Funciona en la zona pública de AWS.
  - Solo trabaja con SFTP.
  - Su IP pública es dinámica, la gestiona AWS, por lo que es aconsejable acceder mediante DNS.
  - No puede controlarse el acceso con NACL o security groups.
- VPC:
  - Los usuarios acceden con DX o VPN.
  - Pueden configurarse security groups y NACL.
  - Hay dos tipos:
    - Internet, permite el uso de SFTP, FTPS y AS2. Utiliza una IP estática pública.
    - Internal, puede utilizar SFTP, FTP, FTPS y AS2. Como se ve, FTP solo está disponible dentro de una VPC, en el resto de opciones no es válido.
