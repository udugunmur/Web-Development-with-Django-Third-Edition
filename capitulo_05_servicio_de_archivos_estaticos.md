# Parte 1: Primeros pasos con Django

## Capítulo 5: Servicio de Archivos Estáticos

Una aplicación web que solo utiliza lenguaje de marcado de hipertexto (*HyperText Markup Language* o HTML) plano es bastante limitada. Sin embargo, podemos mejorar el aspecto de las páginas web con hojas de estilo en cascada (*Cascading Style Sheets* o CSS) e imágenes, y agregar interacción con JavaScript. A todos estos tipos de archivos los llamamos **archivos estáticos** (*static files*), los cuales se desarrollan y luego se despliegan como parte de la aplicación. Podemos comparar esto con las respuestas dinámicas, que se generan en tiempo real cuando se realiza una solicitud. Todas las vistas que has escrito generan una respuesta dinámica al renderizar una plantilla.

No consideraremos que las plantillas sean archivos estáticos, ya que no se envían literalmente al cliente; en su lugar, se renderizan primero y se envían como parte de una respuesta dinámica.

Durante el desarrollo, los archivos estáticos se crean en la máquina del desarrollador y luego deben trasladarse al servidor web de producción. Si debes trasladar una aplicación a producción en un período de tiempo corto (por ejemplo, unas pocas horas), puede llevar mucho tiempo recopilar todos los recursos estáticos, moverlos al directorio correcto y subirlos al servidor. Al desarrollar aplicaciones web utilizando otros frameworks o lenguajes, es posible que debas colocar manualmente todos tus archivos estáticos en un directorio específico que aloje tu servidor web. Hacer cambios en la URL desde la que se sirven los archivos estáticos podría significar actualizar valores en todo tu código.

Django puede administrar los recursos estáticos por nosotros para facilitar este proceso. Proporciona herramientas para servirlos con su servidor de desarrollo durante el desarrollo. Cuando tu aplicación pasa a producción, también puede recopilar todos tus recursos y copiarlos en una carpeta para que los aloje un servidor web dedicado. Esto te permite mantener tus archivos estáticos segregados de una manera significativa durante el desarrollo y agruparlos automáticamente para el despliegue.

Esta funcionalidad la proporciona la aplicación integrada `staticfiles` de Django. Agrega varias características útiles para trabajar y servir archivos estáticos:
- La etiqueta de plantilla `static` crea automáticamente la URL estática para un recurso y la incluye en tu HTML.
- Una vista (llamada `static`) sirve archivos estáticos en desarrollo.
- Los buscadores de archivos estáticos (*static file finders*) se pueden utilizar para personalizar dónde se encuentran los recursos en tu sistema de archivos.
- El comando de gestión `collectstatic` encuentra todos los archivos estáticos y los traslada a un solo directorio para su despliegue.
- El comando de gestión `findstatic` muestra qué archivo estático del disco se ha cargado para una solicitud en particular. Esto también nos ayuda a depurar si un archivo en particular no se está cargando.

En los ejercicios y actividades de este capítulo, agregaremos archivos estáticos (imágenes y CSS) a la aplicación Bookr. Cada archivo se almacenará dentro del directorio del proyecto Bookr durante el desarrollo. Necesitaremos generar una URL para cada uno para que las plantillas puedan hacer referencia a ellos y el navegador pueda descargarlos. Una vez que se haya generado la URL, Django necesitará servir estos archivos. Cuando despleguemos la aplicación Bookr en producción, todos los archivos estáticos deben encontrarse y trasladarse a un directorio donde el servidor web de producción pueda servirlos. Si hay archivos estáticos que no se cargan como se esperaba, necesitamos un método para determinar cuál es la causa.

En este capítulo, comenzarás aprendiendo la diferencia entre respuestas estáticas y dinámicas. Luego verás cómo la aplicación `staticfiles` de Django ayuda a administrar los archivos estáticos. A medida que continúes trabajando en la aplicación Bookr, la mejorarás con imágenes y CSS. Verás las diferentes formas en que puedes estructurar tus archivos estáticos para tu proyecto y examinarás cómo Django los consolida para el despliegue en producción. Django incluye herramientas para hacer referencia a archivos estáticos en plantillas; verás cómo estas herramientas ayudan a reducir la cantidad de trabajo que debes realizar al desplegar la aplicación en producción. Después de esto, explorarás el comando `findstatic`, que se puede utilizar para depurar problemas con tus archivos estáticos. Más adelante, obtendrás una descripción general de cómo escribir código para almacenar archivos estáticos en un servicio remoto. Finalmente, verás el almacenamiento en caché de recursos web y cómo Django puede ayudar con la invalidación de la caché.

