# Snow

Snowball, snowball edge y snowmobile son partes de la misma familia de productos, enfocados a la transferencia de grandes cantidades de datos a o desde AWS utilizando almacenamiento físico.

## Snowball

Lo almacenado está cifrado con KMS.

Capacidad de 50TB a 80TB. Aconsejable utilizarlo si necesitamos transferir entre 10 TB y 10 PB de datos; podemos solicitar varios dispositivos snowball.

Te conectas a él con un cable de red físico.

Solo ofrece almacenamiento, no computación.

## Snowball edge

Proporciona almacenamiento y computación, es decir, procesar los datos que se transfieren.

Da más capacidad de almacenamiento y velocidad de transferencia que snowball.

Hay 3 tipos:

- Storage optimized, puedes elegir la opción EC2 que añade 1 TB SSD para ejecutar EC2.
- Compute optimized. El almacenamiento es muy rápido, aconsejable de necesitar altos requisitos de cómputo.
- Compute con GPU, es como compute optimized pero añade GPU, beneficioso por ejemplo para actividades en paralelo.

## Snowmobile

Es un data center en un camión. Conectas este data center con tu equipo para transferir la información.

Utilizado de necesitar transferir más de 10 PB. Máximo permite 100 PB de datos.

Al tratarse de un camión, si tenemos los datos en diferentes localizaciones, no es aconsejable este método (a menos que sea una gran cantidad de lugares), conviene utilizar snowball edge en su lugar.
