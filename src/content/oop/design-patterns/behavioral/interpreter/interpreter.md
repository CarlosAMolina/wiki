## Motivación

Se trata de un componente que procesa texto. Lo realiza mediante dos procesos:

- Lexing: convertir el texto en tokens. Ejemplo: `3*(4+5) -> Lit[3] Star Lparen Lit[4] Plus Lit[5] Rparen`
- Parsing: convertir los tokens en object oriented constructs que recorreremos y realizaremos una acción sobre ellos como mostrar su valor o evaluarlos. El ejemplo anterior, la conversión a constructs quedaría `MultiplicationExpression[Integer[3], AdditionExpression[Integer[4], Integer[5]]]`.

## Ejemplo

En el siguiente ejemplo se puede ver cómo crear un Interpreter de operaciones matemáticas escritas en un string.

Ejemplo <https://github.com/CarlosAMolina/design-patterns/blob/main/behavioral-patterns/interpreter/handmade.py>.

