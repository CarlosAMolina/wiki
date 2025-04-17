# Cognito

## Introducción

Servicio de AWS que proporciona para web/mobile apps::

- Authentication.
- Authorization.
- Gestión de usuarios.

Formado por dos partes:

- User pool.
- Identity pool

## User pool

Su objetivo es que el usuario inicie sesión y obtenga un JWT. No permite utilizar los servicios de AWS.

Muchos servicios de AWS no son compatibles con JWT y debe usarse credenciales gestionada por AWS. Api Gateway sí es compatible con JWT.

Como hemos dicho, user pool no da acceso a recursos de AWS, pero sí permite:

- Gestión del directorio de usuarios y perfiles.
- Sign-up y sign-in. Puede usarse una interfaz web personalizada.
- MFA.
- Oauth (ejemplo, Google, Facebook, etc), SMAL.
- Otras características de seguridad como detectar leak de credenciales.

## Identity pool

Utiliza IAM roles para dar credenciales temporales de AWS para acceder a los servicios.

Tipos de roles:

- Unauthenticated role. Para unauthenticated identities. Permite usar servicios de AWS sin logearnos. Por ejemplo una app pueda ver un ranking de jugadores almacenados en DynamoDB.
- Authenticated role. Para identidades federadas. Autoriza a usuarios logeados mediante user pools u Oauth; web identity federation es convertir el token dado por un tercero en las credenciales temporales de AWS.

Por cada servicio de terceros que usemos para logearnos (Google, Facebook, etc), debe establecerse una configuración distinta para trabajar con él.

Gracias a esto evitamos la limitación del número de IAM users máximos que permite una cuenta de AWS.
