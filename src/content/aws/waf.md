# WAF

## Introducción

WAF = Web Application Firewall.

Es un firewall de layer 7. Protege por ejemplo contra SQL Injection, Cross-Site Scripting, etc.

Puede proteger servicios de AWS globales como CloudFront y regionales como ALB, AppSync, API Gateway, etc. No protege EC2 o lambda.

Es posible almacenar los logs en S3 (puede tardar hasta 5 minutos), cloudwatch, firehose, etc.

## WEBACL

WEBACL = Web Access Control List.

Es donde se configura el WAF.

Está formado por Rule Groups y estos por rules.

El WEBACL puede actualizarse manualmente o mediante programas que, por ejemplo analizan los logs generados.

Puede funcionar en modo que por defecto el tráfico que no matchee las reglas sea bloqueado o permitido.

Las rules requieren un procesado, por lo que consumen recursos, hay un límite en los recursos que pueden emplearse. Existe la unidad WCU (Web ACL Cpacity Units), no confundir won WCU de dynamodb; indica la complejidad de las reglas, tiene un valor de 1.500 por defecto que puede incrementarse abriendo un ticket.

Los WEBACL se asocian con recursos, esto puede tardar un tiempo porque ha de propagarse la configuración a los edge locations. Pero si ya existe el WEBACL, ajustarlo lleva menos tiempo que asociarlo.

Un recurso puede tener asociado solo un WEBACL pero un WEBACL puede estar asociado a distintos recursos.

## Rule groups

Son grupos de reglas.

No tiene acciones configuradas por defecto; se definen cuando los grupos o las reglas se añaden a los WEBACLs.

Algunas son gestionadas por AWS o el Marketplace, otras por nosotros, otras por el dueño del servicio (shield, manager del firewall, etc.).

Son una entidad separada de los WEBACL, un rule group puede utilizarse en múltiples WEBACL y un WEBACL puede tener varios rule groups.

## Rules

Están formados por 3 partes:

- Type: define cómo funciona.
  - Regular. Realizan una acción cuando matchean algo:
  - Rate-based. Detectan si algo ocurre con un cierto rate. Ejemplo, detectamos 100 conexiones ssh en 5 minutos.
- Statement: qué matchea. Puede trabajar con IP, header, cookie, query parameters, paths, body (solo los primeros 8 kbytes), HTTP method, etc. Puede aplicar expresiones regulares y combinar múltiples condiciones.
- Action: qué lleva a cabo al haber un matcheo. Ejemplo:
  - Permitir. Con los de tipo rate-based no puede aplicarse allow, no tiene sentido.
  - Bloquear.
  - Mostrar un captcha.
  - Llevar un conteo de las detecciones.
  - Dar una respuesta específica, las cabeceras modificadas contienen el término x-amzn-waf.
  - Aplicar un label, este label se utiliza dentro del WEBACL para que otras reglas tomen decisiones en caso de que existan. De aplicarse un action de bloquear o permitir, no se ejecutan más reglas.

## Coste

Al mes se cobra:

- 5$ por cada WEBACL.
- 1$ por cada regla.
- 0.6$ por cada millón de peticiones y por WEBACL.
- De aplicar reglas inteligentes hay costes adicionales: bot control, captcha, control de fraude, etc.
