## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Method chain](#method-chain)
  - [Broker chain](#broker-chain)
    - [Command Query Separation](#command-query-separation)
    - [Broker chain example](#broker-chain-example)

## Motivación

Es utilizado cuando hay un grupo de componentes que pueden procesar un evento. Por ejemplo, al pulsar un botón, el encargado de gestionar el evento puede ser el mismo botón, o el elemento caja padre en el que se encuentre, o la ventana en la que se encuentre, etc.

Se puede hacer que un componente pare la cadena de ejecución.

## Ejemplo

### Method chain

Consiste en aplicar el Chain of Responsibility mediante una cadena de referencias.

Tenemos un juego con una clase `Creature` y otra clase `CreatureModifier` que puede hacer cambios en el ataque y defensa de la criatura.

La clase `CreatureModifier` no realiza ninguna modificación, pero almacena y se utilizará para ir llamando a los modificadores que se aplicarán a `Creature`.

También, hay una clase `NoBonusesModifier` que, tras llamarla, impedirá que el resto de modificadores añadidos tengan efecto; es decir, corta la cadena.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/chain-of-responsibility/method_chain.py>.

### Broker chain

Consiste en aplicar el Chain of Responsibility con un constructor centralizado.

Para este ejemplo, hay que introducir el Command Query Separation (CQS)

#### Command Query Separation

Es la división de dos conceptos:

- Command: solicitar una acción. En el ejemplo de `Method chain` corresponde a cambiar el valor del atributo de una clase (aumentar el ataque de la clase `Creature`).
- Query: solicitar información. En el ejemplo de `Method chain` corresponde a preguntar cual es el valor del atributo de una clase (solicitar el valor de ataque de la clase `Creature`).

Con CQS, en lugar de realizar una acción o tomar el valor de algo de manera directa, se puede pedir a terceras partes. Gracias al patrón Chain of Responsibility, es posible tener a otros listeners que estén encargados de ello e incluso pueden modificar el comportamiento.

#### Broker chain example

Siguiendo con el ejemplo de `Method chain`, si quisiéramos cambiar los atributos sin tener que llamar a los modificadores (invocar a la Chain of Responsibility) de manera explícita, podemos utilizar CQS, donde se invocará a la Chain of Responsibility automáticamente gracias a la clase `Event` y a centralizar la construcción en lo conocido como event broker, que en el ejemplo es la clase `Game`.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/chain-of-responsibility/broker_chain.py>.
