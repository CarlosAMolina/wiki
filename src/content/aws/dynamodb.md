# DynamoDB

## Introducción

DDB = DynamoDB.

Es un NoSQL Database as a Service (no tienes que gestionar servidores o infraestructura).

Gestiona información como key/value o documents.

Es un servicio público.

Tiene resilencia sobre las AZs y de manear opcional global.

Es muy rápida, basada en SSD.

Ofrece backups y cifrado.

Muy utilizada para serverless, permite integración event-driven, es decir tomar acciones cuando los datos en db cambian.

Se accede a sus datos mediante consola, CLI o API. No soporta SQL, aunque sí PartiQL (un lenguaje parecido a SQL).

Se cobra por lecturas, escrituras, storage y características.

## Tabla

Formada por items, cada item puede tener máximo 400KB (combinación de pk, atributos, nombre de los atributos, etc).

Todos los items comparten la misma primary key; podemos pensar en un item como en una fila en una db relacional.

Puede tener infinitos items.

Un item está formado por:

- Clave primaria. Puede ser:
  - Simple: es la partition key.
  - Compuesta: combinación partition key más sort key; se llama SKey. En este caso lo que debe ser único en la tabla es la combinación de la partition y la sort keys.
- Atributos. Pueden tener cualquier valor y tener diferente esquema en cada item.

## Backups

Hay 2 tipos: on-demand y pitr.

### On-Demand

Se realiza una copia manual.

Permite seleccionar el cifrado.

Guarda los datos de la tabla y su configuración.

Debemos eliminarla nosotros.

Útil para llevar una tabla a otras regiones.

### PITR

PITR = point in time recovery.

Por defecto no está activado. Tras hacerlo guarda datos durante 35 días; cada segundo realiza un backup.

## Capacidad

Al hablar de capacidad, nos referimos al rendimiento o velocidad, no al espacio.

Al provisionar una tabla, tienes dos modos disponibles, tras elegir uno luego puedes cambiar a otro:

- On-demand.
- Provision.

### On-demand

No sabes la carga, se ajusta automáticamente.

Cobro por millón de unidades R o W.

Es más caro que el modo provision.

### Provision

Configuras lo deseado.

Se cobra por RCU (read capacity units), WCU (write capacity units), storage y características.

Cada operación consume mínimo 1 RCU o WCU.

