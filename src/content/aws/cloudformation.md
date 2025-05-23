# CloudFormation

## Introducción

CFN = CloudFormation

Es un servicio para gestionar la infraestructura utilizando plantillas.

## Plantillas

Utilizan formato YAML o JSON.

En la plantilla tenemos distintos apartados: versión, descripción, recursos, etc.

Hay algunas peculiaridades como que indicar el número de versión es opcional, pero en caso de hacerlo, entonces es obligatorio que el siguiente apartado sea el de descripción.

### Plantillas no portables

Una plantilla no es portable si por ejemplo no permite:

- Aplicar la misma plantilla varias veces. Por ejemplo, si hardcodeamos el nombre de un bucket S3, solamente podremos lanzar el stack una vez porque el resto dará error al ya existir el bucket.
- Aplicar la plantilla en distintas regiones. Por ejemplo, al hardcodear el AMI ID de una máquina EC2, como el ID cambia entre regiones, la plantilla solo podrá utilizarse en una región.
- Aplicarla en distintas cuentas de AWS.

### Parámetros de plantilla y pseudo parámetros

Son maneras de dar inputs a la plantilla, para influenciar a los recursos a crear y su configuración.

Ayudan a conseguir plantillas portables.

Los parámetros de la plantilla y los pseudo parámetros pueden utilizarse juntos en la plantilla, se complementan entre sí.

#### Parámetros de la plantilla

Se especifica en la plantilla dentro de la sección `Parameters`.

Permiten recibir inputs de fuentes externas como AWS console, CLI o API, cuando el stack es creado o actualizado. Por ejemplo, en la AWS console, tendremos una pantalla donde introducir los valores o elegirlos de una lista.

Pueden configurarse con valores:

- Por defecto.
- Permitidos. Ejemplo, lista de plantillas permitidas.
- Mínimo y máximo valor.
- No echo. Para no mostrar el valor introducido, por ejemplo contraseñas.
- Etc.

Gracias a SSM, podemos acceder a valores de los activos. Por ejemplo, como el AMI ID de un EC2 cambia en cada región, indicamos el tipo de EC2 y la plantilla obtendrá su valor. [Documentación](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-public-parameters.html).

#### Pseudo parámetros

[Documentación pseudo parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html).

En lugar de indicar su valor un usuario o un proceso al crearse el stack, los da AWS basados en el entorno. Ejemplo:
  - `AWS::Region`. Toma el valor de la región en la que se aplique la plantilla.
  - `AWS::AccountId`. ID de la cuenta utilizada para crear el stack.
  - `AWS StackName` y `AWS StackId`. Valores del stack que se está creando.

### Intrinsic Functions

[Documentación](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html).

Son funciones para gestionar los stacks, se utilizan en las plantillas para dar valores a propierties cuando estos valores no están disponibles hasta que se ejecuta la plantilla para crear el stack.

Ejemplo:

- `Ref` y `Fn::GetAtt`. Para referenciar un recurso desde otro. Por ejemplo, para que una subred esté dentro de una VPC que crearemos.
  - `Ref`. Al utilizarlo en parámetros o pseudo parámetros de una plantilla, devuelve su valor. Al utilizarlo en un recurso lógico, por ejemplo al crear una instancia EC2, normalmente devuelve su ID físico, depende del tipo de recurso.
  - `GetAtt`. Devuelve un atributo asociado al recurso, por ejemplo la IP pública de una instancia.
- `Fn::Join` y `Fn::Split`. Para modificar strings. Ejemplo al crear un EC2 con nombre DNS, podemos añadirle el protocolo `http` para tener una URL a la que acceder.
- `Fn::GetAZs` y `Fn::Select`. Lista las AZ en una región y permite elegir una; en realidad devuelve las AZ en las que el default VPC tiene subredes por lo que de no estar configurado correctamente, puede que no devuelva todas las AZ de la región.
- Condicionales (if, and equals, not, or). Para provisionar recursos en base a condiciones. Ejemplo, si estamos en producción o development, elegir distintos tamaños de instancias.
- `Fn::Base64`. Para dar valores que necesitan codificarse en base 64, por ejemplo UserData.
- `Fn::Sub`. Modificar texto en base a runtime information. Ejemplo el ID de una instancia EC2 con `${Instance.InstanceId}`.
- `Fn:Cidr`. Para crear valores de red. Puedes indicar el rango de red, número de subredes a generar, etc.
- Etc.

Pueden utilizarse en conjunto.

### Mappings

Es otra característica que ayuda a crear plantillas portables.

El mapeo relaciona claves con valores. Ejemplo, tener el valor para producción.

