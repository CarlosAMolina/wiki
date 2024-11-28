# Cuenta AWS

## Creación de una cuenta

Hay que facilitar:

- Dirección de email: es única por cuenta.
- Tarjeta de crédito: puede utilizarse en varias cuentas.

## Qué es una cuenta de AWS

Podemos pensar en ella como un contenedor de:

- Identidades: es decir, de usuarios.
- Recursos: lo que creamos al utilizar los servicios de AWS.

Se pueden tener distintas cuentas de AWS: ejemplo una para desarrolladores, otra para testers, para producción, etc. Al estar los recursos separados en cada cuenta, se tienen ventajas como limitar el impacto de cuentas comprometidas.

## Usuarios de la cuenta

### Tipos

#### Root account

Cada cuenta de AWS tiene un `account root user`.

#### Otras cuentas que no son root account

Gracias a IAM pueden crearse:

- Usuarios.
- Grupos.
- Roles.

### Permisos

El root account tiene control total sobre la cuenta de AWS y sus recursos. No puede restringirse a lo que tiene acceso por lo que por seguridad es mejor utilizar esta cuenta lo menos posible.

El resto de usuarios por defecto tienen denegado el acceso a los recursos, pero se les puede dar acceso.

También está la posibilidad de otorgar acceso entre AWS accounts.

## Login

Si desde la página de AWS seleccionamos iniciar sesión, nos llevará a una pantalla con 3 cajitas a rellenar:

- ID de cuenta (12 dígitos) o alias de cuenta.
- Nombre de usuario.
- Contraseña.

Estas cajitas sirven para hacer login con un usuario que no es el root account, es decir con un usuario de IAM. Para acceder con el root account, hacemos click en `Iniciar sesión con el email del usuario raíz`, en este caso se nos pedirá una dirección de email.

### Diferentes logins en el mismo navegador

Normalmente, para hacer login en dos cuentas o usuarios de IAM distintos, abrimos nuestro navegador para un login y luego una navegación privada para otro.

En Firefox, podemos instalar [la extensión Firefox Multi-Account Containers](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/) que permite hacer distintos logins en el mismo navegador.

### Obtención de los valores de login

#### ID de cuenta (12 dígitos) o alias de cuenta

Debemos estar logeados, accedemos al siguiente servicio de AWS: IAM > Dashboard.

En esta pantalla tenemos la sección `AWS Account` con los siguientes valores:

- `AccountID`. Es un valor dado por AWS.
- `Account Alias`. Este valor podemos modificarlo pero debe ser único a nivel global; es decir, no puede existir ninguna cuenta de AWS en el mundo con el valor que utilicemos. Al cambiar este ID, se modificará la URL de login `Sign-in URL for IAM users in this account`.
- `Sign-in URL for IAM users in this account`. Esta URL podemos utilizarla en lugar de la genérica de Login de AWS; de hacerlo, aparecerá rellena automáticamente la cajita `ID de cuenta (12 dígitos) o alias de cuenta` en el login.

#### Nombre de usuario

Este es el valor configurado cuando desde IAM hemos creado un usuario.

Podemos ver los usuarios creados en: IAM > Users. El nombre aparece en la columna `User name`.

El nombre de usuario debe ser único pero dentro de nuestra cuenta de AWS. Es decir, puede que haya otras cuentas de AWS que tengan usuarios con el mismo nombre.

## AWS organizations

Es un servicio de AWS que permite gestionar distintas cuentas de AWS.

Creas un organization desde una cuenta de AWS, pero esta organization no se encuentra dentro de la cuenta. La cuenta desde la que se crea se llama management account o master; solo hay una por organization.

Una vez creada la organization, vamos añadiendo cuentas en ella. Las cuentas pasan de llamarse standard accounts a member accounts. También puedes crear cuentas nuevas desde la organization.

Las cuentas se agrupan en contenedores, que tienen una estructura jerárquica, en el nivel superior está el contenedor llamado organization root compuesto por cuentas de aws y subcontenedores llamados organizational units, los cuales pueden tener otros organizational units.

Las organization ofrecen el servicio `service controll polycies (scp)` para gestionar qué pueden hacer las cuentas de la organization.

En una organizacion pueden utilizarse los roles para dar acceso a un IAM User a otras cuentas.

Hay una cuenta llamada loging account donde te identificas y desde ella puedes acceder al resto; lo que se hace es asumir un rol de otra cuenta.

### Consolidated billing

En una organization, los member accounts dejan de tener una factura propia, todas pasan al management account.

### SCP

Scp = service controll polycies

Sirven para restringir lo que las cuentas de AWS pueden hacer; no es capaz de dar permisos pero indican qué permisos pueden ser otorgados.

Es un archivo json que puede ser asociado al root container, a las organization units o las cuentas AWS individualmente, afecta a todo lo que haya por debajo en la jerarquía.

La management account no se ve afectada por el SCP. Pero sí se aplican al root account.

Pueden utilizarse de dos maneras:

- Allow list: por defecto se bloquean los accesos y hay que activarlos específicamente.
- Deny list: por defecto se permiten los accesos y se deniegan explícitamente.

El comportamiento por defecto es deny list, SCP tiene por defecto la política FullAWSAccess que no restringe. De eliminar esta política, debe otorgarse permisos automáticamente.
