# GuardDuty

Es un servicio de seguridad.

Tras activarlo, funciona continuamente para detectar actividad no autorizada o extraña en la cuenta o en los activos.

El servicio recibe logs de los servicios compatibles (DNS, VPC, CloudTrail) y utiliza inteligencia artificial para analizarlo. No todos los servicios son compatibles con GuardDuty. Aprende de manera autónoma aunque podemos aplicar ciertas configuraciones.

Al haber una detección, puede notificar por SNS o ejecutar un proceso, por ejemplo una lambda que actualice las ACL.

Permite múltiples cuentas, la cuenta donde se active será la cuenta master y podrá invitar a otras cuentas; la configuración/gestión se realiza en la cuenta master.
