# ARN

## Introducción

ARN = Amazon Resource Name

Identifica de manera única un recurso en AWS.

## Formato

arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id

- partition: la partición en la que se encuentra el recurso para regiones de AWS estandard es`aws` pero hay casos como china que es `aws-cn`.
- service: es el producto (s3, iam, etc.).
- región: algunos servicios no necesitan especificarlo.
- account-id: algunos servicios no necesitan especificarlo como por ejemplo S3 porque es globalmente único a todas las cuentas.
- El resto a partir del `resource-id` varía según el producto.


## ARN en IAM policies

Si utilizamos `/*`, el objetivo cambia:

- arn: aws:s3:::bucketname. La policy aplica al bucket pero no a los objetos del bucket. Por ejemplo, para hacer una política de crear un bucket.
- arn: aws:s3:::bucketname/*. La policy aplica al los objetos del bucket pero no al bucket. Por ejemplo, para hacer una política de interactuar con los objetos del bucket.

La diferencia entre `::` y `*` es:

- `::`: significa que no hace falta especificar el valor en el ARN.
- `*`: significa que aplica a todo en esa posición.
