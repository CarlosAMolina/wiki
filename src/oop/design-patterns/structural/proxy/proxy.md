## Contenidos

- [Motivación](#motivación)
- [Proxy VS Decorator](#proxy-vs-decorator)
- [Ejemplo](#ejemplo)
  - [Protection proxy](#protection-proxy)
  - [Virtual proxy](#virtual-proxy)

## Motivación

Controla el acceso a un objeto.

Crea una interfaz que permita:

- Sin modificar un objeto, se le añaden nuevas funcionalidades o cambian las existentes.
- Acceder a un recurso aunque este tenga características complejas de uso, por ejemplo, que sea costoso de inicializar.

El proxy posee la mima interfaz que el objeto con el que trabaja.

Hay diferentes tipos de proxies:

- Communication proxy: se cambia el comportamiento de un objeto.
- Loggin proxy.
- Caching proxy.
- Etc.

## Proxy VS Decorator

- El proxy proporciona una interfaz idéntica a la de la clase a modificar. El decorador ofrece una interfaz mejorada, da más funcionalidades.
- El decorador agrega o hace referencia a la clase con la que trabaja; el proxy no lo necesita.
- El proxy puede trabajar con un objeto que no existe, por ejemplo, en lazy loading trabaja con un objeto que no está inicializado hasta que es utilizado.

## Ejemplo

### Protection proxy

Controla el acceso a un recurso.

Para asegurar que una clase solo pueda ser utilizada bajo ciertas condiciones, en lugar de modificar esa clase, podemos crear un proxy que se encargue de llamar a la clase pero realizando previamente las comprobaciones.

Ejemplo: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/proxy/protection_proxy.py>.

### Virtual proxy

Virtual quiere decir que parece que es igual a la clase a la que le añade funcionalidad. Pero se comporta de manera distinta gracias a la nueva funcionalidad añadida.

Ejemplo, lazy loading en el que no inicializamos una imagen hasta que hagamos uso de ella, por ejemplo, hasta que no la dibujemos: <https://github.com/CarlosAMolina/design-patterns/blob/main/structural-patterns/proxy/virtual_proxy.py>.

