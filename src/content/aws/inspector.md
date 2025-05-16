# AWS Inspector

Es un servicio que escanea:

- Hosts y su sistema operativo. Ejemplo instancias EC2 o contenedores. Lo que busca son vulnerabilidades y desviaciones respecto a buenas prácticas. Necesita para ello un agente.
- Tráfico de red. En este modo no es necesario un agente, pero de tenerlo, la información será más detallada al ofrecer visibilidad del sistema operativo.

El resultado es un reporte de lo encontrado, ordenado por prioridad.

El análisis puede tardar entre 15 minutos y 1 día.

Las reglas vienen en paquetes que determinan qué revisar:

- Si la red tiene acceso público. Puede revisar EC2, ALB, DX, VPC, VGW, ACL, etc.
- Puertos utilizados. De querer saber si el puerto está escuchando, es necesario un agente para conocer qué hace el sistema operativo.
- Las  reglas que revisan el host necesitan un agent.
- CVE (Common Vulnerabilities en Exposures).
- CIS (Center of Internet Security) Benchmakrs. Son unas guías de buenas prácticas para que una empresa cumpla con la seguridad requerida.
- Guía de buenas prácticas proporcionadas por AWS. Ejemplo deshabilitar root login por ssh.
