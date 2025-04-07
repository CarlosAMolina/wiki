# EventBridge

Este servicio está reemplazando a los CloudWatch Events.

Permite lo mismo que CloudWatch Events y además gestionar eventos de terceros y de aplicaciones propias.

Con él se pueden crear:

- Reglas que, al detectar eventos con las características especificadas, los llevan a uno o más destinos. Ejemplo a una lambda. Las reglas se asocian a un event bus.
- Enviar eventos configurados de manera periódica. Es un cron.

Un evento tiene formato JSON.

## Event bus

Es un stream de eventos generados por servicios compatibles.

Tanto CloudWatch Events como EventBridge tienen un event bus por defecto para la cuenta de AWS.

En:

- CloudWatch Events solo hay un event bus, es implícito, no se muestra.
- EventBridge puede tener event bus adicionales y podemos interactuar con ellos.
