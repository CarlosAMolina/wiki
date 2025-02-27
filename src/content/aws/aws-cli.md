# AWS CLI

## Creación Access Key

Como se explicó en la sección de IAM:

click esquina superior derecha en nuestro usuario > Security credentials > Access keys: Create access key > Seleccionar: Command Line Interface (CLI).

## Instalación AWS CLI

Explicado en [este link](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

## Configuración

La cuenta puede configurarse especificando un nombre de perfil o podemos configurarla de manera general.

En el siguiente ejemplo se configura de manera general:

```bash
aws configure
```

Nos hará unas preguntas. Por ejemplo, para la región por defecto podemos utilizar `eu-west-1` y para el output por defecto simplemente pulsamos enter.

Para probar la configuración ejecutamos por ejemplo el siguiente comando, de no dar error la configuración ha sido correcta:

```bash
aws s3 ls
```

A continuación, se muestra un ejemplo de configuración y uso con un nombre de perfil, hay que utilizar la opción `--profile`:

```bash
aws configure --profile nombre_perfil
aws s3 ls --profile nombre_perfil
```

Explicación del orden de preferencia donde se busca la configuración y credenciales: [link](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-configure-quickstart-precedence).
