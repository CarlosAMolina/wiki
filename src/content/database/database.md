## Contenidos

- [Mostrar el table definition](#mostrar-el-table-definition)
  - [Mostrar table definition en PostgreSQL](#mostrar-table-definition-en-postgresql)

## CAP theorem

Establece que cualquier base de datos puede ofrecer como máximo 2 de los siguientes puntos:

- Consistency: cada lectura debe devolver el dato más actualizado (la escritura más reciente) o un error.
- Availability: las lecturas no obtendrán ningún error pero no se garantiza devolver el dato más actualizado.
- Partition Tolerant (resilence): el sistema puede estar formado por diferentes particiones de red y debe seguir funcionando aunque haya errores al transmitir la información entre ellas.

## ACID and BASE transactions

Son modelos de transacciones enfocados en distintos puntos del teorema CAP.

### ACID

Enfocado en consistency.

El nombre ACID viene de:

  - A: atomic. Significa que en una transacción, todas las parte se ejecutan correctamente o ninguna de ellas.
  - C: consistent. La base de datos debe cambiar de un estado válido a otro; no está permitido ningún estado intermedio.
  - I: isolated. En caso de ejecutarse simultáneamente, las transacciones no interfieren entre ellas, el resultado debe ser como si se hubieran ejecutado de manera secuencial.
  - D: durable. En el momento que la base de datos informa que transacción haya sido completada, debe estar así aunque haya error en el sistema, fallo de alimentación, etc. Para ello puede almacenarse en memoria no volátil.

Muy utilizado en bases de datos relacionales. Limita la escalabilidad.

### BASE

Está enfocado en availability.

## Mostrar el table definition

### Show table definition in PostgreSQL

El siguiente comando es independiente de `psql`:

```bash
pg_dump -U {USER} {DATABASE} -t {TABLE} -s;
```

Ejemplo:

```bash
pg_dump -U postgres rustwebdev -t accounts --schema-only;
```

Recursos:

<https://stackoverflow.com/questions/2593803/how-to-generate-the-create-table-sql-statement-for-an-existing-table-in-postgr>

