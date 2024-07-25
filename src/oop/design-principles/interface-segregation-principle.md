# Interface Segregation Principle

## Contenidos

- [Abreviatura](#abreviatura)
- [Definición](#definición)
- [Utilidad](#utilidad)
- [Ejemplo](#ejemplo)

## Abreviatura

- ISP: Interface Segregation Principle

## Definición

Una interfaz no debe tener más métodos de los que necesite quien la utilice.


## Utilidad

Todos los métodos de una clase derivada de una interfaz pueden utilizarse, evitando confusiones al trabajar con dicha clase.

También, evita modificar partes que dependan de otras que no necesitan cuando estas últimas sufren cambios.


## Ejemplo

Una interfaz encargada de describir un equipo multifunción realiza acciones como imprimir, enviar un fax y escanear un documento.

Puede que haya clases que deriven de esta interfaz que solo necesiten imprimir y escanear, pero no enviar un fax.

Al ser clases que no implementan todos los métodos de la interfaz, de llamarlos causarán error o no se produce el comportamiento esperado.

Solución:

- Crear una clase por cada acción (clase impresora, clase fax y clase escáner) y las clases derivadas heredan solo de las que necesiten.

Código de ejemplo: <https://github.com/CarlosAMolina/design-principles>
