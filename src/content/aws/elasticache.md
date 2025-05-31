# ElastiCache

Es una base de datos en memoria para aplicaciones que requieren un alto rendimiento.

Para utilizarlo, la aplicación debe ser modificada para trabajar con este servicio. Por ejemplo, para ser capaz de pedir la información a la base de datos original en caso de no estar cacheada.

Ofrece dos engines:

- Memcached:
  - Solo trabaja con datos simples como strings.
  - No replica datos. La alternativa es que se haga manualmente.
  - No tiene backups o la opción de restaurar a un estado anterior.
  - Es multi-thread, lo que da más performance con multi core cpus.
- Redis:
  - Puede trabajar con datos más complejos como listas.
  - Puede replicar datos en entre instancias de Redis y distintas AZ.
  - Ofrece backups y restore.
  - Ofrece transacciones, es decir, realizar múltiples operaciones como una sola; en caso de fallar una, no se ejecuta ninguna, esto es necesario en aplicaciones que requieren alta consistencia.

Utilizado:

- Cuando hay altas operaciones de lectura y se requiere baja latencia.
- Para reducir carga de base de datos que puede ser económicamente costosa. Al solicitar a elasticache en lugar de db, también se consigue alto escalado.
- Guardar información de sesión de los usuarios y construir aplicaciones stateless.
