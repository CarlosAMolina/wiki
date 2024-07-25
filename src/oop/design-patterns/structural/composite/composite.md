## Contenidos
- [Motivación](#motivación)
- [Ejemplo](#ejemplo)
  - [Figuras geométricas](#figuras-geométricas)
  - [Red neuronal](#red-neuronal)

## Motivación

Los objetos pueden relacionarse mediante herencia o por composición (objetos se guardan como atributos de otros).

Algunos objetos compuestos y no compuestos tienen comportamientos similares. Con el patrón de diseño `Composite` gestionamos estos objetos de manera uniforme.

Su objetivo es tratar objetos individuales y objetos que sean composición de varios objetos de la misma manera.

## Ejemplo

### Figuras geométricas

Podemos trabajar con cada figura (círculo, triángulo, etc) en una clase individual o agruparlas en una clase padre de la que hereden y que irá guardando las figuras; esta clase padre también puede almacenar otras instancias de la clase padre que agrupen otros figuras.

La clase padre se encargará de realizar operaciones sobre todas las clases que guarde.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/composite/geometric_shapes.py>.

### Red neuronal

La clase `Neuron` se conecta con otras instancias de la clase `Neuron` mediante sus atributos `inputs` y `outputs`

Las clases `Neuron` pueden agruparse en una clase `NeuronLayer`.

Para conectar neuronas en una layer, creamos una función que itere sobre todas las neuronas y las relacione. Este es el primer paso, el siguiente es dejar de usar esta función y llevarla como método de una clase `Connectable` de la que hereden `Neuron` y `NeuronLayer` para que no tengan que usar una función externa.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/composite/neural_networks.py>.
