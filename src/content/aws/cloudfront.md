# CloudFront

## Introdución

CF = CloudFront.

Es un CDN (Content Delivery Network), por lo que se encarga de cachear contenido en diferentes partes del mundo para ofrecer la respuesta más cercana posible a los clientes/viewers y de este modo mejorar el rendimiento.

Es un CDN global capaza de cachear contenido estático y dinámico.

Puede trabajar con ACM (Aws Certificate Manager), lo que permite utilizar certificados SSL en CF, por ejemplo HTTPS.

Es importante destacar que, CloudFront es solo para operaciones de descarga. No tiene caché de escritura, las operaciones de subida se comunican directamente con el origen (explicado en la sección de términos).

## Términos

### Origen

Localización original del contenido.

Puede ser S3 o cualquier servidor con una IP pública.

Si en un distribution tienes varios orígenes, puedes crear un origin group, esto ofrece resilencia. En el behaviour se escoge entre utilizar un origin o un origin group.

Tipos de origin:

- AWS S3 bucket.
- AWS media package channel endpoint.
- AWS media store container endpoint.
- Resto. Son orígenes propios, como web servers.

S3 bucket es la manera más sencilla de integrar con CloudFront, ofrece otras opciones de configuración que si eligiéramos un custom origin. Por ejemplo podemos configurar que el bucket sea solo accesible a través de CF; también pueden añadirse cabeceras a la petición.

De usar S3 origin, el origin protocol será igual que el viewer protocol, ejemplo HTTP o HTTPS.

De utilizar un custom origin, se puede configurar:

- Mínimo protocolo SSL.
- Origin protocol, a diferencia que en S3 origin, no es obligatorio que sea igual al viewer protocol.
- Puerto.
- A diferencia de un origin S3, no puede configurarse directamente para que el origin solo sea accesible con CF. Pero pueden configurarse para añadir unas caberas propias y de este modo hacer un filtrado en origin en base a ellas.

Al configurar el origin en el distribution:

- De utilizar el nombre del bucket, el origen será de tipo S3.
- De especificar el nombre del dominio DNS del sitio web estático, para CF el origen será un custom origin, creo que aunque sea S3, CF lo verá como un custom origin.

### Behaviour

Los orígenes no están asociados directamente a los distribution sino que lo están a los `behaviours` y estos a los distribution.

Los behaviours son parte de los distributions.

Es donde se definen patrones para matchear paths de objetos. El patrón por defecto matchea todos los objetos. En estos patrones se indica el path de los objetos.

Permite tener distintas configuraciones en un distribution, una para cada path matcheado.

Podemos configurar:

- Orígenes.
- Caché.
- Restringir acceso. Por ejeplo a rutas que contienen certificado. Se utiliza signed cookies o signed URLs. Opciones:
  - Trusted key groups. Es el modo actual que debe usarse.
  - Trusted signer. Es el modo legacy.
- Políticas del protocolo que utilizar:
  - HTTP y HTTPS.
  - Que HTTP se rediriga a HTTPS.
  - Solo HTTPS.
- Métodos HTTP que utilizar: GET, POST, etc.
- Cifrado, desde el punto de entrada del edge location al resto de la red de CF.
- Lambda edge functions.
- TTL.
- Etc.

Si una petición matchea un behaviour, se utiliza el behaviour matcheado, en caso contrario se acaba utilizando el behaviour por defecto.

Por ejemplo, tenemos un bucket con imágenes que no deben ser públicas, creamos un behaviour `img/*` y las peticiones a contenido en este path utilizará una configuración establecida para ello.

### Distribution

Es la unidad de configuración de CF.

En esta configuración se indica:

- Cuál es el origen. Pueden indicarse uno o más orígenes.
- Tiene como mínimo un behaviour, puede tener más.
- La localización de los edge locations a utilizar.
- AWS WAF.
- Nombre de dominio.
- Certificado SSL.
- Security policy (versión de SSL o TLS).

