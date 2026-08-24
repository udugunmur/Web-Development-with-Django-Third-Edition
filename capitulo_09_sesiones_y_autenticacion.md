# Parte 2: Creación de aplicaciones web con Django

## Capítulo 9: Sesiones y Autenticación

Hasta ahora, hemos utilizado Django para desarrollar aplicaciones dinámicas que permiten a los usuarios interactuar con los modelos de la aplicación, pero no hemos intentado proteger estas aplicaciones del uso no deseado. Por ejemplo, nuestro proyecto `bookr` permite a los usuarios no autenticados agregar reseñas y cargar archivos multimedia. Este es un problema de seguridad crítico para cualquier aplicación web en línea, ya que deja el sitio abierto a la publicación de spam u otro material inapropiado y al vandalismo del contenido existente. Queremos que la creación y modificación de contenido se limite estrictamente a los usuarios autenticados que se hayan registrado en el sitio.

La aplicación de autenticación proporciona a Django los modelos para representar usuarios, grupos y permisos. También proporciona middleware, funciones de utilidad, decoradores y mixins que nos ayudan a integrar la autenticación de usuarios en nuestras aplicaciones. Además, la aplicación de autenticación nos permite agrupar y nombrar ciertos conjuntos de usuarios.

En el Capítulo 4 (*Introducción al Administrador de Django*), usamos la aplicación Admin para crear un grupo de usuarios de mesa de ayuda (*help desk*) con los permisos *Can view log entry*, *Can view permission*, *Can change user* y *Can view user*. Se podía hacer referencia a esos permisos en nuestro código mediante sus nombres en código (*codenames*) correspondientes: `view_logentry`, `view_permissions`, `change_user` y `view_user`. En este capítulo, aprenderemos cómo personalizar el comportamiento de Django según los permisos de usuario específicos.

Los permisos controlan lo que los diferentes tipos de usuarios pueden hacer en tu aplicación. Los permisos se pueden asignar a grupos o directamente a usuarios individuales. Desde el punto de vista administrativo, es más limpio asignar permisos a grupos. Los grupos facilitan el modelado de roles y estructuras organizacionales. Si se crea un nuevo permiso, lleva menos tiempo modificar algunos grupos que recordar asignarlo a un subconjunto de usuarios.

Ya estamos familiarizados con la creación de usuarios y grupos y la asignación de permisos utilizando varios métodos, como la opción de instanciar usuarios y grupos a través del modelo mediante scripts y la comodidad de crearlos a través de la aplicación de administración de Django. La aplicación de autenticación también nos ofrece formas programáticas de crear y eliminar usuarios, grupos y permisos, y asignar relaciones entre ellos.

A medida que avancemos en este capítulo, aprenderemos a usar la autenticación y los permisos para implementar la seguridad de la aplicación y cómo almacenar datos específicos del usuario para personalizar su experiencia. Esto nos ayudará a proteger el proyecto `bookr` de cambios no autorizados en el contenido y hacerlo contextualmente relevante para diferentes tipos de usuarios. Agregar esta seguridad básica a nuestro proyecto `bookr` es crucial antes de considerar implementarlo en Internet.

Este capítulo comienza con una breve introducción al middleware antes de profundizar en los conceptos de modelos de autenticación y motores de sesión. Implementarás el modelo de autenticación de Django para restringir los permisos a un conjunto específico de usuarios. Luego, aprenderás cómo aprovechar la autenticación de Django para proporcionar un enfoque flexible a la seguridad de las aplicaciones. Después de eso, aprenderás cómo Django admite múltiples motores de sesión para retener los datos del usuario. Al final de este capítulo, serás competente en el uso de sesiones para retener información sobre interacciones pasadas del usuario y mantener las preferencias del usuario para cuando se vuelvan a visitar las páginas.

Cubriremos los siguientes temas en este capítulo:
- Middleware
- Almacenamiento de contraseñas en Django
- Decoradores de autenticación y redirección
- Mejora de plantillas con datos de autenticación
- Sesiones

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter09` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Middleware

La autenticación, así como la gestión de sesiones (sobre la que aprenderemos en la sección *Sesiones*), se gestionan mediante lo que se conoce como una pila de middleware (*middleware stack*). Antes de implementar la autenticación en nuestro proyecto `bookr`, aprendamos un poco sobre esta pila de middleware y sus módulos.

En el Capítulo 3 (*Vistas, Configuración de URLs y Plantillas*), analizamos la implementación de Django del proceso de solicitud/respuesta (*request/response*), junto con su funcionalidad de vistas y renderizado. Además de esto, otra característica que juega un papel extremadamente importante cuando se trata del procesamiento web central de Django es el middleware. El middleware de Django se refiere a una variedad de componentes de software que intervienen en este proceso de solicitud/respuesta para integrar funcionalidades importantes como seguridad, administración de sesiones y autenticación.

Por lo tanto, cuando escribimos una vista en Django, no tenemos que configurar explícitamente una serie de características de seguridad importantes en los encabezados de respuesta. Estas adiciones al objeto de respuesta las realiza automáticamente la instancia de `SecurityMiddleware` después de que la vista devuelve su respuesta. Como los componentes de middleware envuelven la vista y realizan una serie de preprocesamientos en la solicitud y posprocesamientos en la respuesta, la vista no se sobrecarga con una gran cantidad de código repetitivo y podemos concentrarnos en codificar la lógica de la aplicación en lugar de preocuparnos por el comportamiento del servidor de bajo nivel. En lugar de incorporar estas funcionalidades en el núcleo de Django, la implementación de una pila de middleware en Django permite que estos componentes sean tanto opcionales como reemplazables.

#### Módulos de middleware

Cuando ejecutamos el subcomando `startproject`, se agrega una lista predeterminada de módulos de middleware a la variable `MIDDLEWARE` en el archivo `<proyecto>/settings.py`, de la siguiente manera:

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]
```

Esta es una pila de middleware mínima que es adecuada para la mayoría de las aplicaciones de Django. La siguiente lista detalla el propósito general de cada módulo:
- **SecurityMiddleware**: Proporciona mejoras de seguridad comunes, como manejar redirecciones SSL y agregar encabezados de respuesta para evitar ataques comunes.
- **SessionMiddleware**: Habilita el soporte de sesiones y asocia transparentemente una sesión almacenada con la solicitud actual.
- **CommonMiddleware**: Implementa una gran cantidad de características misceláneas, como rechazar solicitudes de la lista `DISALLOWED_USER_AGENTS`, implementar reglas de reescritura de URL y configurar el encabezado `Content-Length`.
- **CsrfViewMiddleware**: Agrega protección contra falsificación de solicitudes entre sitios (*Cross-Site Request Forgery*, CSRF).
- **AuthenticationMiddleware**: Agrega el atributo `user` al objeto de solicitud (`request`).
- **MessageMiddleware**: Agrega soporte para mensajes temporales (*flash messages*).
- **XFrameOptionsMiddleware**: Protege contra ataques de secuestro de clics (*clickjacking*) mediante el encabezado `X-Frame-Options`.

Los módulos de middleware se cargan en el orden en que aparecen en la lista `MIDDLEWARE`. Esto tiene sentido porque queremos llamar primero al middleware que se ocupa de los problemas de seguridad iniciales para que las solicitudes peligrosas se rechacen antes de que se produzca un procesamiento posterior. Django también incluye varios otros módulos de middleware que realizan funciones importantes, como el uso de compresión de archivos gzip, configuración de redirección y configuración de caché web.

Este capítulo está dedicado a discutir dos aspectos importantes del desarrollo de aplicaciones con estado que se implementan como componentes de middleware: `SessionMiddleware` y `AuthenticationMiddleware`.

El método `process_request` de `SessionMiddleware` agrega un objeto `session` como un atributo del objeto `request`. El método `process_request` de `AuthenticationMiddleware` agrega un objeto `user` como un atributo del objeto `request`.

Es posible escribir un proyecto de Django sin estas capas de la pila de middleware si un proyecto no requiere autenticación de usuario o un medio para preservar el estado de las interacciones individuales. Sin embargo, la mayor parte del middleware predeterminado juega un papel importante en la seguridad de la aplicación. Si no tienes una buena razón para cambiar los componentes de middleware, es mejor mantener estas configuraciones iniciales. La aplicación Admin requiere que se ejecuten `SessionMiddleware`, `AuthenticationMiddleware` y `MessageMiddleware`, y el servidor Django arrojará errores como estos si la aplicación Admin está instalada sin ellos:

```text
django.core.management.base.SystemCheckError: SystemCheckError: System check identified some issues:
ERRORS:
?: (admin.E408) 'django.contrib.auth.middleware.AuthenticationMiddleware' must be in MIDDLEWARE in order to use the admin application.
?: (admin.E409) 'django.contrib.messages.middleware.MessageMiddleware' must be in MIDDLEWARE in order to use the admin application.
?: (admin.E410) 'django.contrib.sessions.middleware.SessionMiddleware' must be in MIDDLEWARE in order to use the admin application.
```

