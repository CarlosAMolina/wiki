# KMS

## Introducción

KMS = Key Management Service.

Es un servicio regional y público. Aunque ofrece algunas opciones multi región, por ejemplo el replicar las claves criptográficas en otras regiones.

Se utiliza para:

- Crear, almacenar y gestionar claves criptográficas (simétricas y asimétricas).
- Realizar operaciones criptográficas.

## KMS Keys

Estas claves criptográficas nunca abandonan el servicio KMS, interactuamos con ellas mediante APIs.

Pueden cifrar y descifrar datos de hasta 4 KB.

Pueden ser generadas por el usuario o por el servicio que las necesita.

## DEKs

DEKs = Data Encryption Keys.

Son otro tipo de claves criptográficas que puede generar KMS. Permiten trabajar con datos mayores a 4 KB.

Estas claves no se almacenan en KMS. Funcionamiento:

- AWS genera:
  - Una versión en texto plano del DEK.
  - Una versión cifrada del DEK (se cifra y descrifra con una KMS Key).
- Se cifra la información con el DEK plaintext. Decir que el cifrado no lo realiza AWS (tampoco el descrifrado), lo hace el usuario o un servicio encargado de ello; KMS se encarga de cifrar o descifrar el DEK. Puede cifrarse uno o más datos, por ejemplo, S3 genera un DEK distinto por cada objeto.
- Después de que la información es cifrada, AWS descarta el DEK plaintext.
- El usuario almacena el DEK cifrado y los datos cifrados.

## Permisos

En KMS se utiliza el key policy, es un resource policy.

A diferencia de otros servicios de AWS donde con un identity policy se confía en la cuenta, en KMS a cada KMS Key hay que asignarle explícitamente una política que indica el principal que puede utilizarla.

Normalmente se utiliza una combinación de key policy e identiy policy (aunque hay otras opciones como combinar key policies y grant access):

- Key policy: especifica el principal que puede utilizarla.
- Identity policy: para permitir a usuarios IAM interactuar con la key.

Los permisos están muy divididos. Hay permisos diferentes para:

- Crear las keys.
- Cifrar.
- Descifrar.
