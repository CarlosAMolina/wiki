# AWS comprehend

## Introducción

Es un servicio NLP (natural language processing) de AWS.

Como input recibe un documento, pensemos en ello como texto. La salida ofrece información que ha detectado en el input, puede ser:

- Entidad, indica el tipo de detección como personas, organizaciones, etc.
- Frases.
- Idioma utilizado.
- PPI (información personal). Como nombre, fechas, información bancaria, teléfono, etc.
- Sentimientos. La cantidad de texto con sentimiento neutral, positivo, negativo o mezcla de ellos.

Estos resultados tienen asociado un valor de confidence, que muestra la seguridad con que el resultado es correcto. Su valor va de 0 a 1 (máximo).

Utiliza machine learning para crear los resultados. Pueden ser modelos pre entrenados o propios.

Para cargas de trabajo pequeñas da resultados en tiempo real. De ser la carga mayor, emplea trabajos asíncronos.

Interactuamos con él mediante consola, CLI o APIS.
