## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Tree transversal](#tree-transversal)
  - [Array-backed property](#array-backed-property)

## Motivación

Iterar es acceder a los elementos de un objeto (lista, diccionario, etc.). El iterator es una clase externa utilizada para realizar esta iteracción.

Esta clase tiene una referencia al elemento actual y sabe cómo acceder a un elemento diferente.

Para implementarlo es necesario:

- `__iter__()`: para exponer el iterator.
- `__next__()`: para devolver cada uno de los elementos iterados.
- `raise StopIteration`: cuando no hay objetos con los que iterar.

## Ejemplo

### Tree transversal

Los iterators estáticos no pueden ser recursivos ya que, al llegar a cierto estado, se para la iteracción. No guardan el estado, por lo que no se puede continuar la iteracción más tarde a partir de cierto elemento.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/iterator/tree_traversal.py>.

### Array-backed property

En Python se llama `List-Backed Property`.

Consiste en que, si una clase tiene atributos y hacemos operaciones con ellos, es mejor definirlos de modo que, de añadir más atributos, no haya que modificar todos los lugares donde se hagan operaciones, sino que se trabaje con ellos automáticamente.

En el siguiente ejemplo se consigue esto guardando los atributos en una lista y utilizando properties.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/iterator/list_backed_property.py>.
