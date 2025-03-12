# DB AWS

## Tipos de bases de datos

### Relacionales

La información se guarda en tablas que tienen una estructura fija, a esta estructura se le llama esquema.

Las columnas se las conoce como atributo.

No son aconsejables para guardar datos que cambian sus relaciones, como las relaciones de una red social por ejemplo Facebook.

La mayoría de bases de datos relacionales poseen un almacenamiento de tipo row que tiene sus ventajas e inconvenientes:

- Ideales para hacer operaciones en las filas: añadir, actualizar o eliminar. Por tanto, a las db con almacenamiento en filas se les llama OLTP (online transaction processing).
- Hay que buscar la fila con el valor y luego se lee toda la fila hasta obtener el atributo deseado.
- Para obtener un atributo de todas las filas, has de leerlas todas.

### No relacionales

#### Key-value

Almacenan un key y un value, las keys son únicas. Puedes guardar en el valor cualquier tipo: string, JSON, imagen, etc. Ejemplo como key guardo un datetime y en value el número de cookies que se han eliminado en ese momento.

Donde se guarda la información no tiene esquema.

Los datos guardados no tienen estructura, no hay tablas ni tablas que relacionan unas con otras.

Es una base de datos:

- Escalable: los datos pueden dividirse en diferentes servidores.
- Muy rápida al no tener estructura.
- Es muy habitual para caché en memoria.

#### Wide Column Store

Es una variación del tipo key-value, por lo que también es muy rápida. Ejemplo en AWS: DynamoDB.

Tienen una o más keys, la llamada `partition key` y otras opcionales; en DynamoDB a las opcionales se les llama sort key. Todas las filas tienen que mantener el mismo número de keys y la combiinación debe ser única (no estoy seguro de si en toda la base de datos o solo en las tablas que explicamos a continuación).

Ofrecen grupos de items llamados tablas, pero no son las mismas tablas en en una base de datos relacional. Los items de una tablas pueden tener atributos (un item puede tener varios atributos) y estos pueden ser distintos para cada item.

#### Document

La información se guarda en documentos, los documentos son una estructura de datos como JSON o XML.

Documentos en la misma base de datos pueden tener distinta estructura.

La key es única en la base de datos y el value es el documento.

Puedes interactuar con los valores del documento, es aconsejable cuando interactúas con el documento entero o con sus atributos y pueden cambiar. Por ejemplo para almacenar contactos o pedidos de una tienda. Permite hacer queries a los datos del documento, aunque estos tengan distintos niveles de profundidad.

Pueden organizarse los documentos en una jerarquía.

#### Column

Ejemplo de este tipo: Redshift.

Ofrecen ventajas sobre las limitaciones de las bases de datos con un almacenamiento en filas (visto en la sección de las bases de datos relacionales).

La información es la misma que en db con almacenamiento en filas pero los datos se almacenan agrupándolos en columnas, lo que implica:

- Ineficiente para procesar con transacciones.
- Ideales para reportes o analytics sobre valores de un mismo atributo. Ejemplo los productos vendidos en un periodo, cuales se vendieron más, etc.

#### Graph

Al contrario que las bases de datos relacionales donde las relaciones se guardan y hay que calcularlas al solicitar los datos, en las de tipo graph la relación está almacenada con los propios datos y no hay que calcularlas, por lo que son más rápidas para saber las relaciones.

Los items guardados se llaman nodos, pueden tener atributos (son una key un valor). Ejemplo el nodo persona tiene nombre y fecha de cumpleaños.

Los nodos están relacionados mediante los edges, que tienen un nombre y una dirección (de qué nodo a qué nodo apunta), también pueden tener información guardada en keys-values. Ejemplo el edge indica que un nodo persona trabaja para un nodo empresa (ejemplo, el edge se llama `:works_for`) y además guarda desde cuando (ejemplo `startdate: 2025-01-01`).

Pueden guardar gran cantidad de relaciones y complejas.

Ejemplo de uso en una red social para relacionar a los usuarios.

## Db en ec2

Normalmente es una mala idea, pero hay situaciones en que puede ser necesario:

- Para poder acceder al sistema operativo de la base de datos ya que las soluciones de DB que ofrece AWS no lo permiten.
- Querer hace configuraciones que no lo permitan bases de datos gestionadas por AWS.
- AWS no ofrece la versión deseada de la base de datos.
- AWS no ofrece la arquitectura que necesitamos (por ejemplo para conseguir una replicación o resilencia específicas).

Los puntos negativos de ejecutarlo en una instancia EC2 es:

- Mayor gestión por nuestra parte, realizar backups, etc.
- El EC2 funciona en una AZ por lo que la db también y si la AZ falla, fallará la db.
- EC2 no es serverless, es más complicado de escalar.
- Perdemos características y mejoras de las db que ofrece AWS.
