# DynamoDB

## Introducción

DDB = DynamoDB.

Es un NoSQL Database as a Service (no tienes que gestionar servidores o infraestructura).

Gestiona información como key/value o documents.

Es un servicio público.

Puedes configurar el rendimiento o que se ajuste bajo demanda. Al hablar de capacidad, nos referimos al rendimiento o velocidad, no al espacio.

Tiene resilencia sobre las AZs y de manear opcional global.

Es muy rápida, basada en SSD.

Ofrece backups y cifrado.

Muy utilizada para serverless, permite integración event-driven, es decir tomar acciones cuando los datos en db cambian.

Se accede a sus datos mediante consola, CLI o API. No soporta SQL, aunque sí PartiQL (un lenguaje parecido a SQL).

Se cobra por RCU (lectura), WCU (escritura), storage y características.

## Tabla

Formada por items, cada item puede tener máximo 400KB.

Todos los items comparten la misma primary key; podemos pensar en un item como en una fila en una db relacional.

Puede tener infinitos items.

Un item está formado por:

- Clave primaria. Puede ser:
  - Simple: es la partition key.
  - Compuesta: combinación partition key más sort key; se llama SKey. En este caso lo que debe ser único en la tabla es la combinación de la partition y la sort keys.
- Atributos. Pueden tener cualquier valor y tener diferente esquema en cada item.

## Backups

Hay 2 tipos: on-demand y pitr.

### On-Demand

Se realiza una copia manual.

Permite seleccionar el cifrado.

Guarda los datos de la tabla y su configuración.

Debemos eliminarla nosotros.

Útil para llevar una tabla a otras regiones.

### PITR

PITR = point in time recovery.

Por defecto no está activado. Tras hacerlo guarda datos durante 35 días; cada segundo realiza un backup.
