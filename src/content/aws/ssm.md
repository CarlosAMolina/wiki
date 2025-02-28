# SSM

SSM = AWS Systems Manager

## SSM Parameter Store

Permite guardar configuración y secretos.

Es la manera recomendable de trabajar con secretos que necesiten otros servicios.

Se guarda en forma key - value. El value puede ser:

- Strings.
- StringList.
- SecureString: el value está cifrado con KMS; son necesarios permisos para usar KMS de querer descrifrar el valor (los permisos para KMS son aparte de los permisos para usar Parameter Store).

Permite guardar utilizando:

- Versionado.
- Estructura jerárquica. Para ello el nombre del key debe tener `/`, ejemplo /wordpress/DBUser.

Para acceder a ellos, al ser un servicio de AWS se utiliza IAM por lo que hay que tener permisos para acceder a este servicio; aunque los parámetros pueden ser públicos, por ejemplo los ID de las AMI.

Pueden utilizarlo:

- Apps.
- Lambda.
- EC2.

Cambios en los parámetros pueden crear eventos.

### Interactuar desde la línea de comandos

Ejemplo:

```bash
# Acceder a un valor
aws ssm get-parameters --names /wordpress/DBUser
# Acceder a los valores dentro de un path y descrifrar los valores
aws ssm get-parameters-by-path --path /wordpress/ --wih-decryption
```
