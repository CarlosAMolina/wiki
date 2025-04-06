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

## Networking

Tiene dos modos: público y VPC.

### Público

Por defecto se sitúan en al red pública de AWS.

Ofrece el mejor rendimiento al no necesitar configuraciones específicas de VPC.

No puede acceder a recursos en VPC a menos que el VPC tenga una IP pública y sus security controls permitan acceso externo.

### VPC

Al estar en una VPC se le aplican todas las reglas del VPC; es decir, puede acceder a lo que contenga la VPC pero no a recursos externos a menos que la VPC esté configurada para ello.

La lambda también necesita que el rol le otorgue ec2 network permissions para crear network interfaces en la VPC.

Las lambdas no se encuentran dentro de la VPC sino que están en una VPC propia del servicio Lambda y al ejecutarse deben conectarse al VPC.

Para conectarse al VPC, antes AWS lo gestionaba de modo que al ejecutarse crean un ENI para conectarse al VPC, esto consume tiempo. También, de ejecutar la lambda en paralelo o de manera concurrente, cada ejecución necesita una ENI, lo que no es una solución escalable.

Actualmente, este problema se soluciona porque en lugar de utilizar un ENI por ejecución, AWS analiza las lambdas en la región y en la cuenta del usuario para agruparlas en una combinación de subredes y security groups, por cada una de estas combinaciones se usa una ENI, por ejemplo:

- Si las lambdas usan distintas subredes y el mismo security group. Se necesita un ENI por subred.
- Si las lambdas usan la misma subred y el mismo security group. Se necesita un solo ENI.

En la arquitectura actual, la ENI se crea al configurar la lambda o cuando se modifica la configuración de red, por lo que no supone retardo cuando se invoca la lambda. Crear la ENI requiere 90 segundos.
