## Contenidos
- [Instalar Django](#instalar-django)
- [Installed version](#installed-version)
- [Comando django-admin](#comando-django-admin)
- [Iniciar servidor](#iniciar-servidor)
- [Shell](#shell)
- [Proyecto y app](#proyecto-y-app)
  - [Proyecto](#proyecto)
  - [App](#app)
    - [Incluir la app en settings.py](#incluir-la-app-en-settings.py)
- [Vistas](#vistas)
  - [URLs](#urls)
    - [URLconf](#urlconf)
    - [Proyecto](#proyecto)
    - [include()](#include)
    - [path()](#path)
- [Crear usuario admin](#crear-usuario-admin)
- [settings.py](#settings.py)
  - [Zona horaria](#zona-horaria)
  - [Apps instaladas en el proyecto](#apps-instaladas-en-el-proyecto)
- [models.py](#models.py)
- [Referencias](#referencias)


## Instalar Django

```bash
sudo apt-get install python3-pip
pip3 install Django
```

Para verificar que la instalación fue correcta, comprobar la versión.

## Installed version

```bash
python3 -m django --version
django-admin --version
```

## Comando django-admin

De no funcionar el comando `django-admin`, añadir su path a las variables de entorno. Ejemplo, añadir al final del archivo `.bashrc`:

```bash
export P3_DJANGO=path/a/django-admin
```

Comprobar que funciona el comando:

```bash
django-admin
```

## Iniciar servidor

```bash
python3 manage.py runserver 8081
```

De no especificar el puerto, por defecto se usa el 8000.

## Shell

```bash
python3 manage.py shell
```

## Proyecto y app

### Proyecto

Un proyecto es un conjunto de apps y su configuración.

Para crear un nuevo proyecto, ejecutar la siguiente orden, creará todo lo necesario para un proyecto:

```bash
django-admin3 startproject NOMBRE-PROYECTO
```

El nombre del proyecto no puede tener los valores `django` o `test`, suele ser todo minúsculas.

El nombre de la carpeta que contiene todo el proyecto se puede cambiar.

### App

Una app es una aplicación web que hace algo: blog, encuesta, base de datos pública, etc.

Crear una nueva aplicación, el siguiente comando crea todo lo necesario:

```bash
python manage.py startapp NOMBRE-APP
```

El nombre de la app suele ser todo minúsculas.

#### Incluir la app en settings.py

Tras crear la app, debe modificarse el archivo `settings.py`, a la variable `INSTALLED_APPS` hay que añadir los valores de la app.

El formato a usar es `nameApp.apps.nameAppConfig`. Por ejemplo, para la app `commons` sería `commons.apps.CommonsConfig`.

Estos valores pueden verse en `NOMBRE-PROYECTO/djangoProject/NOMBRE-APP/apps.py`. Hay que usar el nombre de la clase y el valor de la variable `name`.

## Vistas

Van dentro de la carpeta de la app. Ejemplo, si la app se llama `polls`, el path es `polls/views.py`.

Son llamadas con las URLs.

### URLs

Ejemplo URLs de un proyecto:

<http://127.0.0.1:8081/NOMBRE-APP/>

<http://127.0.0.1:8081/admin/>

#### URLconf

Para llamar a una vista, con los llamados `URLconf`, mapeamos una URL y una vista.

En la variable `urlpatterns` del archivo `NOMBRE-APP/urls.py` se van añadiendo los paths, se leen de arriba a abajo y el que coincida con el da URL ejecuta lo que tiene indicado.

#### Proyecto

Se indica en la carpeta del proyecto. Ejemplo `NOMBRE-PROYECTO/urls.py`.

#### include()

Para llamar a otros URLconf.

Corta la URL y se envía al URLconf especificado para procesarse.

Debe usarse siempre excepto para `admin.site.urls`.

#### path()

Sus argumentos son:

- Route

  Obligatorio.

  Es el urlpattern a matchear.

- View

  Obligatorio.

  Cuando hay un matcheo se llama a la función `view` correspondiente con un objeto `HttpRequest` como primer argumento y se le pasa el resto de valores tomados.

- kwargs

  Opcional.

  Para pasar valores en un diccionario.

- Name

  Opcional.

  Para dar un nombre a la URL y llamarla con él en cualquier parte de Django.

  Facilita cambiar sus características desde un solo archivo.

## Crear usuario admin

```bash
python3 manage.py createsuperuser
```

## settings.py

El archivo está en la ruta del proyecto. Ejemplo: `NOMBRE-PROYECTO/settings.py`.

### Zona horaria

Ejemplo, utilizar CET:

```bash
TIME_ZONE = 'CET'
```

### Apps instaladas en el proyecto

Indicado en la variable `INSTALLED_APPS`, lo que haya en esta variable se tiene en cuenta al ejecutar el comando `migrate`.

## models.py

Es el diseño de la base de datos, las variables donde se guardan valores corresponden con las columnas de la base de datos.

Path: `NOMBRE-APP/models.py`.

Crear las migraciones para aplicar unos cambios:

```bash
python3 manage.py makemigrations
```

Aplicar cambios a la base de datos:

```bash
python3 manage.py migrate
```

El comando `migrate` crea las tablas necesarias (según configuración en `settings.py`) para cada aplicación indicada en `settings.py` > `INSTALLED_APPS`.

## Referencias

Guía

<https://docs.djangoproject.com/en/2.1/intro/>

Instalación

<https://docs.djangoproject.com/en/2.1/intro/install/>

<https://askubuntu.com/questions/643417/message-cannot-find-installed-version-of-python-django-or-python3-django>

Archivos creados con el comando `startproject` (vistas, URLs...)

<https://docs.djangoproject.com/en/2.1/intro/tutorial01/>

settings.py

<https://docs.djangoproject.com/en/2.1/intro/tutorial02/>
