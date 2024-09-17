## Contenidos
- [Qué es](#qué-es)
- [Ejemplo](#ejemplo)

## Qué es

Es un método que genera un objeto.

## Ejemplo

Si guardamos en una clase `Point` las coordenadas de un punto, estas pueden ser cartesianas o polares.

En lugar de tener un método `__init__` que reciba parámetros indicando el tipo de coordenadas, creamos 2 métodos que inicialicen el objeto `Point` según el tipo de coordenadas.

De esta manera se consiguen ventajas como:

- El nombre de los métodos facilita saber qué tipo de coordenadas se utilizan.
- El nombre de los parámetros también quedará más claro ya que unos pueden ser `x` e `y` y otros `rho` y `theta` al estar en distintos métodos.

Código de ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/creational-patterns/factories/factory.py>.