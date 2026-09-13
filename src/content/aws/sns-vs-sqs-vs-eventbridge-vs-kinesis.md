# SNS VS SQS VS EventBridge VS Kinesis

## Introducción

Referencia: https://desplegando.cloud/

Ideas iniciales:

- No compiten entre sí, resuelven problemas distintos
- SNS no es la versión vieja de EventBridge.
- Kinesis no es SQS pero más caro.
- No compiten entre sí, cada uno existe para un problema diferente.

Analogías:

- SQS es una fila del banco. Dejas un mensaje y el cajero (un solo consumidor) lo atiende cuando puede.
- SNS es un grupo de WhatsApp. Mandas un mensaje y todos los que están en el grupo lo reciben al mismo tiempo. Si alguien no está mirando el teléfono, se lo pierde.
- EventBridge es un operador telefónico inteligente. Recibe la llamada y la conecta con el departamento correcto según lo que dices, no solo según a quién iba dirigida.
- Kinesis es una cinta transportadora. Los datos fluyen sin parar, en orden, y varias estaciones de trabajo pueden leer de la misma cinta al mismo tiempo, incluso volver atrás si hace falta.

## SQS

Tu opción cuando necesitas desacoplar dos servicios y absorber picos de tráfico sin perder nada, los mensajes quedan guardados hasta 14 días.

Tiene dos opciones:

- Standard. Rapidísimo, orden no garantizado.
- FIFO. Orden garantizado, con límite de velocidad. No lo uses si el mismo mensaje tiene que llegar a varios consumidores a la vez.

## SNS

El rey del fan-out (distribuir una entrada a varios destinos): publicas una vez y disparas varias acciones en simultáneo (un email, una alerta, un log).

SNS no tiene memoria, si el suscriptor no está disponible en ese momento, el mensaje se pierde.

Por eso el combo más clásico en arquitecturas serverless es SNS + SQS: fan-out inmediato con la resiliencia de la cola detrás.

## EventBridge

Es como SNS pero con cerebro, puede leer el contenido del evento y rutearlo con reglas, ejemplo: si el pedido es de más de $500, mándalo a auditoría.

Es también el que trae conectores nativos para Stripe, Shopify, Auth0 y otros SaaS, si estás arrancando una arquitectura event-driven desde cero, probablemente sea tu punto de partida.

## Kinesis

La mayoría de los proyectos no lo necesitan, es para flujo masivo y continuo de datos: logs de miles de servidores, clicks de millones de usuarios, sensores IoT.

La diferencia clave con SQS es que los datos no desaparecen al leerlos, puedes hacer replay si algo sale mal.

## Árbol de decisión

- ¿Necesitas que varios servicios reciban el mismo evento?
  - Sin lógica de ruteo: SNS.
  - Con lógica de ruteo según el contenido: EventBridge
- ¿Necesitas absorber picos y que el consumidor procese a su ritmo? SQS
- ¿Tienes un flujo continuo y masivo de datos que necesitas poder re-leer? Kinesis
- ¿Tu caso es simple y no necesitas nada sofisticado? → Empieza con SNS + SQS y evoluciona si hace falta.
