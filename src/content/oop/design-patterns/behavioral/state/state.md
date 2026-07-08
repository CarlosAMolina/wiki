## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Detectar el estado de un objeto](#detectar-el-estado-de-un-objeto)
  - [Handmade state machine](#handmade-state-machine)
  - [Switch based state machine](#switch-based-state-machine)

## Motivación

Se trata de un patrón en el que el comportamiento de un objeto es determinado por su estado. El estado del objeto va cambiando.

Un constructor que maneja el estado y las transacciones es llamado state machine.

## Ejemplo

### Detectar el estado de un objeto

En este ejemplo vemos si una bombilla está encendida o apagada.

Se trata de un ejemplo clásico porque cada estado es una clase, y estas clases manejan la transición a otro estado.

Puntos negativos de este ejemplo es que puede resultar algo complejo el manejar la transacción de un estado a otro mediante las clases que hemos creado, sería mas sencillo hacerlo directamente con unos métodos `on` y `off`.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/state/classic.py>.

### Handmade state machine

Definimos unos estados, unos triggers que indican a qué estado cambiar y unas reglas para establecer los posibles estados a los que un estado puede ir.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/state/handmade.py>.

### Switch based state machine

Al contrario que el ejemplo `Handmade state machine`, este no necesita estructuras como triggers o reglas para el cambio entre estados, solo se definen los estados.

Tiene el inconveniente de poder quedar una lógica compleja de entender al haber muchas condiciones de cambio para los estados.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/state/switch_based.py>.