Puede que una key devuelva un valor o que haya key de 1º y 2º nivel, por ejemplo obtener el AMI ID de una máquina, la key de 1º nivel es la región y la de 2º el tipo de arquitectura. Ejemplo:

```
Mappings:
  Region Map:
    us-east-1:
      HVM64: "ami-1234..."
      HVMG2: "ami-4321..."
```

En la plantilla se especifica en la sección `Mappings`, y se accede a los valores con el intrinsic function `FindInMap`.

### Outputs

Es una sección opcional en la plantilla donde declaramos valores a mostrar o a los que hacer referencia.

Permite:

- Mostrar información en la AWS CloudFormation console o CLI, por ejemplo, ayuda a conocer los objetos creados.
- De esta manera son accesibles desde el parent stack cuando anidamos stacks en otros.
- Puede exportarse, lo que permite referencias cross-stack.

### Condiciones

Las plantillas permiten definir condiciones que se evaluan antes de crear un recurso y determina si crearlo o no. El usuario puede indicar valores que usar en las condiciones gracias a los template parameters.

Pueden combinarse varias condiciones.

### DependsOn

Utilizado para que un recurso sea creado después de otro o de otros.

CFN para ser más eficiente crea los recursos en paralelo. Tiene en cuenta que si un recurso necesita de otro, por ejemplo una máquina EC2 se encuentra en una VPC, primero creará el recurso que no  necesite de otro. Estas dependencias implícitas se definen con el Intrinsic Function `Ref`.

Con DependsOn podemos definir la dependencia de manera explícita.

Hay casos en que no utilizamos `Ref` para definir un recurso, por ejemplo un ENI; si este depende de otro, en este caso de un API Gateway; y no se ha creado, el stack fallará, a veces funcionará porque los recursos se crearon en el orden correcto y otras fallará; ocurrirá lo mismo al eliminar el stack. Para evitar estos errores aleatorios es necesario emplear `DependsOn`.

## Stack

Stack es lo que CFN crea a partir de una plantilla.

El stack contiene los recursos de manera lógica y los creará de manera física.

El stack mantiene los recursos físicos y lógicos sincronizados, es decir, crea, actualiza o elimina los recursos de manera física según cambie la plantilla. Para ello, cuando actualicemos la plantilla hemos de actualizar el stack con los cambios.

Con la misma plantilla pueden crearse más de un stack.

Al eliminar un stack se borran sus recursos lógicos, lo que implica que también se eliminarán los físicos.

## Recursos

[Referencias de los recursos de CFN](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html).

### Recurso lógico

Es un recurso definido en una plantilla de CloudFormation.

Definen qué queremos crear y CFN se encarga de cómo crearlos.

### Recurso físico

Es un recurso físico creado al crear un stack de CloudFormation. El stack crea un recurso físico por cada recurso lógico.

## cfn-signal

Puede ser que una recurso aparezca como creado correctamente pese a no ser así; por ejemplo una instancia EC2 que tras iniciarse, se indica que se ha creado pero además tiene tiene que ejecutar UserData y aunque este proceso falle, la instancia ya aparecerá como creada.

Para evitarlo, configuramos CFN con un CreationPolicy para que esperece recibir un cierto número de sucess signals antes de marcar el recurso como completado de manera correcta; también puede indicarse un teimout de hasta 12 horas de margen para recibirlas. En caso de recibir una signal de fallo, la creación habrá fallado.

Gracias a cfn-signal se envían estas señales.

También pueden utilizarse WaitConditions para que no se pase a un estado hasta que se reciba una señal o pase cierto tiempo. El WaitConditions genera una preSigned URL para recibir signals, a esta preSigned URL puede enviarse otra información que utilizar en la creación del Stack.
 
## Multi stack

En un stack pueden definirse como máximo 500 recursos.

No permite utilizar los recursos creados en otros stacks de manera sencilla. Por ejemplo, hacer referencia a una VPC creada en otro stack.

Para crear proyectos multi stack, tenemos dos opciones:

- Nested stack.
- Cross stack.

### Nested stack

Hay un stack que es el stack parent o root stack. Este stack puede crear stack hijos, los cuales son capaces de crear otros hijos.

La sub stack se define en la stack padre como si fuera otro recurso, para ello:

- El tipo es `AWS::CloudFormation::Stack`.
- Se indica un TemplateUrl.

Los outputs del nested stack pueden ser utilizados por el root stack y enviarlos a otro nested stack.

