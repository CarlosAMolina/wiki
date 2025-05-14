# CloudHSM

## Introducción

HSM = Hardware Secruity Module.

Es un producto hardware enfocado a gestionar claves criptográficas. AWS lo provisiona (en la nube u on premise) y lo gestiona el cliente; AWS no tiene acceso a la parte donde las claves securizadas están almacenadas ni donde se realizan operaciones criptográficas, tampoco puede AWS realizar updates de software.

## Arquitectura

No está desplegado en una VPC de AWS normal sino especial llamada AWS CloudHSM VPC.

En esta VPC especial desplegamos un dispositivo HSM, por defecto no es high available por lo que hay que crear un cluster en el que desplegaremos un device en cada AZ. La información importante como claves y políticas en un HSM es sincronizado con el resto de nodos HSM del cluster.

Se creará un ENI en la VPC del cliente por cada HSM en el CloudHSM VPC, para tener high availability cada AZ del cliente debería tener conexión con las ENI de las otras AZ.

Para acceder al HSM cluster, hay que instalar un AWS CloudHSM Client, y nos conectamos con el cluster HSM mediante API que empleará estándares como PKCS, JCE (Java Cryptography Extensions), CNG (Microsoft CrytoNG).

## KMS vs CloudHSM

CloudHSM es parecido a KMS, hay que saber cuándo elegir cada uno.

Cumple con el estandard de seguridad FIPS 140-2 nivel 3. KMS cumple con el L2 y parte de L3.

KMS puede utlizar CloudHSM como un key store.

No hay integración nativa entre los servicios de AWS y CloudHSM, por ejemplo, no puede utilizarse para Server Side Encription en S3, pero sí podemos cifrarlos on premise y luego cargarlos en S3.

CloudHSM puede:

- Emplearse para realizar las operaciones criptográficas SSL/TLS en lugar de realizarlas en las máquinas EC2 y se ahorra carga de cómputo.
- Habilitar TDE (Transparent Data Encryption) para bases de datos Oracle, así el cifrado es gestionado por nosotros y permite cumplir con regulaciones estrictas.
- Para gestionar las claves privadas de un Issuing Certificate Authority (CA) que sea nuestro.
