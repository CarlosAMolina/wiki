# Cuentas AWS

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
