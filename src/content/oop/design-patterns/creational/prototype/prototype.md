## Contenidos

- [Motivación](#motivación)
- [Tipos](#tipos)
  - [Prototype](#prototype)
  - [Prototype Factory](#prototype-factory)
- [Ejemplo](#ejemplo)
  - [Ejemplo Prototype](#ejemplo-prototype)
  - [Ejemplo Prototype Factory](#ejemplo-prototype-factory)

## Motivación

Se utiliza para crear un objeto copiando otro en lugar de crearlo desde cero.

## Tipos

### Prototype

El prototype es un objecto con valores que otros objetos tendrán en común.

Si dos objetos van a tener los mismos datos, debemos evitar hacer un reference assigment, porque al modificar un objeto también se modificaría el otro. El objeto final no debe referenciar al inicial.

En Python, la solución es utilizar `copy.deepcopy` de modo que se copien recursivamente los atributos de un objeto en otro y cuando se modifiquen en un objeto el otro no se ve alterado.

### Prototype Factory

En lugar de crear el prototype con datos comunes y hacer el `copy.deepcopy` en varias partes del proyecto, podemos mover esto a un único componente; un factory que se encargue de crear los objetos finales. Por tanto, este factory facilita el uso del prototype.

## Ejemplo

### Ejemplo Prototype

Supongamos que tenemos una clase `address` para guardar datos de dirección (calle, ciudad, etc) y otra clase `Person` que tiene un atributo donde se guarda la dirección. Si hay dos personas con la misma dirección, no es correcto inicializar la dirección en una variable y pasárselo a cada clase persona, porque al modificar la dirección de una persona, también se modificará la dirección de la otra. Con `copy.deepcopy` esto queda solucionado.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/prototype/prototype.py>

### Ejemplo Prototype Factory

Tenemos dos oficinas, una principal y otra auxiliar. Los empleados de cada oficina tendrán la misma dirección, cambiando solo el número de despacho (suite).

Con el Prototype Factory la creación de estos trabajadores se facilita.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/prototype/prototype_factory.py>