Ahora que conocemos los módulos de middleware, veamos una forma en que podemos habilitar la autenticación en nuestro proyecto utilizando las vistas y plantillas de la aplicación de autenticación.

#### Implementación de vistas y plantillas de autenticación

Encontramos el formulario de inicio de sesión en la aplicación Admin en el Capítulo 4 (*Introducción al Administrador de Django*). Este es el punto de entrada de autenticación para los usuarios del personal que tienen acceso a la aplicación Admin. También necesitamos crear una capacidad de inicio de sesión para los usuarios comunes que desean realizar reseñas de libros. Afortunadamente, la aplicación de autenticación viene con las herramientas para hacer esto posible.

A medida que trabajamos con los formularios y las vistas de la aplicación de autenticación, encontraremos una gran flexibilidad en su implementación. Somos libres de implementar nuestras propias páginas de inicio de sesión, definir políticas de seguridad muy simples o detalladas a nivel de vista y autenticarnos contra autoridades externas.

La aplicación de autenticación existe para admitir muchos enfoques diferentes de autenticación, de modo que Django no imponga rígidamente un único mecanismo. Para un usuario primerizo que encuentra la documentación, esto puede resultar bastante desconcertante. En su mayor parte en este capítulo, seguiremos los valores predeterminados de Django, pero se indicarán algunas de las opciones de configuración importantes.

El objeto de configuración de un proyecto de Django contiene atributos para el comportamiento del inicio de sesión. `LOGIN_URL` especifica la URL de la página de inicio de sesión. `'/accounts/login/'` es el valor predeterminado. `LOGIN_REDIRECT_URL` especifica la ruta a donde se redirige un inicio de sesión exitoso. La ruta predeterminada es `'/accounts/profile/'`.

La aplicación de autenticación proporciona formularios y vistas estándar para llevar a cabo tareas de autenticación típicas. Estos formularios se encuentran en `django.contrib.auth.forms`, mientras que las vistas se encuentran en `django.contrib.auth.views`.

Las vistas son referenciadas por estos patrones de URL presentes en `django.contrib.auth.urls`:

```python
urlpatterns = [
    path('login/', views.LoginView.as_view(), name='login'),
    path('logout/', views.LogoutView.as_view(), name='logout'),
    path('password_change/', views.PasswordChangeView.as_view(), name='password_change'),
    path('password_change/done/', views.PasswordChangeDoneView.as_view(), name='password_change_done'),
    path('password_reset/', views.PasswordResetView.as_view(), name='password_reset'),
    path('password_reset/done/', views.PasswordResetDoneView.as_view(), name='password_reset_done'),
    path('reset/<uidb64>/<token>/', views.PasswordResetConfirmView.as_view(), name='password_reset_confirm'),
    path('reset/done/', views.PasswordResetCompleteView.as_view(), name='password_reset_complete'),
]
```

Si este estilo de vistas parece desconocido, es porque son vistas basadas en clases (*class-based views*) en lugar de las vistas basadas en funciones con las que nos hemos encontrado anteriormente. Aprenderemos más sobre las vistas basadas en clases en el Capítulo 11 (*Plantillas Avanzadas y Vistas Basadas en Clases*). Por ahora, ten en cuenta que la aplicación de autenticación hace uso de la herencia de clases para agrupar la funcionalidad de las vistas y evitar una gran cantidad de código repetitivo.

Si queremos mantener las URLs y vistas predeterminadas presupuestas por la aplicación de autenticación y la configuración de Django, podemos incluir las URLs de la aplicación de autenticación en los `urlpatterns` de nuestro proyecto.

Al adoptar este enfoque, nos hemos ahorrado mucho trabajo. Solo necesitamos incluir las URLs de la aplicación de autenticación en nuestro archivo `<proyecto>/urls.py` y asignarlo al espacio de nombres `'accounts'`. La designación de este espacio de nombres garantiza que nuestras URLs inversas correspondan a los valores de plantilla predeterminados de las vistas. Las vistas de `authlib` contienen referencias de código a plantillas llamadas `password_reset_done` y `password_reset_complete`, por lo que sus rutas también deben incluirse explícitamente en `urlpatterns`:

```python
from django.contrib import admin, auth
…
urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include(("django.contrib.auth.urls", "auth"), namespace="accounts")),
    path("accounts/password_reset/done/", auth.views.PasswordResetDoneView.as_view(), name="password_reset_done",),
    path("accounts/reset/done/", auth.views.PasswordResetCompleteView.as_view(), name="password_reset_complete",),
    path("", reviews.views.index),
    path("book-search/", reviews.views.book_search, name="book_search"),
    path("", include("reviews.urls")),
]
```

Aunque la aplicación de autenticación viene con formularios y vistas, carece de las plantillas necesarias para renderizar estos componentes como HTML. La Figura 9.1 enumera las plantillas que necesitamos para implementar la funcionalidad de autenticación en nuestro proyecto. Afortunadamente, la aplicación Admin implementa un conjunto de plantillas que podemos utilizar para nuestros propósitos.

Cuando decimos código fuente de Django, nos referimos al directorio donde reside tu instalación de Django. Si instalaste Django en un entorno virtual (como se detalla en el Capítulo 1, *Introducción a Django*), puedes encontrar estos archivos de plantilla en `<nombre de tu entorno virtual>/lib/python3.X/site-packages/django/contrib/admin/templates/registration/`. Siempre que tu entorno virtual esté activado y Django esté instalado en él, también puedes recuperar la ruta completa al directorio `site-packages` ejecutando el comando `python -c "import sys; print(sys.path)"` en una terminal.

Podríamos simplemente copiar los archivos de plantilla del código fuente de Django en el directorio `django/contrib/admin/templates/registration` y `django/contrib/admin/templates/admin/login.html` en el directorio `templates/registration` de nuestro proyecto:

*Figura 9.1: Rutas predeterminadas para plantillas de autenticación*

Solo necesitamos copiar las plantillas que son dependencias de las vistas, y debemos evitar copiar los archivos `base.html` o `base_site.html`.

Esto da un resultado prometedor al principio, pero tal como están, las plantillas de administración no satisfacen nuestras necesidades precisas, como podemos ver en la página de inicio de sesión (Figura 9.2):

*Figura 9.2: Un primer intento de pantalla de inicio de sesión de usuario*

Como estas páginas de autenticación heredan de la plantilla `admin/base_site.html` de la aplicación Admin, siguen el estilo de la aplicación Admin. Preferiríamos que estas páginas sigan el estilo del proyecto `bookr` que hemos desarrollado. Podemos hacer esto siguiendo estos tres pasos en cada plantilla de Django que hayamos copiado de la aplicación Admin a nuestro proyecto:
1. El primer cambio que se debe hacer es reemplazar la etiqueta `{% extends "admin/base_site.html" %}` con `{% extends "base.html" %}`, ya que queremos heredar de la plantilla base del proyecto `bookr`.
2. Dado que `template/base.html` solo contiene las definiciones de bloques `title`, `brand` y `content`, debemos examinar las sustituciones de bloques adicionales de nuestras plantillas en la carpeta `bookr`. No estamos usando el contenido de los bloques `userlinks` y `breadcrumbs` en los diseños de plantilla de nuestra aplicación, por lo que estos bloques se pueden eliminar por completo. Algunos de estos bloques, como `content_title` y `reset_link`, contienen contenido HTML que es relevante para nuestra aplicación, por lo que deben conservarse.
   Por ejemplo, la plantilla `password_change_done.html` contiene los bloques `userlinks` y `breadcrumbs`:
   ```django
   {% extends "admin/base_site.html" %}
   {% load i18n %}

   {% block userlinks %}
       {% url 'django-admindocs-docroot' as docsroot %}
       {% if docsroot %}<a href="{{ docsroot }}">{% translate 'Documentation' %}</a> / {% endif %}
       {% translate 'Change password' %} /
       <form id="logout-form" … ></form>
       {% include "admin/color_theme_toggle.html" %}
   {% endblock %}

   {% block breadcrumbs %}
       <div class="breadcrumbs">
           <a href="{% url 'admin:index' %}">Home</a> › Password change'
       </div>
   {% endblock %}

   {% block content %}
       <p>{% translate 'Your password was changed.' %}</p>
   {% endblock %}
   ```
   Se simplificará a esta plantilla en el proyecto `bookr`:
   ```django
   {% extends "base.html" %}
   {% load i18n %}

   {% block content %}
       <p>{% translate 'Your password was changed.' %}</p>
   {% endblock %}
   ```