[Documentación sobre RCU y WCU](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ProvisionedThroughput.html#ItemSizeCalculations.Writes).

1 RCU consume 4KB o más, aunque la lectura sea de menos KB se redondea a 4; por lo que es conveniente agrupar queries para reducir coste. Son 4KB por segundo.

1 WCU consume 1KB o más, se redondea a 1 KB. Es por segundo.

Cada tabla tienen un RCU y WCU burst pool de 300 segundos.

#### Read operations

En DynamoDB los datos se replican en las AZ, en los llamados Storage Nodes. Hay un nodo lider y si falla este, otro pasa a ser el lider; los datos se guardan en el node lider y se replican al resto, esto tarda milisegundos.

Hay dos modos:

- Strongly consistency read. Asegura que todos los nodos tienen la información actualizada porque siempre se lee del lider node.
- Eventually consistency read. No es seguro de que todos los nodos tengan la información actualizada. Se leen datos de cualquier nodo. Permite leer el doble de datos que el strongly consistency read para el mismo RCU, es 50% más barato que strongly consistency.

Para calcular la provisión, es necesario saber la media de los ítems a recibir por segundo y la media de su tamaño:

1º. RCU por ítem = redondear enteros (tamaño ítem / 4 KB)
2º. Multiplicar valor anterior por el número de lecturas por segundo. Este valor es para strongly consistency, de ser eventually consistency, dividimos el resultado entre dos.

En caso de aumentar la carga de lecturas, podemos incrementar el RCU.

#### Write operations

Para determinar la provisión, necesitamos conocer la media de los items por segundo que serán escritos y la media de su tamaño, con esto el cálculo es:

1º. WCU por ítem = redondear enteros (tamaño del item / 1 KB)
2º. WCU total = multiplicar valor anterior por el número de items por segundo.

## Query

La petición debe pedir una PK o combinación de PK y SK.

La capacidad consumida es la suma de la información obtenida. Ejemplo, si la query obtiene 2 items, uno de 2.5KB y otro de 1KB, el consumo será de 1 RCU.

Del item recibido podemos elegir obtener atributos específicos en lugar de todos, pero los KB consumidos corresponden a todo el item, es decir, filtrar atributos no lo reduce.

Para hacer una petición sobre determinados valores, no es posible con query ya que solo permite solicitar un valor de las claves primarias, hemos de usar scan.

## Scan

Es menos eficiente que utilizar query, pero ofrece mayor flexibilidad.

Analiza todos los items de una tabla. Por lo que la capacidad consumida es de todos los iems.

Podemos aplicar cualquier filtro sobre los atributos de un item, por ejemplo que un valor esté comprendido entre otros valores que digamos nosotros.

## Índices

Permiten que las queries sean más eficientes.

Aunque `query` es la opción más eficiente, solo puede trabajar con un valor de PK en cada petición y con una o varias SK. Gracias a los índices, creamos vistas alternativas en los datos de las tablas que permiten hacer otros tipos de peticiones, se pueden indicar los atributos a utilizar.

Hay dos tipos:

- LSI.
- GSI.

Se recomienda usar GSI al ser más flexible y LIS cuando necesitemos strong consistency.

Al crearla puedes elegir utilizar: todos los atributos, solo algunos o solo las keys. De solicitar un atributo no añadido, se obtendrá el valor, pero por detrás se harán peticiones muy costosas.

### LSI

LSI = Local Secondary Indexes.

Permite crear SK alternativos sobre el mismo PK. Son como una vista de la tabla, donde se emplea la misma PK y una SK diferente.

Estos índices son sparse, lo que implica que únicamente los items con valor en la SK serán añadidos al índice. Esto es una ventaja porque de usar peticiones scan sobre esta vista, se leerán menos datos y no tendrá tanto coste.

Deben crearse junto a la tabla, más tarde no es posible.

Máximo 5 por tabla.

Comparte el RCU y WCU de la tabla.

### GSI

GSI = Global Secondary Indexes

Permite crear distintos PK y SK alternativos.

Pueden crearse en cualquier momento.

Máximo 20 por tabla.

Tienen su propio RCU y WCU.

Como los LSI, son sparse. Solo se añadirán a la nueva visa los items con valor en la nueva PK y SK.

Estas tablas son siempre eventually consistent porque la replicación entre la tabla y las vistas es asíncrona.

## Streams & triggers

Los streams almacenan cronológicamente los cambios que ha tenido un item cuando se hace insert, update o delete.

Almacena las últimas 24 horas.

Se activa por tabla.

Hay 4 opciones view types disponibles:

- keys_only. Se guarda la PK y de manera opcional la SK del item.
- new_image. Almacena el item tras el cambio.
- old_image. Guarda el item antes de cambiar.
- new_and_old_images. Guarda el item antes y después del cambio.

Los streams son la base de los triggers, que permiten invocar a lambdas cuando nuevas entradas se añaden al stream. En este momento se genera un evento que contiene información de lo que ha cambiado, los datos que contiene depende del view type.

Utilizado para reporting, analytics, aggregation, messaging o notificación.

## Global tables

Las tablas se crean en distintas regiones y son añadidas al mismo global table.

Cuando haya un cambio en cualquiera de las tablas que forman parte del global table, se replicará en todas las demás, tarda menos de un segundo, aunque estén en otras regiones.

Todas las tablas pueden utilizarse para escritura y lectura.

En caso de conflicto, se utiliza el último dato escrito.

Solo puedes tener lecturas strongly consistent en la misma región, en otras será eventually consistent.

Proporciona high availability, disaster recovery a nivel global y continuidad de negocio global.

## DAX

DAX = DynamoDB Accelerator.

Es una caché en memoria, utilizarlo requiere menos gestión que una caché de db normal ya que está integrado de manera nativa con DynamoDB y usan la misma SDK.

Para emplearlo, la aplicación debe hacer uso del DAX SDK.

Pese a que DynamoDB es un servicio público, DAX se despliega en una VPC, los nodos deben estar en todas las AZ para tener high availability.

En una AZ está el nodo primario y se replica al resto de nodos en las otras AZ, los cuales funcionan como read replicas. El nodo primario se comunica con DynamoDB. Si falla el nodo primario, una réplica pasará a ser el primario.

Hay dos tipos de caché:

- Item caché. Cachea los items de manera individual.
- Query caché. Almacena colecciones de operaciones que son el resultado de una operación query o scan; también guarda los parámetros utilizados en la query original.

Al escribir datos en DynamoDB, puedes elegir que estos se escriban en DAX, también irán a DynamoDB.

Con DAX se reduce el coste si la aplicación requiere altas cargas de peticiones de lectura.

El tiempo de respuesta disminuye de un segundo a nanosegundos.

Puede escalar añadiendo más instancias o incrementando su tamaño.

No debe usarse en aplicaciones que requieren strong consistency.

## TTL

Especifica el tiempo para que un item se elimine de una tabla.

Un proceso se encarga de marcar como expirados los items y otro proceso de eliminarlos. También se elimina de los índices y queda guardada la operación en el stream de haberlo. Además, puede configurarse un stream para guardar los eventos de eliminación, lo que es útil para conocer las TTL aplicadas; tiene una duración de 24 horas.

No supone costre adicional.
