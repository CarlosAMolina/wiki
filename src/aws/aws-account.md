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