Cubriremos los siguientes temas principales en este capítulo:
- Servicio de archivos estáticos
- Introducción a los buscadores de archivos estáticos (*static files finders*)
- Generación de URLs estáticas con la etiqueta de plantilla `static`
- FileSystemFinder
- Buscadores de archivos estáticos: uso durante `collectstatic`
- Modo prefijado de `STATICFILES_DIRS`
- El comando `findstatic`
- Servicio de los archivos más recientes (para la invalidación de caché)
- Motores de almacenamiento personalizados (*custom storage engines*)

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter05` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos descritos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Django y el servicio de archivos estáticos

En la introducción, mencionamos que Django incluye una función de vista llamada `static` que sirve archivos estáticos. El primer punto importante a destacar es que Django no tiene la intención de servir archivos estáticos en producción. No es la función de Django y, en producción, esta tarea se adapta mejor al servidor web. Si Django está leyendo desde el sistema de archivos y enviando un archivo, entonces no tiene ninguna ventaja sobre un servidor web normal, que probablemente tendrá un mejor rendimiento en esta tarea. Además, si sirves archivos estáticos con Django, mantendrás ocupado el proceso de Python durante la duración de esta solicitud y no podrá atender las solicitudes dinámicas para las que es más adecuado.

Por estas razones, la vista `static` de Django está diseñada únicamente para su uso durante el desarrollo y no funcionará si la configuración de `DEBUG` es `False`. Dado que durante el desarrollo normalmente solo tenemos a una persona accediendo al sitio a la vez (el desarrollador), Django puede servir archivos estáticos sin problemas. Discutiremos cómo la aplicación `staticfiles` admite el despliegue en producción en breve. Además, todo el proceso de despliegue en producción se tratará en el Capítulo 18 (*Despliegue de Django*).

Un mapeo de URL a la vista `static` se configura automáticamente cuando se ejecuta el servidor de desarrollo de Django, siempre que tu archivo `settings.py` conste de los siguientes elementos:
- Tenga `DEBUG` establecido en `True`.
- Contenga `"django.contrib.staticfiles"` en su lista `INSTALLED_APPS`.

Ambas configuraciones se establecen de forma predeterminada en una aplicación recién creada.

El mapeo de URL que se crea es aproximadamente equivalente a tener el siguiente mapeo en tus `urlpatterns`:

```python
path(settings.STATIC_URL, django.conf.urls.static)
```

Cualquier URL que comience con `settings.STATIC_URL` (que es `/static/` de forma predeterminada) se mapea a la vista `static`.

Podrías usar la vista `static` sin tener `staticfiles` en `INSTALLED_APPS`, pero deberás configurar un mapeo de URL equivalente manualmente.

A continuación, hablaremos sobre cómo Django localiza archivos estáticos mediante los buscadores de archivos estáticos (*Static Files Finders*).

---

### Sección: Introducción a los buscadores de archivos estáticos (Static files finders)

Los buscadores de archivos estáticos pueden considerarse como un componente de complemento (*plugin*) de la arquitectura de Django. Son clases que implementan métodos para convertir rutas de URL en rutas de archivos, iterando a través del directorio del proyecto para encontrar archivos estáticos. Hay tres situaciones en las que Django necesita localizar archivos estáticos en el disco, utilizando buscadores de archivos estáticos para hacerlo:
1. La primera es cuando la vista `static` de Django recibe una solicitud para cargar un archivo estático en particular; aquí, necesita convertir la ruta en la URL en una ubicación en el disco. Por ejemplo, supongamos que la ruta de la URL es `/static/logo.png`, y se convierte en la ruta `bookr/static/logo.png` en el disco. Como señalamos en la sección anterior, esto solo ocurre durante el desarrollo. En un servidor de producción, Django no debería recibir esta solicitud, ya que el servidor web la gestionará directamente.
2. La segunda es cuando se utiliza el comando de gestión `collectstatic`. Esto recopila todos los archivos estáticos en el directorio del proyecto y los copia en un solo directorio para que los sirva el servidor web de producción. `bookr/static/logo.png` se copiará en la raíz del servidor web, por ejemplo, `/var/www/bookr/static/logo.png`. El buscador de archivos estáticos contiene código para localizar todos los archivos estáticos dentro del directorio de tu proyecto.
3. El tercer uso de los buscadores de archivos estáticos es durante la ejecución del comando de gestión `findstatic`. Esto es similar al primer uso, donde acepta el nombre de un archivo estático (como `logo.png`), pero imprime la ruta completa (`bookr/static/logo.png`) en la terminal en lugar de cargar el contenido del archivo.

Django viene con algunos buscadores integrados, pero también puedes escribir los tuyos propios si deseas almacenar archivos estáticos en un diseño de directorio personalizado. La lista de buscadores que utiliza Django está definida por la configuración `STATICFILES_FINDERS` en `settings.py`. En este capítulo, cubriremos el comportamiento de los buscadores de archivos estáticos predeterminados, `AppDirectoriesFinder` y `FileSystemFinder`, en la subsección *AppDirectoriesFinder* y en la sección *FileSystemFinder*, respectivamente.

Si buscas en `settings.py`, no verás la configuración `STATICFILES_FINDERS` definida de forma predeterminada. Esto se debe a que Django utilizará su valor predeterminado integrado para la configuración, que se define como:

```python
[
    "django.contrib.staticfiles.finders.FileSystemFinder",
    "django.contrib.staticfiles.finders.AppDirectoriesFinder"
]
```

Si agregas la configuración `STATICFILES_FINDERS` a tu archivo `settings.py` para incluir un buscador personalizado, asegúrate de incluir estos valores predeterminados si los estás utilizando.

En esta sección, analizaremos primero los buscadores de archivos estáticos y su uso en el primer caso: responder a una solicitud. Luego, presentaremos algunos conceptos más y volveremos al comportamiento de `collectstatic` y cómo utiliza los buscadores de archivos estáticos.

#### Buscadores de archivos estáticos: utilizados durante una solicitud

Cuando Django recibe una solicitud para un archivo estático (recuerda que Django solo servirá archivos estáticos durante el desarrollo), se consultará cada buscador de archivos estáticos que se haya definido hasta que se encuentre un archivo en el disco. O, si ninguno de los buscadores puede localizar un archivo, la vista `static` devolverá una respuesta HTTP `404 Not Found`.

Por ejemplo, la URL de la solicitud será algo así como `/static/main.css` o `/static/reviews/logo.png`. Se consultará a cada buscador por turno con la ruta de la URL y devolverá una ruta como `bookr/static/main.css` para el primer archivo y `bookr/reviews/static/reviews/logo.png` para el segundo.

Cada buscador utilizará su lógica para convertir de una ruta de URL a una ruta del sistema de archivos; discutiremos esta lógica en la sección *AppDirectoriesFinder* (que viene a continuación) y en la sección *FileSystemFinder* (que vendrá más adelante).

#### AppDirectoriesFinder

La clase `AppDirectoriesFinder` se utiliza para buscar archivos estáticos dentro de cada directorio de la aplicación, en un directorio llamado `static`. La aplicación debe aparecer en la configuración `INSTALLED_APPS` en tu archivo `settings.py` (hicimos esto en el Capítulo 1). Como también mencionamos en el Capítulo 1, es bueno que las aplicaciones sean autónomas. Al permitir que cada aplicación tenga un directorio `static`, podemos continuar con el diseño autónomo al almacenar también archivos estáticos específicos de la aplicación dentro del directorio de la aplicación.

Antes de usar `AppDirectoriesFinder`, explicaremos un problema que puede ocurrir si varios archivos estáticos tienen el mismo nombre y también cómo resolver este problema.

#### Espacio de nombres de archivos estáticos (Static file namespacing)

En la sección anterior, *Buscadores de archivos estáticos: utilizados durante una solicitud*, hablamos sobre servir un archivo llamado `logo.png`. Esto proporcionaría un logotipo para la aplicación `reviews`. El nombre de archivo (`logo.png`) podría ser bastante común; podrías imaginar que si agregáramos una aplicación `store` (para comprar libros), también tendría un logotipo. Sin mencionar que las aplicaciones de Django de terceros también podrían querer usar un nombre común, como `logo.png`. El problema que estamos a punto de describir podría aplicarse a cualquier archivo estático que tenga un nombre común, como `styles.css` o `main.js`.

Consideremos los ejemplos de `reviews` y `store`. Podemos agregar un directorio `static` en cada una de estas aplicaciones. Luego, cada directorio `static` tendría un archivo `logo.png` (aunque sería un logotipo diferente). La estructura del directorio se muestra en la Figura 5.1:

*Figura 5.1 – Distribución de directorios con directorios static dentro de los directorios de las aplicaciones*

La ruta de URL que usamos para descargar el archivo estático es relativa al directorio `static`. Por lo tanto, no está claro a qué `logo.png` se hace referencia si hacemos una solicitud HTTP para `/static/logo.png`. Django verificará el directorio `static` para cada aplicación por turno (en el orden en que se especifican en la configuración `INSTALLED_APPS`). El primer `logo.png` que localice, lo servirá. No hay forma, en este diseño de directorio, de especificar qué `logo.png` deseas cargar.

Podemos resolver este problema asignando un espacio de nombres (*namespacing*) a nuestros archivos estáticos. Este es el proceso de usar otro directorio dentro del directorio `static`, con el mismo nombre que la aplicación. La aplicación `reviews` tiene un directorio `reviews` dentro de su directorio `static`, y la aplicación `store` tiene un directorio `store` dentro de su directorio `static`. Luego, los archivos `logo.png` respectivos se mueven dentro de estos subdirectorios. La nueva estructura de directorios se muestra en la Figura 5.2:

*Figura 5.2 – Distribución de directorios con directorios con espacio de nombres*

Para cargar un archivo específico, también debemos incluir el directorio con espacio de nombres. Para el logotipo de `reviews`, la ruta de la URL es `/static/reviews/logo.png`, que se asigna a `bookr/reviews/static/reviews/logo.png` en el disco. De manera similar, para el logotipo de `store`, la ruta es `/static/store/logo.png`, que se asigna a `bookr/store/static/store/logo.png`. Es posible que hayas notado que las rutas de ejemplo para el archivo `logo.png` ya tenían un espacio de nombres en la sección *Buscadores de archivos estáticos: utilizados durante una solicitud*.

Ahora que hemos introducido `AppDirectoriesFinder` y el espacio de nombres de archivos estáticos, podemos usarlos para servir nuestro primer archivo estático. En el primer ejercicio de este capítulo, crearemos un nuevo proyecto de Django para un sitio comercial básico. Luego serviremos un archivo de logotipo desde una aplicación llamada `landing` que crearemos en este proyecto. La clase `AppDirectoriesFinder` se utiliza para buscar archivos estáticos dentro de cada directorio de la aplicación, en un directorio llamado `static`. La aplicación debe aparecer en la configuración `INSTALLED_APPS` en tu archivo `settings.py`.

La forma más sencilla de servir un archivo estático es desde un directorio de aplicación. Esto se debe a que no necesitamos realizar ningún cambio en la configuración. En su lugar, solo necesitamos crear los archivos en el directorio correcto y se servirán utilizando la configuración predeterminada de Django.

#### El proyecto del sitio de negocios (The business site project)

Para los ejercicios de este capítulo, crearemos un nuevo proyecto de Django y lo usaremos para demostrar los conceptos de archivos estáticos. El proyecto será un sitio comercial básico con una página de destino (*landing page*) simple que tiene un logotipo. El proyecto tendrá una aplicación llamada `landing`.

Puedes consultar el Ejercicio 1.01 del Capítulo 1 para refrescar tu memoria sobre la creación de un proyecto de Django.

#### Ejercicio 5.01 – Servicio de un archivo desde el directorio de una aplicación

En este ejercicio, agregarás un archivo de logotipo para la aplicación `landing`. Esto se hará colocando un archivo `logo.png` en un directorio `static` dentro del directorio de la aplicación `landing`. Una vez que hayas hecho esto, puedes probar que el archivo estático se esté sirviendo correctamente y confirmar la URL que lo servirá:

1. Comienza creando el nuevo proyecto de Django. Puedes reutilizar el entorno virtual de Bookr que ya tiene Django instalado.
2. Abre una nueva terminal y activa el entorno virtual.
3. Ejecuta el comando `django-admin` en la terminal para iniciar un proyecto de Django llamado `business_site`:
   ```bash
   django-admin startproject business_site
   ```
   No habrá ninguna salida. Este comando estructurará el proyecto Django en un nuevo directorio llamado `business_site`.
4. Crea una nueva aplicación Django en este proyecto mediante el comando de gestión `startapp`. La aplicación debe llamarse `landing`. Para hacer esto, entra al directorio `business_site` y luego ejecuta lo siguiente:
   ```bash
   python manage.py startapp landing
   ```
   El comando creará el directorio de la aplicación `landing` dentro del directorio `business_site`.
5. Inicia PyCharm y abre el directorio `business_site`.
   *Figura 5.3 – El proyecto business_site*
   Sigue el enlace para agregar una nueva configuración de ejecución:
   *Figura 5.4 – Run/Debug Configurations*
   Crea una nueva configuración de ejecución para ejecutar `manage.py runserver` para el proyecto:
   *Figura 5.5 – Run/Debug Configurations para runserver*
   Puedes probar que la configuración se haya establecido correctamente haciendo clic en el botón **Run** y luego visitando `http://127.0.0.1:8000/` en tu navegador. Deberías ver la pantalla de bienvenida de Django.
