# Liskov Substitution Principle

## Contenidos

- [Abreviatura](#abreviatura)
- [Definición](#definición)
- [Utilidad](#utilidad)
- [Ejemplo](#ejemplo)

## Abreviatura

- LSP: Liskov Substitution Principle

## Definición

Si un programa utiliza una interfaz de una clase, esta interfaz puede ser sustituida por cualquier clase que derive de ella sin que haya resultados incorrectos.

## Utilidad

Cumpliendo este principio, evitamos programas con mecanismos complejos para tener en cuenta diferentes características de las clases que derivan de la misma interfaz.

## Ejemplo

En este ejemplo veremos resultados incorrectos por una función que es válidas para la clase padre pero no para una hija.

Tenemos una clase `Rectángulo` y otra `Cuadrado` que hereda de la primera. Estas clases tienen las propiedades `alto` y `ancho` pero, en el caso del cuadrado estos valores se igualan al modificar el otro.

También, las clases tienen un método para calcular el área multiplicando las propiedades `alto` y `ancho`.

Si un programa tiene una función que, guarda en una variable el valor de `alto` y luego calcula el área sin usar el método para ello, sino multiplicando la variable por un nuevo valor que se le vaya a dar a `ancho`, en el caso del cuadrado puede que no tenga en cuenta que `alto` y `ancho` deben valer igual y el resultado del área es incorrecto.

Por tanto, `Cuadrado` no debe derivar de `Rectángulo` porque `alto` y `ancho` en `Rectángulo` son independientes pero no ocurre esto para `Cuadrado`, lo que genera errores cuando el programa crea estar trabajando con un rectángulo, no sea así y no tenga esta característica en cuenta.

Solución:

- No usar la clase derivada `Cuadrado`, trabajar solo con `Rectángulo`. Si `Cuadrado` deriva de `Rectángulo` habrá que ir añadiendo comprobaciones que compliquen el programa.
- Usar factory method que devuelva un rectángulo o un cuadrado.

Código de ejemplo: <https://github.com/CarlosAMolina/design-principles>
