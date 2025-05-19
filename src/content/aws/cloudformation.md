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

