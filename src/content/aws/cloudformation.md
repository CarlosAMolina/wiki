# CloudFormation

## Introducción

CFN = CloudFormation

Es un servicio para gestionar la infraestructura utilizando plantillas.

## Plantillas

Utilizan formato YAML o JSON.

En la plantilla tenemos distintos apartados: versión, descripción, recursos, etc.

Hay algunas peculiaridades como que indicar el número de versión es opcional, pero en caso de hacerlo, entonces es obligatorio que el siguiente apartado sea el de descripción.

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

