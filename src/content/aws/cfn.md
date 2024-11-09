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

El stack contiene los recuros de manera lógica. A partir del stack, AWS crea, actualiza o elimina los recursos de manera física.

A partir de la misma plantilla se pueden crear más de un stack.

Al eliminar un stack se borran los recursos lógicos y físicos que creó.

## Recursos

- [Referencias de los recursos de CFN](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
