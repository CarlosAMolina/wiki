# Lambda

## Introducción

Lambda es un servicio FaaS (Function as a Service), lo que significa que está enfocado a procesos de corta duración.

Un lambda function:

- Es un código que lambda ejecuta.
- Utiliza un runtime, por ejemplo Python 3.11. Pueden crearse runtimes propios gracias a las layers.
- Se ejecuta en un runtime environment. A este le definimos los recursos a utilizar:
  - Memoria. De 128 MB a 10.240 MB.
  - CPU. Según la cantidad de memoria, AWS asigna una cantidad de CPU.
  - Almacenamiento. Va de 512 MB a 10.240 MB.
- Tiene un tiempo de funcionamiento máximo de 15 minutos.

La lambda se despliega en AWS como un paquete.

Cuando se invoca:

- Se crea el runtime environment.
- El paquete de la lambda se lanza en el runtime environment.
- Los datos y los environments no se mantienen entre invocaciones. Hay situaciones en que se usa el mismo runtime environment, pero como norma general, se crea uno nuevo.

El cobro es en base a los recursos asignados y el tiempo de ejecución.

La lambda utiliza roles para interactuar con otros servicios.
