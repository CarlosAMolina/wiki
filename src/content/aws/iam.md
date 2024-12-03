# IAM

## Introducción

IAM = Identity Access Management

## Qué puede crear

- Usuarios: para dar acceso a una persona o una aplicación.
- Grupos: cuando queramos dar acceso a un grupo de usuarios con características en común, por ejemplo a los desarrolladores o al departamento de recursos humanos
- Roles: utilizados por servicios o para dar acceso desde el exterior a mi cuenta. Ejemplo: otorgar a todas las máquinas EC2 acceso a S3.

## Policy

IAM utiliza los `Policy` que es una configuración que permite o niega el acceso a servicios de AWS.

Los `policy` solo tienen efecto cuando se adjunta a una entidad. Las entidades son:

- Usuarios IAM.
- Grupos IAM.
- Roles IAM.

### Resource policies

Pueden asignarse políticas a recursos, como por ejemplo a S3 buckets.

Estas políticas apuntan al ARN de una entidad (un IAM User o un IAM Role) para dar o denegar acceso al recurso. Los IAM Groups no son true identity por lo que no pueden referenciarse como un principal en una política.

### IAM Policy Document

Tienen formato JSON.

En caso de haber reglas que se solapen con otras, la que se aplica es según el siguiente orden de prioridad (de mayor a menor):

- Se deniega algo de manera explícita.
- Se permite algo de manera explícita.
- Si no hay reglas que prohíban o permitan la acción explícitamente, entonces la acción está denegada (a excepción de la cuenta root).

Si un usuario tiene políticas asociadas y pertenece a un grupo con políticas; al usuario se le aplican todas las políticas en conjunto.

### Tipos

#### Inline policy

Cuando a cada identidad le aplicas un policy document; para actualizar hay que hacerlo a cada policy individualmente.

Suele utilizarse para aplicar excepciones.

#### Managed policy

Un policy document se aplica a varias entidades.

Hay dos tipos:

- Creadas y gestionadas por AWS (AWS Managed Policy).
- Creadas y gestionadas por nosotros (Customer Managed Policy).

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

## Principal

Es una:

- Persona física.
- Dispositivo.
- Aplicación o proceso.

Que intenta autenticarse en AWS; estas personas y aplicaciones pueden encontrarse dentro o fuera de AWS.

Cuando el principal se ha autenticado, se llama `authenticated identity`; al cual se le asignan unas policies, este es el proceso de autorización que da acceso a los recursos de AWS.

## IAM Users

Un usuario IAM es una identidad utilizada por cualquier cosa (personas, aplicaciones, etc.) que necesite acceso a AWS durante un periodo largo de tiempo.

Permite autenticación mediante user&password o con access keys.

Se utiliza cuando el objetivo es un solo principal.

## IAM Groups

Son contenedores para IAM Users, para organizarlos; facilita su gestión.

Los grupos no tienen credenciales y no podemos hacer login en un grupo.

Un IAM user puede estar en varios grupos.

Pueden tener asociadas políticas de tipo inline y managed; los usuarios de los grupos pueden seguir teniendo sus políticas propias. Como vimos antes, al no ser los grupos true identities, no se les puede referenciar desde una resource policy.

No hay número máximo de usuarios que pueden pertenecer a un grupo.

Por defecto, no existe un grupo en IAM que englobe a todos los usuarios; de querer usar grupos, hay que crearlos.

No puede haber grupos dentro de grupos.

Por defecto el máximo de grupos por cuenta es de 300, pero se puede solicitar ampliación.

### Limitaciones

- Máximo 5000 IAM Users por cuenta. Es un hard limit, no puede solicitarse ampliación.
- Un IAM user puede ser miembro de 10 grupos. Es un hard limit, no puede solicitarse ampliación.

Para casos que sobrepasan estas limitaciones, se utiliza IAM Roles o Identity Federation.

## IAM Role

Es un tipo de identidad IAM en una cuenta de AWS, otro tipo de entidad eran los IAM users.

A diferencia de los IAM Users, un rol se utiliza en estas situaciones:

- Cuando hay múltiples principals.
- Cuando el acceso es por un breve periodo de tiempo.

Un rol no representa a alguien, sino que representa el nivel de acceso.

Un rol tiene 2 tipos de políticas:

- Trust policy: controla las identidades que pueden asumir el rol.
- Permissions policy.

A un rol no se hace login, sino que se asume el rol.

Al asumir un rol, se generan credenciales temporales mediante el servicio STS (Secure Token Service).

AWS ofrece el producto AWS Organizations para gestionar múltiples cuentas de AWS. Los roles se utilizan aquí para logearse en una sola cuenta y poder acceder a otras sin necesidad de volver a hacer login.

Ejemplo dónde usar IAM Role:

- En funciones lambda. Como una lambda no sabes cuantas ejecuciones va a atener (no sabes el número de principals), es una buena manera de darle acceso a cloudwath, s3, etc.
- Si hay un usuario con permisos limitados y en situaciones excepcionales requiere más privilegios para realizar una acción puntual.
- Permitir a cuentas externas acceso a recursos de AWS asumiendo un rol. Por ejemplo:
  - ID Federation, donde los usuarios del active directory de una empresa puede asumir el rol. Por ejemplo, en una organización con más de 5.000 usuarios porque IAM tiene de límite 5.000 usuarios.
  - En web identity federation, por ejemplo para que los usuarios de una app puedan utilizarla porque interactúa con base de datos en AWS. Los usuarios se logean con su cuenta de gamil, facebook, twitter, etc. en la app.
- Para que identidades de una cuenta de AWS puedan realizar acciones en recursos de otra cuenta de AWS.

Gracias a los roles tenemos las ventajas de:

- No es necesario almacenar credenciales que pueden perderse en una fuga de información.
- No necesitamos crear nuevos usuarios, pueden utilizarse cuentas externas ya existentes.
- Es posible trabajar con gran número de usuarios.

### Service-linked roles

Son roles que están asociados a un servicio de AWS específico.

Se diferencian de los roles en que no pueden ser eliminados hasta que no es utilizado por ningún servicio.

#### PassRole Permission

Es posible que un usuario asigne un rol ya existente a un servicio aunque el usuario no tenga los permisos que ese rol ofrece. Esto en una policy aparece con el action `iam:PassRole`.

En CloudFormation, este utiliza los permisos de la entidad que lo ejecuta para interactuar con AWS, por lo que el usuario debe tener los permisos no solo para crear el stack, sino también los permisos sobre los recursos que el stack creará. Gracias a service-linked roles, un usuario que no tenga todos los permisos puede ejecutar el stack pasándole el rol que necesite.
