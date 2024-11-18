# IAM

## Introducción

IAM = Identity Access Management

## Qué puede crear

- Usuarios: para dar acceso a una persona o una aplicación.
- Grupos: cuando queramos dar acceso a un grupo de usuarios con características en común, por ejemplo a los desarrolladores o al departamento de recursos humanos
- Roles: utilizados pro servicios o para dar acceso desde el exterior a mi cuenta. Ejemplo: otorgar a todas las máquinas EC2 acceso a S3.

## Policy

IAM utiliza los `Policy` que es una configuración que permite o niega el acceso a servicios de AWS.

Los `policy` solo tienen efecto cuando se adjunta a una entidad. Las entidades son:

- Usuarios IAM.
- Grupos IAM.
- Roles IAM.

### IAM Policy Document

Tienen formato JSON.

En caso de haber reglas que se solapen con otras, la que se aplica es según el siguiente orden de prioridad (de mayor a menor):

- Se deniega algo de manera explícita.
- Se permite algo de manera explícita.
- Si no hay reglas que prohíban o permitan la acción explícitamente, entonces la acción está denegada (a excepción de la cuenta root).

## Funciones

- ID Provider (IDP): para crear y gestionar identidades.
- Authenticate: probar que soy la identidad que digo ser.
- Authorize: permitir o restringir acceso. Gracias los policy.

Recordar que el root account no puede ser restringido.

## Características

- Es un servicio global, no es específica de cada región.
- No tiene coste pero posee ciertos límites.
- No tiene control directo sobre cuentas/usuarios externos.
- Permite utilizar identity federation (logearnos en los servicios de AWS con el Active Directory de nuestra empresa o nuestra cuenta de FaceBook, Google, etc) y MFA.

## IAM Access Keys

Se utilizan para acceder a los servicios de AWS mediante terminal.

Cada usuario puede tener como máximo dos access keys.

Están formadas por:

- Access Key ID: puede entenderse como el user name cuando hacemos login en la consola de AWS. No es información sensible.
- Secret Access Key: puede entenderse como el password cuando hacemos login en la consola de AWS.

En AWS, se crean desde: click esquina superior derecha en nuestro usuario > Security credentials > Access keys: Create access key.

Se crean para cada usuario de IAM. Por ejemplo, podemos crearlas en el usuario de IAM administrador.
