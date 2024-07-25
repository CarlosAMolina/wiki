## Contenidos

- [Motivación](#motivación)
- [Tipos](#tipos)
  - [Singleton Allocator](#singleton-allocator)
  - [Singleton Decorator](#singleton-decorator)
  - [Singleton Metaclass](#singleton-metaclass)
  - [Monostate](#monostate)
- [Problemas](#problemas)
- [Ejemplo](#ejemplo)
  - [Ejemplo Singleton Allocator](#ejemplo-singleton-allocator)
  - [Ejemplo Singleton Decorator](#ejemplo-singleton-decorator)
  - [Ejemplo Singleton Metaclass](#ejemplo-singleton-metaclass)
  - [Ejemplo Monostate](#ejemplo-monostate)
  - [Ejemplo Problemas Testing](#ejemplo-problemas-testing)

## Motivación

Se utiliza para inicializar un objeto una única vez.

Ejemplo, cuando la inicialización consume muchos recursos o requiere tiempo, con Singleton evitamos que la inicialización ocurra más de una vez y siempre se usará la misma instancia. Por ejemplo, cargar una base de datos en memoria.

## Tipos

### Singleton Allocator

Controlamos si un objeto ya se ha inicializado:

- No se ha inicializado, inicializamos el objeto.
- Sí se ha inicializado, devolvemos ese objeto inicializado.

Se utiliza el magic method `__new__`, pero si la clase tiene el método `__init__`, hay que tener cuidado porque, aunque se devuelva la misma instancia de la clase, el método `__init__` se ejecutará cada vez que se inicialice el objeto.

### Singleton Decorator

Soluciona el problema de Singleton Allocator por el que se inicia el `__init__`.

### Singleton Metaclass

Igual que Singleton Decorator pero utiliza una clase intermedia en lugar de un decorator.

Posiblemente es la mejor opción para crear un Singleton.

### Monostate

Es diferente a las anteriores implementaciones, consiste en guardar en una variable los atributos de la clase y todas las instancias usarán dicha variable.

No es muy recomendable, mejor usar Singleton Decorator o Singleton Metaclass.

## Problemas

Si tenemos un Singleton para conectar a base de datos, a la hora de realizar tests, supone un problema porque el Singleton obligará a utilizar una base de datos y la información en ella puede cambiar; sería necesario crear una clase que sustituya al Singleton para trabajar con datos estáticos.

## Ejemplo

### Ejemplo Singleton Allocator

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/singleton/singleton_allocator.py>

### Ejemplo Singleton Decorator

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/singleton/singleton_decorator.py>

### Ejemplo Singleton Metaclass

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/singleton/singleton_metaclass.py>

### Ejemplo Monostate

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/singleton/monostate.py>

### Ejemplo Problemas Testing

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/singleton/singleton_testing.py>

