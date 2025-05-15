# Macie

## Introducción

Es un servicio de seguridad. Permite detectar, monitorizar y proteger S3, por si por ejemplo lo hemos configurado mal y puede haber fugas de información.

Automáticamente detecta los tipos datos, por ejemplo información individual, financiera, etc.

Emplea una arquitectura multi account; una cuenta es la administradora y gestiona Macie para otras cuentas, permitiendo analizar los buckets de todas las cuentas que compongan la organización.

## Identificadores

Es lo que utiliza para que automáticamente detecte los tipos datos, por ejemplo información individual, financiera, etc.

Pueden ser de dos tipos:

- Gestionados:
  - Son patrones y modelos de inteligencia artificial
  - Gestionados por AWS.
  - [Link a su documentación](https://docs.aws.amazon.com/macie/latest/user/managed-data-identifiers.html).
- Propias:
   - Son expresiones regulares.
   - Creadas por nosotros.
   - [Link a su documentación](https://docs.aws.amazon.com/macie/latest/user/custom-data-identifiers.html).
   - Permite definir palabras y una distancia para especificar lo cercano que una expresión regular puede matchear para considerarse válida.
   - También puede indicarse una lista negra de palabras.

## Discover jobs

Creamos discovery jobs para utilizar estos identificadores y encontrar lo que matchee en los buckets, las detecciones pueden utilizarse para interactuar con otros servicios, por ejemplo EventBridge.

En los discovery jobs quedan especificados los buckets que analizar.

## Detecciones

Son de dos tipos:

- Policy. Cuando la configuración de S3 cambia de modo que la seguridad puede reducirse; por ejemplo, deshabilitar el cifrado.
- Datos sensibles. Cuando descubre datos sensibles en los buckets S3. Ejemplo credenciales como keys ssh.

[Link a la documentación](https://docs.aws.amazon.com/macie/latest/user/findings-types.html).