6. Abre `settings.py` en el directorio `business_site` y agrega `'landing'` a la configuración `INSTALLED_APPS`.
7. En PyCharm, haz clic con el botón derecho en el directorio `landing` en el panel del proyecto y selecciona **New | Directory**.
8. Ingresa `static` y presiona la tecla Enter:
   *Figura 5.6 – Asignación del nombre static al directorio*
9. Haz clic con el botón derecho en el directorio `static` que acabas de crear y selecciona **New | Directory** nuevamente.
10. Ingresa `landing` y presiona Enter. Esto implementará el espacio de nombres del directorio de archivos estáticos, como comentamos anteriormente:
    *Figura 5.7 – Asignación del nombre reviews al nuevo directorio para implementar el espacio de nombres*
11. Descarga el archivo `logo.png` de la carpeta `Chapter05` en el repositorio de GitHub de este libro y muévelo al directorio `landing/static/landing`.
12. Inicia el servidor de desarrollo de Django. Si aún no se está ejecutando, navega a `http://127.0.0.1:8000/static/landing/logo.png`. Deberías ver la imagen que se sirve en tu navegador, lo que ilustra que has configurado el servicio de archivos estáticos correctamente:
    *Figura 5.8 – Imagen servida por Django*

Ahora que has visto que la carga de imágenes estáticas funciona correctamente, aprenderemos más sobre cómo evitar codificar estas URLs de forma fija (*hardcoding*). En la siguiente sección, verás cómo Django puede generar automáticamente URLs estáticas correctas para su uso en plantillas.

---

### Sección: Generación de URLs estáticas con la etiqueta de plantilla static

En el Ejercicio 5.01, configuraste un archivo de imagen para que Django lo sirviera. Vimos que la URL de la imagen era `http://127.0.0.1:8000/static/landing/logo.png`, que podrías usar dentro de una plantilla HTML. Por ejemplo, para mostrar la imagen con una etiqueta `img`, podrías usar este código en tu plantilla:

```html
<img src="http://127.0.0.1:8000/static/landing/logo.png">
```

O, dado que Django también sirve los medios y tiene el mismo host que la respuesta de plantilla dinámica, puedes simplificar esto incluyendo solo la ruta, de la siguiente manera:

```html
<img src="/static/landing/logo.png">
```

Ambas direcciones (URLs y rutas) se han codificado de forma rígida (*hardcoded*) en la plantilla; es decir, incluimos la ruta completa al archivo estático y hacemos suposiciones sobre dónde se aloja el archivo. Esto funciona bien con el servidor de desarrollo de Django o si alojas tus archivos estáticos y el sitio web de Django en el mismo dominio. Para un mejor rendimiento a medida que tu sitio se vuelve más popular, podrías considerar servir archivos estáticos desde su propio dominio o red de entrega de contenido (*Content Delivery Network* o CDN).

Una CDN es un servicio que puede alojar partes o la totalidad de tu sitio web por ti. Proporciona varios servidores web y puede acelerar sin problemas la carga de tu sitio web. Por ejemplo, podría servir archivos a un usuario desde el servidor que esté geográficamente más cercano a él. Hay varios proveedores de CDN y, según cómo estén configurados, es posible que requieran que especifiques un determinado dominio desde el cual servir tus archivos estáticos.

Consideremos, por ejemplo, un enfoque de separación común: usar un dominio diferente para servir archivos estáticos. Alojas tu sitio web principal en `https://www.example.com` pero deseas servir archivos estáticos desde `https://static.example.com/`. Durante el desarrollo, podríamos usar solo la ruta al archivo del logotipo, como en el ejemplo que acabamos de ver. Pero cuando desplegamos en el servidor de producción, nuestras URLs deberían cambiar para que incluyan el dominio, de esta manera:

```html
<img src="https://static.example.com/landing/logo.png">
```

Dado que todos los enlaces están codificados de forma fija, esto debería hacerse para cada URL en nuestras plantillas, cada vez que despleguemos en producción. Sin embargo, una vez que se hayan cambiado, la URL ya no funcionará en el servidor de desarrollo de Django. Afortunadamente, Django ofrece una solución a este problema.

#### La etiqueta de plantilla static

La aplicación `staticfiles` proporciona una etiqueta de plantilla, `static`, para generar dinámicamente la URL a un archivo estático dentro de una plantilla. Dado que todas las URLs se generan dinámicamente, podemos cambiar la URL para todas ellas cambiando solo una configuración (`STATIC_URL` en `settings.py`). Además, más adelante, presentaremos un método para invalidar las cachés del navegador para archivos estáticos que se basa en el uso de la etiqueta de plantilla `static`.

La etiqueta `static` es muy simple: toma un solo argumento, que es la ruta relativa al proyecto a un recurso estático. Luego generará esta ruta, precedida por la configuración `STATIC_URL`. Pero primero, debe cargarse en la plantilla con la etiqueta de plantilla `{% load static %}`.

Django tiene un conjunto de etiquetas y filtros de plantilla predeterminados que automáticamente pone a disposición de cada plantilla. Django (y las librerías de terceros) también proporcionan conjuntos de etiquetas que no se cargan automáticamente. En estos casos, debemos cargar estas etiquetas y filtros de plantilla adicionales en una plantilla antes de poder usarlos. Esto se puede hacer con la etiqueta de plantilla `load`, que debe ubicarse cerca del inicio de una plantilla (aunque debe ser posterior a la etiqueta de plantilla `extends` si se utiliza una). La etiqueta de plantilla `load` toma uno o más paquetes/librerías para cargar. He aquí un ejemplo:

```django
{% load package_one package_two package_three %}
```

Esto cargaría el conjunto de etiquetas de plantilla y filtros proporcionado por los paquetes `package_one`, `package_two` y `package_three`.

La etiqueta de plantilla `load` debe utilizarse en la plantilla real que requiere el paquete cargado. Por ejemplo, si tu plantilla extiende otra plantilla y esa plantilla base ha cargado un paquete determinado, tu plantilla dependiente no tiene acceso automáticamente a ese paquete. Tu plantilla aún debe cargar el paquete para acceder al nuevo conjunto de etiquetas. La etiqueta de plantilla `static` no forma parte del conjunto predeterminado, por lo que debemos cargarla.

Luego, se puede utilizar para interpolar en cualquier lugar dentro del archivo de plantilla. Por ejemplo, de forma predeterminada, Django usa `/static/` como `STATIC_URL`. Si quisiéramos generar la URL estática para nuestro archivo `logo.png`, usaríamos la etiqueta en una plantilla como esta:

```django
{% static 'landing/logo.png' %}
```

La salida dentro de la plantilla sería esta:

```text
/static/landing/logo.png
```

#### Un ejemplo de la etiqueta de plantilla static

Esto quedará más claro con un ejemplo, así que veamos cómo se podría usar la etiqueta `static` para generar una URL para varios recursos diferentes.

Podemos incluir el logotipo como una imagen en la página con una etiqueta `img`, de la siguiente manera:

```django
<img src="{% static 'landing/logo.png' %}">
```

Esto se renderiza en la plantilla de la siguiente manera:

```html
<img src="/static/landing/logo.png">
```

