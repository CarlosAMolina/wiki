# CloudFront

## Introdución

CF = CloudFront.

Es un CDN (Content Delivery Network), por lo que se encarga de cachear contenido en diferentes partes del mundo para ofrecer la respuesta más cercana posible a los clientes y de este modo mejorar el rendimiento.

Puede trabajar con ACM (Aws Certificate Manager), lo que permite utilizar certificados SSL en CF, por ejemplo HTTPS.

Es importante destacar que, CloudFront es solo para operaciones de descarga. No tiene caché de escritura, las operaciones de subida se comunican directamente con el origen (explicado en la sección de términos).

## Términos

### Origen

Localización original del contenido.

Puede ser S3 o cualquier servidor con una IP pública.

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

Una vez tenemos el distribution, se despliega en los edge location a los que pueden hacer peticiones los clientes. Es decir, lo que configuramos se despliega fuera de CF, pero la configuración está en CF.

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

El cache HIT es la frecuencia con que el edge location entrega contenido a los clientes. Cuanto mayor sea, menor es la carga en el origen y mejores respuestas tendrá el cliente.

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

- Para el viewer protocol, los certificados deben ser emitidos por un Certificate Authority (CA) o ACM en la región us-east-1.
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
