## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Command](#command)
  - [Composite Command](#composite-command)

## Motivación

Tener un objeto que represente una operación de modo que tiene toda la información para realizarla. Es decir, encapsular los detalles de una operación en un objeto separado.

Esto permite acciones como:

- Deshacer una asignación.
- Guardar logs de lo llevado a cabo.

Ejemplo de usos en GUI, al hacer o deshacer acciones, copiar al pulsar control+c, etc.

Tras encapsular los detalles de la operación en un objeto, definimos la instrucción para aplicar el comando; esta se puede definir en el Command o en otro lugar. También se pueden establecer instrucciones para deshacer lo realizado por el Command.

## Ejemplo

### Command

Tenemos una clase `BankAccount` que permite sacar o guardar dinero pero no guarda las operaciones realizadas. Crearemos una interfaz para realizar las operaciones de sacar o guardar dinero que permita guardar logs e incluso deshacer las operaciones.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/command/command.py>.

### Composite Command

A veces también llamado `macro` por ser una secuencia de Commands a ejecutar unos tras otros.

Para hacer transferencias entre dos cuentas bancarias, en lugar de crear solamente un Command para la primera cuenta bancaria y otro Command para la segunda, se debe utilizar una clase composite para evitar errores.

En el ejemplo se muestra cómo controlar que se tienen las cantidades adecuadas para realizar las operaciones.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/command/composite_command.py>.