Alternativamente, podríamos usar la etiqueta `static` para generar la URL de un archivo CSS vinculado, de la siguiente manera:

```django
<link href="{% static 'path/to/file.css' %}" rel="stylesheet">
```

Esto se representará así:

```html
<link href="/static/path/to/file.css" rel="stylesheet">
```

Se puede utilizar en una etiqueta `script` para incluir un archivo JavaScript, utilizando la siguiente línea de código:

```django
<script src="{% static 'path/to/file.js' %}"></script>
```

Esto se renderiza de la siguiente manera:

```html
<script src="/static/path/to/file.js"></script>
```

Incluso puedes usarlo para generar un enlace a un archivo estático para descargarlo, como hemos hecho aquí:

```django
<a href="{% static 'path/to/document.pdf' %}">Download PDF</a>
```

Ten en cuenta que esto no generará el contenido del PDF real; solo creará un enlace a un archivo ya existente. Cubriremos la generación de archivos PDF e imágenes en el Capítulo 13.
Esto se procesa de la siguiente manera:

```html
<a href="/static/path/to/document.pdf">Download PDF</a>
```

Al hacer referencia a estos ejemplos, ahora podemos demostrar la ventaja de usar la etiqueta `static` en lugar de la codificación fija. Cuando estemos listos para desplegar en producción, simplemente podemos cambiar el valor de `STATIC_URL` en `settings.py`. Ninguno de los valores de las plantillas necesita cambiarse.

Por ejemplo, podemos cambiar `STATIC_URL` a `https://static.example.com/`, y luego, cuando la página se renderice a continuación, los ejemplos que hemos visto se actualizarán automáticamente. La siguiente línea muestra esto para la imagen:

```html
<img src="https://static.example.com/landing/logo.png">
```

Lo siguiente es para el enlace CSS:

```html
<link href="https://static.example.com/path/to/files.css" rel="stylesheet">
```

Para el script, es lo siguiente:

```html
<script src="https://static.example.com/path/to/file.js"></script>
```

Y finalmente, lo siguiente es para el enlace de descarga:

```html
<a href="https://static.example.com/path/to/document.pdf">Download PDF</a>
```

Ten en cuenta que en todos estos ejemplos se pasa una cadena literal como argumento (está entre comillas). También puedes usar una variable como argumento, por ejemplo, si estuvieras renderizando una plantilla con un contexto, como en este código de ejemplo:

```python
def view_function(request):
    context = {"image_file": "logofile.png"}
    return render(request, "example.html", context)
```

Estamos renderizando la plantilla `example.html` con una variable llamada `image_file`. Esta variable tiene un valor de `logofile.png`.
Pasarías esta variable a la etiqueta `static` sin comillas:

```django
<img src="{% static image_file %}">
```

Se renderizaría así (asumiendo que cambiamos `STATIC_URL` nuevamente a `/static/`):

```html
<img src="/static/logo.png">
```

La etiqueta de plantilla también se puede usar con el sufijo `as [variable]` para asignar el resultado a una variable para su uso posterior en la plantilla. Esto puede resultar útil si la búsqueda de archivos estáticos lleva mucho tiempo y deseas hacer referencia al mismo archivo estático varias veces (como incluir una imagen en varios lugares).

La primera vez que hagas referencia a la URL estática, asígnale un nombre de variable para guardarla. En este caso, estamos creando la variable `logo_path`:

```django
<img src="{% static 'logo.png' as logo_path %}">
```

Esto se renderiza igual que los ejemplos que hemos visto antes:

```html
<img src="/static/logo.png">
```

Sin embargo, luego podemos usar la variable asignada (`logo_path`) nuevamente más adelante en la plantilla:

```django
<img src="{{ logo_path }}">
```

Esto se renderiza de la misma manera nuevamente:

```html
<img src="/static/logo.png">
```

Esta variable es ahora simplemente una variable de contexto normal en el ámbito de la plantilla y se puede utilizar en cualquier lugar de la plantilla. Sin embargo, ten cuidado, ya que podrías anular una variable que ya se haya definido (aunque esta es una advertencia general al usar cualquiera de las etiquetas de plantilla que asignan variables, por ejemplo, `{% with %}`).

En el siguiente ejercicio, pondremos en práctica la plantilla estática agregando una plantilla al proyecto `business_site` y luego incluyendo la imagen de ejemplo.

#### Ejercicio 5.02 – Uso de la etiqueta de plantilla static

En el Ejercicio 5.01, probaste servir `logo.png` desde el directorio `static`. En este ejercicio, continuarás con el proyecto del sitio comercial y crearás un archivo `index.html` como plantilla para nuestra página de destino. Luego, incluirás el logotipo en esta página usando la etiqueta de plantilla `{% static %}`:

1. En PyCharm (asegúrate de estar en el proyecto `business_site`), haz clic con el botón derecho en el directorio `landing` y crea una nueva carpeta llamada `templates`.
2. Haz clic con el botón derecho en el nuevo directorio `templates` y selecciona **New | HTML File**. Selecciona archivo HTML 5 y nómbralo `index.html`:
   *Figura 5.9 – Nuevo index.html*
3. Abre el archivo `index.html` y, primero, carga la biblioteca de etiquetas estáticas para que la etiqueta `static` esté disponible en la plantilla. Haz esto con la etiqueta de plantilla `load`. En la segunda línea del archivo (justo después de `<!DOCTYPE html>`), agrega esta línea para cargar la biblioteca estática:
   ```django
   {% load static %}
   ```
4. También puedes hacer que la plantilla sea un poco más agradable con contenido adicional. Introduce el texto `Business Site` dentro de las etiquetas `<title>`:
   ```html
   <title>Business Site</title>
   ```
5. Luego, dentro del body, agrega un elemento `<h1>` con el texto `Welcome to my Business Site`:
   ```html
   <h1>Welcome to my Business Site</h1>
   ```
6. Debajo del texto del encabezado, usa la etiqueta de plantilla `{% static %}` para establecer la fuente de una imagen `<img>`. La usarás para hacer referencia al logotipo del Ejercicio 5.01:
   ```django
   <img src="{% static 'landing/logo.png' %}">
   ```
7. Finalmente, para completar un poco el sitio, agrega un elemento `<p>` debajo de `<img>`. Dale un texto sobre el negocio:
   ```html
   <p>Welcome to the site for my Business. For all your Business needs!</p>
   ```
   Aunque el texto y el título adicionales no son demasiado importantes, nos dan una idea de cómo usar la etiqueta de plantilla `{% static %}` alrededor del resto del contenido. Guarda el archivo.
8. A continuación, configura una URL para usar y representar la plantilla. También utilizarás la clase integrada `TemplateView` para renderizar la plantilla sin tener que crear una vista. Abre `urls.py` en el directorio del paquete `business_site`. Al principio del archivo, importa `TemplateView` de la siguiente manera:
   ```python
   from django.views.generic import TemplateView
   ```
   También puedes eliminar la línea `from django.contrib import admin` ya que no la estamos usando en este proyecto:
   ```python
   from django.contrib import admin
   ```
9. Agrega un mapa de URL desde la URL raíz a `TemplateView`. El método `as_view` de `TemplateView` toma `template_name` como argumento, que se utiliza de la misma manera que una ruta que podrías pasar a la función `render`. Tus `urlpatterns` deberían verse así:
   ```python
   urlpatterns = [
       path("", TemplateView.as_view(template_name="index.html")),
   ]
   ```
10. Guarda el archivo `urls.py`. Inicia el servidor de desarrollo de Django si aún no se está ejecutando. Navega a `http://127.0.0.1:8000/` en tu navegador. Deberías ver tu nueva página de destino, como se muestra en la Figura 5.10:
    *Figura 5.10 – Mi sitio de negocios con un logotipo*

En este ejercicio, agregamos una plantilla base para `landing` y cargamos la biblioteca `static` en la plantilla. Una vez cargada la biblioteca estática, pudimos usar la etiqueta de plantilla `static` para cargar una imagen. Luego, pudimos ver el logotipo de nuestra empresa renderizado en el navegador.

Hasta ahora, la carga de archivos estáticos ha utilizado `AppDirectoriesFinder` porque no requería ninguna configuración adicional para usarlo. En la siguiente sección, veremos `FileSystemFinder`, que es más flexible pero requiere una pequeña cantidad de configuración.

---

### Sección: FileSystemFinder

