# SES

SES = Simple Email Service.

Servicio de AWS para mandar emails.

Tiene un sandbox para evitar spam; en él, es necesario confirmar las direcciones que pueden utilizarse:

- Confirmar sender.
- Confirmar receiver.

El sandbox puede desactivarse.

Para utilizar SES, por ejemplo creamos una lambda que con boto3 usa el servicio.
