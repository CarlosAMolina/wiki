## Contenidos
- [Qué es](#qué-es)
- [Motivación](#motivación)
- [Ejemplo](#ejemplo)


## Qué es

Un factory es cualquier entidad que se encargue de la creación de un objeto.

Normalmente es stateless, no necesita tener atributos, es normal que los métodos enfocados a la creación de objetos sean estáticos, por lo que no es necesario inicializarlo.


## Motivación

Es una manera de realizar el patrón de diseño `factory method` pero respetando el `single responsability principle`. Es decir, cuando se tienen varios `factory methods`, los sacamos de la clase, normalmente se llevan a otra clase, aunque puede ser cualquier coas que cree un objeto, por ejemplo un método que tome una lambda como parámetro que se encargue de crear objetos.

De ser una clase, puede ser una externa o una inner class.


## Ejemplo

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/factories/factory.py>.