Hasta ahora, hemos aprendido sobre `AppDirectoriesFinder`, que carga archivos estáticos dentro de los directorios de aplicaciones de Django. Sin embargo, esperamos que las aplicaciones bien diseñadas sean autónomas y, por lo tanto, solo deben contener archivos estáticos de los que dependen. Si tenemos otros archivos estáticos que se utilizan en todo el sitio web o en diferentes aplicaciones, debemos almacenarlos fuera del directorio de la aplicación.

Como regla general, tu CSS probablemente sea coherente en todo el sitio y podría mantenerse en un directorio global. Algunas imágenes y código JavaScript podrían ser específicos de las aplicaciones, por lo que se almacenarían en el directorio `static` de esa aplicación. Sin embargo, esto es solo un consejo general: puedes almacenar archivos estáticos en cualquier lugar que tenga más sentido para tu proyecto.

En nuestra aplicación de sitio comercial, almacenaremos un archivo CSS en un directorio estático del sitio, ya que se utilizará no solo en la aplicación `landing` sino también en todo el sitio a medida que agreguemos más aplicaciones.

Django proporciona soporte para servir archivos estáticos desde directorios arbitrarios utilizando su buscador de archivos estáticos `FileSystemFinder`. Los directorios pueden estar en cualquier lugar del disco. Por lo general, tendrás un directorio `static` dentro del directorio de tu proyecto, pero si tu empresa tiene un directorio estático global que se utiliza en muchos proyectos diferentes (incluidas aplicaciones web que no son de Django), también podrías utilizarlo.

`FileSystemFinder` utiliza la configuración `STATICFILES_DIRS` en el archivo `settings.py` para determinar en qué directorios buscar archivos estáticos. Esto no está presente cuando se crea el proyecto y debe ser configurado por el desarrollador. Hay dos opciones para construir esta lista:
1. Establecer una lista de directorios.
2. Establecer una lista de tuplas en la forma de `(prefijo, directorio)`.

En `business_site`, agregaremos un directorio `static` dentro del directorio del proyecto (es decir, en el mismo directorio que contiene la aplicación `landing` y el archivo `manage.py`). Podemos usar la variable `BASE_DIR` al construir la lista para asignar a `STATICFILES_DIRS`:

```python
STATICFILES_DIRS = [BASE_DIR / "static"]
```

También mencionamos anteriormente en esta sección que es posible que desees establecer múltiples rutas de directorio en esta lista; por ejemplo, si tuvieras algunos datos estáticos para toda la empresa compartidos por múltiples proyectos web. Simplemente agrega directorios adicionales a la lista `STATICFILES_DIRS`:

```python
STATICFILES_DIRS = [
    BASE_DIR / "static",
    "/Users/username/projects/company-static/"
]
```

Cada uno de estos directorios se comprobaría, en el orden especificado, para encontrar un archivo coincidente. Si existiera un archivo en ambos directorios, se serviría el primero que se encuentre. Por ejemplo, si existieran los archivos `static/main.css` (dentro del directorio del proyecto `business_site`) y `/Users/username/projects/company-static/bar/main.css`, una solicitud de `/static/main.css` serviría el `main.css` del proyecto `business_site` ya que es el primero en la lista.

En nuestro sitio de negocios (y más adelante con Bookr), solo usaremos un directorio estático en esta lista, por lo que no tendremos que preocuparnos por este problema.

En el siguiente ejercicio, agregaremos un directorio `static` con un archivo CSS adentro. Luego, configuraremos la opción `STATICFILES_DIRS` para que se pueda servir desde el directorio estático.

#### Ejercicio 5.03 – Servicio desde un directorio estático del proyecto

En este ejercicio, configurarás tu proyecto para servir archivos estáticos desde un directorio específico y luego usarás la etiqueta de plantilla `{% static %}` nuevamente para incluirlo en la plantilla:

1. Abre el proyecto `business_site` en PyCharm si aún no está abierto. Luego, haz clic con el botón derecho en el directorio del proyecto `business_site` (el directorio `business_site` de nivel superior, no el directorio del paquete `business_site`) y selecciona **New | Directory**.
2. En el cuadro de diálogo New Directory, ingresa `static` y haz clic en OK.
3. Haz clic con el botón derecho en el directorio `static` que acabas de crear y selecciona **New | File**.
4. En el cuadro de diálogo Name New File, ingresa `main.css` y haz clic en OK.
5. El archivo en blanco `main.css` debería abrirse automáticamente. Ingresa un par de reglas CSS simples para centrar el texto y establecer una fuente y un color de fondo, de la siguiente manera:
   ```css
   body {
       font-family: Arial, sans-serif;
       text-align: center;
       background-color: #f0f0f0;
   }
   ```
   Ahora guarda y cierra `main.css`.
6. A continuación, abre `business_site/settings.py`. Aquí, establece una lista de directorios para la configuración `STATICFILES_DIRS`. En este caso, la lista tendrá un solo elemento. Define una nueva variable llamada `STATICFILES_DIRS` en la parte inferior del archivo `settings.py` usando el siguiente código:
   ```python
   STATICFILES_DIRS = [BASE_DIR / "static"]
   ```
7. Inicia el servidor de desarrollo de Django si no se está ejecutando. Puedes verificar que la configuración sea correcta comprobando si puedes cargar el archivo `main.css`. Ten en cuenta que esto no tiene espacio de nombres, por lo que la URL es `http://127.0.0.1:8000/static/main.css`. Abre esta URL en tu navegador y verifica que el contenido coincida con lo que acabas de ingresar y guardar:
   *Figura 5.11 – CSS servido por Django*
8. Ahora, debes incluir `main.css` en tu plantilla `index`. Abre `index.html` en la carpeta `templates`. Antes de la etiqueta de cierre `</head>`, agrega esta etiqueta `<link>` para cargar el CSS:
   ```html
   <link rel="stylesheet" href="{% static 'main.css' %}">
   ```
   Esto vincula el archivo `main.css`, utilizando la etiqueta de plantilla `{% static %}`.
9. Carga `http://127.0.0.1:8000/` en tu navegador; deberías ver que el color de fondo, las fuentes y la alineación cambian:
   *Figura 5.12 – CSS aplicado con fuentes personalizadas visibles*

En este ejercicio, colocamos algunas reglas CSS en un archivo y las servimos mediante el `FileSystemFinder` de Django. Esto se logró creando un directorio `static` dentro del directorio del proyecto `business_site` y especificándolo en la configuración de Django mediante la opción `STATICFILES_DIRS`. Vinculamos el archivo `main.css` a la plantilla `index.html` mediante la etiqueta de plantilla `static`.

A continuación, veremos el otro caso de uso de los buscadores de archivos estáticos: encontrar y copiar archivos estáticos para el despliegue en producción al ejecutar el comando de gestión `collectstatic`.

---

### Sección: Buscadores de archivos estáticos: uso durante collectstatic

Una vez que hayamos terminado de trabajar en nuestros archivos estáticos, debemos moverlos a un directorio específico que pueda ser servido por nuestro servidor web de producción. Luego podemos desplegar nuestro sitio web copiando nuestro código Django y los archivos estáticos a nuestro servidor web de producción. En el caso de `business_site`, querremos mover `logo.png` y `main.css` (junto con otros archivos estáticos que el propio Django incluye) a un solo directorio que se pueda copiar al servidor web de producción. Esta es la función del comando de gestión `collectstatic`.

Sin tener que ejecutar `collectstatic`, un servidor web no podría mapear una URL a una ruta. Por ejemplo, no sabría que `main.css` debe cargarse desde el directorio estático del proyecto mientras que `logo.png` debe cargarse desde el directorio de la aplicación `landing`; no tiene ningún concepto del diseño del directorio de Django. Al ejecutar el comando de gestión `collectstatic`, Django utiliza cada buscador para enumerar los archivos estáticos en el disco. Cada archivo estático que se encuentra se copia luego en el directorio `STATIC_ROOT` (también definido en `settings.py`). Esto es un poco como el proceso inverso de manejar una solicitud. En lugar de obtener una ruta de URL y asignarla a una ruta del sistema de archivos, la ruta del sistema de archivos se copia en una ubicación que el servidor web frontend puede predecir. Esto permite que el servidor web frontend maneje una solicitud de un archivo estático independientemente de Django.

Un servidor web frontend es un software diseñado para enrutar solicitudes a aplicaciones (como Django) o leer archivos estáticos del disco. Pueden gestionar las solicitudes más rápido, pero no pueden generar contenido dinámico de la misma manera que Django. Los servidores web frontend son software como Apache HTTPD, Nginx y Lighttpd.

