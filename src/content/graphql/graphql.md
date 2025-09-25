# GraphQL

## Types

Listas (obtenido del libro `Learning GraphQL`, capítulo 4, apartado `Connections and Lists`):

Declaración | Definición
------------|---------------------------------------------------
[Int]       | Lista de enteros que pueden ser nulos.
[Int!]      | Lista de enteros que no pueden ser nulos.
[Int]!      | Lista no nula de enteros que pueden ser nulos.
[Int!]!     | Lista de enteros que no puede devolver nulos. Sí puede devolver una lista vacía; ya que es un array sin valores, lo cual es diferente a contener elementos nulos.

El signo de exclamación `!` indica que lo que hay a la izquierda no puede ser nulo.

## Recursos

- [Eve Porcello. GraphQL email course](https://graphqlworkshop.com/)
- [Librerías para distintos lenguajes](https://graphql.org/code/).
- Ejemplos de API:
    - [Star Wars GraphQL API online](https://github.com/graphql/swapi-graphql/).
    - [Fake ski resort GraphQL API online](https://snowtooth.moonhighway.com/).
