## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Volver a un estado que hemos guardado](#volver-a-un-estado-que-hemos-guardado)
  - [Undo/redo](#undoredo)

## Motivación

Para poder navegar a través de los cambios que se han hecho en un sistema, una alternativa es guardar todos los cambios y utilizar el patrón `Command` para deshacerlos y otra es guardar snapshots del sistema, esta última opción es el patrón `Memento`.

Con memento podemos guardar estados y cambiar entre ello y también implementar característica de undo/redo.

## Ejemplo

### Volver a un estado que hemos guardado

Una clase que sea una cuenta bancaria a la que se añade y quita dinero, gracias a memento podemos movernos hacia delante y hacia atrás al estado de la cuenta bancaria en diferentes momentos.

En este ejemplo no tenemos guardado el estado inicial del sistema, en el siguiente ejemplo de `Undo/redo` sí lo tenemos guardado.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/memento/memento.py>.

### Undo/redo

Para poder ser capaces de añadir características de deshacer (undo) y rehacer (redo), debemos guardar todos los estados del sistema.

El guardar todos los estados del sistema puede suponer un problema desde el punto de vista de la memoria consumida.

Al contrario que en el anterior ejemplo, en este sí tenemos guardado el estado inicial.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/memento/undo_redo.py>.
