# Parte 4: Despliegue de Django

## Capítulo 15: Despliegue de un proyecto Django

Ahora que hemos examinado los aspectos cruciales del desarrollo de aplicaciones en Django, podemos examinar el proceso de despliegue de Django. Django cuenta con importantes utilidades para desplegar un proyecto y existe una variedad de opciones de despliegue que examinaremos. Finalmente, realizaremos paso a paso el despliegue de una aplicación en un proveedor de plataforma como servicio (*Platform-as-a-Service*, PaaS).

En este capítulo, cubriremos los siguientes temas:
- Arquitectura del servidor
- La lista de verificación de despliegue de Django
- Configuración de un proyecto para el despliegue
- Opciones de despliegue para Django

Comenzaremos con una breve discusión sobre las arquitecturas de servidores.

---

### Sección: Requisitos técnicos

Todos los archivos de código se encuentran en la carpeta `Chapter15` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Arquitectura del servidor

Hasta ahora, hemos estado utilizando el servidor de desarrollo de Django tanto para ejecutar el código Python de Bookr como para servir los archivos estáticos y multimedia. Hemos mencionado a lo largo del libro que el servidor de desarrollo de Django no es adecuado para su uso en producción. No está diseñado para ejecutar múltiples procesos ni manejar muchos usuarios y, lo que es más importante, no se ejecuta de forma segura cuando la configuración `DEBUG` se establece en `False`.

Comenzaremos este capítulo observando la arquitectura utilizada en un servidor web de producción; es decir, donde las funciones del servidor de desarrollo de Django se dividen y son manejadas por dos aplicaciones diferentes. Hay un servidor web frontend (por ejemplo, NGINX, lighttpd o Apache), que recibe la solicitud del navegador, y el servidor de aplicaciones (por ejemplo, Gunicorn o uWSGI), que ejecuta el código Python. El servidor web frontend decide cómo debe manejarse la solicitud. Si es una solicitud de un archivo estático o multimedia, el servidor web frontend puede manejar la solicitud por sí mismo; simplemente puede leer y enviar el archivo. Si la solicitud es para código específico de Python, se reenviará al servidor de aplicaciones para que la maneje. Luego, la aplicación Django analiza la URL y otros datos HTTP, genera un objeto `HttpRequest` mediante una vista y envía la respuesta de vuelta al servidor web frontend. El servidor web frontend luego pasa la respuesta de vuelta al navegador.

La siguiente figura profundiza en este proceso:

*Figura 15.1: Validación de solicitudes*

#### Gunicorn

Gunicorn es un servidor de aplicaciones fácil de usar. Sus configuraciones se pueden controlar con indicadores de línea de comandos o un archivo de configuración. En su uso más simple, se puede ejecutar así:

```bash
gunicorn bookr.wsgi:application
```

Esto inicia Gunicorn usando el WSGI de Django.

WSGI (*Web Server Gateway Interface*) describe un estándar para que Python se comunique con servidores web. Esto significa que cualquier script de Python WSGI debería poder comunicarse con cualquier servidor WSGI, y cualquier combinación de servidor frontend y servidor de aplicaciones se puede utilizar junta, siempre que ambos "hablen WSGI". Por ejemplo, en lugar de usar Gunicorn, podríamos usar uWSGI. Su configuración sería diferente, pero tiene el mismo propósito.

#### Whitenoise

Para un entorno de producción con tráfico significativo, se requiere un servidor web frontend como NGINX para manejar el contenido estático y multimedia. Sin embargo, si estamos configurando un sitio con una demanda modesta, podemos instalar un middleware de Django llamado Whitenoise que cumplirá esta función, y se puede ejecutar un servidor de aplicaciones (como Gunicorn) sin un servidor web frontend.

Desde 2019, Django introdujo la Interfaz de Puerta de Enlace de Servidor Asíncrono (*Asynchronous Server Gateway Interface*, ASGI), que se utiliza para conectarse con servidores asíncronos como Uvicorn y Daphne. Los servidores asíncronos se utilizan con frecuencia para conexiones de larga duración y se ven con frecuencia en aplicaciones como chatbots, notificaciones de sitios, paneles de actualización en vivo y servicios de transmisión (*streaming*). ASGI está fuera del alcance de este libro.

En la siguiente sección, examinaremos la lista de verificación de despliegue en preparación para un despliegue con Gunicorn/Whitenoise.

---

### Sección: La lista de verificación de despliegue

Antes de desplegar una aplicación Django, es necesario configurarla de una manera más robusta, ya que ya no es accesible solo para el desarrollador, sino que está expuesta a todo Internet.

La documentación de Django proporciona una lista de verificación de seguridad que debe seguirse antes de desplegar una aplicación en un entorno de producción: [https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/](https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/).

Un entorno de desarrollo normalmente no cumplirá con las necesidades de seguridad de un entorno de producción. Discutiremos las tareas de la lista de verificación de despliegue de Django que se requieren para reforzar un proyecto de Django y luego las abordaremos para la aplicación Bookr en el Ejercicio 15.01.

