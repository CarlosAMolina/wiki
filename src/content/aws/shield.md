# AWS Shield

Protege contra ataques DDOS.

Hay dos tipos: standard y advanced.

Standard:
- Es gratuito y está activado por defecto.
- Actúa en el límite de la región o edge.
- Gestiona ataques típicos en las layer 3 y 4 del modelo OSI.

Advaced:
- Cuesta 3.000$ al mes por organización.
- Protege CF, R53, Global accelerator, cualquier servicio que use EIPs (ejemplo EC2), ALBs, CLBs, NLBs.
- Puedes configurar grupos de servicios que proteger, lo que disminuye la gestión requerida de protegerlos individualmente.
- No protege los servicios de manera automática, hay que especificarlo.
- Ofrece protección contra costes producidos por ataques no bloqueados, por ejemplo si una instancia EC2 escala por acciones maliciosas.
- Tiene un enfoque proactivo, el AWS Shield Response Team (SRT) tomará acciones ante ataques o extrañas detecciones.
- Se integra con WAF.
