## Contenidos
- [Motivación](#motivación)
- [Ejemplo](#ejemplo)


## Motivación

Trata de adaptar una interfaz a otra a usar.

## Ejemplo

Hay una librería externa con una clase `Point`, pero nuestra aplicación utiliza la clase `Line` con la que formamos la clase `Rectangle`. A la hora de representar gráficamente un rectángulo, la función que hace la representación, trabaja con la clase `Point` en lugar de `Line`; gracias al adapter transformamos `Line` en una serie de clases `Point`.

Este ejemplo tiene el problema de que volvemos a generar los puntos de una línea aunque esta ya exista, una mejora es añadir caché para evitar esto.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/adapter/adapter.py>.


