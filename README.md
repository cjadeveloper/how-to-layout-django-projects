# Guía de diseño de proyectos en Django

## Cambios al diseño de projectos por default de Django 2.x

Tomé de guía la estructura de diseño que se explica en el [libro](https://www.twoscoopspress.com/products/two-scoops-of-django-1-11) de Daniel y Audrey Roy Greenfield. También uso [python-decouple](https://pypi.org/project/python-decouple/), una librería que ayuda a separar los parámetros de configuración del código fuente en un archivo de entorno, y que la conocí en [ésta](https://simpleisbetterthancomplex.com/2015/11/26/package-of-the-week-python-decouple.html) entrada del gran blog de Vitor Freitas.

## Set de herramientas en mis proyectos

- [Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html) que es un conjunto de extensiones para [virtualenv](https://pypi.org/project/virtualenv/) de Ian Bicking.
- [Visual Studio Code](https://code.visualstudio.com/) como IDE multiplataforma para escribir el código.
- [python-decouple](https://pypi.org/project/python-decouple/) para separar configuraciones y código.
- [Git](https://git-scm.com/) como control de versiones.
- [EditorConfig](https://editorconfig.org/) que me ayuda a definir y mantener estilos de codificación consistentes entre diferentes editores e IDEs.

## Nueva estructura de directorios

- Lo primero que hago es crear un archivo de entorno `.env` en la raiz del proyecto con los parámetros que queremos ocultar y adicionamos la línea .env al archivo `.gitignore` para no enviar este archivo de entorno al repositorio remoto. Seguimos las instrucciones del [blog](https://simpleisbetterthancomplex.com/2015/11/26/package-of-the-week-python-decouple.html) para hacer uso en el settings.py de nuestro archivo de entorno.
- Cambiamos el nombre de la carpeta del sitio (directorio de trabajo externo, también conocido como carpeta de repositorio) de `myapp` a `myapp_project`.
- Creamos una carpeta `config` dentro de ella y movemos los archivos de la carpeta `myapp` a ella.
- Dentro de esta carpeta `config`, creamos otra de nombre `settings` que contendrá los archivos de configuración por cada entorno de ejecución.
- Para ello vamos a dividir `settings.py` en `base.py`, `local.py` y `production.py`. En principio con esto estaría bien. Luego podremos incorporar `test.py` más adelante para lo necesario a las pruebas. Por ahora lo de las pruebas vamos a ponerlo en `base.py` que contendrá lo común a todos los niveles y será el que se extienda en los demás módulos.
- Editamos `manage.py` para que busque la configuración acorde a cada entorno dentro de nuestro nuevo archivo de configuraciones

Cambiamos esto

```py
#!/usr/bin/env python

# blablabla

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myapp.settings")
    # blablabla
```

Por esto

```py

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img alt="Licencia Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by/3.0/88x31.png" /></a><br />Esta obra está bajo una <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Licencia Creative Commons Atribución 3.0 No portada</a>.
#!/usr/bin/env python

def get_env_variable(var_name):
    from decouple import config

    try:
        return config(var_name)
    except KeyError:
        error_msg = f'Establezca la variable {var_name} en el archivo de entorno .env'
        raise ImproperlyConfigured(error_msg)


if __name__ == "__main__":
    DJANGO_EXECUTION_ENVIRONMENT = get_env_variable('DJANGO_EXECUTION_ENVIRONMENT')
    if DJANGO_EXECUTION_ENVIRONMENT == 'LOCAL':
        os.environ.setdefault(
            "DJANGO_SETTINGS_MODULE", "config.settings.local")
    if DJANGO_EXECUTION_ENVIRONMENT == 'PRODUCTION':
        os.environ.setdefault(
            "DJANGO_SETTINGS_MODULE","config.settings.production")

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
```

- Por último, en nuestro archivo de entorno, .env, especificamos la variable de entorno `DJANGO_EXECUTION_ENVIRONMENT = "LOCAL"` así Django irá a buscar la configuración del archivo `"config.settings.local"` como le indicamos.

### Creación de nuevas aplicaciones

Para que cada app nueva que creamos se coloque dentro de nuestra carpeta de aplicaciones, osea dentro de `myapp`, ejecutaremos `django-admin startapp new_app` como siempre, pero desde dentro de la carpeta de aplicaciones.

La ventaja que brinda este diseño es que nos quedan todas las apps ordenadas, dentro de la una carpeta de aplicaciones que, por convención, la llamamos igual que el proyecto. Así, si el projecto es grande y hay muchas aplicaciones, va a ser más fácil, navegar por las mismas dentro de una sola carpeta.

Como siempre, va a haber una app llamada `core` que va a ser la que contenga el nucleo de nuestra app.

#### Archivo `apps.py`

En cada archivo `apps.py` debemos cambiar el nombre de la app por default

```py
class RegistrationConfig(AppConfig):
    name = 'app_name'
```

a

```py
class RegistrationConfig(AppConfig):
    name = 'myapp.app_name'
```

#### Archivo `urls.py`

En el archivo `urls.py` de cada nueva app devemos agregar, antes de la lista `urlpatterns`

```py
app_name = "app_name"
```

### Creación de templates

Para trabajar con los templates de cada app, vamos a crear una carpeta `templates` dentro de la carpeta de aplicaciones `myapp`. Para organizar los templates, podemos incluirlos dentro de carpetas con los nombres de las apps.

Luego en el archivo de settings `base.py`, cambiamos la lista TEMPLATES que vienen por defecto con el proyecto de Django

```py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        # blablabla
    },
]

```

Por este otro, que incluye el path de nuestro directorio de apps

```py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            str(APPS_DIR.path('templates')),
        ],
        # blablabla
    },
]
```

