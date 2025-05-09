# DataSync

## Introducción

Permite transferencia de grandes cantidades de datos hacia o desde AWS y viceversa.

Ejemplo de uso: migraciones, transferencia de datos a procesar, archivar datos, disaster recovery, etc.

Cada:

- Agente puede gestionar 10 GB por segundo de transferencia de datos.
- Job puede gestionar 50 millones de archivos.

También transfiere metadatos como permisos y timestamps.

Permite validación de datos que garantizan que se han transferido sin errores.

## Características

- Servicio escalable.
- Ofrece transferencias incrementales y programadas.
- Puede configurar la velocidad de transferencia para evitar problemas.
- Ofrece compresión y cifrado.
- Recuperación automática de errores durante la transferencia.
- Integración con servicios de AWS como S3, EFS, FSx.
- Proporciona en algunos servicios transferencia entre regiones.
- El coste es por giga transferido.

## Componentes

### DataSync Agent

Funciona en la parte on premise, en una plataforma de virtualización como VMWare y se comunica con:

- El almacenamiento SAN/NAS on premise mediante NFS o SMB.
- El AWS DataSync Endpoint.

### Task

Un task es un job en DataSnc, configura:

- Qué va a sincronizarse.
- A qué velocidad.
- Origen y destino.

### Location

Cada task tiene dos localizaciones FROM y TO.

Ejemplo: NFS, SMB, AWS EFS, AWS FSx, AWS S3.