A continuación, daremos un resumen de los elementos de la lista de verificación.

#### Ejecutar manage.py check --deploy

Cuando iniciamos el servidor de desarrollo de Django con el comando `manage.py runserver`, podemos notar una línea en la consola que dice:

```text
System check identified 0 issues (0 silenced).
```

Esta línea es la salida del subcomando `check`, que se utiliza para detectar e informar sobre problemas de configuración en un proyecto de Django.

Podemos invocarlo independientemente del servidor de desarrollo con este comando:

```bash
$ python manage.py check
```

Se puede utilizar para evaluar el estado de una aplicación Django para el despliegue en producción incluyendo el indicador `--deploy`:

```bash
$ python manage.py check --deploy
```

Esto nos alertará sobre problemas de seguridad en la configuración de nuestro proyecto que deberemos abordar en un entorno de producción antes del despliegue, como se ve en la Figura 15.2.

*Figura 15.2: Advertencias de despliegue*

Podemos usar este comando para evaluar nuestros esfuerzos para proteger una configuración de Django y la reevaluaremos después de crear un entorno de producción robusto en el Ejercicio 15.01.

#### Abandonar manage.py runserver

La funcionalidad interna `runserver` de Django es una herramienta de desarrollo útil, pero como servidor de un solo proceso, no se puede desplegar en un entorno de producción. Un despliegue en producción requiere un servidor web de alta concurrencia. Examinaremos el uso de `gunicorn` para nuestro despliegue.

#### Configuraciones críticas

Estas son las configuraciones más cruciales en un proyecto de Django que deben abordarse antes del despliegue:

##### SECRET_KEY
En el comando de gestión `check`, vimos la siguiente advertencia:

```text
?: (security.W009) Your SECRET_KEY has less than 50 characters, less than 5 unique characters, or it's prefixed with 'django-insecure-' indicating that it was generated automatically by Django. Please generate a long and random value, otherwise many of Django's security-critical features will be vulnerable to attack.
```

Cuando se crea un nuevo proyecto de Django, se genera una `SECRET_KEY` menos segura. Antes del despliegue, es necesario crear una clave más segura para el entorno de producción. Django viene con una función llamada `get_random_secret_key()` que produce una clave que se ajusta a sus estándares de mejores prácticas actuales. Se puede invocar de la siguiente manera:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

También debemos tener en cuenta que esta clave no esté presente en el código fuente del proyecto, y podemos hacer cumplir esto usando una variable de entorno, `decouple` o un archivo de secretos.

##### DEBUG
Establecer `DEBUG` en `True` en un entorno de desarrollo es una forma invaluable de recibir información útil, como rastreos de pila (*tracebacks*) al encontrar errores en solicitudes web. Sin embargo, esta misma información puede ser una vulnerabilidad de seguridad si está disponible en un entorno de producción, ya que puede revelar fragmentos de código fuente, dependencias y configuraciones.

#### Configuraciones específicas del entorno

##### ALLOWED_HOSTS
Hasta ahora hemos dejado el valor de `ALLOWED_HOSTS` como una lista vacía en `settings.py`. Sin embargo, es obligatorio que no esté vacío en un despliegue en producción:

```text
?: (security.W020) ALLOWED_HOSTS must not be empty in deployment.
```

De hecho, cuando `DEBUG` se establece en `False`, una lista vacía para `ALLOWED_HOSTS` generará una excepción:

```text
$ python manage.py runserver
CommandError: You must set settings.ALLOWED_HOSTS if DEBUG is False.
```

La configuración `ALLOWED_HOSTS` se introdujo en Django como un mecanismo para detener los ataques de envenenamiento de encabezados de host (*host header poisoning*). Aquí es donde una solicitud HTTP contendría un encabezado a un sitio malicioso y se usaría para generar un enlace falso posterior en una página HTML o correo electrónico al sitio malicioso.

Si estás desplegando en un servidor con un dominio conocido, aquí es donde debes agregarlo:

```python
ALLOWED_HOSTS = ['example.com']
```

Si estás desplegando en un entorno donde el dominio aún no se conoce, se puede usar un comodín, pero esto es menos seguro y debe reemplazarse cuando el dominio sea evidente:

```python
ALLOWED_HOSTS = ['*']
```

##### CACHES
El almacenamiento en caché web es el mecanismo que permite almacenar las respuestas de solicitudes realizadas con frecuencia para su uso repetido con el fin de evitar recálculos innecesarios que afectarían los recursos de la CPU, el sistema de archivos o la base de datos.