Para algunos ejemplos específicos de cómo funciona `collectstatic`, usaremos los dos archivos de los ejercicios anteriores: `landing/static/landing/logo.png` y `static/main.css`.

Supongamos que `STATIC_ROOT` se ha establecido en un directorio que está siendo servido por un servidor web normal; esto sería algo como `/var/www/business_site/static`. El destino de estos archivos sería `/var/www/business_site/static/reviews/logo.png` y `/var/www/business_site/static/main.css`, respectivamente.

Ahora, cuando llega una solicitud de un archivo estático, el servidor web podrá servirlo fácilmente porque las rutas se asignan de manera consistente:
- `/static/main.css` se sirve desde el archivo `/var/www/business_site/static/main.css`.
- `/static/reviews/logo.png` se sirve desde el archivo `/var/www/business_site/static/reviews/logo.png`.

Esto significa que la raíz del servidor web es `/var/www/business_site/` y las rutas estáticas se cargan directamente desde el disco de la manera habitual en que un servidor web cargaría archivos.

Nunca sirvas archivos directamente desde el directorio del proyecto Django configurando la raíz de tu servidor web en este directorio. Existe un riesgo de seguridad al compartir todo el directorio de tu proyecto Django, ya que haría posible descargar `settings.py` u otros archivos confidenciales. La ejecución de `collectstatic` copiará los archivos a un directorio que se puede mover fuera del directorio del proyecto Django a la raíz del servidor web por motivos de seguridad.

El comando `collectstatic` no toma en consideración el uso de etiquetas de plantilla `static`. Recopilará todos los archivos estáticos dentro de los directorios estáticos, incluso aquellos que tu proyecto no incluye dentro de una plantilla.

En el siguiente ejercicio, veremos el comando `collectstatic` en acción.

#### Ejercicio 5.04 – Recopilación de archivos estáticos para producción

En este ejercicio, crearemos una ubicación de almacenamiento temporal para copiar los archivos estáticos. Este directorio se llamará `static_production_test` y se ubicará dentro del directorio del proyecto `business_site`:

1. En PyCharm, crea un directorio temporal para colocar los archivos recopilados. Haz clic con el botón derecho en el directorio del proyecto `business_site` y selecciona **New | Directory**.
2. En el cuadro de diálogo New Directory, ingresa `static_production_test` y presiona Enter.
3. Abre `settings.py` y, en la parte inferior del archivo, define una nueva configuración para `STATIC_ROOT`. Establécela en la ruta del directorio que acabas de crear:
   ```python
   STATIC_ROOT = BASE_DIR / "static_production_test"
   ```
4. En una terminal, ejecuta el comando de gestión `collectstatic`:
   ```bash
   python manage.py collectstatic
   ```
   Deberías ver una salida como la siguiente:
   ```text
   129 static files copied to '/Users/chrisguest/business_site/static_production_test'.
   ```
   Esto puede parecer mucho si esperabas que copiara solo dos archivos, pero recuerda que copiará todos los archivos para todas las aplicaciones instaladas. En este caso, como tienes instalada la aplicación de administración de Django, la mayoría de los 129 archivos son para respaldarla.
5. Revisemos el directorio `static_production_test` para verificar lo que se ha creado.
   *Figura 5.13 – Directorio de destino del comando collectstatic*
   Deberías notar tres elementos dentro de él:
   - El directorio `admin`: Contiene archivos de la aplicación de administración de Django (organizado en subcarpetas para `css`, `fonts`, `img` y `js`).
   - El directorio `landing`: Este es el directorio estático de tu aplicación `landing`, que contiene `logo.png`.
   - El archivo `main.css`: Proviene del directorio estático de tu proyecto. Dado que no lo colocaste dentro de un directorio de espacio de nombres, se ha colocado directamente dentro de `STATIC_ROOT`.

En este ejercicio, recopilamos todos los archivos estáticos de `business_site` (incluidos los archivos estáticos de administración que incluye Django). Se copiaron en el directorio definido por la configuración `STATIC_ROOT`.

---

### Sección: Modo prefijado de STATICFILES_DIRS

Como se mencionó anteriormente, la configuración `STATICFILES_DIRS` también acepta elementos como tuplas en la forma de `(prefijo, directorio)`. Estos modos de operación no son mutuamente excluyentes; `STATICFILES_DIRS` puede contener elementos sin prefijo (cadenas) o con prefijo (tuplas). Básicamente, esto te permite mapear un determinado prefijo de URL a un directorio.

En Bookr, no tenemos suficientes recursos estáticos para justificar la configuración de esto, pero puede ser útil si deseas organizar tus recursos estáticos de manera diferente. Por ejemplo, puedes guardar todas tus imágenes en un determinado directorio y todo tu CSS en otro directorio. Es posible que debas hacer esto si utilizas una herramienta de generación de CSS de terceros, como Node.js con LESS.

He aquí un ejemplo donde se agregan dos directorios con prefijo a esta configuración: uno para servir imágenes y otro para servir CSS:

```python
STATICFILES_DIRS = [
    BASE_DIR / "static",
    ("images", BASE_DIR / "static_images"),
    ("css", BASE_DIR / "static_css"),
]
```

Además del directorio `static` que ya se estaba sirviendo sin prefijo, hemos agregado el servicio del directorio `static_images` con un prefijo de `images` y el directorio `static_css` con un prefijo de `css`.

Luego, podemos servir tres archivos (`main.js`, `main.css` y `main.jpg`) desde los directorios `static`, `static_css` y `static_images`, respectivamente:

*Figura 5.14 – Distribución de directorios para usar con URLs con prefijo*

En términos de acceder a estos a través de una URL, el mapeo es el siguiente:

*Figura 5.15 – Mapeo de una URL a un archivo según el prefijo*

Django enruta cualquier URL estática con un prefijo al directorio que coincide con ese prefijo.

Cuando se utiliza con la etiqueta de plantilla `static`, usa el prefijo y el nombre del archivo, no el nombre del directorio. He aquí un ejemplo:

```django
{% static 'images/main.jpg' %}
```

Cuando los archivos estáticos se recopilan mediante el comando `collectstatic`, se mueven a un directorio con el nombre del prefijo, dentro de `STATIC_ROOT`:

*Figura 5.16 – Mapeo desde la ruta en el directorio del proyecto a la ruta en STATIC_ROOT*

Django crea los directorios de prefijo dentro de `STATIC_ROOT`. Debido a esto, las rutas se pueden mantener consistentes, incluso cuando se usa un servidor web y no se enruta la búsqueda de URL a través de Django.

---

### Sección: El comando findstatic

La aplicación `staticfiles` también proporciona un comando de gestión adicional: `findstatic`. Este comando te permite ingresar la ruta relativa a un archivo estático (la misma que se usaría dentro de una etiqueta de plantilla `static`) y Django te dirá dónde se encontraba ese archivo. También se puede utilizar en modo detallado (*verbose*) para mostrar los directorios por los que está buscando.

Este comando es principalmente útil para fines de depuración/solución de problemas. Si se carga el archivo incorrecto o no se puede encontrar un archivo en particular, puedes usar este comando para intentar averiguar el motivo.

#### Ejercicio 5.05 – Búsqueda de archivos mediante findstatic

En este ejercicio, ejecutarás el comando `findstatic` con varias opciones y comprenderás qué significa su salida:

1. Abre una terminal y navega hasta el directorio del proyecto `business_site`.
2. Ejecuta el comando `findstatic` sin opciones para ver la ayuda:
   ```bash
   python manage.py findstatic
   ```
   Se mostrará la siguiente salida de ayuda:
   ```text
   usage: manage.py findstatic [-h] [--first] [--version] [-v {0,1,2,3}] [--settings SETTINGS] [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color] [--skip-checks] staticfile [staticfile ...]
   manage.py findstatic: error: Enter at least one label.
   ```
3. Busquemos un archivo que sabemos que existe: `main.css`:
   ```bash
   python3 manage.py findstatic main.css
   ```
   El comando anterior genera la ruta en la que se encontró `main.css`:
   ```text
   Found 'main.css' here:
     /Users/chrisguest/business_site/static/main.css
   ```
4. Intentemos buscar un archivo sin especificar el espacio de nombres, `logo.png`:
   ```bash
   python manage.py findstatic logo.png
   ```
   Django mostrará un error que indica que no se pudo encontrar el archivo:
   ```text
   No matching file found for 'logo.png'.
   ```
   Django no puede localizar este archivo porque le hemos asignado un espacio de nombres: debemos incluir la ruta relativa completa.