Una ventaja de nested stacks es que la plantilla de los sub stacks puede utilizarse en otros stacks, apuntando a su TemplateUrl; se crearán recursos diferentes desde la misma plantilla. Es decir, se reutiliza la plantilla, no el recurso.

Permite evitar la limitación de los 500 recursos máximo de emplear solo un stack. Puedes tener más de 2.500 recursos pero en un despliegue solo puedes crear, eliminar o actualizar 2.500.

Debemos utilizarla cuando los recursos tienen un ciclo de vida relacionado (se crean y eliminan juntos), si los recursos se actualizan de manera independiente unos de otros, es mejor no usar este método.

### Cross stack

En este caso se reutilizan recursos creados por otro stack.

Consiste en exportar los outputs de un stack para que podamos emplearlos en otros. El nombre utilizado en la exportación debe ser único por región y cuenta de AWS.

Para utilizar este valor, se emplea el `!ImportValue` intrinsic function. Solo puede utilizarse en la misma región y cuenta de AWS.

Ejemplo de uso:

- Aplicaciones que necesitan compartir o reutilizar recursos, como por ejemplo una VPC.
- Cuando los recursos tienen diferente ciclo de vida.

## StackSets

Permite modificar infraestructura en distintas regiones y cuentas de AWS.

La plantilla empleada en un StackSet es una plantilla CFN normal.

StackSets se encuentran en la cuenta llamada administradora, administradora significa solamente que es donde residen los StackSets.

Los StackSets tienen referencias a las instancias stack, un stack instance es un contenedor que guarda lo que ocurre en un stack particular; si el stack falla, en el stack instance se indicará el motivo.

Los stack instances y los stacks se encuentran en los target accounts; se llama así a las cuentas donde se despliegan los recursos. Cada stack se encuentra en una región y una cuenta de AWS.

Al crearlo se utiliza roles gestionados por CFN o en conjunto por CFN y aws organizations.

Términos:

- Concurrent accounts. Número de cuentas de AWS en que pueden desplegarse los recursos simultáneamente.
- Failure tolerance. Número de stacks que pueden fallar antes de marcar el StackSet como fallido.
- Retain Stacks. Configura la eliminación de instancias stacks.

Usos:

- Habilitar AWS Config.
- AWS Config rules: MFA, EIPS, cifrado EBS.
- Crear Roles IAM para acceso cross-account.

## DeletionPolicy

De no tener este atributo, CFN por defecto borrará el recurso cuando lo eliminamos de la plantilla.

Permite:

- Mantener un recurso al eliminar el stack, es decir, el recurso no se modifica.
- Hacer un backup si el recurso es compatible con esta opción, por ejemplo:
  - Es útil para no perder información en RDS, EBS, etc. Nosotros seremos responsables de eliminar estos backups.
  - EC2 no es compatible con esta opción de backup.

Eso solo aplica a operaciones de eliminación, no a las que actualizan recursos; además si un recursos se reemplaza (no es una operación de eliminación), se perderán sus datos.

## Stack Roles

CFN utiliza los permisos de la identidad que ha ejecutado el stack.

Por tanto, necesitamos permisos para interactuar con stacks y los recursos de AWS.

CFN es capaz de asumir un rol para obtener sus permisos. Un IAM rol puede pasarse al stack mediante PassRole. Así conseguimos role separation de modo distintas cuentas de AWS pueden lanzar stacks sin tener permisos sobre todos los recursos implicados.

## cfn-init

Con `AWS::CloudFormation::Init` y cfn-init es posible dar configuración/metadatos a una instancia EC2.

Es una alternativa a bootstrapping con UserData, pero UserData no es algo nativo de CFN. Con UserData indicamos qué hacer; con cfn-init solo definimos lo que queremos y CFN lo llevará acabo, si algo ya existe no lo volverá a crear, es mejor que UserData ya que con este último habría que escribir las comprobaciones necesarias.

La configuración se indica en la template mediante `AWS::CloudFormation::Init`, que estará dentro de la sección `Metadata`en la sección `EC2Instance`.

cfn-init:

- Es un script instalado en el sistema operativo del EC2. Se envía mediante órdenes en UserData, pero órdenes sencillas para ejecutar este script indicándole datos como la región de AWS.
- Soporta todo los tipos de metadatos de Linux y algunos de Windows.

Solo se ejecuta una vez, si actualizamos la plantilla y volvemos a ejecutarla, cfn-init no se volverá a ejecutar.

## cfn-hup

El usuario es responsable de instalarlo, es un demonio que detecta cambios en los metadatos de EC2 y así se ejecutar de nuevo cfn-init; ya que como se dijo, cfn-init solo se ejecuta la primera vez que se lanza el stack.