Con el distribution se crea un nombre de dominio para este distribution que termina en `cloudfront.net`, ejemplo `https:/d123456abcdef8sdf.cloudfront.net`. Es el modo por defecto que utilizamos para acceder a un distribution, es un CNAME en el DNS. Será único en cada distribution que creemos.

El distribution también puede utilizar un dominio nuestro, es decir, un Alternate Domain Name. Para ello hemos de verificar que el domino es nuestro, para lo que se requiere un certificado SSL (se use o no se use HTTPS) que coincida con el nombre utilizado, el certificado se crea o importa con ACM.

Una vez tenemos el distribution, se despliega en los edge location a los que pueden hacer peticiones los viewers. Es decir, lo que configuramos se despliega fuera de CF, pero la configuración está en CF.

### Edge location

Son las piezas de la infraestructura global donde la información está cacheada.

Si lo solicitado no se encuentra en el edge location, se solicita al regional edge cache. Y el edge location lo cachea al recibirlo.

### Regional Edge Cache

Es otra capa de caché.

Es una versión más grande que un edge location, hay menos de ellos y pueden cachear más cantidad de datos.

Se encuentra ete los edge location y el origen. Sopora los edge location en la misma área geográfica.

Se utilizan para cachear datos de acceso menos frecuente. Es beneficioso cuando ofrece mejor resultados que acceder directamente al origen, y en despliegues globales.

Como se indicó en los edge location, se utiliza el regional edge cache si los edge location no tienen la información solicitada en su caché. Si la información tampoco está en el regional edge cache, este la solicita al origen y la cachea.

### Viewer protocol

Es la conexión entre el cliente o viewer y el edge location.

### Origin protocol

Es la conexión entre el edge location y el origen.

## TTL

Controlan el tiempo que el contenido se encuentra en los edge locations.

Cuando el TTL ha pasado, el objeto no es eliminado inmediatamente del edge location; al solicitar el objeto en el edge location, este consulta con el origen si el objecto ha cambiado:

- Si no ha cambiado, el origen responde con un `304 Not Modified`. Se mantiene la versión del edge location como la actual.
- La versión del origen es distinta. Devuelve un `200 OK`.

El cache HIT es la frecuencia con que el edge location entrega contenido a los viewers. Cuanto mayor sea, menor es la carga en el origen y mejores respuestas tendrá el viewer.

Los objetos tienen por defecto un validity period o TTL de 24 horas; tras este tiempo, el objeto se marca como expirado.

Podemos configurar un TTL mínimo y uno máximo. Estos no afectan al TTL, indican el rango de posibles valores que puede tener un objeto, porque cada objeto puede tener un TTL distinto, se especifica en las cabeceras. Si las cabeceras sobrepasan estos límites, se usa el valor de los límites.

Cabeceras para indicar cuándo un objeto se considera expirado:

- Cache-Control max-age. Son segundos.
- Cache-Control s-maxage. Son segundos.
- Expires. Fecha y hora.

Configurar una o ambas de las cabeceras `max-age` y `s-maxage` causa el mismo efecto.

Las cabeceras se configuran según el origen:

- S3. Se indica en los metadatos de los objetos; mediante API, CLI o con la consola de AWS.
- Aplicación propia. La aplicación o el web server establece las cabeceras.

## Cache Invalidation

Al utilizarlo, se marcan como expirados los objetos especificados mediante un path, independientemente de su TTL. Ejemplo:

- /images/dog1.png
- /images/dog*
- /images/*
- /*

Se realiza en el distribution, por lo que afecta a todos los edge locations del distribution y es algo que lleva tiempo, no se aplica de manera inmediata.

Tiene un coste, independiente del número de objetos a los que aplica el cambio.

Si de manera habitual estamos invalidando la caché de objetos, es mejor utilizar versionado en los nombres. Ejemplo, dog_v1.png, dog_v2.png, etc. Y actualizar la aplicación para apuntar a la versión deseada. Versionado en el nombre tiene ventajas como:

- No afecta que el objeto esté cacheado en el navegador del usuario ya que al tener otro nombre, hay que solicitar el nuevo.
- Los logs son mas claros al saber qué objeto se utiliza al tener cada uno distinto nombre.
- Mantenemos todas las versiones de los objetos, por lo que son consistentes en todos los edge locations y podemos utilizar cualquiera independientemente.
- Reduce costes al no tener que utilizar cache invalidation continuamente.

Versionado en los nombres es diferente al versionado de S3 porque este utiliza el mismo nombre para distintas versiones.

## SSL

Podemos habilitar que se use SSL por defecto al acceder al distribution.

CF emplea un certificado SSL por defecto que utiliza `*.cloudfront.net`, por lo que los distributions configurados para trabajar con la dirección terminada en esto, harán uso del certificado.

Hay dos conexiones SSL:

- Desde el cliente/viewer a CF. El certificado aplicado a CF debe coincidir con el nombre DNS de lo utilizado por el cliente para acceder a CF.
- De CF al origen. El certificado instalado en los orígenes debe coincidir con el nombre del DNS que CF utiliza para conectarse con el origen.

Ambas conexiones, y cualquier conexión intermedia, necesitan certificados públicos válidos, no sirven certificados autofirmados:

- Para el viewer protocol, los certificados deben ser emitidos por un Certificate Authority (CA) o ACM en la región us-east-1. Esto es obligatorio solo si ACM debe trabajar con CF (explicado en el apartado de ACM).
- Respecto al origin protocol, en el caso de:
  - S3 no hay que preocuparse por el certificado, lo gestiona de manera nativa.
  - ALB. Puede usar certificados de una CA o de ACM.
  - Si el origen es un EC2 o es on-premise. Solo puede utilizar certificados de CA, no de ACM.

### SNI

SNI = Server Name Indication.

Antes de 2003, si un servidor con una IP tenía varios dominios, el cliente indicaba el dominio deseado en las cabeceras de la petición. Como el cifrado se inicia en la conexión TCP, y las cabeceras se encuentran en la layer 7/aplicación, después de la conexión TCP, no es posible usar SSL para verificar la identidad del host y esto implicaba que cada dominio con SSL tuviera su propia dirección IP.

Por esto apareció SNI, es una extensión de TLS que permite que un cliente indique a un servidor el dominio a visitar. Esto habilita que distintos dominios usen la misma IP; cada dominio debe tener su propio certificado SSL.

Pero no todos los navegadores soportan SNI.

CF no tiene coste adicional por usar SNI pero sí para IPs dedicadas en los edge location (600$ al mes por cada distribution), esta última opción es la que permitiría que todos los navegadores puedan usar HTTPS contra una IP con varios dominios.

## OAI

### Introducción

Utilizado para securizar los origin, evita que alguien acceda a ellos directamente sin usar CF.

La manera recomendada de evitar que se acceda directamente al origin es OAC, OAI es el modo antiguo.

Es un tipo de identidad, con diferencias respecto a IAM user o IAM role.

### Origin S3

Se asocia el OAI con los CF Distributions y lo usan los edge locations, de modo que cuando estos acceden a un origin S3, el CF es el OAI; al ser una identidad, se le pueden aplicar políticas del bucket para permitir el acceso a los recursos y los accesos sin OAI están denegados.

### Custom origin

A diferencia de S3, no pueden utilizarse identities.

Hay dos opciones, pueden utilizarse ambas:

- Configurar los edge locations para modificar las peticiones de los clientes, de modo que les añade unas cabeceras que espera el origin para ofrecer el contenido. Estas nuevas cabeceras son entre el edge location y el origin, y al estar las peticiones protegidas con HTTPS, nadie excepto CF y el origin pueden conocer su valor.
- Configurar un firewall en el origen que solo acepte el rango de IPs de CF.

## Private Behaviours

Un CF distribution se crea con un behaviour. Este behaviour puede ser público o privado.

De ser privado, se accede al contenido utilizando cookies firmadas o URLs firmadas.

Un signer es una entidad que puede crear signed URLs o cookies, por ejemplo, una lambda que realice esta función.

Cuando un signer se asocia a un behaviour, este es privado. Para configurar esto:

- La opción antigua era crear un Trusted Signer. En esta opción el Account Root User genera un CloudFront Key. Al tener esta key, la cuenta actúa como Trusted Signer.
- El modo actual es emplear trusted key groups, son los que deciden las keys que pueden utilizarse para obtener signed key groups o cookies. Este método ofrece las ventajas de:
  - No tener que utilizar el Account Root User.
  - Ofrece más flexibilidad en la configuración: puede gestionarse con la API de CF y permite crear múltiples keys.

### Signed URLs vs Cookies

Signed URLs:

- Dan acceso a un solo objeto.
- Algunos clientes no permiten trabajar con cookies.
- Ofrecen una URL diferente por lo que de querer que la URL no cambie, es mejor utilizar las cookies.

Cookies:

- Permite gestionar varios objetos, incluso dividiéndolos en grupos, por ejemplo, las imágenes de gatos.

### Arquitectura private behaviours

Por ejemplo, un usuario se logea a través de API Gateway, la lambda signer genera una cookie que se guarda en el navegador del usuario y lo usan las peticiones a CF de modo que se habilita el acceso.

En este ejemplo es importante que el origin esté protegido con OAI para evitar que se salte esta seguridad, es decir, acceder a todo el contenido sin restricción.

## OAC

OAC = Origin Access Control

Es la manera recomendada de evitar que se acceda directamete al origin. OAI es el modo antiguo.

Puede configurarse de dos maneras:

- En la consola de AWS, en el menu de la izquierda: Security > Origin access.
- En la consola de AWS, en la parte de CloudFront, seleccionamos un Distribution.

De ese modo generamos un policy que debemos utilizar en el bucket S3 origin, por lo que debemos modificar las políticas de este. Esta política indica que el bucket solo aceptará peticiones de CF.

## Lambda@Edge

Permite a CF ejecutar lambdas ligeras en CF edge locations para modificar el tráfico entre el viewer y el origen.

Lambda@Edge puede ejecutar una lambda en cada una de estas peticiones que hay al utilizar CF:

- Viewer request: del viewer al edge location. La lambda se ejecuta después de que CF recibe la petición del viewer.
- Origin request: del edge location al origin. Se ejecuta la lambda antes de que CF envíe la petición al origin.
- Origin response: del origin al edge location. Se ejecuta la lambda después de que CF recibe la respuesta del origin.
- Viewer response: del edge location al viewer. Se ejecuta la lambda antes de que la respuesta se lleva al cliente.

Las lambdas en un edge location no tienen todas las características de una lambda normal de AWS:

- Solo permite Node.js y Python.
- No pueden trabajar con lambda layers.
- Funciona en AWS public space, por lo que no puede acceder a VPCs.
- Las funciones tienen otros límites de tamaño y tiempo de ejecución que las lambdas habituales:
  - Viewer request/response. Máximo memoria 128 MB y 5 segundos de timeout.
  - Origin request/response. Máximo memoria igual que una lambda normal y 30 segundos de timeout.

Ejemplos de uso ([link](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html#lambda-examples-redirecting-examples)):

- Viewer request:
  - A/B testing. Suele utilizarse con viewer request. Permite ofrecer distintas respuestas sin necesitar que el cliente visite distintas URLs, el cliente utiliza una URL y la lambda apuntará a otras. Ejemplo, devolver distintas versiones de imágenes, cada una aleatoriamente o en base a un porcentaje de visitas.
- Lambdas en origin request:
  - Migrar entre S3 origins. Por ejemplo, vamos cambiando el porcentaje del tráfico que accede al nuevo origin.
  - Distintas respuestas según el dispositivo que hace la petición. Por ejemplo, enviar una imagen de mayor calidad si la pantalla del viewer lo permite.
  - Distinto contenido según el país del viewer.
