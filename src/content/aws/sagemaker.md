# Sagemaker

Es un servicio compuesto por productos para desarrollar modelos de machine learning (ML).

Conceptos:

- Sage Maker Studio. Para creación, entrenamiento, debug y monitorización de modelos. Es un IDE para el ciclo de vida de un modelo de ML.
- Sagemaker Domain. Es como un contenedor para desarrollar un proyecto, contiene volúmenes EFS , usuarios, aplicaciones, políticas, VPC, etc.
- Containers. Son contenedores docker desplegados en instancias EC2 de ML, contienen el entorno necesario (sistema operativo, librerías y herramientas).
- Hosting. Sagemaker permite desplegar modelos y ser un endpoint que usar por el resto de aplicaciones.

Sagemaker no tiene un coste propio, sino que se cobra por los servicios creados por Sagemaker. [Enlace](https://aws.amazon.com/es/sagemaker/pricing/) sobre el coste.
