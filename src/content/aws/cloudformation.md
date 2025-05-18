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

Los parámetros de la plantilla y los pseudo parámetros pueden utilizarse juntos en la plantilla, se complementan entre sí.

Ayudan a conseguir plantillas portables.

#### Parámetros de la plantilla

Se especifica en la plantilla dentro de la sección `Parameters`.

Permiten recibir inputs de fuentes externas como AWS console, CLI o API, cuando el stack es creado o actualizado. Por ejemplo, en la AWS console, tendremos una pantalla donde introducir los valores o elegirlos de una lista.

Pueden configurarse con valores:

- Por defecto.
- Permitidos. Ejemplo, lista de plantillas permitidas.
- Mínimo y máximo valor.
- No echo. Para no mostrar el valor introducido, por ejemplo contraseñas.
- Etc.

#### Pseudo parámetros

[Documentación pseudo parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html).

En lugar de indicar su valor un usuario o un proceso al crearse el stack, los da AWS basados en el entorno. Ejemplo:
  - `AWS::Region`. Toma el valor de la región en la que se aplique la plantilla.
  - `AWS::AccountId`. ID de la cuenta utilizada para crear el stack.
  - `AWS StackName` y `AWS StackId`. Valores del stack que se está creando.

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

