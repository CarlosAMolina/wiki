# Secrets manager

## Introducción

Permite gestionar valores secretos (contraseñas, API Keys, etc.) en AWS; están cifrados con KMS.

Suele confundirse con SSM Parameter Store ya que en este también pueden guardarse strings protegidos.

Permite rotación automática de los valores, mediante una lambda que los actualiza; SSM Parameter Store no soporta esto.

Puede utilizarse mediante:

- Consola.
- CLI.
- API.
- SDK. Esto es importante porque puede integrarse con aplicaciones.
- Hay servicios de AWS, como RDS, que son capaces de utilizarlo directamente, así se beneficia de la rotación de credenciales.
