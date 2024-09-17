## Contents

- [Show table definition](#show-table-definition)
  - [Show table definition in PostgreSQL](#show-table-definition-in-postgresql)

## Show table definition

### Show table definition in PostgreSQL

The following command is independent of `psql`:

```bash
pg_dump -U {USER} {DATABASE} -t {TABLE} -s;
```

Example:

```bash
pg_dump -U postgres rustwebdev -t accounts --schema-only;
```

Resources:

<https://stackoverflow.com/questions/2593803/how-to-generate-the-create-table-sql-statement-for-an-existing-table-in-postgr>

