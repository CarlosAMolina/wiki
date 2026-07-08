## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Intrusive](#intrusive)
  - [Reflective](#reflective)
  - [Classic](#classic)
  - [Classic refined](#classic-refined)

## Motivación

Permite añadir una nueva funcionalidad no solo a una clase sino a una jerarquía de clases completa.

## Ejemplo

### Intrusive

Modificamos las clases a las que queramos añadir funcionalidad. En el siguiente ejemplo, hacemos que muestren contenido con `print`.

Esta manera de implementar el patrón no cumple con el principio Open Closed.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/visitor/intrusive.py>.

### Reflective

En lugar de modificar las clases a las que añadir funcionalidad, creamos una clase para dicha funcionalidad.

En el siguiente ejemplo continuamos con el caso de mostrar una suma. Tampoco cumple el principio Open Closed porque la clase que hace el print deberá modificarse en caso de que ampliemos el programa con una expresión resta.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/visitor/reflective.py>.

### Classic

Es la implementación explicada en el libro Gang of Four Design Patterns y utilizada en la mayoría de lenguajes de programación.

Creamos una clase externa que realiza el print.

Utiliza lo conocido como double-dispatch, en el siguiente ejemplo, esto se ven al hacer `ae.left.accept` porque se acaba llamando a la clase `DoubleExpression` o a la `AdditionExpression` y en ellas llamamos a visitor pasándole `self`, es decir, pasándole la clase afectada.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/visitor/classic.py>.

### Classic refined

Modificamos el ejemplo de `classic.py` dejando de usar unos métodos que no son necesarios y añadiendo una clase que realiza la operación de sumar.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/visitor/classic_refined.py>.