3. Del mismo modo, hay patrones de URL inversa que deben cambiar para reflejar la ruta actual, por lo que `{% url 'login' %}` se reemplaza con `{% url 'accounts:login' %}`.

Teniendo en cuenta estas consideraciones, el siguiente ejercicio se centrará en transformar la plantilla de inicio de sesión de la aplicación Admin en una plantilla de inicio de sesión para el proyecto `bookr`.

El módulo `i18n` se utiliza para crear contenido multilingüe. Si tienes la intención de desarrollar contenido multilingüe para tu sitio web, deja la importación de `i18n`, las etiquetas `translate` y las declaraciones `translateblock` en las plantillas. Por brevedad, no las cubriremos en detalle en este capítulo.

#### Ejercicio 9.01 – Reutilización de la plantilla de inicio de sesión de la aplicación Admin

Comenzamos este capítulo sin una página de inicio de sesión para nuestro proyecto. Al agregar los patrones de URL para la autenticación y copiar las plantillas de la aplicación Admin a nuestro proyecto, podemos implementar la funcionalidad de una página de inicio de sesión. Pero esta página de inicio de sesión no es satisfactoria ya que se copia directamente de la aplicación Admin y está desconectada del diseño de Bookr. En este ejercicio, seguiremos los pasos necesarios para reutilizar la plantilla de inicio de sesión de la aplicación Admin para nuestro proyecto. La nueva plantilla de inicio de sesión deberá heredar su estilo y formato directamente del archivo `templates/base.html` del proyecto `bookr`:

1. Crea un directorio dentro de tu proyecto para `templates/registration`. La plantilla de inicio de sesión de Admin se encuentra en el directorio de código fuente de Django en la ruta `django/contrib/admin/templates/admin/login.html`. Comienza con una etiqueta `extends`, una etiqueta `load`, importando los módulos `i18n` y `static`, y una serie de extensiones de bloque que anulan los bloques definidos en la plantilla secundaria, `django/contrib/admin/templates/admin/base.html`. En el siguiente bloque de código se muestra un fragmento truncado del archivo `login.html`:
   ```django
   {% extends "admin/base_site.html" %}
   {% load i18n static %}

   {% block title %}{% if form.errors %} {% translate "Error:" %} {% endif %} {{ block.super }}{% endblock %}

   {% block extrastyle %}…{% endblock %}

   {% block bodyclass %}…{% endblock %}

   {% block usertools %}{% endblock %}

   {% block nav-global %}{% endblock %}

   {% block content_title %}{% endblock %}

   {% block breadcrumbs %}{% endblock %}

   {% block content %}
       {% if form.errors and not form.non_field_errors %}
           …
       {% endif %}
       <div id="content-main">
           …
           <form action="{{ app_path …>{% csrf_token %}
               …
           </form>
       </div>
   {% endblock %}
   ```
2. Copia esta plantilla de inicio de sesión de Admin, `django/contrib/admin/templates/admin/login.html`, en `templates/registration` y comienza a editar el archivo con PyCharm.
3. Como la plantilla de inicio de sesión que estás editando se encuentra en `templates/registration/login.html` y extiende la plantilla base (`templates/base.html`), reemplaza el argumento de la etiqueta `extends` en la parte superior de `templates/registration/login.html` con lo siguiente:
   ```django
   {% extends "base.html" %}
   ```
4. No necesitamos la mayor parte del contenido de este archivo. Solo conserva el bloque `content`, que contiene el formulario de inicio de sesión. El resto de la plantilla consistirá en cargar las bibliotecas de etiquetas `i18n` y `static`:
   ```django
   {% load i18n static %}

   {% block title %}{% if form.errors %}{% translate "Error:" %} {% endif %}{{ block.super }}{% endblock %}

   {% block content %}
       …
   {% endblock %}
   ```
5. Ahora, debes reemplazar las rutas y los patrones de URL inversa en `templates/registration/login.html` con los que sean apropiados para tu proyecto. Como no tienes una variable `app_path` definida en tu plantilla, es necesario reemplazarla con la URL inversa para el inicio de sesión, `'accounts:login'`. Por lo tanto, considera la siguiente línea:
   ```django
   <form action="{{ app_path }}" method="post" id="login-form">
   ```
   Esta línea cambia de la siguiente manera:
   ```django
   <form action="{% url 'accounts:login' %}" method="post" id="login-form">
   ```
6. No se ha definido ningún `'admin_password_reset'` en las rutas de tu proyecto, por lo que se reemplazará con `'accounts:password_reset'`.
   Considera la siguiente línea:
   ```django
   {% url 'admin_password_reset' as password_reset_url %}
   ```
   Esto cambia a la siguiente línea:
   ```django
   {% url 'accounts:password_reset' as password_reset_url %}
   ```
7. Tu plantilla de inicio de sesión se verá de la siguiente manera:
   ```django
   {% extends "base.html" %}
   {% load i18n static %}

   {% block title %}{% if form.errors %}{% translate "Error:" %} {% endif %}{{ block.super }}{% endblock %}

   {% block content %}
       {% if form.errors and not form.non_field_errors %}
           <p class="errornote">
               {% if form.errors.items|length == 1 %}{% translate "Please correct the error below." %}{% else %}{% translate "Please correct the errors below." %}{% endif %}
           </p>
       {% endif %}
   ```
   Puedes encontrar el código completo de este archivo en la carpeta `Chapter09` en el repositorio de GitHub de este libro.
8. Como se mencionó anteriormente, para usar las vistas de autenticación estándar de Django, debemos agregarles asignaciones de URL. Abre el archivo `urls.py` en el directorio del proyecto `bookr`, asegúrate de que `auth` esté importado de `django.contrib` y agrega las tres rutas a los patrones de URL:
   ```python
   from django.contrib import admin, auth
   …
   urlpatterns = [
       …
       path("accounts/", include(("django.contrib.auth.urls", "auth"), namespace="accounts")),
       path("accounts/password_reset/done/", auth.views.PasswordResetDoneView.as_view(), name="password_reset_done",),
       path("accounts/reset/done/", auth.views.PasswordResetCompleteView.as_view(), name="password_reset_complete",),
       …
   ]
   ```
9. Ahora, cuando visites el enlace de inicio de sesión en `http://127.0.0.1:8000/accounts/login/`, verás esta página:
   *Figura 9.3: La página de inicio de sesión de Bookr*

Al completar este ejercicio, has creado la plantilla requerida para la autenticación sin privilegios de administrador en tu proyecto.

Antes de continuar, deberás asegurarte de que el resto de las plantillas en el directorio `registration` sigan el estilo del proyecto `bookr`; es decir, no heredan de la plantilla `admin/base_site.html` de la aplicación Admin. Ya has visto cómo se hizo esto con las plantillas `password_change_done.html` y `login.html`. Continúa y aplica lo que has aprendido en este ejercicio (y en la sección anterior) al resto de los archivos en el directorio `registration`. Alternativamente, puedes descargar los archivos modificados de la carpeta `Chapter09` en el repositorio de GitHub de este libro.

Ahora que hemos modificado nuestro proyecto para habilitar la autenticación, aprenderemos sobre el almacenamiento de contraseñas y sesiones en Django.

---

### Sección: Almacenamiento de contraseñas en Django

Django no almacena contraseñas en forma de texto sin formato en la base de datos. En cambio, las contraseñas se procesan con un algoritmo de hashing, como PBKDF2/SHA256, BCrypt/SHA256 o Argon2. Como los algoritmos de hashing son una transformación unidireccional, esto evita que la contraseña de un usuario se descifre a partir del hash almacenado en la base de datos. Esto a menudo sorprende a los usuarios que esperan que un administrador del sistema recupere su contraseña olvidada, pero es una práctica recomendada en el diseño de seguridad. Por lo tanto, si consultamos la base de datos para obtener la contraseña, veremos algo como esto:

*Figura 9.4 – Hashes de contraseñas en una base de datos SQLite3 de Django*

Los componentes de esta cadena son `<algoritmo>$<iteraciones>$<sal>$<hash>`. Dado que varios algoritmos de hashing se han visto comprometidos con el tiempo y a veces necesitamos trabajar con requisitos de seguridad obligatorios, Django es lo suficientemente flexible como para adaptarse a nuevos algoritmos y puede mantener datos cifrados en múltiples algoritmos.

#### La página de perfil y el objeto request.user

Cuando un inicio de sesión es exitoso, la vista de inicio de sesión redirige a `/accounts/profile`. Sin embargo, esta ruta no está incluida en el `auth.url` existente, ni la aplicación de autenticación proporciona una plantilla para ella. Para evitar un error de *Page not Found* (Página no encontrada), se requiere una vista y un patrón de URL apropiado.

