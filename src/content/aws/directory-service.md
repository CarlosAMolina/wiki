# Directory Service

## Qué es un directorio

Un directorio almacena objetos de identidad como usuarios, grupos, servidores, file shares, con una estructura jerárquica (un árbol invertido) que puede llamarse dominio.

Múltiples directorios pueden agruparse en lo llamado como forest.

Los directorio se utilizan mucho en entornos Windows, permite autenticarse a distintos dispositivos con el mismo usuario y contraseña, ofreciendo una gestión centralizada de activos.

Ejemplo de directorios:

- AD DS (Microsoft Active Directory Domain Services).
- SAMBA, una alternativa a AD DS open source, es muy popular y ofrece algunas compatibilidades con AD DS.

## Introducción en AWS

Ofrece directorio gestionado donde almacena usuarios, objetos y otra configuración. Al ser gestionado, ahora trabajo a los administradores, es el equivalente a AWS RDS con las bases de datos.

Funciona en una VPC, es un servicio privado.

Ofrece high availability, está desplegado en varias AZs.

Ejemplo de servicios que pueden utilizarlo: EC2.

Hay otros servicios de AWS que lo necesitan para funcionar, por ejemplo Amazon Workspaces, que es la versión de AWS de Citrix.

Puede funcionar en tres modos:

- Aislado, es decir, no se comunica con soluciones on premise.
- Integrado con sistemas on premise.
- Actuar como un proxy a sistemas on premises de modo que estos utilice servicios de AWS.

## Modos de funcionamiento

### Simple AD

Implementación de Samba 4, compatible con algunas funciones de Microsoft AD.

Es la opción más económica.

Por ejemplo, si unos usuarios quieren conectarse a Amazon Workspaces, necesitamos tener directories en AWS, estos los implementamos con Simple AD.

Permite entre 500 y 5000 usuarios e integración con servicios de AWS como EC2, workspaces, etc.

Utilizamos este modo por defecto de no requerir alguna de las características de los otros modos.

### AWS Managed Microsoft AD

Una implementación de Microsoft AD DS.

Es como simple AD pero permite conectar con directorios on premise mediante VPN o Direct Connect.

La localización principal es en AWS, por lo que si falla la VPN, los servicios de AWS seguirán funcionando.

Empleamos esta opción si las aplicaciones en AWS necesitan MS AD DS.

### AD Connector

Hace de proxy devolviendo peticiones a un directorio on-premise.

Útil por ejemplo si tenemos directorios on premise pero no queremos tenerlos en AWS y hay servicios de AWS, por ejemplo AWS Workspaces, que requieren de un directorio.

La localización principal es on premise. Necesita un AD connector en AWS y conexión privada (ejemplo VPN) de AWS a la localización on premise. El AD connector es el proxy entre los servicios de AWS y la conexión privada a la localización on premise; solo hace de proxy, no tiene otras funcionalidades como por ejemplo autenticación.
