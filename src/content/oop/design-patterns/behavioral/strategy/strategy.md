## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Procesador de texto](#procesador-de-texto)

## Motivación

Permitir en run-time que cada componente use el objeto que más le convenga.

Para implementarlo:

- Definimos una clase para uso general (alto nivel).
- Definimos la interfaz que cada clase strategy debe seguir. Las clases strategies tienen el comportamiento a bajo nivel (a nivel de detalle).
- La clase de alto nivel usará los métodos de los strategies.

Puede que haya casos en que los diferentes strategies interactúen entre ellos.

El patrón Strategy consigue su objetivo mediante composición. En cambio, el patrón de diseño Template Method tiene el mismo objetivo pero utiliza herencia.

## Ejemplo

### Procesador de texto 

Tenemos un procesador de texto que se encarga de convertir una lista de strings a un formato Html o Markdown. El procesador de texto es una clase que recibe un strategy (el strategy de Html o para Markdown) e irá guardando los elementos de la lisa aplicando las modificaciones que indique el strategy.

Los strategies son clases que saben cómo establecer el formato Html o Markdown.

La clase para procesar el texto es de alto nivel por ofrecer un comportamiento general para todas las listas de strings, mientras que los strategies son de bajo nivel al contener los detalles del formato Html o Markdown.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/strategy/strategy.py>.

