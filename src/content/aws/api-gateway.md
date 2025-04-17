# API Gateway

## Introducción

Es un punto de entrada para que las aplicaciones interactúen con nuestros servicios. No se trata de la API que desarrollemos, sino el punto de entrada a ella.

Es un intermediario entre los clientes y las integraciones (los que realizan las acciones).

Puede utilizarse para conectarse a aplicaciones monolíticas legacy y ayudar en el proceso de convertirlas a serverless.

Puede trabajar directamente con JWT.

## Características

- Tiempo máximo 29 segundos, si la petición y respuesta dura más, tenemos error de timeout.
- High available.
- Escalable.
- Gestiona authorization.
- Permite configurar throttling. La frecuencia con la que los usuarios pueden utilizarla.
- API Gateway Cache.
- CORS. Para controlar la seguridad de llamadas entre navegadores.
- Compatible con OpenAPI spec.
- Integración directa con otros servicios de AWS sin necesidad de que tengamos que desarrollar algo de back. Ejemplo: DynamoDB, SNS, SF, Lambda, HTTP Endpoints, etc.
- Puede trabajar con servicios de AWS y on-premise.
- Puede trabajar con APIS que utilizan HTTP, REST y WebSockets.

## Funcionamiento

Nos conectamos a su API Endpoint DNS.

Fases de funcionamiento:

1. Petición del cliente al API Gateway. Realiza 3 acciones sobre las peticiones del cliente:
  - Authorization. Es opcional. Tiene varias opciones como:
    - Cognito. El cliente recibe un token de Cognito que valida el API Gateway.
    - Lambda. Para aplicar una autorización que implementemos. La lambda devuelve un IAM Policy y un principal identifier.
  - Validación.
  - Transformación para que pueda utilizarse por las integraciones.
2. Llamar los servicios que ofrecen las integraciones.
3. Respuesta al cliente. Realiza 3 acciones:
  - Transformación.
  - Preparar.
  - Devolver al cliente.

Guarda en CloudWatch los logs de las peticiones y respuestas; así como métricas de cliente e integraciones.

## Tipos de endpoints

- Edge-Optimized. Las peticiones son enviadas al CloudFront POP más cercano.
- Regional. No usan el CloudFront network, sino que los clientes emplean el mismo endpoint. Para clientes y servicios en la misma región donde hay poca carga.
- Privado. Solo es accesible en una VPC mediante una interfaz endpoint.

## Stage

Al desplegar una configuración en API Gateway, se realiza en un stage.

Podemos tener diferentes, por ejemplo uno de pro y otro de dev. Y esos tener distintas versiones que apunten a diferentes versiones de los servicios.

Permiten realizar el canary deployment. En este caso, el despliegue se realiza en una parte reducida del stage, no en todo el stage. La parte reducida se llama canary; esta parte puede ir ajustándose y finalmente convertirse en un nuevo stage.

## Errores

[Link documentación AWS a códigos de error](https://docs.aws.amazon.com/apigateway/latest/api/CommonErrors.html).

- 4XX. Errores que se dan en la parte del cliente. Ejemplo: petición incorrecta, permisos erróneos, etc.
  - 400 - Bad Request. Error genérico; no especifica qué ocurre.
  - 403 - Acceso denegado. Por ejemplo no está autorizado o se filtra por el WAF.
  - 429 - Se ha superado el máximo de peticiones permitidas en cierto tiempo.
- 5XX. La petición es correcta pero ocurre error en el lado del servidor.
  - 502 - Bad Gateway. El servicio backend devuelve una respuesta incorrecta.
  - 503 - Service Unavailable.
  - 504 - La integración da timeout. El backend ha tardado más de los 29 segundos máximos que permite el API Gateway.

## Caché

Se configura por stage.

Tamaño de 500MB a 237GB.

Por defecto tiene una duración de 300 segundos. Puede configurarse de 0 a 3.600 segundos.

Puede ser cifrado.

Solo se hacen llamadas al backend cuando la caché expira; lo que ofrece mayor velocidad de respuesta y menor uso de recursos, por lo que también hay menor coste.
