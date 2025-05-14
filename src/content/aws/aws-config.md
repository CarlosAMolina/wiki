# AWS config

Al activarlo, se guarda un histórico con las configuraciones de los recursos, cada vez que haya un cambio se almacena un nuevo item. 

Los datos guardados son:

- Configuración del recurso.
- Relación con otros recursos.
- Quién hizo los cambios.

El histórico se guarda en el S3 config bucket.

Permite detectar cambios con respecto a lo que establezcamos, pero no evita que se realicen las modificaciones. Por ejemplo, en un security group podemos permitir X puertos, si se añaden otros diferentes, lo detectará pero no evitará que se añadan.

Monitoriza una región y una cuenta de AWS aunque puede trabajar con multi región y multi cuenta.

Pueden establecerse config rules (gestionadas por AWS o propias, estas últimas se revisan con funciones lambda), y las configuraciones serán comparadas con estas, notificando a AWS Config si la configuración se matchea o no. A partir de aquí puede invocar a SNS o EventBridge y de este modo desencadena lambdas que realicen acciones en base a los cambios; también puede emplearse SSM para modificar estos cambios, lambda es bueno para cambios relacionados con la cuenta y SSM para temas sobre configuración de instancias.