Si bien no hemos abordado las capacidades de almacenamiento en caché de Django en este libro, el almacenamiento en caché es una estrategia importante para una aplicación web que se puede ajustar después del despliegue. Puedes encontrar más información en el siguiente enlace: [https://docs.djangoproject.com/en/6.0/topics/cache/](https://docs.djangoproject.com/en/6.0/topics/cache/).

##### DATABASES
Durante el desarrollo, es satisfactorio trabajar con una base de datos SQLite, pero en un entorno de producción, donde múltiples solicitudes HTTP requerirán sus propias conexiones de base de datos, necesitaremos pasar de una base de datos liviana a un despliegue listo para producción.

Debemos evitar dejar los parámetros de conexión a la base de datos en un archivo como `settings.py`. El servidor de base de datos debe configurarse para permitir solo conexiones desde servidores de aplicaciones y dispositivos que alojan copias de seguridad.

##### EMAIL_BACKEND y configuraciones relacionadas
Hablamos sobre la configuración del correo electrónico en el Capítulo 10 (*Configuración de un servidor de correo electrónico con Django*) y dimos un ejemplo con una cuenta de Gmail. La configuración del correo electrónico es necesaria para automatizar las tareas de administración (como el restablecimiento de contraseñas) y las comunicaciones de la aplicación.

##### Archivos estáticos
Hasta ahora, hemos estado utilizando el servidor de desarrollo para servir archivos estáticos. En un entorno de producción, necesitaremos definir un directorio `STATIC_ROOT` donde `collectstatic` los copiará durante el despliegue.

##### Archivos multimedia
Los archivos multimedia son subidos por los usuarios y nunca se debe confiar en ellos. Por lo tanto, por ejemplo, no desarrolles una aplicación Django que interprete archivos Python subidos.

Considera cuál es tu estrategia de copia de seguridad para el contenido subido.

##### HTTPS
La fuga de datos a través de conexiones HTTP inseguras es un problema perenne en la seguridad web. Estas dos configuraciones obligan al uso de conexiones HTTPS para transmitir datos confidenciales:
- **CSRF_COOKIE_SECURE**: Establecer esta variable en `True` hace cumplir la transmisión de la cookie CSRF a través de HTTPS. Por lo general, necesitamos que esté configurada en `False` mientras desarrollamos en `http://localhost:8000` y no tenemos una conexión segura disponible, pero es imperativo configurarla en `True` en producción para que la cookie CSRF no se transmita a través de un protocolo inseguro.
- **SESSION_COOKIE_SECURE**: De manera similar, `SESSION_COOKIE_SECURE=True` hace cumplir que la cookie de sesión se transmita a través de HTTPS.

#### Optimizaciones de rendimiento

Estas son algunas de las optimizaciones de rendimiento que deben considerarse antes del despliegue:

##### Sesiones
Se recomienda utilizar un paquete de sesión en caché, ya que tiene la ventaja de reducir el impacto en la base de datos. Además, cuando desarrollamos nuestra aplicación, generamos una tasa baja de interacción del usuario, pero en un entorno de producción hay una actividad de sesión significativa y las tablas de la base de datos que mantienen esto pueden crecer incrementalmente si los datos de sesión antiguos no se purgan periódicamente.

##### CONN_MAX_AGE
La configuración `CONN_MAX_AGE` permite que las conexiones de base de datos persistan en un hilo del servidor web entre solicitudes. Esto significa que si un hilo de trabajo está ocupado atendiendo solicitudes secuencialmente, no necesita realizar conexiones de base de datos independientes para cada una, ya que se ha establecido una conexión persistente.

##### TEMPLATES
El almacenamiento en caché de plantillas mejora el rendimiento de Django, ya que significa que las plantillas no necesitan leerse del disco y compilarse en Python para cada solicitud, sino que permanecen en la memoria como objetos de plantilla de Python. El comportamiento predeterminado desde Django 4.1 es habilitar el almacenamiento en caché de plantillas, y se implementa internamente en Django, independientemente de los mecanismos de almacenamiento en caché, y no requiere la configuración explícita de una caché (como es el caso del almacenamiento en caché de sesiones o vistas).

Esto significa que el cargador de plantillas predeterminado está configurado en `django.template.loaders.cached.Loader` a menos que se haya deshabilitado explícitamente para fines de depuración. Si has deshabilitado la carga de plantillas en el entorno de desarrollo, asegúrate de volver a habilitarla durante el despliegue.

#### Informe de errores

##### LOGGING
Durante la depuración, normalmente vemos los registros en la consola y los errores de rastreo en el navegador de nuestra aplicación. Un entorno de producción debe producir un registro que sea más persistente y rastreable.

##### ADMINS y MANAGERS
El comportamiento predeterminado del registro de errores de Django es enviar un correo electrónico a todos en la lista `ADMINS` si hay un error 500. `ADMINS` es una lista de direcciones de correo electrónico configuradas en el archivo `<project>/settings.py`. Por defecto está vacía, pero es bueno agregar direcciones de correo electrónico relevantes de desarrolladores y administradores para recibir información sobre errores 500, ya que esta información ya no se muestra en el navegador cuando `DEBUG` es `False`.

Del mismo modo, la lista `MANAGERS` se configura en el archivo `<project>/settings.py`. Los destinatarios de esta lista reciben correos electrónicos sobre enlaces rotos en el proyecto cuando `BrokenLinkEmailsMiddleware` está habilitado:

```python
ADMINS = ['systemadmin@bookr.example.com', 'alice@example.com']
MANAGERS = ['contentmanager@bookr.example.com', 'david@example.com', 'eve@example.com']
```

##### Personalización de las vistas de error predeterminadas
Django incluye plantillas predeterminadas para errores 400, 403, 404 y 500. Puede ser preferible representar estos errores de una manera coherente con el resto del diseño de tu proyecto. Esto se puede hacer agregando tus propios archivos de plantilla `400.html`, `403.html`, `404.html` y `500.html` en la carpeta raíz de plantillas del proyecto.

Con estas consideraciones en mente, podemos proceder a configurar nuestro proyecto Bookr para el despliegue en el siguiente ejercicio.

#### Ejercicio 15.01 – Configuración del proyecto Bookr para el despliegue

En este ejercicio, tenemos el desafío de aplicar la lista de verificación de despliegue de Django a nuestro proyecto Bookr antes de que esté listo para un entorno de producción:

1. Nuestra primera preocupación con el proyecto Bookr es que el archivo `bookr/settings.py` contiene el valor de la configuración `SECRET_KEY`. Una opción más segura es cargarlo desde una variable de entorno o un archivo externo, para que quede desacoplado del código fuente del proyecto Django. También agregaremos una variable en nuestra configuración que actúa como un interruptor. Se llamará `ENVIRONMENT` y su valor será `'development'` o `'production'`. En el Capítulo 10, presentamos el módulo `decouple` y lo importamos en `bookr/settings.py`. Iniciamos un archivo `.env` para incluir las configuraciones confidenciales o específicas del entorno:
   `bookr/settings.py`:
   ```python
   from decouple import config

   SECRET_KEY = config('SECRET_KEY')
   ENVIRONMENT = config('ENVIRONMENT')
   ```
2. Deberemos agregar las configuraciones `SECRET_KEY` y `ENVIRONMENT` al archivo `.env` que creamos en el directorio de nivel superior de nuestro proyecto:
   `.env`:
   ```ini
   SECRET_KEY=....
   ENVIRONMENT=development
   ```
3. Mientras que el archivo `.env` existente se utiliza en nuestro entorno de desarrollo, podemos crear otro archivo `.env` en una ubicación segura donde almacenaremos los valores que establecemos para producción. En este ejercicio, nos referiremos a estos archivos como `.env de desarrollo` y `.env de producción` para evitar confusiones.
   Podemos usar este comando para generar un valor seguro para la clave secreta en el `.env de producción` antes de agregarlo al archivo:
   ```bash
   python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
   ```
   `.env (producción)`:
   ```ini
   SECRET_KEY=....
   ENVIRONMENT=production
   ```
4. Usando el interruptor `ENVIRONMENT`, podemos asegurarnos de que `DEBUG` solo esté configurado en `True` en entornos de desarrollo:
   ```python
   DEBUG = True if config('ENVIRONMENT') == 'development' else False
   ```
5. En la actualidad, nuestro proyecto no tiene hosts permitidos configurados. Podemos usar la función `config` para cargar el valor desde el archivo `.env`. Por ahora, solo agregaremos un valor comodín para `ALLOWED_HOSTS` en el `.env de producción`, pero si tuviéramos un nombre de dominio para nuestro servidor de producción, aquí es donde lo agregaríamos. Por defecto, `decouple.config` carga valores del archivo `.env` como cadenas. Como `ALLOWED_HOSTS` es una lista, necesitamos un método para convertirlo. Podemos usar el método `Csv()` de decouple que analiza una configuración utilizando el formato de valores separados por comas. Por lo tanto, si el archivo `.env` contiene `ALLOWED_HOSTS=a.site,b.site`, `config` lo analizará como `['a.site', 'b.site']`:
   ```python
   from decouple import config, Csv

   ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
   ```
   El archivo `.env de desarrollo` contendrá `ALLOWED_HOSTS=`, que se analizará como una lista vacía.
   Estableceremos provisionalmente el comodín en el archivo `.env de producción` agregando `ALLOWED_HOSTS=*`.
6. La configuración actual de la base de datos apunta a una base de datos SQLite3 en la carpeta del proyecto, `db.sqlite3`. Mantendremos esto para el desarrollo, pero agregaremos una configuración de PostgreSQL en nuestro entorno de producción. En la actualidad, las configuraciones que usaremos aquí son marcadores de posición, pero las cambiaremos más adelante durante el despliegue. Para conectarnos a una base de datos PostgreSQL, necesitamos instalar la biblioteca `psycopg2-binary`:
   ```bash
   pip install psycopg2-binary
   ```
   La forma estándar de configurar un conector de base de datos de Django en el archivo `settings.py` es especificar una base de datos predeterminada que contenga claves para `ENGINE`, `NAME`, `USER`, `PASSWORD`, `HOST` y `PORT`. Configurar seis parámetros diferentes para una base de datos es un enfoque propenso a errores, especialmente si los datos se dividen entre los archivos `settings.py` y `.env`, y sería preferible especificar una única URL de conexión a la base de datos. Aunque Django no tiene esta funcionalidad lista para usar, existe un módulo de Python que la implementa: `dj-database-url`.
   Podemos instalarlo usando `pip`:
   ```bash
   pip install dj-database-url
   ```
   Una vez instalado, podemos especificar una cadena de base de datos utilizando la función `parse`:
   ```python
   dj_database_url.parse(
       'postgres://user:password@host:5432/database',
       conn_max_age=600,
       conn_health_checks=True
   )
   ```
7. Nuestra intención es usar PostgreSQL en producción, pero nos gustaría tener la opción de ejecutarlo también en nuestro entorno de desarrollo, por lo que no queremos usar la variable `ENVIRONMENT` para alternar entre los dos.
   Una mejor estrategia es verificar si la URL de la base de datos está definida en el archivo `.env` y seguirla si está presente, o recurrir a una base de datos SQLite3 si está ausente.
   También configuraremos los parámetros `conn_max_age` y `conn_health_checks` de la función `dj_database_url.parse`:
   `bookr/settings.py`:
   ```python
   DATABASE_URL = config('DATABASE_URL', default='')
   CONN_MAX_AGE = config('CONN_MAX_AGE', default=0, cast=int)
   CONN_HEALTH_CHECKS = config('CONN_HEALTH_CHECKS', default=False, cast=bool)

   if DATABASE_URL:
       DATABASES = {
           'default': dj_database_url.parse(
               DATABASE_URL,
               conn_max_age=CONN_MAX_AGE,
               conn_health_checks=CONN_HEALTH_CHECKS
           )
       }
   else:
       DATABASES = {
           'default': {
               'ENGINE': 'django.db.backends.sqlite3',
               'NAME': BASE_DIR / 'db.sqlite3',
           }
       }
   ```
   Por lo tanto, si no tenemos la intención de utilizar una base de datos externa para el entorno de desarrollo, simplemente no incluimos estas configuraciones en el `.env de desarrollo` y el proyecto se configurará con una base de datos SQLite3.
   Si tenemos una base de datos PostgreSQL, podríamos incluir algo similar a lo que sigue en el archivo `.env`:
   `.env`:
   ```ini
   DATABASE_URL=postgres://<user>:<password>@<host>:<port>/<database>
   CONN_MAX_AGE=600
   CONN_HEALTH_CHECKS=True
   ```
8. Nuestro servidor de desarrollo sirve los archivos desde `STATICFILES_DIRS`. Sin embargo, en nuestro entorno de producción, esta funcionalidad se desacopla del servidor Gunicorn y requiere un servidor independiente como NGINX para servir los archivos estáticos o el uso de middleware como Whitenoise, que coopta Gunicorn para tal fin. Para facilitar el despliegue, instalaremos Whitenoise:
   ```bash
   pip install whitenoise
   ```
   También necesitaremos especificar este middleware en el archivo `settings.py` y asignar una ubicación para `STATIC_ROOT` donde el proceso `collectstatic` pueda guardar los archivos durante el despliegue:
   `bookr/settings.py`:
   ```python
   MIDDLEWARE = [
       'django.middleware.security.SecurityMiddleware',
       'whitenoise.middleware.WhiteNoiseMiddleware',
       …
   ]
   …
   STATIC_ROOT = BASE_DIR / 'staticfiles'
   ```
   Si bien no realizaremos ningún cambio en `MEDIA_ROOT` y `MEDIA_URL` en esta configuración, vale la pena señalar que no todos los entornos de despliegue alojados utilizan entornos de almacenamiento de archivos persistentes.
9. En la configuración de HTTPS, queremos asegurarnos de que las cookies de CSRF y las sesiones solo se sirvan a través de HTTPS en producción. Como nuestro entorno de desarrollo utiliza HTTP (`http://localhost:8000/`), no queremos forzar este comportamiento en un entorno de desarrollo:
   `bookr/settings.py`:
   ```python
   if config('ENVIRONMENT') == 'development':
       CSRF_COOKIE_SECURE = False
       SESSION_COOKIE_SECURE = False
   else:
       CSRF_COOKIE_SECURE = True
       SESSION_COOKIE_SECURE = True
   ```

Estos son los cambios esenciales que deben realizarse en los archivos `bookr/settings.py` y `.env` para prepararse para el despliegue.

Puedes encontrar el código completo en la carpeta `Chapter15` en el repositorio de GitHub de este libro.

---

### Sección: Opciones de despliegue para Django

Si bien nos hemos decidido por una base de datos y un servidor web de alto rendimiento, como Gunicorn, todavía se nos presentan varias opciones para el despliegue del proyecto Django. Existe una variedad de soluciones que se enumeran a continuación.

#### Despliegue manual

Tradicionalmente, los administradores de sistemas con conocimiento del entorno de línea de comandos de Linux desplegaban los proyectos de Django manualmente. Las bases de datos y los servidores web eran configurados y mantenidos manualmente por estos administradores. Este tipo de enfoque puede resultar atractivo si tienes una infraestructura existente en la que desplegar, pero para muchos desarrolladores de hoy en día requiere recursos y conjuntos de habilidades adicionales fuera de sus capacidades.

#### Despliegue en contenedores

El despliegue en contenedores implica desarrollar "contenedores" para la configuración de módulos de servidores virtuales ligeros que especifican los requisitos del servidor y el código de la aplicación necesarios para el despliegue mediante módulos de plataforma preexistentes.

Docker es la plataforma de contenedorización de nivel de entrada y se conecta con un ecosistema de herramientas de alto rendimiento como Docker Compose, Docker Swarm y Kubernetes.

Este es un ejemplo de una especificación de archivo Docker (`Dockerfile`) necesaria para desplegar y ejecutar un proyecto Django mínimo:

```dockerfile
# Specify a pre-built docker container
FROM python:3.12
# Set the working directory in the container
WORKDIR /usr/src/app
# Copy project code into the container's working directory
COPY . .
# Install the dependencies from the requirements file
RUN pip install --no-cache-dir -r requirements.txt
# Expose port 8000 to the host machine
EXPOSE 8000
# Command to run the gunicorn production server
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "bookr:wsgi"]
```

#### Plataforma como servicio (PaaS)

Los proveedores de plataforma como servicio (*Platform-as-a-Service*, PaaS) ofrecen un entorno de alojamiento que se puede utilizar sin tener experiencia en administración de sistemas. Se trata de plataformas en la nube que alojarán un entorno de Python capaz de ejecutar un proyecto de Django y sus servicios auxiliares, como un servidor de base de datos, copias de seguridad y alojamiento de archivos estáticos.

Los proveedores de PaaS de nivel de entrada populares incluyen Render.com, PythonAnywhere.com, Seenode.com y Railway.com. Los proveedores de nube establecidos, incluidos DigitalOcean, AWS y Google Cloud Platform, también atienden configuraciones de PaaS.

Muchos de estos proveedores tienen documentos y videos detallados sobre cómo desplegar proyectos de Django en sus plataformas. En el siguiente ejercicio, realizaremos un tutorial de un despliegue de Django con la PaaS Render.com.

#### Ejercicio 15.02 – Despliegue del proyecto Bookr

En este ejercicio, trabajaremos en los pasos involucrados en el despliegue de un proyecto Django en un entorno PaaS. Crearemos un repositorio para el proyecto Bookr usando GitHub.com y luego usaremos Render.com para alojarlo. Esta es una estrategia de despliegue simple que implica aprovechar los servicios web y no requiere experiencia en DevOps o administración de sistemas.

Al momento de escribir este artículo, Render.com tiene un nivel gratuito en su estructura de precios. Existen algunas limitaciones con esta opción, ya que no brinda acceso a la línea de comandos, el almacenamiento de archivos no es persistente y la base de datos se restablece después de 30 días, con la pérdida de datos.

Hay información sobre los niveles de precios en [https://render.com/pricing](https://render.com/pricing).

Para seguir estas instrucciones, deberás crear una cuenta de [github.com](https://github.com/) si aún no tienes una. El ejercicio ha sido estructurado de modo que no se requieren conocimientos de Git en la línea de comandos (o una interfaz gráfica de usuario). Sin embargo, si tienes experiencia con Git, puedes crear el repositorio de la forma que te resulte más familiar.

Se asume que has seguido los ejemplos de código de este libro y tienes una versión funcional del proyecto Django Bookr en tu computadora local. Si no la tienes y deseas seguir el despliegue, puedes descargar el código de la carpeta `Chapter15` en el repositorio de GitHub de este libro. Esta versión se desarrolló con Python 3.12.4.

1. Inicia sesión en [github.com](https://github.com/) y crea un nuevo repositorio seleccionando **New repository** en el menú `+`, como en la Figura 15.3:
   *Figura 15.3: Adición de un nuevo repositorio en GitHub*
2. Ahora, vemos un formulario en blanco donde ingresamos los detalles de nuestro nuevo repositorio, como en la Figura 15.4.
   En la sección General del formulario, podemos asignarle al repositorio un nombre como `bookr`. En la sección Configuration, haz lo siguiente:
   - Establece **Choose visibility** en **Public** (esto simplifica el proceso de despliegue de Render.com).
   - Para **Add .gitignore**, selecciona **Python** en el menú desplegable.
   Ahora podemos hacer clic en el botón **Create repository** en la parte inferior de la pantalla:
   *Figura 15.4: Creación de un nuevo repositorio*
3. Esto creará un repositorio con un solo archivo llamado `.gitignore`. Este archivo le indica a Git qué tipos de archivos no se deben rastrear en el repositorio. El `.gitignore` de Python enumera los archivos de base de datos SQLite, los directorios `__pycache__` y una variedad de otros archivos del sistema que son generados por el entorno de ejecución o el sistema operativo pero que no deben mantenerse como parte del repositorio de código.
4. Si seleccionamos la opción **Upload files** en el menú `+`, llegamos a una página con una zona de colocación para la funcionalidad de arrastrar y soltar:
   *Figura 15.6: La zona de colocación de GitHub*
5. En nuestro Finder o Explorador de archivos, podemos navegar dentro de la carpeta del proyecto `bookr`, seleccionando todos los archivos y carpetas relevantes excepto el archivo `db.sqlite3`, `__pycache__` o el archivo `.env`:
   *Figura 15.7: Los archivos seleccionados para arrastrar y soltar desde el sistema de archivos*
   Estos archivos se pueden arrastrar a la zona de colocación marcada como **Drag files here** para agregarlos a tu repositorio.
6. Ahora, podemos ver los archivos que se han agregado a este repositorio:
   *Figura 15.8: Los archivos agregados al repositorio de GitHub*
   Aunque tenemos un archivo `.gitignore`, este paso de arrastrar y soltar lo elude y se habrán incluido algunos archivos no deseados, como los archivos `.pyc` descritos. Estos se pueden eliminar manualmente de la confirmación de Git haciendo clic en el icono de papelera junto a cada archivo.
7. En la parte inferior de esta página se encuentra el cuadro de diálogo **Commit changes**. Podemos agregar un mensaje de confirmación y una descripción extendida opcional antes de hacer clic en el botón verde **Commit changes**:
   *Figura 15.9: El cuadro de diálogo Commit changes de GitHub*
8. Ahora hemos cargado nuestro proyecto Bookr en un repositorio de Git llamado `bookr` que será visible públicamente en `https://github.com/<username>/bookr`:
   *Figura 15.10: El repositorio que contiene el proyecto Bookr*
9. Ahora que tenemos nuestro repositorio creado, podemos crear una cuenta en [render.com](https://render.com/) y crear los servicios necesarios para el despliegue. Podemos utilizar el nivel de cuenta gratuito, **Hobby - For hobbyists and students**. Después de haber creado tu cuenta, verás el panel de control donde puedes crear nuevos servicios.
   Render.com ofrece *Static Sites*, *Web Services*, *Private Services*, *Background Workers*, *Cron Jobs*, *Postgres* y *Key Value* como sus componentes de PaaS. Para este ejercicio, desplegaremos Postgres y un servicio web.
   *Figura 15.11: El panel de control de Render.com*
10. Selecciona **New Postgres** en el panel de Render. Verás un formulario para configurar y desplegar tu nueva base de datos. Asigna un nombre de instancia y un nombre de base de datos adecuados, como `pg-bookr` y `bookr`, respectivamente. Selecciona una región que esperas que esté más cerca de tu base de usuarios. En la actualidad, Render ofrece sitios de alojamiento en América del Norte, Europa y Asia:
    *Figura 15.12: El formulario de nueva base de datos*
11. Por ahora, podemos desplazarnos hacia abajo y seleccionar el tipo de instancia **Free**. Al momento de escribir este artículo, Render no conserva las instancias de Postgres gratuitas después del período de prueba de 30 días y no brinda acceso a sus servicios de copia de seguridad alojados. Una instancia de Postgres se puede actualizar a un nivel de pago durante este tiempo. El nivel gratuito actualmente ofrece 1 GB de almacenamiento, lo cual es suficiente para un despliegue inicial de muchas aplicaciones:
    *Figura 15.13: Opciones de planes de Render.com*
12. Una vez que hayas confirmado que el costo del total mensual es el esperado, haz clic en el botón **Create Database**:
    *Figura 15.14: Total mensual de Render.com*
13. Verás un resumen de la instancia de base de datos creada. Toma nota de la fecha de vencimiento de la base de datos, ya que deberás actualizar a un nivel de pago antes de este momento o correrás el riesgo de perder tus datos:
    *Figura 15.15: Resumen de la base de datos PostgreSQL*
14. Haz clic en los iconos de ver y copiar de **Internal Database URL**:
    *Figura 15.16: URL de base de datos interna*
    Agrega esta cadena al archivo `.env de producción`:
    ```ini
    DATABASE_URL=postgresql://bookr_<…>_user:<long password>/bookr_<…>
    ```
    Nuestro servicio de Postgres ya está configurado. Podemos comenzar a aprovisionar el servicio web.
15. Necesitamos agregar una configuración `PYTHON_VERSION` a nuestro archivo `.env`. Esto refleja la versión de Python con la que se ejecuta nuestro proyecto Django. Render utilizará esta información para aprovisionar una versión adecuada de Python en el servicio web:
    ```ini
    PYTHON_VERSION=3.12.4
    ```
16. Desde el panel de Render, haz clic en **Add Web Service**. Se te dirigirá a un formulario con los detalles del servicio web. En **Source Code**, incluye la URL de tu repositorio de Git. Completa los campos **Name**, **Language** y **Branch** con `bookr`, `Python 3` y `main`, respectivamente. Selecciona la misma región que hiciste para el servicio de Postgres:
    *Figura 15.17: El nuevo formulario de servicio web*
17. En **Environment Variables**, existe la opción de agregar desde `.env` (**Add from .env**). Verás un cuadro de diálogo donde puedes pegar tu archivo `.env`. Las variables de entorno se extraerán del archivo y se mostrarán en el formulario:
    *Figura 15.18: Variables de entorno*
18. Debajo de Environment Variables hay una sección del formulario que contiene **Build Command** y **Start Command**. Nuestro comando de compilación implicará inicialmente declaraciones para instalar los requisitos, crear la estructura de la base de datos con el subcomando `migrate`, `collectstatic` y crear un superusuario. Estas declaraciones se pueden concatenar en una sola instrucción de compilación mediante el operador `&&`. Puedes configurar tu contraseña en la instrucción utilizando una variable de entorno de Linux, `DJANGO_SUPERUSER_PASSWORD`. Asegúrate de incluir tu propio nombre de usuario y contraseña:
    ```bash
    pip install -r requirements.txt && python manage.py migrate && python manage.py collectstatic --noinput && DJANGO_SUPERUSER_PASSWORD='<HardP@ssw0rd!>' python manage.py createsuperuser --noinput --username <yourname> --email <yourname@example.com>
    ```
    En compilaciones posteriores después del despliegue inicial, puedes omitir la última cláusula en este comando de compilación, ya que el superusuario ya se habrá creado.
    Para nuestro entorno, el comando de inicio será `gunicorn bookr.wsgi`. Asegúrate de haber incluido `gunicorn` en el archivo `requirements.txt`:
    *Figura 15.19: Directorio raíz, comandos de compilación e inicio*
19. Ahora puedes hacer clic en el botón **Deploy Web Service** en la parte inferior del formulario y comenzará el despliegue:
    *Figura 15.20: Compilación del servicio web*
20. Ten en cuenta que en esta pantalla se proporciona una URL para el despliegue de destino (por ejemplo, `https://bookr-9vi8.onrender.com`). Al visitar este enlace durante el despliegue, verás el estado del despliegue del servicio:
    *Figura 15.21: Estado del despliegue del servicio en Render*
21. Eventualmente, si el despliegue tiene éxito, verás la página web familiar que hemos estado desarrollando:
    *Figura 15.22: El proyecto desplegado*
22. Mantén un ojo en la pantalla de compilación para detectar cualquier error. Es posible que hayas introducido un parámetro de configuración o una variable de entorno de forma incorrecta. Incluso puede haber errores en tu código fuente que deban corregirse en el repositorio antes de volver a desplegar. Este es también el punto en el que queremos agregar el dominio asignado a la variable de entorno `ALLOWED_HOSTS` en lugar de usar el comodín:
    ```ini
    ALLOWED_HOSTS=<yourdomain>.onrender.com
    ```
23. Al reconstruir después de un error o una reconfiguración, la forma más segura de hacerlo es haciendo clic en **Clear build cache & deploy** en el menú **Manual Deploy**:
    *Figura 15.23: Clear build cache & deploy*
24. Si tu despliegue parece exitoso, confirma que la base de datos se haya migrado y que el superusuario se haya creado iniciando sesión con los detalles de nombre de usuario y contraseña que proporcionaste en el Paso 18:
    *Figura 15.24: El proyecto Django en producción*

Con esto, nuestro despliegue en producción está completo.

---

### Sección: Resumen

En este capítulo, hemos examinado arquitecturas simples y estrategias de despliegue para un proyecto de Django y hemos aprendido el proceso de reconfigurarlo para que pase de ser una instancia de desarrollo a un despliegue listo para producción. Siguiendo la lista de verificación de despliegue, hemos trabajado en los pasos de un despliegue en un proveedor de PaaS, Render.com.

A lo largo de este libro, hemos cubierto en profundidad todo lo necesario para configurar Python para instalar Django: crear una aplicación para almacenar datos y aceptar entradas de datos a través de formularios con validación, administrar el sistema y los datos a través del sistema de administración de Django, y configurar un servidor para servir archivos estáticos de modo que la aplicación pueda estilizarse con CSS, mejorarse con JavaScript y mostrar imágenes. Hemos cubierto cómo gestionar usuarios y autenticación, y una personalización profunda para garantizar que Django esté configurado para adaptarse mejor al proyecto que se está llevando a cabo. También vimos cómo proporcionar exportaciones de datos en formatos como CSV y PDF, concluyendo con cómo poner nuestra aplicación a disposición de los visitantes alojándola en un servidor de producción.
