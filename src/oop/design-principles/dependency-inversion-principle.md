# Dependency Inversion Principle

## Contenidos

- [Abreviatura](#abreviatura)
- [Definición](#definición)
- [Utilidad](#utilidad)
- [Ejemplo](#ejemplo)

## Abreviatura

- DIP: Dependency Inversion Principle

## Definición

Clases o módulos de alto nivel no deben depender de los de bajo nivel. Deben depender de abstracciones, es decir, de interfaces en lugar de implementaciones.

## Utilidad

De esta manera se pueden cambiar clases sin afectar a otras.

## Ejemplo

Tenemos una clase cuya responsabilidad es guardar relaciones entre padres e hijos, inicialmente utiliza como sistema de almacenamiento una lista de tuplas. Esta es una clase de bajo nivel porque gestiona las relaciones y cómo se almacena la información.

También, hay una clase para hacer búsquedas específicas en estas relaciones, se trata de una clase de alto nivel porque gestiona los requisitos del programa. Ejemplo, buscar todos los hijos de un padre con cierto nombre.

Si la clase que hace la búsqueda utiliza directamente el mecanismo para guardar la información, en este caso la lista de tuplas, habrá aspectos de bajo nivel en una clase de alto nivel.

Esto es un problema porque, al cambiar la manera de almacenar la información (por ejemplo, en lugar de la lista utilizar un diccionario o una base de datos, o al hacer tests sustituir la base de datos por una lista para hacer tests en local, etc.), habrá que cambiar también la clase que realiza las búsquedas, es decir, cambios en la clase de bajo nivel implica cambios en las de alto nivel.

Solución:

- No depender de las implementaciones internas. La clase que guarda las relaciones debe proporcionar un método para buscar en la información abstrayéndonos de cómo se almacena.

  Para ello, lo primero es definir una interfaz para la clase que guarde las relaciones con un método para buscar relaciones, de modo que la clase que hace las búsquedas llame a ese método en lugar de usar directamente la implementación del almacenamiento.

Código de ejemplo: <https://github.com/CarlosAMolina/design-principles>
