## Contents

- [Mostrar el table definition](#mostrar-el-table-definition)
  - [Mostrar table definition en PostgreSQL](#mostrar-table-definition-en-postgresql)

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