5. Intenta buscar el `logo.png` nuevamente, pero esta vez usando la ruta completa:
   ```bash
   python manage.py findstatic landing/logo.png
   ```
   Django puede encontrar el archivo ahora:
   ```text
   Found 'landing/logo.png' here:
     /Users/chrisguest/business_site/landing/static/landing/logo.png
   ```
6. Puedes buscar varios archivos a la vez agregando cada archivo como argumento:
   ```bash
   python manage.py findstatic landing/logo.png missing-file.js main.css
   ```
   El estado de ubicación para cada archivo se muestra de la siguiente manera:
   ```text
   No matching file found for 'missing-file.js'.
   Found 'landing/logo.png' here:
     /Users/chrisguest/business_site/landing/static/landing/logo.png
   Found 'main.css' here:
     /Users/chrisguest/business_site/static/main.css
   ```
7. El comando se puede ejecutar con una verbosidad de 0, 1 o 2. De forma predeterminada, se ejecuta en la verbosidad 1. Disminuir la verbosidad a 0 solo genera las rutas que localiza sin ninguna información adicional:
   ```bash
   python manage.py findstatic -v0 landing/logo.png missing-file.js main.css
   ```
   La salida muestra solo las rutas encontradas:
   ```text
   /Users/chrisguest/business_site/landing/static/landing/logo.png
   /Users/chrisguest/business_site/static/main.css
   ```
8. Para obtener más información sobre en qué directorios está buscando Django el archivo solicitado, aumenta la verbosidad a 2:
   ```bash
   python manage.py findstatic -v2 landing/logo.png missing-file.js main.css
   ```
   La salida contiene mucha más información, incluidos los directorios que se han buscado:
   *Figura 5.17 – findstatic ejecutado con verbosidad 2, mostrando exactamente qué directorios se buscaron*

---

### Sección: Servicio de los archivos más recientes (para la invalidación de caché)

La idea básica del almacenamiento en caché es que algunas operaciones pueden tardar mucho tiempo en realizarse. Podemos acelerar un sistema almacenando los resultados de la operación en un lugar al que sea más rápido acceder para que la próxima vez que los necesitemos, se puedan recuperar rápidamente.

Es posible que hayas notado que la primera vez que visitas un sitio web en particular, la carga puede ser lenta, pero luego, la próxima vez, se carga mucho más rápido. Esto se debe a que tu navegador ha almacenado en caché algunos (o todos) los archivos estáticos que el sitio necesita cargar.

El servidor web frontend debe configurarse para enviar encabezados HTTP especiales como parte de una respuesta de archivo estático, como `Cache-Control` (`no-cache` o `max-age=seconds`) o el encabezado `Expires`.

Uno de los problemas más difíciles en informática es la invalidación de la caché (*cache invalidation*). Por ejemplo, si cambiamos `logo.png`, ¿cómo sabe nuestro navegador que debe descargar la nueva versión?

Django proporciona una solución integrada. Durante la fase `collectstatic`, cuando se copian los archivos, Django puede agregar un hash de su contenido al nombre del archivo. Por ejemplo, el archivo de origen, `logo.png`, se copiará en `static_production_test/landing/logo.f30ba08c60ba.png`. Esto solo se hace cuando se utiliza el motor de almacenamiento `ManifestFilesStorage`.

Dado que el nombre del archivo solo cambia cuando cambia el contenido, el navegador siempre descargará el nuevo contenido.

Un hash es una función unidireccional que genera una cadena de longitud fija, independientemente de la longitud de la entrada. Django usa MD5 para el hash de contenido.

Puedes elegir el motor de almacenamiento cambiando el valor de `STATICFILES_STORAGE` en `settings.py`. La clase que implementa la funcionalidad de adición de hash es `django.contrib.staticfiles.storage.ManifestStaticFilesStorage`.

El uso de este motor de almacenamiento no requiere que realices ningún cambio en tus plantillas HTML, siempre que incluyas recursos estáticos con la etiqueta de plantilla `static`. Django genera un archivo de manifiesto (`staticfiles.json`, en formato JSON) que contiene un mapeo entre el nombre de archivo original y el nombre de archivo con hash. Insertará automáticamente el nombre de archivo con hash al usar la etiqueta de plantilla `static`:

```django
<img src="{% static 'reviews/logo.png' %}">
```

Cuando se procesa la página, el hash más reciente se recuperará de `staticfiles.json` y la salida será así:

```html
<img src="/static/landing/logo.f30ba08c60ba.png">
```

Mientras que, si no hubiéramos utilizado la etiqueta `static` y en su lugar hubiéramos codificado la ruta de forma fija, siempre aparecería como está escrita:

```html
<img src="/static/landing/logo.png">
```

Dado que esto no contiene un hash, nuestro navegador no verá que la ruta cambia y, por lo tanto, nunca intentará descargar el nuevo archivo.

Django conserva la versión anterior de los archivos con el hash antiguo cuando se ejecuta `collectstatic`, de modo que las versiones anteriores de tu aplicación aún puedan hacer referencia a él si lo necesitan. La versión más reciente del archivo también se copia sin hash para que las aplicaciones que no son de Django puedan hacer referencia a él sin necesidad de buscar el hash.

#### Ejercicio 5.06 – Exploración del motor de almacenamiento ManifestFilesStorage

En este ejercicio, actualizarás temporalmente `settings.py` para usar `ManifestFilesStorage`, luego ejecutarás `collectstatic` para ver cómo se generan los archivos con un hash:

1. En PyCharm (todavía en el proyecto `business_site`), abre `settings.py` y agrega una configuración `STATICFILES_STORAGE` en la parte inferior del archivo:
   ```python
   STATICFILES_STORAGE = "django.contrib.staticfiles.storage.ManifestStaticFilesStorage"
   ```
2. Abre una terminal, navega hasta el directorio del proyecto `business_site` y ejecuta el comando `collectstatic`:
   ```bash
   python manage.py collectstatic
   ```
   Si tu directorio `static_production_test` no está vacío, se te pedirá que permitas la sobrescritura de los archivos existentes:
   *Figura 5.18 – Mensaje para permitir la sobrescritura durante la recopilación estática*
   Simplemente escribe `yes` y luego presiona Enter para permitir la sobrescritura.
   La salida de este comando te dirá la cantidad de archivos que se copiaron, así como la cantidad que se procesó y se le agregó el hash al nombre del archivo:
   ```text
   0 static files copied to '/Users/chrisguest/business_site/static_production_test', 129 unmodified, 101 post-processed.
   ```
3. Los archivos estáticos se copiaron en el directorio `static_production_test` como antes; sin embargo, ahora hay dos copias de cada archivo: una con el nombre del hash y otra sin él:
   *Figura 5.19 – Directorio static_production_test expandido con nombres de archivo con hash*
4. Hagamos un cambio en el archivo `main.css` y veamos cómo cambia el hash. Agrega algunas líneas en blanco al final del archivo y guárdalo. Vuelve a ejecutar el comando `collectstatic` en una terminal:
   ```bash
   python3 manage.py collectstatic
   ```
   Una vez más, confirma con `yes`:
   ```text
   You have requested to collect static files at the destination location as specified in your settings:

       /Users/chrisguest/business_site/static_production_test

   This will overwrite existing files!
   Are you sure you want to do this?

   Type 'yes' to continue, or 'no' to cancel: yes
   1 static file copied to '/Users/chrisguest/business_site/static_production_test', 128 unmodified, 101 post-processed.
   ```
5. Mira dentro del directorio `static_production_test` nuevamente. Deberías ver que el archivo antiguo con el hash anterior se conservó y que se agregó un nuevo archivo con un nuevo hash:
   *Figura 5.20 – Se agregó otro archivo main.css con el hash más reciente*
6. Ahora, examina el archivo `staticfiles.json` que genera Django. Abre `static_production_test/staticfiles.json`. Desplázate hasta el final del archivo; deberías ver una entrada para el archivo `main.css`. He aquí un ejemplo:
   ```json
   "main.css": "main.efb556103718.css"
   ```

---

### Sección: Motores de almacenamiento personalizados (Custom storage engines)

En la sección anterior, configuramos el motor de almacenamiento en `ManifestFilesStorage`. Esta clase la proporciona Django, pero también es posible escribir un motor de almacenamiento personalizado. Por ejemplo, podrías escribir un motor de almacenamiento que suba tus archivos estáticos a una CDN, Amazon S3 o un depósito de Google Cloud cuando ejecutes `collectstatic`.