Cada solicitud de Django tiene un objeto `request.user`. Si la solicitud la realiza un usuario no autenticado, `request.user` será un objeto `AnonymousUser`. Si la solicitud la realiza un usuario autenticado, entonces `request.user` será un objeto `User`. Esto facilita la recuperación de información de usuario personalizada en una vista de Django y su renderizado en una plantilla.

En el siguiente ejercicio, agregaremos una página de perfil a nuestro proyecto `bookr`.

#### Ejercicio 9.02 – Adición de una página de perfil

En este ejercicio, agregaremos una página de perfil a nuestro proyecto. Para hacerlo, debemos incluir la ruta a ella en nuestros patrones de URL y también incluirla en nuestras vistas y plantillas. La página de perfil simplemente mostrará los siguientes atributos del objeto `request.user`:
- `username`
- `first_name` y `last_name`
- `date_joined`
- `email`
- `last_login`

Realiza los siguientes pasos para completar este ejercicio:

1. Agrega `bookr/views.py` al proyecto. Necesita una función de perfil trivial para definir nuestra vista:
   ```python
   from django.shortcuts import render

   def profile(request):
       return render(request, 'profile.html')
   ```
2. En la carpeta de plantillas de la carpeta principal de tu proyecto `bookr`, crea un nuevo archivo llamado `profile.html`. En esta plantilla, se puede hacer referencia fácilmente a los atributos del objeto `request.user` mediante una notación como `{{ request.user.username }}`:
   ```django
   {% extends "base.html" %}

   {% block title %}Bookr{% endblock %}

   {% block content %}
       <h2>Profile</h2>
       <div>
           <p>
               Username: {{ request.user.username }} <br>
               Name: {{ request.user.first_name }} {{ request.user.last_name }}<br>
               Date Joined: {{ request.user.date_joined }} <br>
               Email: {{ request.user.email }}<br>
               Last Login: {{ request.user.last_login }}<br>
           </p>
       </div>
   {% endblock %}
   ```
   También agregamos un bloque que contiene los detalles del perfil del usuario. Más importante aún, nos aseguramos de que `profile.html` extienda `base.html`.
3. Finalmente, esta ruta debe agregarse en la parte superior de la lista `urlpatterns` en `bookr/urls.py`. Primero, importa las nuevas vistas y luego agrega una ruta que vincule la URL `accounts/profile/` a `bookr.views.profile`:
   ```python
   from bookr.views import profile

   urlpatterns = [
       …
       path('accounts/profile/', profile, name='profile'),
       path("", reviews.views.index),
       path("book-search/", reviews.views.book_search, name="book_search"),
       path('', include('reviews.urls'))
   ]
   ```

Este es un buen comienzo en una página de perfil de usuario. Creamos una cuenta para Alice en la sección de operaciones CRUD usando la aplicación de administración de Django en el Capítulo 4 (*Introducción al Administrador de Django*). Cuando Alice inicia sesión y visita `http://localhost:8000/accounts/profile/`, se renderiza como se muestra en la Figura 9.5. Recuerda, si es necesario iniciar el servidor, usa el comando `python manage.py runserver`:

*Figura 9.5: Alice visita su perfil de usuario*

Con esto, hemos visto cómo podemos redirigir a un usuario a su página de perfil una vez que haya iniciado sesión correctamente. Ahora, analicemos cómo podemos dar acceso al contenido solo a usuarios específicos.

---

### Sección: Decoradores de autenticación y redirección

Ahora que hemos aprendido a permitir que usuarios comunes inicien sesión en nuestro proyecto, podemos descubrir cómo restringir el contenido a usuarios autenticados. El módulo de autenticación viene con algunos decoradores útiles que se pueden usar para proteger las vistas de acuerdo con la autenticación o el acceso del usuario actual.

Actualmente, si Alice cerrara sesión en Bookr, la página de perfil aún se renderizaría y mostraría detalles vacíos. En lugar de que esto suceda, sería preferible que cualquier visitante no autenticado sea dirigido a la pantalla de inicio de sesión:

*Figura 9.6: Un usuario no autenticado visita un perfil de usuario*

La aplicación de autenticación viene con decoradores útiles para agregar comportamiento de autenticación a las vistas de Django. En esta situación de asegurar nuestra vista de perfil, podemos usar el decorador `login_required`:

```python
from django.contrib.auth.decorators import login_required

@login_required
def profile(request):
    …
```

Ahora, si un usuario no autenticado visita la URL `/accounts/profile`, será redirigido a `http://localhost:8000/accounts/login/?next=/accounts/profile/`.

Esta URL lleva al usuario a la URL de inicio de sesión. El parámetro `next` en las variables GET le dice a la vista de inicio de sesión a dónde redirigir después de un inicio de sesión exitoso. El comportamiento predeterminado es redirigir a la vista actual, pero esto se puede anular especificando el argumento `login_url` en el decorador `login_required`. Por ejemplo, si tuviéramos alguna necesidad de redirigir a una página diferente después del inicio de sesión, podríamos haberlo indicado explícitamente en la llamada del decorador de esta manera:

```python
@login_required(login_url='/accounts/profile2')
```

Si hubiéramos reescrito nuestra vista de inicio de sesión para esperar que la URL de redirección se especificara en un argumento de URL diferente a `'next'`, podríamos explicarlo en la llamada del decorador con el argumento `redirect_field_name`:

```python
@login_required(redirect_field_name='redirect_to')
```

A menudo hay situaciones en las que una URL debe restringirse a usuarios o grupos que cumplan una condición específica. Considera el caso en el que tenemos una página para que los usuarios del personal vean cualquier perfil de usuario. No queremos que esta URL sea accesible para todos los usuarios, por lo que queremos limitar esta URL a usuarios o grupos con el permiso `'view_user'` y reenviar las solicitudes no autorizadas a la URL de inicio de sesión:

```python
from django.contrib.auth.decorators import (login_required, permission_required)
…

@permission_required('view_group')
def user_profile(request, uid):
    user = get_object_or_404(User, id=uid)
    permissions = user.get_all_permissions()
    return render(request, 'user_profile.html', {'user': user, 'permissions': permissions})
```

Entonces, con este decorador aplicado a nuestra vista `user_profile`, un usuario no autorizado que visite `http://localhost:8000/accounts/users/123/profile/` sería redirigido a `http://localhost:8000/accounts/login/?next=/accounts/users/123/profile/`.

Desde Django 5.1, está disponible un componente de middleware adicional llamado `LoginRequiredMiddleware`. Esto se puede activar agregando `"django.contrib.auth.middleware.LoginRequiredMiddleware"` a la lista `MIDDLEWARE` en el archivo `settings.py` de un proyecto. Esto reforzará el requisito de inicio de sesión de forma predeterminada para las vistas en el proyecto. Para permitir que usuarios no validados llamen a una vista, se deberá emplear el decorador `@login_not_required`.

A veces, sin embargo, necesitamos estructurar permisos condicionales más sutiles que no entren en el ámbito de estos dos decoradores. Para este propósito, Django proporciona un decorador personalizado que toma una función arbitraria como argumento. El decorador `user_passes_test` requiere un argumento `test_func`:

```python
user_passes_test(test_func, login_url=None, redirect_field_name='next')
```

He aquí un ejemplo en el que tenemos una vista, `veteran_features`, que solo está disponible para los usuarios que han estado registrados en el sitio durante más de un año:

```python
from django.contrib.auth.decorators import (login_required, permission_required, user_passes_test)
from django.utils import timezone
import datetime
…

def veteran_user(user):
    now = timezone.now()
    if user.date_joined is None:
        return False
    return now - user.date_joined > datetime.timedelta(days=365)

@user_passes_test(veteran_user)
def veteran_features(request):
    user = request.user
    permissions = user.get_all_permissions()
    return render(request, 'veteran_profile.html', {'user': user, 'permissions': permissions})
```

A veces, la lógica en nuestras vistas no se puede manejar con uno de estos decoradores y necesitamos aplicar la redirección dentro del flujo de control de la vista. Podemos hacer esto usando la función auxiliar `redirect_to_login`. Toma los mismos argumentos que los decoradores, como se muestra en el siguiente fragmento:

```python
redirect_to_login(next, login_url=None, redirect_field_name='next')
```

Consolidaremos nuestro conocimiento de los decoradores de autenticación aplicándolos al proyecto `bookr` en el siguiente ejercicio.

#### Ejercicio 9.03 – Adición de decoradores de autenticación a las vistas

Habiendo aprendido sobre la flexibilidad de los decoradores de autenticación y permisos de la aplicación de autenticación, ahora nos dispondremos a utilizarlos en la aplicación Reviews. Debemos asegurarnos de que solo los usuarios autenticados puedan editar reseñas y que solo los usuarios del personal puedan editar editoriales. Hay varias formas de hacer esto, por lo que intentaremos algunos enfoques. Todo el código de estos pasos se encuentra en el archivo `reviews/views.py`:

1. Tu primer instinto para resolver este problema sería pensar que el método `publisher_edit` necesita un decorador apropiado para hacer cumplir que el usuario tenga el permiso `edit_publisher`. Para esto, podrías hacer fácilmente algo como esto:
   ```python
   from django.contrib.auth.decorators import permission_required
   …

   @permission_required('edit_publisher')
   def publisher_edit(request, pk=None):
       …
   ```
2. Usar este método está bien y es una forma de agregar verificación de permisos a una vista. También puedes utilizar un método ligeramente más complicado pero más flexible. En lugar de utilizar un decorador de permisos para hacer cumplir los derechos de permisos en el método `publisher_edit`, puedes crear una función de prueba que requiera un usuario del personal (*staff*) y aplicar esta función de prueba a `publisher_edit` con el decorador `user_passes_test`. Escribir una función de prueba permite una mayor personalización sobre cómo validas los derechos de acceso o permisos de los usuarios. Si realizaste cambios en tu archivo `views.py` en el Paso 1, no dudes en comentar el decorador (o eliminarlo) y escribir la siguiente función de prueba en su lugar:
   ```python
   from django.contrib.auth.decorators import user_passes_test
   …

   def is_staff_user(user):
       return user.is_staff

   @user_passes_test(is_staff_user)
   def publisher_edit(request, pk=None):
       …
   ```
3. Asegúrate de que se requiera iniciar sesión para las funciones `review_edit` y `book_media` agregando el decorador apropiado:
   ```python
   …
   from django.contrib.auth.decorators import (
       login_required, user_passes_test)
   …

   @login_required
   def review_edit(request, book_pk, review_pk=None):
       …

   @login_required
   def book_media(request, pk):
       …
   ```
4. En el método `review_edit`, agrega lógica a la vista que requiera que el usuario sea un usuario del personal o el propietario de la reseña. La vista `review_edit` controla el comportamiento tanto de la creación de reseñas como de las actualizaciones de reseñas. La restricción que estamos desarrollando solo se aplica al caso en el que se actualiza una reseña existente. Por lo tanto, el lugar para agregar código es después de que se haya recuperado con éxito un objeto `Review`. Si el usuario no es una cuenta del personal o el creador de la reseña no coincide con el usuario actual, debemos generar un error `PermissionDenied`:
   ```python
   …
   from django.core.exceptions import PermissionDenied
   from PIL import Image
   from django.contrib import messages
   …

   @login_required
   def review_edit(request, book_pk, review_pk=None):
       book = get_object_or_404(Book, pk=book_pk)
       if review_pk is not None:
           review = get_object_or_404(Review, book_id=book_pk, pk=review_pk)
           user = request.user
           if (not user.is_staff and review.creator.id != user.id):
               raise PermissionDenied
       else:
           review = None
       …
   ```
5. Ahora, cuando un usuario que no es del personal intente editar la reseña de otro usuario, se generará un error Forbidden (Prohibido), como en la Figura 9.7. En la siguiente sección, veremos cómo aplicar la lógica condicional en las plantillas para que los usuarios no sean llevados a páginas a las que no tienen permiso suficiente para acceder:
   *Figura 9.7: El acceso está prohibido para usuarios que no pertenecen al personal*

En este ejercicio, utilizamos decoradores de autenticación para proteger las vistas en una aplicación Django. Los decoradores de autenticación que se aplicaron proporcionaron un mecanismo simple para restringir las vistas a usuarios que carecen de los permisos necesarios, usuarios que no son del personal y usuarios no autenticados. Los decoradores de autenticación de Django proporcionan un mecanismo robusto que sigue el marco de roles y permisos de Django, mientras que el decorador `user_passes_test` proporciona una opción para desarrollar autenticación personalizada.

Ahora que hemos utilizado decoradores para controlar el flujo de autenticación de la aplicación, podemos personalizar las plantillas con datos de autenticación de usuario.

---

### Sección: Mejora de plantillas con datos de autenticación

En el Ejercicio 9.02 – Adición de una página de perfil, vimos que podemos pasar el objeto `request.user` a la plantilla para renderizar los atributos del usuario actual en el HTML. También podemos adoptar el enfoque de proporcionar diferentes representaciones de plantillas según el tipo de usuario o los permisos que posea un usuario. Considera que queremos agregar un enlace de edición que solo aparezca para los usuarios del personal. Podríamos aplicar una condición `if` para lograr esto:

```django
{% if user.is_staff %}
    <p><a href="{% url 'review:edit' %}">Edit this Review</a> </p>
{% endif %}
```

Si no nos tomáramos el tiempo para renderizar enlaces condicionalmente según los permisos, los usuarios tendrían una experiencia frustrante al navegar por la aplicación, ya que muchos de los enlaces en los que hacen clic conducirían a páginas 403 Forbidden. El siguiente ejercicio mostrará cómo podemos usar plantillas y autenticación para presentar enlaces contextualmente apropiados en nuestro proyecto.

#### Ejercicio 9.04 – Alternar enlaces de inicio y cierre de sesión en la plantilla base

En la plantilla base del proyecto `bookr`, ubicada en `templates/base.html`, tenemos un enlace de marcador de posición para cerrar sesión en el encabezado. Está codificado en HTML de la siguiente manera:

```html
<li class="nav-item">
    <a class="nav-link" href="#">Logout</a>
</li>
```

No queremos que aparezca el enlace de cierre de sesión después de que un usuario haya cerrado sesión. Por lo tanto, este ejercicio tiene como objetivo aplicar lógica condicional a la plantilla para que los enlaces de inicio y cierre de sesión se alternen, según si el usuario está autenticado:

1. Edita el archivo `templates/base.html`. Copia la estructura del elemento de lista Logout y crea un elemento de lista Login. Luego, reemplaza los enlaces de marcador de posición con las URLs correctas para las páginas de Logout y Login: `/accounts/logout` y `/accounts/login`, respectivamente, de la siguiente manera:
   ```html
   <li class="nav-item">
       <a class="nav-link" href="/accounts/logout">Logout </a>
   </li>
   <li class="nav-item">
       <a class="nav-link" href="/accounts/login">Login</a>
   </li>
   ```
2. Ahora, coloca nuestros dos elementos `li` dentro de un bloque condicional `if … else … endif`. La condición lógica que estamos aplicando es `if user.is_authenticated`:
   ```django
   {% if user.is_authenticated %}
       <li class="nav-item">
           <a class="nav-link" href="/accounts/logout">Logout </a>
       </li>
   {% else %}
       <li class="nav-item">
           <a class="nav-link" href="/accounts/login">Login </a>
       </li>
   {% endif %}
   ```
3. Ahora, visita la página de perfil de usuario en `http://localhost:8000/accounts/profile/`. Cuando estés autenticado, verás el enlace Logout:
   *Figura 9.8: Un usuario autenticado ve el enlace Logout*
4. Ahora, haz clic en el enlace Logout; serás llevado a la página `/accounts/logout`. El enlace Login aparece en el menú, confirmando que el enlace depende contextualmente del estado de autenticación del usuario:
   *Figura 9.9: Un usuario no autenticado ve el enlace Login*

Las solicitudes a `/accounts/logout` requieren una solicitud POST desde Django 5.0. Por lo tanto, verás que en la plantilla de ejemplo en GitHub, hay un formulario con un botón para manejar esto.

Este ejercicio fue un ejemplo simple de cómo se pueden usar las plantillas de Django con información de autenticación para crear una experiencia de usuario contextual y con estado. Tampoco queremos proporcionar enlaces a los que un usuario no tenga acceso o acciones que no estén permitidas para el nivel de permiso del usuario. La siguiente actividad utilizará esta técnica de plantillas para solucionar algunos de estos problemas en Bookr.

---

### Sección: Actividad 9.01 – Contenido basado en autenticación mediante bloques condicionales en plantillas

En esta actividad, aplicarás bloques condicionales a plantillas que modifican el contenido según la autenticación del usuario y el estado del usuario. A los usuarios no se les deben presentar enlaces que no tengan permiso para visitar o acciones que no estén autorizados a llevar a cabo. Los siguientes pasos te ayudarán a completar esta actividad:

1. En la plantilla `book_detail`, en el archivo ubicado en `reviews/templates/reviews/book_detail.html`, oculta los botones **Add Review** y **Media** para los usuarios no autenticados.
2. Además, oculta el encabezado que dice *Be the first one to write a review*, ya que no es una opción para los usuarios no autenticados.
3. En la misma plantilla, haz que el enlace **Edit Review** solo aparezca para el personal o el usuario que escribió la reseña. La lógica condicional para el bloque de plantilla es muy similar a la lógica condicional que usamos en la vista `review_edit` en la sección anterior:
   *Figura 9.10: El enlace Edit Review aparece en la reseña de Alice cuando Alice ha iniciado sesión*
   Cuando un usuario diferente inicia sesión, el enlace **Edit Review** no estará presente:
   *Figura 9.11: No hay enlace Edit Review en la reseña de Alice cuando Bob ha iniciado sesión*
