## Contenidos

- [Motivación](#motivación)
- [Ejemplo](#ejemplo)

## Motivación

Su objetivo es conectar componentes mediante abstracciones.

Evita el `Cartesian product` complexity explosion, es decir, la creación de muchas clases.

Puede pensarse en él como un tipo de encapsulación.

## Ejemplo

Tenemos una clase padre que puede dibujar dos figuras:

- Un círculo.
- Un triángulo.

Además, esta clase puede renderizarlas de dos maneras:

- Renderizado vectorial.
- Renderizado raster (mediante píxeles).

Podríamos implementar 4 clases:

- Renderizado vectorial del círculo.
- Renderizado vectorial del triángulo.
- Renderizado raster del círculo.
- Renderizado raster del triángulo.

Esto es un problema al añadir nuevas figuras y modos de renderizado ya que acabaríamos con una gran cantidad de clases.

Para solucionarlo, podemos trabajar con el concepto de figuras por un lado y el concepto de renderizar por otro. La manera de conectarlos será mediante el bridge; habrá una clase por cada tipo de figura a crear y a estas se les pasará el modo de renderizar.

Crearíamos una clase abstracta `Renderer`, tendrá un método para renderizar cada figura. De ella heredan los tipos de renderizado (vectorial y raster en este ejemplo).

Nota. Esto viola el principio de abierto-cerrado porque habría que añadir nuevo métodos a la clase y subclases por cada nueva figura a renderizar. Pero en este caso es necesario ya que el objetivo de este patrón de diseño es evitar una gran cantidad de clases.

Respecto a las figuras, creamos una clase padre con el método para dibujar la figura y a la que se le pasará al inicializarla el renderer a utilizar. De esta clase heredarán la clase círculo y triángulo.

De este modo, hemos conseguido una api que, para usarla, inicializamos un renderer raster y otro vector y se lo pasamos a la figura con la que queramos trabajar; así reducimos el número de clases.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/bridge/bridge.py>.
