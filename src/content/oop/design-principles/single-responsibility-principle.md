# Single Responsibility Principle

## Contenidos

- [Abreviatura](#abreviatura)
- [Definición](#definición)
- [Utilidad](#utilidad)
- [Ejemplo](#ejemplo)

## Abreviatura

- SRP: Single Responsibility Principle
- SOC: Separation Of Concerns

## Definición

Una clase debe tener una sola responsabilidad y no tomar otras.

Esto no quiere decir que una clase deba hacer solo una cosa (ver ejemplo).

Una clase debe tener solo una razón para cambiar, esa razón tiene que estar relacionada con la responsabilidad principal de la clase.

## Utilidad

Evita el antipatrón `God object` que puede llevar a tener clases muy grandes.

## Ejemplo

Una clase que guarda los elementos de la lista de la compra puede añadir elementos y eliminarlos, hace varias acciones pero tiene una única responsabilidad, la de gestionar los elementos.

De querer exportar la información de la lista de la compra para guardarla en un archivo, esto debe realizarlo otra clase, no la lista que gestiona los elementos porque tendría varias responsabilidades.

Código de ejemplo: <https://github.com/CarlosAMolina/design-principles>
