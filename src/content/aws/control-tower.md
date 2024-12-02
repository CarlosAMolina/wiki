# Control Tower

## Introducción

Su objetivo es la configuración de los entornos de distintas cuentas.

Para conseguirlo, utiliza distintos servicios:

- Organizations.
- IAM Identity Center.
- CloudFormation.
- Etc.

Permite por ejemplo crear automáticamente cuentas de AWS, VPC, etc.

## Términos

- Landing zone: es el entorno multi cuenta. Puede pensarse como AWS Organizations pero mejorado. Ofrece: SSO / ID Federation, Login centralizado, etc.
- Guard Rails: gestiona reglas.
- Account Factory: para crear nuevas cuentas.
- Dashboard: para visualizar todo el entorno.