El siguiente código es un esqueleto breve que indica qué métodos debes implementar para crear un motor de almacenamiento de archivos personalizado:

```python
from django.conf import settings
from django.contrib.staticfiles import storage

class CustomFilesStorage(storage.StaticFilesStorage):
    def __init__(self):
        """The class must be able to be instantiated without any arguments.
        Create custom settings in settings.py and read them instead."""
        self.setting = settings.CUSTOM_STORAGE_SETTING

    def delete(self, name):
        """Implement delete of the file from the remote service."""

    def exists(self, name):
        """Return True if a file with name exists in the remote service."""

    def listdir(self, path):
        """List a directory in the remote service.
        Return should be a 2-tuple of lists, the first a list of directories, the second a list of files."""
        return (["directory1", "directory2"], ["main.css", "document.txt", "image.jpg"])

    def size(self, name):
        """Return the size in bytes of the file with name."""

    def url(self, name):
        """Return the URL where the file of with name can be access on the remote service.
        For example, this might be URL of the file after it has been uploaded to a specific remote host with a specific domain."""

    def _open(self, name, mode="rb"):
        """Return a File-like object pointing to file with name.
        For example, this could be a URL handle for a remote file."""

    def _save(self, name, content):
        """Write the content for a file with name.
        In this method you might upload the content to a remote service."""
```

Después de implementar tu motor de almacenamiento personalizado, puedes activarlo estableciendo su ruta de módulo con puntos en la configuración `STATICFILES_STORAGE` en `settings.py`:

```python
STATICFILES_STORAGE = "myapp.storages.CustomFilesStorage"
```

#### Actividad 5.01 – Adición de un logotipo para Reviews

La aplicación Bookr debe tener un logotipo que sea específico para las páginas de la aplicación Reviews. Esto implicará agregar una plantilla base solo para la aplicación Reviews y actualizar nuestras plantillas actuales de Reviews para que hereden de ella. Luego, incluirás el logotipo de Bookr Reviews en esta plantilla base. Estos pasos te ayudarán a completar esta actividad:

1. Agrega una regla CSS para posicionar el logotipo. Coloca esta regla en el archivo `base.html` existente, después de la regla `.navbar-brand`:
   ```css
   .navbar-brand > img {
       height: 60px;
   }
   ```
2. Agrega una etiqueta de plantilla de bloque `brand` que las plantillas que heredan puedan anular. Coloca esto dentro del elemento `<a>` con la clase `navbar-brand`. El contenido predeterminado del bloque debe dejarse como `Book Review`.
3. Agrega un directorio `static` dentro de la aplicación Reviews, que contenga un directorio con espacio de nombres. Descarga el archivo `logo.png` de Reviews de la carpeta `Chapter05` en el repositorio de GitHub de este libro y colócalo dentro de este directorio.
4. Crea un directorio `templates` para el proyecto Bookr (dentro del directorio del proyecto Bookr). Luego, mueve el `base.html` actual de la aplicación Reviews a este directorio para que se convierta en una plantilla base para todo el proyecto.
5. Agrega la ruta del nuevo directorio de plantillas a la configuración `TEMPLATES["DIRS"]` en `settings.py`.
6. Crea otra plantilla `base.html` específicamente para la aplicación Reviews. Colócala dentro del directorio `templates` de la aplicación Reviews. La nueva plantilla debe extender el archivo `base.html` existente (ahora global).
7. El nuevo archivo `base.html` debe anular el contenido del bloque `brand`. Este bloque debe contener solo un `<img>` cuyo atributo `src` se establece mediante la etiqueta de plantilla `{% static %}`. La fuente de la imagen debe ser el logotipo que agregamos en el Paso 2.

Consulta las siguientes capturas de pantalla para ver cómo deberían verse tus páginas después de estos cambios:
*Figura 5.21 – La página Book List después de agregar el logotipo de Reviews*
*Figura 5.22 – La página Book Details después de agregar el logotipo de Reviews*

#### Actividad 5.02 – Mejoras de CSS

Actualmente, el CSS se mantiene en línea en la plantilla `base.html`. Como práctica recomendada, debe moverse a su propio archivo para que se pueda almacenar en caché por separado y disminuir el tamaño de las descargas de HTML. Como parte de esto, también agregarás algunas mejoras de CSS, como fuentes y colores, y vincularás el CSS de Google Fonts para respaldar estos cambios. Estos pasos te ayudarán a completar esta actividad:

1. Crea un directorio llamado `static` en el directorio del proyecto Bookr. Luego, crea un nuevo archivo dentro de él llamado `main.css`.
2. Copia el contenido del elemento `<style>` de la plantilla principal `base.html` en el nuevo archivo `main.css`, luego elimina el elemento `<style>` de la plantilla. Agrega estas reglas adicionales al final del archivo CSS:
   ```css
   body {
       font-family: 'Source Sans Pro', sans-serif;
       background-color: #e6efe8;
       color: #393939;
   }

   h1, h2, h3, h4, h5, h6 {
       font-family: 'Libre Baskerville', serif;
   }
   ```
3. Vincula el nuevo archivo `main.css` con una etiqueta `<link rel="stylesheet" href="...">`. Usa la etiqueta de plantilla `{% static %}` para generar la URL para el atributo `href`, y no olvides cargar la biblioteca estática.
4. Vincula el CSS de Google Fonts agregando este código a la plantilla base:
   ```html
   <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Libre+Baskerville|Source+Sans+Pro&display=swap">
   ```
5. Actualiza tu configuración de Django para agregar `STATICFILES_DIRS`, que se establece en el directorio estático que creaste en el Paso 1. Cuando hayas terminado, tu aplicación Bookr debería verse como la Figura 5.23:
   *Figura 5.23 – Página principal con una nueva fuente y color de fondo*

#### Actividad 5.03 – Adición de un logotipo global

Ya agregaste un logotipo que se sirve en las páginas de la aplicación Reviews. Tenemos otro logotipo que se utilizará globalmente de forma predeterminada, pero otras aplicaciones podrán anularlo:

1. Descarga el logotipo de Bookr (`logo.png`) de la carpeta `Chapter05` en el repositorio de GitHub de este libro.
2. Guárdalo en el directorio estático principal del proyecto.
3. Edita el archivo principal `base.html`. Ya tenemos un bloque para el logotipo (`brand`), por lo que se puede colocar un `<img>` dentro de aquí. Usa la etiqueta de plantilla `static` para hacer referencia al logotipo que acabas de descargar.
4. Comprueba que tus páginas funcionen. En la URL principal, deberías ver el logotipo de Bookr, pero en las páginas de lista de libros y detalles del libro, deberías ver el logotipo de Bookr Reviews.

Cuando hayas terminado, deberías ver el logotipo de Bookr en la página principal:
*Figura 5.24 – Logotipo de Bookr en la página principal*

Cuando visites una página que tenía el logotipo de Bookr Reviews antes, como la página Book List, aún debería mostrar el logotipo de Bookr Reviews:
*Figura 5.25 – El logotipo de Bookr Reviews todavía aparece en las páginas de Reviews*

---

### Sección: Resumen

En este capítulo, te mostramos cómo usar la aplicación `staticfiles` de Django para buscar y servir archivos estáticos. Usamos la vista `static` integrada para servir estos archivos con el servidor de desarrollo de Django en modo `DEBUG`. Mostramos diferentes lugares para almacenar archivos estáticos mediante el uso de un directorio global para el proyecto o en un directorio específico para la aplicación; los recursos globales deben almacenarse en el primero, mientras que los recursos específicos de la aplicación deben almacenarse en el segundo. Mostramos la importancia de asignar espacios de nombres a los directorios de archivos estáticos para evitar conflictos. Después de servir los recursos, usamos la etiqueta `static` para incluirlos en nuestra plantilla. Luego demostramos cómo el comando `collectstatic` copia todos los recursos en el directorio `STATIC_ROOT`, para el despliegue en producción. Mostramos cómo usar el comando `findstatic` para depurar la carga de archivos estáticos. Para invalidar las cachés automáticamente, consideramos el uso de `ManifestFilesStorage` para agregar un hash del contenido del archivo a la URL del archivo estático. Finalmente, hablamos brevemente sobre el uso de un motor de almacenamiento de archivos personalizado.

Hasta ahora, solo hemos obtenido páginas web utilizando contenido que ya existe. En el próximo capítulo, comenzaremos a agregar formularios para que podamos interactuar con las páginas web enviándoles datos a través de HTTP.