4. Modifica `templates/base.html` para que muestre el nombre de usuario del usuario autenticado actualmente a la derecha del formulario de búsqueda en el encabezado, vinculándolo a la página de perfil del usuario.

Al completar esta actividad, habrás agregado contenido dinámico a la plantilla que refleja el estado de autenticación y la identidad del usuario actual, como se puede ver en la siguiente captura de pantalla:

*Figura 9.12: El nombre de un usuario autenticado aparece después del formulario de búsqueda*

Con esto, hemos revisado los mecanismos de autenticación disponibles en Django y podemos examinar cómo implementa Django las sesiones.

---

### Sección: Sesiones

Vale la pena revisar algo de teoría para comprender por qué las sesiones son una solución común en las aplicaciones web para administrar el contenido del usuario. El protocolo HTTP define las interacciones entre un cliente y un servidor. Se dice que es un protocolo "sin estado" (*stateless*) ya que el servidor no retiene información de estado entre solicitudes. Este diseño de protocolo funcionó bien para entregar información hipertextual en los primeros días de la World Wide Web, pero no se adaptaba a las necesidades de aplicaciones web seguras que entregan información personalizada a usuarios específicos.

Ahora estamos acostumbrados a ver cómo los sitios web se adaptan a nuestros hábitos de navegación. Los sitios de compras recomiendan productos similares a los que hemos visto recientemente y nos informan sobre productos populares en nuestra región. Todas estas funciones requerían un enfoque con estado (*stateful*) para el desarrollo de sitios web. Una de las formas más comunes de implementar una experiencia web con estado es a través de sesiones. Una sesión se refiere a la interacción actual de un usuario con un servidor o aplicación web y requiere que los datos persistan durante la duración de la interacción. Esto puede incluir información sobre los enlaces que el usuario ha visitado, las acciones que ha realizado y las preferencias que ha tomado en sus interacciones.

Si un usuario configura un sitio de blogs con un tema oscuro en una página, se espera que la siguiente página también use el mismo tema. Describimos este comportamiento como "mantener el estado". Una clave de sesión se almacena en el lado del cliente como una cookie del navegador, que se puede identificar con información del lado del servidor que persiste mientras el usuario ha iniciado sesión.

En Django, las sesiones se implementan como una forma de middleware. Cuando creamos inicialmente la aplicación en el Capítulo 4 (*Introducción al Administrador de Django*), el soporte de sesiones se activó de forma predeterminada.

#### El motor de sesiones

La información sobre las sesiones actuales y caducadas debe almacenarse en algún lugar. En los primeros días de la World Wide Web, esto se hacía guardando la información de la sesión en archivos en el servidor, pero a medida que las arquitecturas de los servidores web se volvieron más complejas y sus demandas de rendimiento aumentaron, otras estrategias más eficientes, como una base de datos o almacenamiento en memoria, se han convertido en la norma. De forma predeterminada, en Django, la información de la sesión se almacena en la base de datos de un proyecto.

Este es un valor predeterminado razonable para la mayoría de los proyectos pequeños. Sin embargo, la implementación de sesiones basada en middleware de Django nos brinda la flexibilidad de almacenar la información de sesión de nuestro proyecto de varias maneras para adaptarnos a la arquitectura de nuestro sistema y a los requisitos de rendimiento. Cada una de estas diferentes implementaciones se denomina **motor de sesiones** (*session engine*). Si queremos cambiar la configuración de la sesión, debemos especificar la configuración `SESSION_ENGINE` en el archivo `settings.py` del proyecto:
- **Sesiones en caché** (*Cached sessions*): En algunos entornos, almacenar en caché la información de la sesión en la memoria o en una base de datos es un enfoque adecuado para un alto rendimiento. Django proporciona los motores de sesión `django.contrib.sessions.backends.cache` y `django.contrib.sessions.backends.cached_db` para este propósito.
- **Sesiones basadas en archivos** (*File-based sessions*): Como se indicó anteriormente, esta es una forma algo anticuada de mantener la información de la sesión, pero puede ser adecuada para algunos sitios donde el rendimiento no es un problema y existen razones para no almacenar información dinámica en una base de datos.
- **Sesiones basadas en cookies** (*Cookie-based sessions*): En lugar de mantener la información de la sesión en el lado del servidor, puedes mantenerla completamente en el cliente del navegador web serializando el contenido de la sesión como JSON y almacenándolo en una cookie basada en el navegador.

#### ¿Necesitas advertir sobre el contenido de las cookies?

Todas las implementaciones de sesiones de Django requieren almacenar un ID de sesión en una cookie en el navegador web del usuario.

Independientemente del motor de sesiones empleado, todas estas implementaciones de middleware implican almacenar una cookie específica del sitio en el navegador web. En los primeros días del desarrollo web, no era raro pasar IDs de sesión como argumentos de URL, pero este enfoque se ha evitado en Django por razones de seguridad.

En muchas jurisdicciones, incluida la Unión Europea, los sitios web están obligados por ley a advertir a los usuarios si el sitio establece cookies en sus navegadores. Si existen tales requisitos legislativos en la región donde pretendes operar tu sitio, es tu responsabilidad asegurarte de que el código cumpla con estas obligaciones. Asegúrate de utilizar implementaciones actualizadas y evita el uso de proyectos abandonados que no hayan seguido el ritmo de los cambios legislativos.

