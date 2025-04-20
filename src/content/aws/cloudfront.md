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
- Políticas del protocolo que utilizar: HTTP o/y HTTPS.
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

Con el distribution se crea un nombre de dominio para este distribution que termina en `cloudfront.net`, ejemplo https:/d123456abcdef8sdf.cloudfront.net`. Será único en cada distribution que creemos. También puede utilizar un dominio nuestro.

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
