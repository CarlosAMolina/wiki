## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Usernames](#usernames)
  - [Text formatting](#text-formatting)

## Motivación

Está enfocado a optimizar la memoria almacenada.

En lugar de guardar las características de los objetos en ellos mismos, se guardan en un objeto externo y se hace referencia a él, de este modo se ahorra espacio para objetos con las misas características al no tener que ir repitiendo la misma información.

## Ejemplo

### Usernames

En un videojuego, hay muchos usuarios con el mismo nombre, en lugar de almacenarlos todos, porque se irán repitiendo, guardamos una lista de strings únicos y para guardar cada usuario usamos índices que apunten a esa lista de strings.

Ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/flyweight/users.py>.

### Text formatting

Si tenemos una clase que modifica partes de texto convirtiéndolo a mayúsculas, una implementación es guardar una lista de longitud el número de caracteres que tenga el texto y cada valor indicará si debe estar en mayúsculas.

De ser el texto muy largo esto es un problema. Una solución es guardar el rango a modificar en lugar de guardar la información de cada carácter.

Ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/flyweight/text_formatting.py>.
