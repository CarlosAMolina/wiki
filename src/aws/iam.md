# IAM

## Qué puede crear

- Usuarios: para dar acceso a una persona o una aplicación.
- Grupos: cuando queramos dar acceso a un grupo de usuarios con características en común, por ejemplo a los desarrolladores o al departamento de recursos humanos
- Roles: utilizados pro servicios o para dar acceso desde el exterior a mi cuenta. Ejemplo: otorgar a todas las máquinas EC2 acceso a S3.

## Policy

IAM utiliza los `Policy` que es una configuración que permite o niega el acceso a servicios de AWS.

Los `policy` solo tienen efecto cuando se adjunta a un usuario grupo o rol de IAM.

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