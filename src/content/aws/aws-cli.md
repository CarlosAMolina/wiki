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

## Autenticación con AWS Identity Center

No tiene coste adicional.

Es un buen método porque en nuestro ordenador las credenciales serán temporales.

Configuraremos el IAM Identity Center para poder utilizar por ejemplo AWS CLI y AWS CDK.

1. Nos logeamos en la cuenta de AWS con nuestro usuario IAM, no tiene que ser la cuenta root, yo utilizo la cuenta en la que despliego los servicios, pese a que AWS advierte `We recommend as a security best practice that you do not store resources in this account.`. Es importante que la cuenta tenga `AdministratorAccess` para habilitar Identity Center y que AWS CDK pueda desplegar todos los servicios. Para verificar si tenemos `AdministratorAccess`:
  - Acceder a IAM > Users > seleccionar el deseado.
  - En la pestaña `Permissions` debe aparece `AdministratorAccess`.
2. Accedemos a `IAM Identity Center`.
3. Debemos estar en la región deseada, solo puede configurarse una.
4. Clic en `Enable`.
5. AWS nos indicará si crear una instancia de organización o de cuenta. Elegimos organización para a futuro poder acceder más tipos de instancias; solamente hay que hacer clic en `Enable`.
6. IAM Identity Center setup > Confirm your identity source. Clic en `Confirm identity source`. Vemos `Identity source = Identity Center directory` que significa que AWS será el responsable de almacenar los usuarios.
7. En el menú de la izquierda en `Users` crearemos el usuario para ser utilizado por AWS CLI y CDK, es independiente de los usuarios IAM por lo que pueden utilizarse el mismo nombre de usuario en IAM y en Identity Center al ser diferentes servicios. Clic en `Add user`:
  - Username: cmolina
  - Password: send an email. Escribimos nuestro email.
  - Firstname: Carlos
  - Last Name: Molina
8. Recibiremos un email con instrucciones para configurar la contraseña, esta es la que utilizaremos para logearnos desde la terminal, por lo que hay que guardarla. Podremos añadir un MFA.
9. Accederemos a la pantalla de crear grupos (por ejemplo Developers, Administratos, etc.). No crearemos ninguno, hacemos clic en siguiente.
10. Llega el momento de asignar permisos a la cuenta de Identity Center creada, estos indican lo que puede realizar en la cuenta AWS original. Desde el buscador accedemos a `IAM Identity Center` y, en el menú izquierdo, elegimos Multi-account permissions > AWS accounts, seleccionamos la cuenta de AWS y hacemos clic en `Assign users or groups`. Clic en `Create permission set`, el Permission Set es una plantilla que creará un IAM Role basado en estos permisos. Seleccionamos `Permission set type` y en `Predefined permission set` a `AdministratorAccess` para que AWS CDK pueda crear roles. En la siguiente pantalla utilizamos `Pemission set name` = `AdministratorAccess`, el resto de opciones lo dejamos por defecto.
11. Asignamos el Permission Set y el usuario de Identity Center al usuario de la cuenta AWS. Vamos a `IAM Identity Center` > `AWS accounts`, seleccionamos la cuenta de AWS, clic en `Assign users or groups`, seleccionamos el usuario creado, clic en siguiente y seleccionamos el Permission Set, clic en siguiente y luego en submit. Con esto se creará el rol `AWSReservedSSO_AdministratorAccess_...`que obtendremos al hacer login desde la terminal.

Para acceder a la consola de AWS desde esta cuenta Identity Center:

- Login en la URL awsapps.com/start.
- Seleccionar la cuenta y hacer clic en AdministratorAccess.

Añadimos esta información en el archivo ~/.aws/config:

```
[profile cmolina]  # The profile created in IAM Identity Center
region=eu-south-2
output = json
```

Vamos a configurar el anterior archivo:

```bash
aws configure sso --profile cmolina
# Si aparece `WARNING: Configuring using legacy format (e.g. without an SSO session)....`, vuelve a ejecutar el comando anterior.
# SSO session name (Recommended): cmolina  # This will appear en the ~/.aws/config as [sso-session cmolina], that describes how to authenticate.
# SSO start URL [None]: https://.....awsapps.com/start  # El valor puede verse en IAM Identity Center > Settings > AWS access portal URL.
# SSO region [None]:  # Valor de la región que activamos el IAM Identity Center.
# SSO registration scopes [sso:account:access]:  # Presionamos enter, el valor por defecto `sso:account:access` permite al AWS CLI detectar las cuentas y permission sets asignados al usuario del Identity Center.
```

El navegador web mostrará pantalla de login.

Uso de la CLI:

```
aws sso login --profile cmolina
aws sts get-caller-identity --profile cmolina
aws --profile cmolina s3 ls
aws sso logout
# We can ommit `--profile cmolina` with `export AWS_PROFILE=cmolina`.
```