Para atender estos cambios y requisitos legislativos, existen muchas aplicaciones útiles, como *Django Cookie Consent* y *Django Cookie Law*, que están diseñadas para funcionar con varios marcos legislativos. Puedes obtener más información visitando los siguientes enlaces:
- [https://github.com/jazzband/django-cookie-consent](https://github.com/jazzband/django-cookie-consent)
- [https://github.com/TyMaszWeb/django-cookie-law](https://github.com/TyMaszWeb/django-cookie-law)

Existen muchos módulos de JavaScript que implementan mecanismos de consentimiento de cookies similares.

Con esto en mente, podemos examinar cómo se implementan los datos de sesión en Django.

#### ¿Almacenamiento Pickle o JSON?

Python proporciona el módulo `pickle` en su biblioteca estándar para serializar objetos de Python en una representación de flujo de bytes. Un pickle es una estructura binaria que tiene la ventaja de ser interoperable entre diferentes arquitecturas y diferentes versiones de Python, de modo que un objeto de Python se puede serializar en un pickle en una PC con Windows y deserializar en un objeto de Python en una Raspberry Pi con Linux.

Esta flexibilidad conlleva vulnerabilidades de seguridad y no se recomienda su uso para representar datos no confiables. Considera el siguiente objeto de Python, que contiene varios tipos de datos. Se puede serializar usando `pickle`:

```python
import datetime

data = dict(
    viewed_books=[17, 18, 3, 2, 1],
    search_history=['1981', 'Machine Learning', 'Bronte'],
    background_rgb=(96, 91, 92),
    foreground_rgb=(17, 17, 17),
    last_login_login=datetime.datetime(2019, 12, 3, 15, 30, 30),
    password_change=datetime.datetime(2019, 9, 2, 8, 41, 25),
    user_class='Veteran',
    average_rating=4.75,
    reviewed_books={18, 3, 7}
)
```

Usando el método `dumps` (*dump string*) del módulo `pickle`, podemos serializar el objeto `data` para producir una representación en bytes:

```python
import pickle

data_pickle = pickle.dumps(data)
```

JSON significa *JavaScript Object Notation* (Notación de Objetos de JavaScript). La sintaxis de JSON es un pequeño subconjunto del lenguaje JavaScript. Es un estándar generalizado para mensajería e intercambio de datos, comúnmente utilizado para transferir datos entre navegadores web y servidores. Podemos serializar JSON con un enfoque similar al que describimos con el formato `pickle`:

```python
import json

data_json = json.dumps(data)
```

Debido a que `data` contiene objetos `datetime` y `set` de Python, que no son serializables con JSON, cuando intentamos serializar la estructura, se generará un error de tipo:

```text
TypeError: Object of type datetime is not JSON serializable
```

Para serializar a JSON, podríamos convertir los objetos `datetime` al tipo de cadena y colocar los conjuntos en una lista:

```python
data['last_login_login'] = \
    data['last_login_login'].strftime("%Y%d%m%H%M%S")
data['password_change'] = \
    data['password_change'].strftime("%Y%d%m%H%M%S")
data['reviewed_books'] = list(data['reviewed_books'])
```

Como los datos JSON son legibles por humanos, es fácil examinarlos:

```json
{"viewed_books": [17, 18, 3, 2, 1], "search_history": ["1981", "Machine Learning", "Bronte"], "background_rgb": [96, 91, 92], "foreground_rgb": [17, 17, 17], "last_login_login": "20190312153030", "password_change": "20190209084125", "user_class": "Veteran", "average_rating": 4.75, "reviewed_books": [18, 3, 7]}
```

Ten en cuenta que tuvimos que convertir explícitamente los objetos `datetime` y `set`, pero JSON convierte automáticamente la tupla en una lista. Django se envía con `PickleSerializer` y `JSONSerializer`. Si es necesario modificar el serializador, se puede cambiar configurando la variable `SESSION_SERIALIZER` en el archivo `settings.py` del proyecto:

```python
SESSION_SERIALIZER = 'django.contrib.sessions.serializers.JSONSerializer'
```

#### Ejercicio 9.05 – Examen de la clave de sesión

El propósito de este ejercicio es consultar la base de datos SQLite del proyecto y realizar consultas en la tabla de sesiones para familiarizarse con la forma en que se almacenan los datos de la sesión. Luego crearás un comando de administración de Django para examinar los datos de sesión que se almacenan usando `JSONSerializer`:

1. Inicia la aplicación DB Browser for SQLite y abre tu base de datos `db.sqlite3`:
   *Figura 9.13: Selección de tablas en DB Browser for SQLite*
2. En la pestaña Database Structure, expande la tabla `django_session`:
   *Figura 9.14: Definiciones de tablas en DB Browser for SQLite*
   Esto revela que la tabla `django_session` en la base de datos almacena información de sesión en los campos `session_key`, `session_data` y `expire_date`.
3. Haz clic en la pestaña Browse Data y selecciona la tabla `django_session`:
   *Figura 9.15: Consulta de datos en la tabla django_session*
   Django almacena los datos de la sesión mediante valores firmados criptográficamente. Dado que, de forma predeterminada, usamos la base de datos para almacenar nuestras sesiones, importaremos el modelo `Session`, así como la clase `SessionStore` de `django.contrib.sessions.backends.db`. `SessionStore` tiene un método `decode` que podemos usar para leer el `session_data` codificado del objeto `Session`.
4. Este fragmento de código se puede ejecutar usando el shell de Django ingresando `python manage.py shell` en la línea de comandos para obtener un indicador interactivo:
   ```python
   >>> from django.contrib.sessions.backends.db import SessionStore
   >>> from django.contrib.sessions.models import Session
   >>> session_store = SessionStore()
   >>> sessions = Session.objects.all()
   >>> session_store.decode(sessions[0].session_data)
   {'_auth_user_id': '2', '_auth_user_backend': 'django.contrib.auth.backends.ModelBackend', '_auth_user_hash': 'eb5de6bf54460bafc9e0f555c65d16b17b3fdf754617d1d73529d9605fea2f0c'}
   ```
   Usando este enfoque, podemos crear un comando de administración para examinar los datos de la sesión. Es posible que se almacenen datos confidenciales en las sesiones, por lo que la creación de un comando de administración garantiza que los datos solo sean accesibles para un usuario con acceso de consola al servidor que aloja Django.
5. Con PyCharm, en el directorio `reviews/management/commands` del proyecto `bookr`, crea un archivo de Python llamado `sessioninfo.py`.
6. Comencemos importando los módulos necesarios. Usaremos el comando `pformat` del módulo `pprint` de la biblioteca estándar de Python para formatear los datos de la sesión. Necesitaremos el modelo `User` de `django.contrib.auth.models` y el modelo `Session` de `django.contrib.sessions.models` para consultar objetos `User` y `Session`. La clase `BaseCommand` de `django.core.management.base` proporciona la estructura para nuestro comando de administración de Django:
   ```python
   from pprint import pformat
   from django.contrib.auth.models import User
   from django.contrib.sessions.models import Session
   from django.contrib.sessions.backends.db import SessionStore
   from django.core.management.base import BaseCommand
   ```
7. Como mínimo, se debe definir un comando de usuario subclasificando `BaseCommand` como la clase `Command` y definiendo un método `handle`. Por lo general, los parámetros adicionales de la línea de comandos se definirían en un método `add_arguments`, pero nuestro objetivo es crear un ejemplo muy simple. Se utiliza un atributo `help` de la clase `Command` para proporcionar información cuando el usuario ingresa `python manage.py --help` o `python manage.py sessioninfo --help`:
   ```python
   class Command(BaseCommand):
       help = "List all user sessions and the data that they contain."
   ```
8. Escribe un método `handle` de la clase `Command`. Primero, instancia un objeto `SessionStore`, luego itera a través de los objetos `Session` y decodifica los datos de cada uno usando el método `SessionStore.decode`. La clave `_auth_user_id` de los datos descifrados hace referencia a un `User.id` de la base de datos para que podamos recuperar el usuario usando el modelo `User`. Ahora, para cada sesión, escribe la clave de sesión, el ID de usuario, el nombre de usuario y la dirección de correo electrónico en la consola. Desarrollaremos una utilidad simple de Python para descifrar esta información de sesión. Ten en cuenta que es una práctica recomendada llamar a `self.stdout.write` en un comando de administración en lugar de `print` al escribir en la consola:
   ```python
   def handle(self, *args, **options):
       session_store = SessionStore()
       for session in Session.objects.all():
           data = session_store.decode(session.session_data)
           user = User.objects.get(id=data['_auth_user_id'])
           self.stdout.write(
               f"Session Key: {session.session_key} "
               f"User: {user.id} {user.username} {user.email} "
           )
           self.stdout.write(pformat(data))
   ```
9. Ahora, puedes usar este comando de administración para examinar los datos de sesión que se almacenan en la base de datos. Puedes llamarlo en la línea de comandos de la siguiente manera:
   ```bash
   python manage.py sessioninfo
   ```
   Esto será útil para depurar el comportamiento de la sesión cuando intentes la actividad final:
   *Figura 9.16: Script de Python*
   Este script genera la información de sesión decodificada. Actualmente, la sesión solo contiene tres claves:
   - `_auth_user_backend`: Es una representación de cadena de la clase del backend del usuario. Como nuestro proyecto almacena las credenciales del usuario en el modelo, se utiliza `ModelBackend`.
   - `_auth_user_hash`: Es un hash de la contraseña del usuario.
   - `_auth_user_id`: Es el ID de usuario obtenido del atributo `User.id` del modelo.

Este ejercicio te ayudó a familiarizarte con cómo se almacenan los datos de sesión en Django. Ahora nos centraremos en agregar información adicional a las sesiones de Django.

#### Almacenar datos en sesiones

Ya hemos cubierto la forma en que se implementan las sesiones en Django. Ahora, examinaremos brevemente algunas de las formas en que podemos hacer uso de las sesiones para enriquecer nuestra experiencia de usuario. En Django, `session` es un atributo del objeto `request`. Se implementa como un objeto similar a un diccionario. En nuestras vistas, podemos asignar claves al objeto `session` como en un diccionario típico, como se muestra aquí:

```python
request.session['books_reviewed_count'] = 39
```

Pero existen algunas restricciones. En primer lugar, las claves de la sesión deben ser cadenas, por lo que no se permiten enteros ni marcas de tiempo. En segundo lugar, las claves que comienzan con un guión bajo están reservadas para uso interno del sistema. Los datos se limitan a valores que se pueden codificar como JSON, por lo que algunas secuencias de bytes que no se pueden decodificar como UTF-8 no se pueden almacenar como datos JSON. La otra advertencia es evitar reasignar `request.session` a un valor diferente. Solo debemos asignar o eliminar claves. Por lo tanto, no hagas esto:

```python
# No hagas esto
request.session = {'books_read_count': 30, 'books_reviewed_count': 39}
```

En su lugar, haz esto:

```python
request.session['books_read_count'] = 30
request.session['books_reviewed_count'] = 39
```

Con esas restricciones en mente, investigaremos cómo podemos hacer uso de los datos de sesión en nuestra aplicación Reviews.

#### Ejercicio 9.06 – Almacenamiento de libros vistos recientemente en sesiones

El propósito de este ejercicio es utilizar la sesión para conservar información sobre los 10 libros explorados más recientemente por el usuario autenticado. Esta información se mostrará en la página de perfil del proyecto `bookr`. Cuando se explora un libro, se llama a la vista `book_detail`. En este ejercicio, editaremos `reviews/views.py` y agregaremos lógica adicional al método `book_detail`. Agregaremos una clave a la sesión llamada `viewed_books`. Usando conocimientos básicos de HTML y CSS, la página se puede crear para mostrar los detalles del perfil y los libros vistos almacenados en divisiones separadas de la página, de la siguiente manera:

*Figura 9.17: La página de perfil que incorpora libros vistos*

1. Edita `reviews/views.py` y el método `book_detail`. Solo nos interesa agregar información de sesión para usuarios autenticados, así que agrega una declaración condicional para verificar si el usuario está autenticado y establece `max_viewed_books_length`, la longitud máxima de la lista de libros vistos, en 10:
   ```python
   def book_detail(request, pk):
       …
       if request.user.is_authenticated:
           max_viewed_books_length = 10
   ```
2. Dentro del mismo bloque condicional, agrega código para recuperar el valor actual de `request.session['viewed_books']`. Si esta clave no está presente en la sesión, comienza con una lista vacía:
   ```python
   viewed_books = request.session.get('viewed_books', [])
   ```
3. Si la clave primaria del libro actual ya está presente en `viewed_books`, el siguiente código la eliminará:
   ```python
   viewed_book = [book.id, book.title]
   if viewed_book in viewed_books:
       viewed_books.pop(viewed_books.index(viewed_book))
   ```
4. El siguiente código inserta la clave primaria del libro actual al principio de la lista `viewed_books`:
   ```python
   viewed_books.insert(0, viewed_book)
   ```
5. Agrega la siguiente clave para mantener solo los primeros 10 elementos de la lista:
   ```python
   viewed_books = viewed_books[:max_viewed_books_length]
   ```
6. El siguiente código agregará nuestros `viewed_books` nuevamente a `session['viewed_books']` para que esté disponible en solicitudes posteriores:
   ```python
   request.session['viewed_books'] = viewed_books
   ```
7. Como antes, al final de la función `book_detail`, renderiza la plantilla `reviews/book_detail.html` dados los datos de solicitud y contexto:
   ```python
   return render(request, "reviews/book_detail.html", context)
   ```
   Una vez completada, la vista `book_detail` tendrá este bloque condicional:
   ```python
   def book_detail(request, pk):
       …
       if request.user.is_authenticated:
           max_viewed_books_length = 10
           viewed_books = request.session.get('viewed_books', [])
           viewed_book = [book.id, book.title]
           if viewed_book in viewed_books:
               viewed_books.pop(viewed_books.index(viewed_book))
           viewed_books.insert(0, viewed_book)
           viewed_books = viewed_books[:max_viewed_books_length]
           request.session['viewed_books'] = viewed_books
       return render(request, "reviews/book_detail.html", context)
   ```
8. Modifica el diseño de página y el CSS de `templates/profile.html` para dar cabida a la división `viewed_books`. Como es posible que agreguemos más divisiones a esta página en el futuro, un concepto de diseño conveniente es flexbox. Agregaremos este CSS y separaremos el contenido en instancias `div` anidadas que se organizarán horizontalmente en la página. Nos referiremos a las instancias internas de `div` como instancias de `infocell` y les daremos estilo con bordes verdes y bordes redondeados:
   ```html
   <style>
       .flexrow {
           display: flex;
           border: 2px black;
       }
       .flexrow > div {
           flex: 1;
       }
       .infocell {
           border: 2px solid green;
           border-radius: 5px 25px;
           background-color: white;
           padding: 5px;
           margin: 20px 5px 5px 5px;
       }
   </style>

   <div class="flexrow">
       <div class="infocell">
           <p>Profile</p>
           …
       </div>
       <div class="infocell">
           <p>Viewed Books</p>
           …
       </div>
   </div>
   ```
9. Modifica el elemento `div` de `viewed_books` en `templates/profile.html` para que, si hay libros presentes, sus títulos se muestren y se vinculen a las páginas de detalles de libros individuales. Esto se renderizará de la siguiente manera:
   ```html
   <a href="/books/1">Advanced Deep Learning with Keras </a><br>
   ```
   Se debe mostrar un mensaje si la lista está vacía. Todo el `div`, incluida la iteración a través de `request.session.viewed_books`, se verá así:
   ```html
   <div class="infocell">
       <p>Viewed Books</p>
       <p>
           {% for book_id, book_title in request.session.viewed_books %}
               <a href="/books/{{ book_id }}">{{ book_title }} </a><br>
           {% empty %}
               No recently viewed books found.
           {% endfor %}
       </p>
   </div>
   ```
   Puedes encontrar el código completo para `templates/profile.html` en la carpeta `Chapter09` en el repositorio de GitHub de este libro.

Este ejercicio ha mejorado la página de perfil agregando una lista de libros vistos recientemente. Ahora, cuando visites el enlace de inicio de sesión en `http://127.0.0.1:8000/accounts/profile/`, verás esta página:

*Figura 9.18: Libros vistos recientemente*

Podemos usar el comando de administración `sessioninfo` que desarrollamos en el Ejercicio 9.05 – Examen de la clave de sesión, para examinar la sesión del usuario una vez que se implemente esta característica. Se puede llamar en la línea de comandos pasando los datos de la sesión como argumento:

```bash
python manage.py sessioninfo
```

Podemos ver que los IDs y títulos de los libros se enumeran en la clave `viewed_books`. Recuerda que los datos codificados se obtienen consultando la tabla `django_session` en la base de datos SQLite:

*Figura 9.19: Los libros vistos se almacenan en los datos de la sesión*

En este ejercicio, utilizamos el mecanismo de sesión de Django para almacenar información efímera sobre las interacciones del usuario con el proyecto de Django. Hemos aprendido cómo se puede recuperar esta información de la sesión del usuario y mostrarla en una vista que informa a los usuarios sobre su actividad reciente.

#### Ejercicio 9.07 – Uso del almacenamiento de sesiones para la página de búsqueda de libros

Las sesiones son una forma útil de almacenar información de corta duración que ayuda a mantener una experiencia con estado en un sitio. Los usuarios vuelven a visitar con frecuencia páginas como formularios de búsqueda, y sería conveniente almacenar la configuración del formulario utilizada más recientemente cuando regresen a esas páginas. En el Capítulo 3 (*Vistas, Configuración de URLs y Plantillas*), desarrollamos una función de búsqueda de libros para el proyecto `bookr`. La página de búsqueda de libros tiene dos opciones para *Search in*: *Title* y *Contributor*. Actualmente, cada vez que se visita la página, el valor predeterminado es *Title*:

*Figura 9.20: Los campos Search y Search in del formulario de búsqueda de libros*

En esta actividad, utilizarás el almacenamiento de sesiones para que cuando se visite la página de búsqueda de libros, `/book-search`, se establezca de forma predeterminada la opción de búsqueda utilizada más recientemente. También agregarás una tercera `infocell` a la página de perfil que contenga una lista de enlaces a los términos de búsqueda utilizados más recientemente. Sigue estos pasos para completar esta actividad:

1. Edita la vista `book_search` y recupera `search_history` de la sesión.
2. Cuando el formulario haya recibido una entrada válida y un usuario haya iniciado sesión, agrega la opción de búsqueda y el texto de búsqueda a la lista del historial de búsqueda de la sesión. Si el formulario no se ha completado (por ejemplo, cuando se visita la página por primera vez), renderiza el formulario con la opción *Search in* utilizada anteriormente seleccionada, es decir, *Title* o *Contributor* (Figura 9.21):
   *Figura 9.21: Selección de Contributor en el campo Search*
3. En la plantilla de perfil, incluye una división `infocell` adicional para *Search History*.
4. Muestra el historial de búsqueda como una serie de enlaces a la página de búsqueda de libros. Los enlaces tomarán esta forma: `/book-search?search=Python&search_in=title`.

Esta actividad te desafiará a aplicar datos de sesión para resolver un problema de usabilidad en un formulario web. Este enfoque tendrá aplicabilidad en muchas situaciones del mundo real y te dará una idea del uso de las sesiones para crear una experiencia web con estado. Después de completar esta actividad, la página de perfil contendrá la tercera `infocell`, como se muestra en la Figura 9.22:

*Figura 9.22: La página de perfil con la infocell Search History*

---

### Sección: Resumen

En este capítulo, examinamos la implementación de autenticación y sesiones de middleware de Django. También aprendimos cómo incorporar la lógica de autenticación y permisos en vistas y plantillas. Ahora, podemos establecer permisos en páginas específicas y limitar su acceso a usuarios autenticados. También examinamos cómo almacenar datos en la sesión de un usuario y renderizarlos en páginas posteriores.

Con esto, tienes las habilidades para personalizar un proyecto de Django y ofrecer una experiencia web personalizada. Puedes limitar el contenido a usuarios autenticados o privilegiados y puedes personalizar la experiencia de un usuario en función de sus interacciones anteriores.

En el próximo capítulo, volveremos a visitar la aplicación Admin y aprenderemos sobre algunas técnicas avanzadas que podemos usar para personalizar nuestro modelo de usuario y aplicar cambios detallados a la interfaz de administración para nuestros modelos.
