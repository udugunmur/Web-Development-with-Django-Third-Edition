# Parte 2: Creación de aplicaciones web con Django

## Capítulo 8: Servicio de Archivos Multimedia y Carga de Archivos

Los archivos multimedia (*media files*) se refieren a archivos adicionales que se pueden agregar después del despliegue para enriquecer tu aplicación Django. Por lo general, son imágenes adicionales que usarías en tu sitio, pero cualquier tipo de archivo (incluidos video, audio, PDF, texto, documentos o incluso HTML) se puede servir como contenido multimedia.

Puedes pensar en ellos como algo intermedio entre los datos dinámicos y los activos estáticos. No son datos dinámicos que Django genera sobre la marcha, como cuando se renderiza una plantilla. Tampoco son los archivos estáticos que incluye el desarrollador del sitio cuando este se despliega. En cambio, son archivos adicionales que los usuarios pueden cargar o que tu aplicación puede generar para su posterior recuperación.

Algunos ejemplos comunes de archivos multimedia (que verás en la Actividad 8.01 – Carga de imágenes y PDF para libros, más adelante en este capítulo) son portadas de libros y archivos PDF de vista previa que se pueden adjuntar a un objeto `Book`. También puedes usar archivos multimedia para permitir que los usuarios carguen imágenes para una publicación de blog o avatares para un sitio de redes sociales. Si quisieras usar Django para crear tu propia plataforma para compartir videos, almacenarías los videos cargados como archivos multimedia. Tu sitio web no funcionará bien si todos estos archivos son archivos estáticos, ya que los usuarios no podrán cargar sus propias portadas de libros, videos, etc., y se quedarán con los que tú desplegaste.

En este capítulo, cubriremos los siguientes temas:
- Configuración para la carga y servicio de archivos multimedia
- Carga de archivos mediante formularios HTML
- Carga de archivos con formularios de Django
- Carga de imágenes con formularios de Django
- Servir archivos subidos (y otros) usando Django
- ModelForm y carga de archivos
- Actividad 8.01 – Carga de imágenes y PDF para libros
- Actividad 8.02 – Mostrar la portada y el enlace de muestra

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter08` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Configuración para la carga y servicio de archivos multimedia

En el Capítulo 5 (*Servicio de Archivos Estáticos*), vimos cómo se puede usar Django para servir archivos estáticos. El servicio de archivos multimedia es bastante similar. Se deben configurar dos ajustes en `settings.py`: `MEDIA_ROOT` y `MEDIA_URL`. Estos son análogos a `STATIC_ROOT` y `STATIC_URL` para servir archivos estáticos. Son los siguientes:
- **MEDIA_ROOT**: Esta es la ruta en el disco donde se almacenarán los archivos multimedia (como los archivos cargados). Al igual que con los archivos estáticos, tu servidor web debe configurarse para servir directamente desde este directorio para quitarle carga a Django.
- **MEDIA_URL**: Esto es similar a `STATIC_URL`, pero como habrás adivinado, es la URL que debe usarse para servir archivos multimedia. Debe terminar en `/`. Generalmente, usarás algo como `/media/`.

Por razones de seguridad, la ruta de `MEDIA_ROOT` no debe ser la misma que la de `STATIC_ROOT`, y `MEDIA_URL` no debe ser la misma que `STATIC_URL`. Si fueran iguales, un usuario podría reemplazar tus archivos estáticos (como archivos JavaScript o CSS) con código malicioso y vulnerar a tus usuarios.

`MEDIA_URL` está diseñada para usarse en plantillas de modo que no tengas la URL codificada de forma fija (*hardcoded*) y se pueda cambiar fácilmente. Por ejemplo, es posible que desees configurarla en un host específico o en una red de entrega de contenido (CDN) cuando realices el despliegue en producción. Hablaremos sobre el uso de `MEDIA_URL` en plantillas en la siguiente sección.

#### Servir archivos multimedia en desarrollo

Al igual que con los archivos estáticos, al servir archivos multimedia en producción, tu servidor web debe configurarse para servir directamente desde el directorio `MEDIA_ROOT` para evitar que Django se quede ocupado atendiendo la solicitud. El servidor de desarrollo de Django puede servir archivos multimedia en desarrollo. Sin embargo, a diferencia de los archivos estáticos, el mapeo de URL y la vista no se configuran automáticamente para los archivos multimedia.

Django proporciona el mapeo de URL `static`, que se puede agregar a tus mapas de URL existentes para servir archivos multimedia. Se agrega a tu archivo `urls.py` de esta manera:

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # tus mapas de URL existentes
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

Esto servirá la configuración `MEDIA_ROOT` definida en `settings.py` en la configuración `MEDIA_URL` que también se define allí. La razón por la que verificamos `settings.DEBUG` antes de anexar el mapeo es para no agregar este mapa en producción.

Por ejemplo, si tu `MEDIA_ROOT` estuviera configurado en `/var/www/bookr/media` y tu `MEDIA_URL` estuviera configurado en `/media/`, entonces el archivo `/var/www/bookr/media/image.jpg` estaría disponible en `http://127.0.0.1:8000/media/image.jpg`.

El mapeo de URL `static` no funciona cuando la configuración `DEBUG` de Django es `False`, por lo que no se puede usar en producción. Sin embargo, como se ha mencionado, en producción, tu servidor web debería atender estas solicitudes, por lo que Django no necesitará manejarlas.

En el primer ejercicio, agregarás `MEDIA_ROOT` y `MEDIA_URL` a tu `settings.py`. Luego, agregarás el mapa de URL para servir archivos multimedia estáticos y agregarás un archivo de prueba para asegurarte de que el servicio multimedia esté configurado correctamente.

#### Ejercicio 8.01 – Configuración del almacenamiento y servicio de archivos multimedia

En este ejercicio, configurarás un nuevo proyecto de Django como proyecto de ejemplo para usar a lo largo de este capítulo. Luego, lo configurarás para que pueda servir archivos multimedia. Harás esto creando un directorio `media` y agregando las configuraciones `MEDIA_ROOT` y `MEDIA_URL`. Luego, configurarás el mapeo de URL para `MEDIA_URL`.

Para verificar que todo esté configurado y se esté sirviendo correctamente, colocarás un archivo de prueba dentro del directorio `media`:

1. Al igual que con los proyectos de Django de ejemplo anteriores que has configurado, puedes reutilizar el entorno virtual `djangoenv` existente. En una terminal, activa el entorno virtual `djangoenv`, usando `pyenv local djangoenv`. Luego, inicia un nuevo proyecto llamado `media_project` usando `django-admin`:
   ```bash
   django-admin startproject media_project
   ```
   Para aprender cómo crear y activar un entorno virtual usando `pyenv`, consulta el Capítulo 1 (*Introducción a Django*) de este libro.
2. Cambia (con `cd`) al directorio `media_project` que se creó, luego usa el comando de administración `startapp` para iniciar una aplicación llamada `media_example`:
   ```bash
   python manage.py startapp media_example
   ```
3. Abre el directorio `media_project` en PyCharm. Configura una configuración de ejecución para el comando `runserver` de la misma manera que para los otros proyectos de Django que has abierto:
   *Figura 8.1: Configuración de Runserver*
   La Figura 8.1 muestra la configuración de Runserver del proyecto en PyCharm.
4. Crea un nuevo directorio llamado `media` dentro del directorio del proyecto `media_project`. Luego, crea un nuevo archivo en este directorio llamado `test.txt`. La estructura de directorios se verá como en la Figura 8.2:
   *Figura 8.2: El directorio media y la disposición de test.txt*
   El archivo `test.txt` también se abrirá automáticamente. Ingresa `Hello world!` en él, luego puedes guardar y cerrar el archivo.
5. Abre `settings.py` dentro del directorio del paquete `media_project`. Al final del archivo, agrega una configuración para `MEDIA_ROOT` usando la ruta al directorio `media` que acabas de crear. Une el nombre del directorio (`media`) a `BASE_DIR`:
   ```python
   MEDIA_ROOT = BASE_DIR / "media"
   ```
6. Directamente debajo de la línea agregada en el paso 5, agrega otra configuración para `MEDIA_URL`; esto debería ser simplemente `"/media/"`:
   ```python
   MEDIA_URL = "/media/"
   ```
   Con estos cambios realizados, `settings.py` debería parecerse al archivo ubicado en la carpeta `Chapter08` en el repositorio de GitHub de este libro.
7. Abre el archivo `urls.py` del paquete `media_project`. Después de la definición de `urlpatterns`, agrega el siguiente código para agregar la URL de servicio multimedia si se ejecuta en modo `DEBUG`. Primero, deberás importar la configuración de Django y la vista de servicio estático agregando las líneas de importación resaltadas encima de la definición de `urlpatterns`:
   ```python
   from django.contrib import admin
   from django.urls import path
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns = [
       path("admin/", admin.site.urls),
   ]
   ```
8. Luego, agrega el siguiente código justo después de tu definición de `urlpatterns` (consulta el bloque de código en el paso anterior) para agregar condicionalmente un mapeo desde la configuración `MEDIA_URL` a la vista estática, que servirá desde `MEDIA_ROOT`:
   ```python
   if settings.DEBUG:
       urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```
   Ahora puedes guardar este archivo.
9. Inicia el servidor de desarrollo de Django, si aún no se está ejecutando, luego visita `http://127.0.0.1:8000/media/test.txt`. Si hiciste todo correctamente, deberías ver el texto `Hello world!` en tu navegador:
   *Figura 8.3: Servir un archivo multimedia*
   Si tu navegador se parece a la Figura 8.3, significa que los archivos multimedia se están sirviendo desde el directorio `MEDIA_ROOT`.

El archivo `test.txt` que creamos fue solo para pruebas, pero lo usaremos en el Ejercicio 8.02 – Configuración de plantillas y uso de `MEDIA_URL` en plantillas, así que no lo elimines todavía.

En este ejercicio, configuramos Django para servir archivos multimedia. Servimos un archivo de prueba solo para asegurarnos de que todo funcionara como se esperaba, y así fue. Ahora veremos cómo podemos generar automáticamente URLs de medios en plantillas.

#### Procesadores de contexto y uso de MEDIA_URL en plantillas

Para usar `MEDIA_URL` en una plantilla, podríamos pasarlo a través del diccionario de contexto de renderizado en nuestra vista. Se muestra un ejemplo en el siguiente bloque de código:

```python
from django.conf import settings

def my_view(request):
    return render(request, "template.html", {"MEDIA_URL": settings.MEDIA_URL, "username": "jbloggs"})
```

Esto funcionará, pero el problema es que `MEDIA_URL` es una variable común que podríamos querer usar en muchos lugares, por lo que tendríamos que pasarla prácticamente en cada vista.

En su lugar, podemos usar un **procesador de contexto** (*context processor*), que es una forma de agregar una o más variables automáticamente al diccionario de contexto en cada llamada a `render`.

Un procesador de contexto es una función que acepta un argumento, el `request` actual. Devuelve un diccionario de información de contexto que se fusionará con el diccionario que se pasó a la llamada de `render`.

Podemos mirar el código fuente del procesador de contexto `media`, que ilustra cómo funciona:

```python
def media(request):
    """
    Add media-related context variables to the context.
    """
    return {"MEDIA_URL": settings.MEDIA_URL}
```

Con el procesador de contexto `media` activado, `MEDIA_URL` se agregará a tus diccionarios de contexto. Podríamos cambiar nuestra llamada a `render`, vista anteriormente, a esto:

```python
return render(request, "template.html", {"username": "jbloggs"})
```

Los mismos datos se enviarían a la plantilla, ya que el procesador de contexto agregaría `MEDIA_URL`.

La ruta completa del módulo hacia el procesador de contexto `media` es `django.template.context_processors.media`. Algunos ejemplos de otros procesadores de contexto que proporciona Django son los siguientes:
- `django.template.context_processors.debug`: Esto devuelve el diccionario `{"DEBUG": settings.DEBUG}`.
- `django.template.context_processors.request`: Esto devuelve el diccionario `{"request": request}`; es decir, simplemente agrega la solicitud HTTP actual al contexto.

Para habilitar un procesador de contexto, se debe agregar su ruta de módulo a la opción `context_processors` de tu configuración `TEMPLATES`. Por ejemplo, para habilitar el procesador de contexto `media`, agrega `django.template.context_processors.media`. Cubriremos cómo hacer esto en detalle en el Ejercicio 8.02 – Configuración de plantillas y uso de `MEDIA_URL` en plantillas.

Una vez habilitado el procesador de contexto `media`, se puede acceder a la variable `MEDIA_URL` dentro de una plantilla como a una variable normal:

```django
{{ MEDIA_URL }}
```

Podrías usarlo, por ejemplo, para obtener una imagen:

```html
<img src="{{ MEDIA_URL }}uploads/image.jpg">
```

Ten en cuenta que, a diferencia de los archivos estáticos, no hay una etiqueta de plantilla para cargar archivos multimedia (es decir, no hay un equivalente a la etiqueta de plantilla `{% static %}`).

También se pueden escribir procesadores de contexto personalizados. Por ejemplo, volviendo a la aplicación Bookr que creamos, es posible que deseemos mostrar una lista de las cinco reseñas más recientes en una barra lateral que se encuentra en cada página. Un procesador de contexto como este ejecutaría el siguiente código:

```python
from reviews.models import Review

def latest_reviews(request):
    return {"latest_reviews": Review.objects.order_by("-date_created")[:5]}
```

Esto se guardaría en un archivo llamado `context_processors.py` en el directorio del proyecto Bookr, y luego se haría referencia a él en la configuración de `context_processors` por su ruta de módulo, `context_processors.latest_reviews`. O podríamos guardarlo dentro de la aplicación `reviews` y referirnos a él como `reviews.context_processors.latest_reviews`. Depende de ti decidir si un procesador de contexto debe considerarse para todo el proyecto o específico de la aplicación. Sin embargo, ten en cuenta que, independientemente de dónde se almacene, una vez activado, se aplica a todas las llamadas de `render` para todas las aplicaciones.

Un procesador de contexto puede devolver un diccionario con varios elementos o incluso cero elementos. Haría esto si tuviera condiciones para agregar elementos solo si se cumplieran ciertos criterios. Por ejemplo, mostrar las últimas reseñas solo si el usuario ha iniciado sesión.

Exploremos esto en detalle en el siguiente ejercicio.

#### Ejercicio 8.02 – Configuración de plantillas y uso de MEDIA_URL en plantillas

En este ejercicio, continuarás con `media_project` y configurarás Django para agregar automáticamente la configuración `MEDIA_URL` a cada plantilla. Para ello, agregarás `django.template.context_processors.media` a la configuración `TEMPLATES` en `context_processors`. Luego, agregarás una plantilla que usa esta nueva variable y una vista de ejemplo para renderizarla. Realizarás cambios en la vista y la plantilla a lo largo de los ejercicios de este capítulo:

1. En PyCharm, abre `settings.py`. Primero, deberás agregar `media_example` a la configuración `INSTALLED_APPS`, ya que no se hizo cuando se configuró el proyecto:
   ```python
   INSTALLED_APPS = [
       # otras aplicaciones truncadas por brevedad
       "media_example",
   ]
   ```
2. Aproximadamente a la mitad del archivo, encontrarás la configuración `TEMPLATES`, que es un diccionario. Dentro de él está el elemento `OPTIONS` (otro diccionario). Dentro de `OPTIONS` está la configuración `context_processors`. Agrega lo siguiente al final de esta lista:
   ```python
   "django.template.context_processors.media"
   ```
   La lista completa debería verse así:
   ```python
   TEMPLATES = [
       {
           "BACKEND": "django.template.backends.django.DjangoTemplates",
           "DIRS": [],
           "APP_DIRS": True,
           "OPTIONS": {
               "context_processors": [
                   "django.template.context_processors.debug",
                   "django.template.context_processors.request",
                   "django.contrib.auth.context_processors.auth",
                   "django.contrib.messages.context_processors.messages",
                   "django.template.context_processors.media",
               ],
           },
       },
   ]
   ```
3. Abre el archivo `views.py` de la aplicación `media_example` y crea una nueva vista llamada `media_example`. Por ahora, solo puede renderizar una plantilla llamada `media-example.html` (crearás esto en el paso 5). Todo el código de la función de vista es el siguiente:
   ```python
   def media_example(request):
       return render(request, "media-example.html")
   ```
4. Guarda `views.py`. Necesitas un mapeo de URL a la vista `media_example`. Abre el archivo `urls.py` del paquete `media_project`. Primero, importa `media_example.views` con las otras importaciones en el archivo:
   ```python
   import media_example.views
   ```
   Luego, agrega `path` en `urlpatterns` para mapear `media-example/` a la vista `media_example`:
   ```python
   path('media-example/', media_example.views.media_example)
   ```
   Tu `urlpatterns` completo debería verse como este bloque de código:
   ```python
   from django.conf.urls.static import static
   import media_example.views

   urlpatterns = [
       path("admin/", admin.site.urls),
       path("media-example/", media_example.views.media_example),
   ]

   if settings.DEBUG:
       urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```
   Puedes guardar y cerrar el archivo.
5. Crea un directorio `templates` dentro del directorio de la aplicación `media_example`. Luego, crea un nuevo archivo HTML dentro del directorio `templates` del proyecto `media_project`. Selecciona HTML File y nombra el archivo `media-example.html`.
6. El archivo `media-example.html` debería abrirse automáticamente. Solo vas a agregar un enlace dentro del archivo al archivo `test.txt` que creaste en el Ejercicio 8.01 – Configuración del almacenamiento y servicio de archivos multimedia. Dentro del elemento `<body>`, agrega el código resaltado:
   ```html
   <body>
       <a href="{{ MEDIA_URL }}test.txt">Test Text File</a>
   </body>
   ```
   Ten en cuenta que no hay `/` entre `MEDIA_URL` y el nombre del archivo; esto se debe a que ya agregamos una barra diagonal final cuando lo definimos en `settings.py`. Puedes guardar el archivo.
7. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego visita `http://127.0.0.1:8000/media-example/`. Deberías ver una página sencilla como en la Figura 8.4:
   *Figura 8.4: Página básica de enlace multimedia*
   Si haces clic en el enlace, accederás a la visualización de `test.txt` y verás el texto `Hello world!` que creaste en el Ejercicio 8.01 – Configuración del almacenamiento y servicio de archivos multimedia (Figura 8.3). Esto significa que has configurado correctamente los ajustes de `context_processors` de Django.

Hemos terminado con `test.txt`, por lo que puedes eliminar el archivo ahora. Usaremos la vista y la plantilla `media_example` en los otros ejercicios, así que consérvalas. En la siguiente sección, hablaremos sobre cómo cargar archivos mediante un navegador web y cómo Django accede a ellos en una vista.

---

### Sección: Carga de archivos mediante formularios HTML

En el Capítulo 6 (*Formularios*), aprendimos sobre los formularios HTML. Discutimos cómo usar el atributo `method` de `<form>` para las solicitudes GET o POST. Hasta ahora, solo hemos enviado datos de texto mediante un formulario, pero también es posible enviar uno o más archivos mediante un formulario.

Al enviar archivos, debemos asegurarnos de que haya al menos dos atributos en el formulario: `method` y `enctype`. Es posible que aún necesites otros atributos, como `action`. Un formulario que admite cargas de archivos podría verse así:

```html
<form method="post" enctype="multipart/form-data">
```

Las cargas de archivos solo están disponibles para solicitudes POST. No son posibles con solicitudes GET, ya que sería imposible enviar todos los datos de un archivo a través de una URL. El atributo `enctype` debe configurarse para que el navegador sepa que debe enviar los datos del formulario como varias partes: una parte para los datos de texto del formulario y partes separadas para cada uno de los archivos que se han adjuntado al formulario. Esta codificación es transparente para el usuario; no saben cómo el navegador está codificando el formulario y no necesitan hacer nada diferente.

Para adjuntar archivos a un formulario, debes crear una entrada del tipo `file`. Puedes escribir manualmente el código HTML de esta manera:

```html
<input type="file" name="file-upload-name">
```

Cuando la entrada se renderiza en el navegador, se ve así cuando está vacía:

*Figura 8.5: Entrada de archivo vacía*

El título del botón puede ser diferente según tu navegador.

Al hacer clic en el botón *Browse…* (o *Examinar...*), se mostrará un cuadro de diálogo para abrir archivos:

*Figura 8.6: Explorador de archivos en macOS*

Después de seleccionar un archivo, el nombre del archivo se muestra en el campo:

*Figura 8.7: Entrada de archivo con cover.jpg seleccionado*

La Figura 8.7 muestra una entrada de archivo en la que se ha seleccionado un archivo llamado `cover.jpg`.

Vimos brevemente cómo se ven los campos de carga de archivos en el navegador. Comencemos a ver el lado de Django en el proceso de carga de archivos.

#### Trabajar con archivos subidos en una vista

Además de los datos de texto, si un formulario también contiene cargas de archivos, Django completará el atributo `request.FILES` con los archivos cargados. El atributo `request.FILES` es un objeto similar a un diccionario cuya clave es el atributo `name` asignado a la entrada de archivo.

En el ejemplo de formulario de la sección anterior, la entrada de archivo tenía el nombre `file-upload-name`. Por lo tanto, el archivo sería accesible en la vista usando `request.FILES["file-upload-name"]`.

Los objetos que contiene `request.FILES` son objetos de tipo archivo (específicamente una instancia de `django.core.files.uploadedfile.UploadedFile`), por lo que para usarlos, debes leer sus datos. Por ejemplo, para obtener el contenido de un archivo cargado en tu vista, puedes escribir lo siguiente:

```python
content = request.FILES["file-upload-name"].read()
```

Una acción más común es escribir el contenido del archivo en el disco. Cuando se cargan archivos, se almacenan en una ubicación temporal (en la memoria si pesan menos de 2.5 MB; de lo contrario, en un archivo temporal en el disco). Para almacenar los datos del archivo en una ubicación conocida, se debe leer el contenido y luego escribirlo en el disco en esa ubicación. `UploadedFile` tiene un método `chunks` que leerá los datos del archivo fragmento a fragmento para evitar que se use demasiada memoria al leer la totalidad del archivo de una sola vez.

Por lo tanto, en lugar de usar simplemente las funciones `read` y `write`, usa el método `chunks` para leer solo pequeños fragmentos del archivo en la memoria a la vez:

```python
with open("/path/to/output.jpg", "wb+") as output_file:
    uploaded_file = request.FILES["file-upload-name"]
    for chunk in uploaded_file.chunks():
        output_file.write(chunk)
```

Ten en cuenta que en algunos de los próximos ejemplos, nos referiremos a este código como la función `save_file_upload`. Asume que la función se define de la siguiente manera:

```python
def save_file_upload(upload, save_path):
    with open(save_path, "wb+") as output_file:
        for chunk in upload.chunks():
            output_file.write(chunk)
```

El código de ejemplo anterior podría luego refactorizarse para llamar a la función:

```python
uploaded_file = request.FILES["file-upload-name"]
save_file_upload(uploaded_file, "/path/to/output.jpg")
```

Cada objeto `UploadedFile` (la variable `uploaded_file` en los fragmentos de código de ejemplo anteriores) también contiene metadatos adicionales sobre el archivo cargado, como el nombre, el tamaño y el tipo de contenido del archivo. Los atributos que te resultarán más útiles son los siguientes:
- **size**: Como sugiere el nombre, este es el tamaño del archivo subido en bytes.
- **name**: Esto se refiere al nombre del archivo subido, por ejemplo, `image.jpg`, `file.txt`, `document.pdf`, etc. Este valor lo envía el navegador.
- **content_type**: El tipo de contenido (tipo MIME) del archivo subido, por ejemplo, `image/jpeg`, `text/plain`, `application/pdf`, etc. Al igual que `name`, este valor lo envía el navegador.
- **charset**: Esto se refiere al juego de caracteres o codificación de texto del archivo subido para archivos de texto. Esto será algo como UTF-8 o ASCII. Una vez más, este valor también lo determina y envía el navegador.

He aquí un ejemplo rápido de acceso a estos atributos (como dentro de una vista):

```python
upload = request.FILES["file-upload-name"]
size = upload.size
name = upload.name
content_type = upload.content_type
charset = upload.charset
```

#### Seguridad y confianza en los valores enviados por los navegadores

Como acabamos de describir, los valores de `name`, `content_type` y `charset` de `UploadedFile` los determina el navegador. Es importante considerar esto porque un usuario malintencionado podría enviar valores falsos en lugar de reales para disfrazar los archivos reales que se están cargando. Django no intenta determinar automáticamente el tipo de contenido o el juego de caracteres del archivo cargado, por lo que depende de que el cliente sea preciso cuando envía esta información.

Si manejamos manualmente el guardado de cargas de archivos sin las comprobaciones adecuadas, podría ocurrir un escenario como este:
1. Un usuario del sitio sube un ejecutable malicioso `malware.exe` pero envía el tipo de contenido `image/jpeg`.
2. Nuestro código verifica el tipo de contenido y lo considera seguro, por lo que guarda `malware.exe` en el archivo `MEDIA_ROOT`.
3. Otro usuario del sitio descarga lo que cree que es la imagen de la portada de un libro, pero en realidad es el ejecutable `malware.exe`. Abre el archivo y su computadora se infecta con malware.

Este escenario se ha simplificado; el archivo malicioso probablemente tendría un nombre que no fuera tan obvio (tal vez algo como `cover.jpg.exe`), pero se ha ilustrado el proceso general.

La forma en que elijas manejar la seguridad de tus cargas dependerá del caso de uso específico, pero para la mayoría de los casos, estos consejos te ayudarán:
- Cuando guardes el archivo en el disco, genera un nombre en lugar de usar el proporcionado por quien lo subió. Debes reemplazar la extensión del archivo con la que esperas. Por ejemplo, si un archivo se llama `cover.exe` pero el tipo de contenido es `image/jpeg`, guarda el archivo como `cover.jpg`. También puedes generar un nombre de archivo completamente aleatorio para mayor seguridad.
- Comprueba que la extensión del nombre de archivo coincida con el tipo de contenido. Este método no es infalible, ya que hay tantos tipos MIME que si estás manejando archivos poco comunes, es posible que no obtengas una coincidencia. El módulo integrado de Python `mimetypes` puede ayudarte aquí. Su función `guess_type` toma un nombre de archivo y devuelve una tupla de tipo MIME (tipo de contenido) y codificación. He aquí un breve fragmento que muestra su uso en una consola de Python:
  ```python
  >>> import mimetypes
  >>> mimetypes.guess_type('file.jpg')
  ('image/jpeg', None)
  >>> mimetypes.guess_type('text.html')
  ('text/html', None)
  >>> mimetypes.guess_type('unknownfile.abc')
  (None, None)
  >>> mimetypes.guess_type('archive.tar.gz')
  ('application/x-tar', 'gzip')
  ```
  Cualquier elemento de la tupla puede ser `None` si el tipo o la codificación no se pueden adivinar. Una vez importado a tu archivo mediante `import mimetypes`, puedes usarlo de esta manera en tu función de vista:
  ```python
  upload = request.FILES["file-upload-name"]
  mimetype, encoding = mimetypes.guess_type(upload.name)
  if mimetype != upload.content_type:
      raise TypeError("Mimetype doesn't match file extension.")
  ```
  Este método funcionará para tipos de archivos comunes, como imágenes, pero como se mencionó, muchos tipos poco comunes pueden devolver `None` para `mimetype`.
- Si esperas cargas de imágenes, usa la biblioteca Pillow para intentar abrir el archivo cargado como una imagen. Si no es una imagen válida, Pillow no podrá abrirla. Esto es lo que hace Django al usar `ImageField` para cargar imágenes. Mostraremos cómo usar esta técnica para abrir y manipular una imagen en el Ejercicio 8.05 – Carga de imágenes mediante formularios de Django.
- También puedes considerar el paquete de Python `python-magic`, que examina el contenido real de los archivos para intentar determinar su tipo. Se instala mediante `pip` y su proyecto de GitHub se encuentra aquí: [https://github.com/ahupp/python-magic](https://github.com/ahupp/python-magic). Una vez instalado e importado a tu archivo con `import magic`, puedes usarlo de esta manera en tu función de vista:
  ```python
  upload = request.FILES["field_name"]
  mimetype = magic.from_buffer(upload.read(2048), mime=True)
  ```
  Luego puedes verificar que `mimetype` esté en una lista de tipos permitidos.

Esta no es una lista definitiva de todas las formas de protección contra la carga de archivos maliciosos. El mejor enfoque dependerá del tipo de aplicación que estés creando. Podrías crear un sitio para alojar archivos arbitrarios, en cuyo caso no necesitarías ningún tipo de verificación de contenido.

Veamos ahora cómo podemos crear un formulario HTML y una vista que permita cargar archivos. Luego los almacenaremos dentro del directorio `media` y recuperaremos los archivos descargados en nuestro navegador.

#### Ejercicio 8.03 – Carga y descarga de archivos

En este ejercicio, agregarás un formulario con un campo de archivo a la plantilla `media-example.html`. Esto te permitirá cargar un archivo a la vista `media_example` usando tu navegador. También actualizarás la vista `media_example` para guardar el archivo en el directorio `MEDIA_ROOT` de modo que esté disponible para su descarga. Luego probarás que todo esto funciona descargando el archivo nuevamente:

1. En PyCharm, abre la plantilla `media-example.html` ubicada dentro de la carpeta `templates`. Dentro del elemento `<body>`, elimina el enlace `<a>` que se agregó en el paso 6 del Ejercicio 8.02 – Configuración de plantillas y uso de `MEDIA_URL` en plantillas. Reemplázalo con un elemento `<form>` (resaltado en el siguiente fragmento de código). Asegúrate de que la etiqueta de apertura tenga `method="post"` y `enctype="multipart/form-data"`:
   ```html
   </head>
   <body>
       <form method="post" enctype="multipart/form-data">
       </form>
   </body>
   ```
2. Inserta la etiqueta de plantilla `{% csrf_token %}` dentro del cuerpo de `<form>`.
3. Después de `{% csrf_token %}`, agrega un elemento `<input>`, con `type="file"` y `name="file_upload"`:
   ```html
   <input type="file" name="file_upload">
   ```
4. Finalmente, antes de la etiqueta de cierre `</form>`, agrega un elemento `<button>` con `type="submit"` y el contenido de texto `Submit`:
   ```html
   <button type="submit">Submit</button>
   ```
   El cuerpo de tu HTML ahora debería verse así:
   ```html
   <body>
       <form method="post" enctype="multipart/form-data">
           {% csrf_token %}
           <input type="file" name="file_upload">
           <button type="submit">Submit</button>
       </form>
   </body>
   ```
   Ahora, guarda y cierra el archivo.
5. Abre el archivo `views.py` de la aplicación `media_example`. Dentro de la vista `media_example`, agrega código para guardar el archivo subido en el directorio `MEDIA_ROOT`. Para esto, necesitas acceder a `MEDIA_ROOT` desde las configuraciones, así que importa las configuraciones de Django en la parte superior del archivo:
   ```python
   from django.conf import settings
   ```
6. El archivo subido solo debe guardarse si el método de solicitud es POST. Dentro de la vista `media_example`, agrega una declaración `if` para validar que `request.method` sea `POST`:
   ```python
   def media_example(request):
       if request.method == "POST":
           …
   ```
7. Dentro de la declaración `if` agregada en el paso anterior, genera la ruta de salida uniendo el nombre del archivo cargado al directorio `MEDIA_ROOT`. Luego, abre esta ruta en el modo `wb` e itera sobre el archivo cargado usando el método `chunks`. Finalmente, escribe cada fragmento en el archivo guardado:
   ```python
   def media_example(request):
       if request.method == 'POST':
           save_path = (settings.MEDIA_ROOT / request.FILES["file_upload"].name)
           with open(save_path, "wb") as output_file:
               for chunk in request.FILES["file_upload"].chunks():
                   output_file.write(chunk)
       return render(request, "media-example.html")
   ```
   Ten en cuenta que se accede al archivo cargado y a sus metadatos desde el diccionario `request.FILES` utilizando la clave que coincide con el nombre asignado a la entrada de archivo (en nuestro caso, es `file_upload`). Puedes guardar y cerrar `views.py`.
8. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego navega a `http://127.0.0.1:8000/media-example/`. Deberías ver el campo de carga de archivos y el botón Submit, como se puede ver aquí:
   *Figura 8.8: El formulario de carga de archivos*
9. Haz clic en *Browse…* (o el equivalente en tu navegador) y selecciona un archivo para cargar. El nombre del archivo aparecerá en la entrada de archivo. Luego, haz clic en Submit.
10. La página se recargará y el formulario volverá a estar vacío. Esto es normal: en segundo plano, el archivo debería haberse guardado.
11. Intenta descargar el archivo que subiste usando `MEDIA_URL`. En este ejemplo, se subió un archivo llamado `cover.jpg`. Se podrá descargar en `http://127.0.0.1:8000/media/cover.jpg`. Tu URL dependerá del nombre del archivo que hayas subido:
    *Figura 8.9: Archivo subido visible dentro de MEDIA_URL*
    Si cargas un archivo de imagen, un archivo HTML u otro tipo de archivo que tu navegador pueda mostrar, podrás verlo dentro del navegador. De lo contrario, tu navegador simplemente lo descargará en el disco nuevamente. En ambos casos, significa que la carga se realizó correctamente.
12. También puedes confirmar que la carga fue exitosa mirando dentro del directorio `media` en el directorio del proyecto `media_project`:
    *Figura 8.10: cover.jpg dentro del directorio media*
    La Figura 8.10 muestra el archivo `cover.jpg` dentro del directorio `media` en PyCharm.

En este ejercicio, agregaste un formulario HTML con `enctype` establecido en `multipart/form-data` para que permitiera la carga de archivos. Contenía una entrada de archivo para seleccionar un archivo para cargar. Luego agregaste una funcionalidad de guardado a la vista `media_example` para guardar el archivo cargado en el disco.

En la siguiente sección, veremos cómo simplificar la generación de formularios y agregar validación mediante formularios de Django para el manejo de la carga de archivos.

---

### Sección: Carga de archivos con formularios de Django

En el Capítulo 6 (*Formularios*), vimos cómo Django facilita la definición de formularios y su renderizado automático en HTML. En el ejemplo anterior, definimos nuestro formulario manualmente y escribimos el HTML. Podemos reemplazar esto con un formulario de Django e implementar la entrada de archivo con un constructor `FileField`.

Así es como se define `FileField` en un formulario:

```python
from django import forms

class ExampleForm(forms.Form):
    file_upload = forms.FileField()
```

El constructor `FileField` puede tomar los siguientes argumentos de palabra clave:
- **required**: Esto debe ser `True` para campos obligatorios y `False` si el campo es opcional.
- **max_length**: Esto se refiere a la longitud máxima del nombre del archivo que se está cargando.
- **allow_empty_file**: Un campo con este argumento se considera válido incluso si el archivo cargado está vacío (tiene un tamaño de 0).

Aparte de estos tres argumentos de palabra clave, el constructor también puede aceptar los argumentos estándar de `Field`, como `widget`. La clase de widget predeterminada para `FileField` es `ClearableFileInput`; esta es una entrada de archivo que puede mostrar una casilla de verificación que se puede marcar para enviar un valor nulo y borrar el archivo guardado en un campo de `Model`.

El uso de un formulario con un constructor `FileField` en una vista es similar a otros formularios, pero cuando el formulario se ha enviado (es decir, `request.method` es `POST`), entonces `request.FILES` también debe pasarse al constructor del formulario. Esto se debe a que Django necesita acceder a `request.FILES` para encontrar información sobre los archivos cargados al validar el formulario.

El flujo básico en una función de vista es, por tanto, similar a este:

```python
def view(request):
    if request.method == "POST":
        # instanciar el formulario con datos POST y archivos
        form = ExampleForm(request.POST, request.FILES)
        if form.is_valid():
            # procesar el formulario y guardar archivos
            form.save()
            return redirect("success-url")
    else:
        # instanciar un formulario vacío como hemos visto antes
        form = ExampleForm()
    # renderizar una plantilla, igual que para otros formularios
    return render(request, "template.html", {"form": form})
```

Al trabajar con formularios y archivos subidos, puedes interactuar con los archivos subidos accediendo a ellos a través de `request.FILES` o a través de `form.cleaned_data`; los valores devolverán el mismo objeto. En el ejemplo anterior, podríamos procesar el archivo subido de esta manera:

```python
if form.is_valid():
    save_file_upload("/path/to/save.jpg", request.FILES["file_upload"])
    return redirect("/success-url/")
```

O, dado que contienen el mismo objeto, puedes usar `form.cleaned_data`:

```python
if form.is_valid():
    save_file_upload("/path/to/save.jpg", form.cleaned_data["file_upload"])
    return redirect("/success-url/")
```

Los datos que se guardan serán los mismos.

En el Capítulo 6 (*Formularios*), experimentaste con formularios y los enviaste con valores no válidos. Cuando la página se actualizó para mostrar los errores del formulario, los datos que habías ingresado previamente se completaron cuando se recargó la página. Esto no ocurre con los campos de archivo; en su lugar, el usuario tendrá que navegar y seleccionar el archivo nuevamente si el formulario no es válido.

En el siguiente ejercicio, pondremos en práctica lo que hemos visto con `FileField` creando un formulario de ejemplo y luego modificando nuestra vista para guardar el archivo solo si el formulario es válido.

#### Ejercicio 8.04 – Carga de archivos con un formulario de Django

En el ejercicio anterior, creaste un formulario en HTML y lo usaste para subir un archivo a una vista de Django. Si intentabas enviar el formulario sin seleccionar un archivo, obtenías una pantalla de excepción de Django. No realizaste ninguna validación en el formulario, por lo que este método es bastante frágil.

En este ejercicio, crearás un formulario de Django con `FileField`, lo que te permitirá usar funciones de validación de formularios para hacer que la vista sea más robusta, además de reducir la cantidad de código:

1. En PyCharm, dentro de la aplicación `media_example`, crea un nuevo archivo llamado `forms.py`. Se abrirá automáticamente. Al inicio del archivo, importa la biblioteca de formularios de Django:
   ```python
   from django import forms
   ```
2. Luego, crea una subclase de `forms.Form` y llámala `UploadForm`. Agrégale un campo `FileField` llamado `file_upload`. Tu clase debería tener este código:
   ```python
   class UploadForm(forms.Form):
       file_upload = forms.FileField()
   ```
   Puedes guardar y cerrar este archivo.
3. Abre el archivo `views.py` de la aplicación `form_example`. Al inicio del archivo, justo debajo de las declaraciones de importación existentes, deberás importar tu nueva clase de esta manera:
   ```python
   from .forms import UploadForm
   ```
4. Si estás en la rama POST de la vista, `UploadForm` debe instanciarse tanto con `request.POST` como con `request.FILES`. Si no pasas `request.FILES`, la instancia del formulario no podrá acceder a los archivos subidos. Debajo de la comprobación `if request.method == "POST"`, instancia `UploadForm` con estos dos argumentos:
   ```python
   form = UploadForm(request.POST, request.FILES)
   ```
5. Las líneas existentes que definen `save_path` y almacenan el contenido del archivo se pueden conservar, pero deben tener una sangría de un bloque y colocarse dentro de una verificación de validez del formulario, de modo que solo se ejecuten si el formulario es válido. Agrega la línea `if form.is_valid():` y luego sangra las otras líneas para que el código se vea así:
   ```python
   if form.is_valid():
       save_path = os.path.join(settings.MEDIA_ROOT, request.FILES["file_upload"].name)
       with open(save_path, "wb") as output_file:
           for chunk in request.FILES["file_upload"].chunks():
               output_file.write(chunk)
   ```
6. Como ahora estás usando un formulario, puedes acceder al archivo cargado a través del formulario. Reemplaza los usos de `request.FILES["file_upload"]` con `form.cleaned_data["file_upload"]`:
   ```python
   if form.is_valid():
       save_path = settings.MEDIA_ROOT / form.cleaned_data["file_upload"].name
       with open(save_path, "wb") as output_file:
           for chunk in form.cleaned_data["file_upload"].chunks():
               output_file.write(chunk)
   ```
7. Finalmente, agrega una rama `else` para manejar solicitudes que no sean POST, que simplemente instancia un formulario sin ningún argumento:
   ```python
   if request.method == "POST":
       …
   else:
       form = UploadForm()
   ```
8. Agrega un argumento de diccionario de contexto a la llamada `render` y establece la variable `form` en la clave `form`:
   ```python
   return render(request, "media-example.html", {"form": form})
   ```
   Ahora puedes guardar y cerrar este archivo.
9. Finalmente, abre la plantilla `media-example.html` y elimina el archivo `<input>` definido manualmente. Reemplázalo con `form`, renderizado usando el método `as_p` (resaltado):
   ```django
   <body>
       <form method="post" enctype="multipart/form-data">
           {% csrf_token %}
           {{ form.as_p }}
           <button type="submit">Submit</button>
       </form>
   </body>
   ```
   No debes cambiar ninguna otra parte del archivo. Puedes guardar y cerrar este archivo.
10. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego navega a `http://127.0.0.1:8000/media-example/`. Deberías ver el campo de carga de archivos y el botón Submit de la siguiente manera:
    *Figura 8.11: El formulario de carga de archivos de Django renderizado en el navegador*
11. Como estamos usando un formulario de Django, obtenemos su validación integrada automáticamente. Si intentas enviar el formulario sin seleccionar un archivo, tu navegador debería evitarlo y mostrar un error, como se puede ver aquí:
    *Figura 8.12: Envío de formulario impedido por el navegador*
12. Finalmente, repite la prueba de carga que realizaste en el Ejercicio 8.03 – Carga y descarga de archivos, seleccionando un archivo y enviando el formulario. Luego deberías poder recuperar el archivo usando `MEDIA_URL`. En este caso, se está cargando nuevamente un archivo llamado `cover.jpg` (consulta la siguiente captura de pantalla):
    *Figura 8.13: Carga de un archivo llamado cover.jpg*
13. Luego puedes recuperar el archivo en `http://127.0.0.1:8000/media/cover.jpg`, y puedes verlo en el navegador de la siguiente manera:
    *Figura 8.14: El archivo cargado mediante el formulario de Django también es visible en el navegador*

En este ejercicio, reemplazamos un formulario creado manualmente con un formulario de Django que contiene `FileField`. Instanciamos el formulario en la vista pasando tanto `request.POST` como `request.FILES`. Luego usamos el método estándar `is_valid` para verificar la validez del formulario y solo guardamos el archivo cargado si el formulario era válido. Probamos la carga de archivos y vimos que podíamos recuperar archivos cargados usando `MEDIA_URL`.

En la siguiente sección, veremos `ImageField`, que es como `FileField` pero específicamente para imágenes.

---

### Sección: Carga de imágenes con formularios de Django

Si deseas trabajar con imágenes en Python, la biblioteca más común que utilizarás se llama Pillow, y esta es la biblioteca que utiliza Django para validar imágenes. El veterano desarrollador de Python Fredrik Lundh lanzó por primera vez la Python Imaging Library (PIL) en 1995. No se mantuvo actualizada y, finalmente, se creó una bifurcación (*fork*) de la biblioteca que todavía se mantiene, llamada Pillow. El proyecto original se suspendió, pero el desarrollo en una bifurcación del proyecto llamada Pillow ha continuado. Pillow se esfuerza por mantener un alto grado de compatibilidad con el paquete PIL original. Por esta razón, Pillow contiene un espacio de nombres muy similar a PIL, y el paquete todavía se llama PIL cuando se instala. Por ejemplo, el objeto `Image` se importa desde PIL de la siguiente manera:

```python
from PIL import Image
```

Los términos Python Imaging Library, PIL y Pillow se suelen utilizar indistintamente. Puedes asumir que si alguien se refiere a PIL, se refiere a la biblioteca Pillow más reciente.

Pillow proporciona varios métodos para recuperar datos sobre imágenes o manipularlas. Puedes conocer el ancho y el alto de las imágenes y escalarlas, recortarlas y aplicarles transformaciones. Hay demasiadas operaciones disponibles para cubrirlas en este capítulo, por lo que solo presentaremos un ejemplo simple (escalar una imagen), que utilizarás en el siguiente ejercicio.

Dado que las imágenes son uno de los tipos de archivos más comunes que un usuario puede querer cargar, Django también incluye `ImageField`. Esto se comporta de manera similar a `FileField`, pero también valida automáticamente que los datos correspondan a un archivo de imagen. Esto ayuda a mitigar los problemas de seguridad en los que esperamos una imagen pero el usuario carga un archivo malicioso.

Un `UploadedFile` de un `ImageField` tiene todos los mismos atributos y métodos que `FileField` (`size`, `content_type`, `name`, `chunks()`, etc.), pero agrega un atributo adicional: `image`. Esta es una instancia del objeto `Image` de PIL que se utiliza para verificar que el archivo que se está cargando sea una imagen válida.

Después de verificar que el formulario sea válido, el objeto `Image` de PIL subyacente se cierra. Esto se hace para liberar memoria y evitar que el proceso de Python mantenga demasiados archivos abiertos, lo que podría causar problemas de rendimiento. Lo que esto significa para el desarrollador es que puedes acceder a algunos de los metadatos sobre la imagen (como el ancho, el alto y el formato), pero no puedes acceder a los datos reales de la imagen sin volver a abrirla.

Para ilustrarlo, tendremos un formulario con `ImageField`, llamado `picture`:

```python
class ExampleForm(forms.Form):
    picture = forms.ImageField()
```

Dentro de la función de vista, se puede acceder al campo `picture` en `cleaned_data` del formulario:

```python
if form.is_valid():
    picture_field = form.cleaned_data["picture"]
```

Luego, se puede recuperar el objeto `Image` del campo de imagen:

```python
image = picture_field.image
```

Ahora que tenemos una referencia a la imagen en la vista, podemos obtener algunos metadatos:

```python
w = image.width # un entero, p. ej. 600
h = image.height # también un entero, p. ej. 420
f = image.format # el formato de la imagen como cadena,
                 # p. ej. "PNG"
```

Django también actualizará automáticamente el atributo `content_type` de `UploadedFile` al tipo correcto para el campo de imagen. Esto sobrescribe el valor que el navegador envió al cargar el archivo.

Intentar usar un método que acceda a los datos reales de la imagen (en lugar de solo a los metadatos) provocará que se genere una excepción. Esto se debe a que Django ya ha cerrado el archivo de imagen subyacente.

Por ejemplo, el siguiente fragmento de código generará `AttributeError`:

```python
image.getdata()
```

En su lugar, debemos volver a abrir la imagen. Los datos de la imagen se pueden abrir con la referencia de `ImageField` después de importar la clase `Image`:

```python
from PIL import Image
image = Image.open(picture_field)
```

Ahora que la imagen se ha abierto, puedes realizar operaciones sobre ella. En la siguiente sección, veremos un ejemplo simple: redimensionar la imagen cargada.

#### Redimensionar una imagen con Pillow

Pillow admite muchas operaciones que podrías querer realizar en una imagen antes de guardarla. No podemos explicarlas todas en este libro, por lo que solo usaremos una operación común: redimensionar una imagen a un tamaño específico antes de guardarla. Esto nos ayudará a ahorrar espacio de almacenamiento y mejorar la velocidad de descarga. Por ejemplo, un usuario puede cargar imágenes de portada grandes en Bookr que son más grandes de lo necesario para nuestros propósitos. Al guardar el archivo (escribiéndolo de nuevo en el disco), debemos especificar el formato a utilizar. Podríamos determinar el tipo de imagen que se cargó con varios métodos (como verificar `content_type` del archivo cargado o `format` del objeto `Image`), pero en nuestro ejemplo, siempre lo guardaremos como un archivo JPEG.

La clase `Image` de PIL tiene un método `thumbnail` que cambiará el tamaño de una imagen a un tamaño máximo manteniendo la relación de aspecto (*aspect ratio*). Por ejemplo, podríamos establecer un tamaño máximo de 50 píxeles (px) por 50 px. Una imagen de 200 px por 100 px se redimensionaría a 50 px por 25 px; la relación de aspecto se mantiene estableciendo la dimensión máxima en 50 px. Cada dimensión se escala por un factor de 0.25:

```python
from PIL import Image
size = 50, 50 # una tupla de ancho, alto a la que redimensionar
image = Image.open(image_field) # abrir la imagen como antes
image.thumbnail(size) # realizar el cambio de tamaño
```

En este punto, el cambio de tamaño se ha realizado únicamente en la memoria. El cambio no se guarda en el disco hasta que se llama al método `save`, de esta manera:

```python
image.save("path/to/file.jpg")
```

El formato de salida se determina automáticamente a partir de la extensión de archivo utilizada, en este caso, JPEG. El método `save` también puede tomar un argumento `format` para anularlo, como en este ejemplo:

```python
image.save("path/to/file.png", "JPEG")
```

A pesar de tener la extensión `.png`, el formato se especifica como JPEG, por lo que la salida estará en formato JPEG. Como puedes imaginar, esto puede ser muy confuso, por lo que podrías decidir limitarte a especificar solo la extensión.

En el siguiente ejercicio, cambiaremos el `UploadForm` con el que hemos estado trabajando para usar `ImageField` en lugar de `FileField`, luego implementaremos el cambio de tamaño de una imagen cargada antes de guardarla en el directorio `media`.

#### Ejercicio 8.05 – Carga de imágenes mediante formularios de Django

En este ejercicio, actualizarás la clase `UploadForm` que creaste en el Ejercicio 8.04 – Carga de archivos con un formulario de Django, para usar `ImageField` en lugar de `FileField` (esto implicará simplemente cambiar la clase del campo). Luego verás que el formulario se renderiza igual en el navegador. A continuación, intentarás cargar algunos archivos que no sean de imagen y verás cómo Django valida el formulario para rechazarlos. Finalmente, actualizarás tu vista para usar PIL para redimensionar la imagen antes de guardarla y luego probarla en acción:

1. Abre el archivo `forms.py` de la aplicación `media_example`. En la clase `UploadForm`, cambia `file_upload` para que sea una instancia de `ImageField` en lugar de `FileField`. Después de la actualización, `UploadForm` debería verse así:
   ```python
   class UploadForm(forms.Form):
       file_upload = forms.ImageField()
   ```
   Guarda y cierra el archivo.
2. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego navega a `http://127.0.0.1:8000/media-example/`. Deberías ver el formulario renderizado y se verá idéntico a cuando usamos `FileField` (mira la siguiente captura de pantalla):
   *Figura 8.15: ImageField se ve igual que FileField*
3. Notarás la diferencia cuando intentes subir un archivo que no sea de imagen. Haz clic en el botón *Browse…* e intenta seleccionar un archivo que no sea de imagen. Según tu navegador o sistema operativo, es posible que no puedas seleccionar nada más que un archivo de imagen, como en la Figura 8.16:
   *Figura 8.16: Solo se pueden seleccionar archivos de imagen*
   Es posible que tu navegador te permita seleccionar una imagen pero muestre un error en el formulario después de la selección. O tu navegador puede permitirte seleccionar un archivo y enviar el formulario, y Django generará `ValidationError`. De cualquier manera, puedes estar seguro de que en tu vista, el método `is_valid` del formulario solo devolverá `True` si se ha cargado una imagen.
   No es necesario que pruebes subir un archivo en este momento, ya que el resultado sería el mismo que en el Ejercicio 8.04 – Carga de archivos con un formulario de Django.
4. Lo primero que deberás hacer es asegurarte de que la biblioteca Pillow esté instalada. En una terminal (asegurándote de que tu entorno virtual esté activado), ejecuta el siguiente comando:
   ```bash
   pip3 install pillow
   ```
   En Windows, esto es `pip install pillow`. Obtendrás una salida como en la Figura 8.17:
   *Figura 8.17: pip3 instalando Pillow*
   Si Pillow ya está instalado, verás el mensaje de salida `Requirement already satisfied`.
5. Ahora, podemos actualizar la vista `media_example` para redimensionar la imagen antes de guardarla. Vuelve a PyCharm y abre el archivo `views.py` de la aplicación `media_example`, luego importa la clase `Image` de PIL agregando esta línea de importación cerca de la parte superior del archivo:
   ```python
   from PIL import Image
   ```
6. Ve a la vista `media_example`. Debajo de la línea que genera `save_path`, elimina las tres líneas que abren el archivo de salida, iteran sobre el archivo subido y escriben sus fragmentos. Reemplaza esto con el código que abre el archivo subido con PIL, le cambia el tamaño y luego lo guarda:
   ```python
   image = Image.open(form.cleaned_data["file_upload"])
   image.thumbnail((50, 50))
   image.save(save_path)
   ```
   La primera línea crea una instancia de `Image` abriendo el archivo cargado, la siguiente realiza la conversión a miniatura (a un tamaño máximo de 50 px por 50 px) y la tercera línea guarda el archivo en la misma ruta de guardado que hemos estado generando en ejercicios anteriores. Puedes guardar el archivo.
7. El servidor de desarrollo de Django aún debería estar ejecutándose desde el paso 2, pero debes iniciarlo si no es así. Luego, navega a `http://127.0.0.1:8000/media-example/`. Verás el conocido `UploadForm`. Selecciona una imagen y envía el formulario. Si la carga y el cambio de tamaño se realizaron correctamente, el formulario se actualizará y volverá a estar vacío.
8. Visualiza la imagen cargada utilizando `MEDIA_URL`. Por ejemplo, un archivo llamado `cover.jpg` se podrá descargar desde `http://127.0.0.1:8000/media/cover.jpg`. Deberías ver que la imagen se ha redimensionado para tener una dimensión máxima de solo 50 px:
   *Figura 8.18: La portada redimensionada*
   Si bien una miniatura de este tamaño puede no ser tan útil, al menos nos permite estar seguros de que el cambio de tamaño de la imagen ha funcionado correctamente.

En este ejercicio, cambiamos `FileField` en `UploadForm` por `ImageField`. Vimos que el navegador no nos permitía subir nada más que imágenes. Luego agregamos código a la vista `media_example` para cambiar el tamaño de la imagen cargada usando PIL.

Hemos fomentado el uso de un servidor web independiente para servir archivos estáticos y multimedia por razones de rendimiento. Sin embargo, en algunos casos, es posible que desees utilizar Django para servir archivos, por ejemplo, para proporcionar autenticación antes de permitir el acceso. En la siguiente sección, analizaremos cómo utilizar Django para servir archivos multimedia.

---

### Sección: Servir archivos subidos (y otros) usando Django

A lo largo de este capítulo y del Capítulo 5 (*Servicio de Archivos Estáticos*), hemos desaconsejado servir archivos utilizando Django. Esto se debe a que ocuparía innecesariamente un proceso de Python solo para servir un archivo, algo que el servidor web es capaz de manejar. Desafortunadamente, los servidores web no suelen proporcionar control de acceso dinámico, es decir, permitir que solo los usuarios autenticados descarguen un archivo. Dependiendo del servidor web utilizado en producción, es posible que puedas hacer que se autentique contra Django y luego sirva el archivo él mismo; sin embargo, la configuración específica de servidores web específicos está fuera del alcance de este libro.

Un enfoque que puedes adoptar es especificar un subdirectorio de tu directorio `MEDIA_ROOT` y hacer que tu servidor web impida el acceso solo a esta carpeta específica. Cualquier medio protegido debe almacenarse dentro de ella. Si haces esto, solo Django podrá leer los archivos del interior. Por ejemplo, tu servidor web podría servir todo lo que se encuentra en el directorio `MEDIA_ROOT`, excepto un directorio `MEDIA_ROOT/protected`.

Otro enfoque sería configurar una vista de Django para servir un archivo específico desde el disco. La vista determinará la ruta del archivo en el disco que se enviará y luego lo enviará mediante la clase `FileResponse`. La clase `FileResponse` toma un identificador de archivo abierto como argumento e intenta determinar el tipo de contenido correcto a partir del contenido del archivo. Django cerrará el identificador de archivo una vez completada la solicitud.

La función de vista aceptará la solicitud y una ruta relativa al archivo que se descargará como parámetros. Esta ruta relativa es la ruta dentro de la carpeta `MEDIA_ROOT/protected`.

En nuestro caso, simplemente verificaremos si el usuario es anónimo (no ha iniciado sesión). Haremos esto verificando la propiedad `request.user.is_anonymous`. Si no ha iniciado sesión, generaremos una excepción `django.core.exceptions.PermissionDenied`, que devuelve una respuesta HTTP 403 Forbidden al navegador. Esto detendrá la ejecución de la vista y no devolverá ningún archivo:

```python
import os.path
from django.conf import settings
from django.http import FileResponse
from django.core.exceptions import PermissionDenied

def download_view(request, relative_path):
    if request.user.is_anonymous:
        raise PermissionDenied
    full_path = os.path.join(settings.MEDIA_ROOT, "protected", relative_path)
    file_handle = open(full_path, "rb")  # Django envía el archivo y luego cierra el identificador
    return FileResponse(file_handle)
```

El mapeo de URL a esta vista podría ser así, utilizando el convertidor de ruta `path` dentro de tu archivo `urls.py`:

```python
urlpatterns = [
    …
    path("downloads/<path:relative_path>", views.download_view)
]
```

Hay muchas formas en las que puedes optar por implementar una vista que envíe archivos. Lo importante es que utilices la clase `FileResponse`, que está diseñada para transmitir el archivo al cliente en fragmentos en lugar de cargarlo todo en la memoria. Esto reducirá la carga en el servidor y disminuirá el impacto en el uso de recursos si debes recurrir al envío de archivos con Django.

Ahora que sabemos cómo cargar y procesar imágenes, veamos cómo almacenarlas en los modelos.

#### Almacenar archivos en instancias de modelos

Hasta ahora, hemos gestionado manualmente la carga y el guardado de archivos. También puedes asociar un archivo con una instancia de modelo asignando la ruta en la que se guardó a un `CharField`. Sin embargo, como ocurre con gran parte de Django, esta capacidad (y más) ya la proporciona la clase `models.FileField`. Una instancia de `FileField` en realidad no almacena los datos del archivo; en su lugar, almacena la ruta donde se guarda el archivo (como lo haría `CharField`), pero también proporciona métodos auxiliares. Estos métodos ayudan a cargar archivos (para que no tengas que abrirlos manualmente) y a generar rutas de disco para ti según el ID de la instancia (u otros atributos).

`FileField` puede aceptar dos argumentos opcionales específicos en su constructor (así como los argumentos básicos de `Field`, como `required`, `unique`, `help_text`, etc.):
- **max_length**: Al igual que `max_length` en el `ImageField` del formulario, esta es la longitud máxima permitida para el nombre del archivo.
- **upload_to**: El argumento `upload_to` tiene tres comportamientos diferentes según el tipo de variable que se le pase. Su uso más simple es con una cadena o un objeto `pathlib.Path`. La ruta simplemente se anexa a `MEDIA_ROOT`.

En este ejemplo, `upload_to` simplemente se define como una cadena:

```python
class ExampleModel(models.Model):
    file_field = models.FileField(upload_to="files/")
```

Los archivos guardados en `FileField` se almacenarán en el directorio `MEDIA_ROOT/files`.

También puedes lograr el mismo resultado utilizando una instancia de `pathlib.Path`:

```python
import pathlib

class ExampleModel(models.Model):
    file_field = models.FileField(upload_to=pathlib.Path("files/"))
```

La siguiente forma de usar `upload_to` es con una cadena que contiene las directivas de formato `strftime` (por ejemplo, `%Y` para sustituir el año actual, `%m` para el mes actual y `%d` para el día actual del mes). La lista completa de estas directivas es extensa y se puede encontrar en [https://docs.python.org/3/library/time.html#time.strftime](https://docs.python.org/3/library/time.html#time.strftime). Django interpolará automáticamente estos valores al guardar el archivo.

Por ejemplo, supongamos que definiste el modelo y `FileField` de esta manera:

```python
class ExampleModel(models.Model):
    file_field = models.FileField(upload_to="files/%Y/%m/%d/")
```

Para el primer archivo subido en un día específico, Django crearía la estructura de directorios para ese día. Por ejemplo, para el primer archivo cargado el 1 de enero de 2025, Django crearía el directorio `MEDIA_ROOT/2025/01/01` y luego almacenaría el archivo cargado allí. El siguiente archivo (y todos los posteriores) cargados el mismo día también se almacenarían en ese directorio. De manera similar, el 2 de enero de 2025, Django creará el directorio `MEDIA_ROOT/2025/01/02` y los archivos se almacenarán allí.

Si tienes muchos miles de archivos cargados todos los días, incluso podrías hacer que los archivos se dividan aún más incluyendo la hora y el minuto en el argumento `upload_to` (`upload_to="files/%Y/%m/%d/%H/%M/"`). Sin embargo, esto puede no ser necesario si solo tienes un pequeño volumen de cargas.

Al utilizar este método del argumento `upload_to`, puedes hacer que Django segregue automáticamente las cargas y evite que se almacenen demasiados archivos dentro de un solo directorio (lo cual puede ser difícil de administrar).

El método final para usar `upload_to` es pasar una función que se llamará para generar la ruta de almacenamiento. Ten en cuenta que esto es diferente de los otros usos de `upload_to`, ya que debe generar la ruta completa, incluido el nombre del archivo, en lugar de solo el directorio. La función toma dos argumentos: `instance` y `filename`. El argumento `instance` es la instancia del modelo a la que está adjunto `FileField`, y `filename` es el nombre del archivo cargado.

He aquí una función de ejemplo que toma los dos primeros caracteres de un nombre de archivo para generar el directorio de guardado. Esto significa que cada archivo cargado se agrupará en directorios principales, lo que puede ayudar a organizar los archivos y evitar que haya demasiados en un solo directorio:

```python
def user_grouped_file_path(instance, filename):
    return "/".join([instance.username, filename[0].lower(), filename[1].lower(), filename])
```

Si se llama a esta función con el nombre de archivo `Test.jpg`, devolverá `<username>/t/e/test.jpg`. Si se llama con `example.txt`, devolverá `<username>/e/x/example.txt`, y así sucesivamente. `username` se recupera de la instancia que se está guardando. Para ilustrar, aquí hay un modelo con `FileField` que usa esta función. También tiene un `username`, que es un `CharField`:

```python
class ExampleModel(models.Model):
    file_field = models.FileField(upload_to=user_grouped_file_path)
    username = models.CharField(unique=True)
```

Puedes usar cualquier atributo de la instancia en la función `upload_to`, pero ten en cuenta que si esta instancia está en proceso de creación, la función de guardado de archivos se llamará antes de que se guarde en la base de datos. Por lo tanto, algunos de los atributos generados automáticamente en la instancia (como `id`/`pk`) aún no estarán completos y no deberían usarse para generar una ruta.

Cualquiera que sea la ruta que devuelva la función `upload_to`, se añade a `MEDIA_ROOT`, por lo que los archivos cargados se guardarían en `MEDIA_ROOT/<username>/t/e/test.jpg` y `MEDIA_ROOT/<username>/e/x/example.txt`, respectivamente.

Ten en cuenta que `user_grouped_file_path` es solo una función ilustrativa que se ha mantenido intencionalmente corta, por lo que no funcionará correctamente con nombres de archivo de un solo carácter o si el nombre de usuario tiene caracteres no válidos. Por ejemplo, si el nombre de usuario tiene un carácter `/`, esto actuaría como un separador de directorios en la ruta generada.

Ahora bien, hemos profundizado en la configuración de `FileField` en un modelo, pero ¿cómo guardamos realmente un archivo subido en él? Es tan fácil como asignar el archivo subido al atributo del modelo, como lo harías con cualquier tipo de valor. He aquí un ejemplo rápido con una vista y el `ExampleModel` simple que estábamos usando como ejemplo anteriormente en esta sección:

```python
class ExampleModel(models.Model):
    file_field = models.FileField(upload_to="files/")

def view(request):
    if request.method == "POST":
        # Crear una nueva instancia de ExampleModel
        m = ExampleModel()
        m.file_field = request.FILES["uploaded_file"]
        m.save()
        return render(request, "template.html")
```

En este ejemplo, creamos un nuevo `ExampleModel` y asignamos el archivo subido (llamado `uploaded_file` en el formulario) a su atributo `file_field`. Cuando guardamos la instancia del modelo, Django escribió automáticamente el archivo con su nombre en la ruta del directorio `upload_to`. Si el archivo subido tenía el nombre `image.jpg`, la ruta de guardado sería `MEDIA_ROOT/upload_to/image.jpg`.

Podríamos haber actualizado con la misma facilidad el campo de archivo en un modelo existente o haber usado un formulario (validándolo antes de guardarlo). He aquí otro ejemplo simple que demuestra esto:

```python
class ExampleForm(forms.Form):
    uploaded_file = forms.FileField()

def view(request, model_pk):
    form = ExampleForm(request.POST, request.FILES)
    if form.is_valid():
        # Obtener una instancia de modelo existente
        m = ExampleModel.object.get(pk=model_pk)
        # almacenar el archivo subido en la instancia
        m.file_field = form.cleaned_data["uploaded_file"]
        m.save()
        return render(request, "template.html")
```

Puedes ver que actualizar `FileField` en una instancia de modelo existente es el mismo proceso que configurarlo en una nueva instancia, y si eliges usar un formulario de Django o simplemente acceder a `request.FILES` directamente, el proceso es igual de simple.

Ahora veremos cómo almacenar imágenes en modelos usando `ImageField`.

#### Almacenar imágenes en instancias de modelos

Si bien `FileField` puede almacenar cualquier tipo de archivo, incluidas imágenes, también existe `ImageField`. Como era de esperar, esto es solo para almacenar imágenes. La relación entre los campos `forms.FileField` y `forms.ImageField` de los formularios es similar a la que existe entre `models.FileField` y `models.ImageField`, lo que significa que `ImageField` extiende `FileField` y agrega métodos adicionales para trabajar con imágenes.

El constructor `ImageField` toma los mismos argumentos que `FileField` y agrega dos argumentos opcionales adicionales:
- **height_field**: Este es el nombre del campo en el modelo que se actualizará con la altura de la imagen cada vez que se guarde la instancia del modelo.
- **width_field**: Esta es la contraparte de ancho de `height_field`: el campo que almacena el ancho de la imagen que se actualiza cada vez que se guarda la instancia del modelo.

Ambos argumentos son opcionales, pero los campos que nombran deben existir si se usan. Es decir, es válido tener `height_field` o `width_field` sin configurar, pero si se establecen con el nombre de un campo que no existe, se producirá un error. El propósito de esto es ayudar a buscar en la base de datos archivos de una dimensión particular.

He aquí un modelo de ejemplo que usa `ImageField`, que actualiza los campos de dimensiones de la imagen:

```python
class ExampleModel(models.Model):
    image = models.ImageField(
        upload_to="images/%Y/%m/%d/",
        height_field="image_height",
        width_field="image_width")
    image_height = models.IntegerField()
    image_width = models.IntegerField()
```

Observa que `ImageField` está utilizando el parámetro `upload_to` con directivas de formato de fecha que se actualizan al guardar. El comportamiento de `upload_to` es idéntico al de `FileField`.

Al guardar una instancia de `ExampleModel`, su campo `image_height` se actualizará con la altura de la imagen y `image_width` con el ancho de la imagen.

No mostraremos ejemplos para configurar los valores de `ImageField` en una vista, ya que el proceso es el mismo que para un `FileField` normal.

#### Trabajar con FieldFile

Cuando accedes al atributo `FileField` o `ImageField` de una instancia de modelo, no obtendrás un objeto de archivo nativo de Python. En su lugar, trabajarás con un objeto `FieldFile`. La clase `FieldFile` es un contenedor alrededor del archivo que agrega métodos adicionales. Sí, puede resultar confuso tener clases llamadas `FileField` y `FieldFile`.

La razón por la que Django usa `FieldFile` en lugar de solo un objeto de archivo es doble. Primero, agrega métodos adicionales para abrir, leer, eliminar y generar la URL del archivo. En segundo lugar, proporciona una abstracción para permitir el uso de motores de almacenamiento alternativos.

#### Motores de almacenamiento personalizados

Analizamos los motores de almacenamiento personalizados en el Capítulo 5 (*Servicio de Archivos Estáticos*), con respecto al almacenamiento de archivos estáticos. No examinaremos los motores de almacenamiento personalizados en detalle para archivos multimedia, ya que el código descrito en el Capítulo 5 para archivos estáticos también se aplica a archivos multimedia. Lo importante a tener en cuenta es que el motor de almacenamiento que estás utilizando se puede cambiar sin actualizar el resto de tu código. Esto significa que puedes tener tus archivos multimedia almacenados en tu disco local durante el desarrollo y luego guardarlos en una CDN cuando tu aplicación se despliegue en producción.

La clase `storage_engine` predeterminada se puede configurar con `DEFAULT_FILE_STORAGE` en `settings.py`. El motor de almacenamiento también se puede especificar campo por campo (para `FileField` o `ImageField`) con el argumento `storage`, como en este ejemplo:

```python
storage_engine = CustomStorageEngine()

class ExampleModel(models.Model):
    image_field = ImageField(storage=storage_engine)
```

Esto demuestra lo que realmente sucede cuando cargas o recuperas un archivo. Django delega en el motor de almacenamiento para escribirlo o leerlo, respectivamente. Esto sucede incluso al guardar en el disco; sin embargo, esto está automatizado y es invisible para el usuario.

#### Leer un FieldFile almacenado

Ahora que hemos aprendido sobre los motores de almacenamiento personalizados, veamos cómo leer desde `FieldFile`. En las secciones anteriores, vimos cómo configurar el archivo en la instancia del modelo. Leer los datos nuevamente es igual de fácil: tenemos un par de métodos diferentes que pueden ayudarnos según nuestro caso de uso.

En los siguientes fragmentos de código, asumamos que estamos dentro de una vista y que hemos recuperado nuestra instancia de modelo de alguna manera, y que está almacenada en una variable `m`:

```python
m = ExampleModel.object.get(pk=model_pk)
```

Podemos leer todos los datos del archivo con el método `read`:

```python
data = m.file_field.read()
```

O podemos abrir manualmente el archivo con el método `open`. Esto podría ser útil si queremos escribir nuestros propios datos generados en el archivo:

```python
with m.file_field.open("wb") as f:
    chunk = f.write(b"test")  # escribir bytes en el archivo
```

Si queremos leer el archivo en fragmentos, podemos usar el método `chunks`. Esto funciona igual que leer fragmentos del archivo cargado, como vimos anteriormente:

```python
for chunk in m.file_field.chunks():
    # asumimos que este método está definido en alguna parte
    write_chunk(open_file, chunk)
```

También podemos abrir manualmente el archivo nosotros mismos utilizando su atributo `path`:

```python
open(m.file_field.path)
```

Si queremos transmitir `FileField` para su descarga, la mejor manera es usar la clase `FileResponse`, como vimos anteriormente. Combina esto con el método `open` en `FileField`. Ten en cuenta que si solo estamos intentando servir un archivo multimedia, solo debemos implementar una vista para hacer esto si estamos intentando restringir el acceso al archivo. De lo contrario, deberíamos simplemente servir el archivo usando `MEDIA_URL` y permitir que el servidor web maneje la solicitud. He aquí cómo escribiríamos `download_view` para usar `FileField` en lugar de la ruta especificada manualmente:

```python
def download_view(request, model_pk):
    if request.user.is_anonymous:
        raise PermissionDenied
    m = ExampleModel.objects.get(pk=model_pk)
    # Django envía el archivo y luego cierra el identificador
    return FileResponse(m.file_field.open())
```

Django abre la ruta correcta y la cierra después de la respuesta. Django también intentará determinar el tipo MIME correcto para el archivo. Asumimos que `FileField` tiene su atributo `upload_to` configurado en un directorio protegido al que el servidor web impide el acceso directo.

#### Almacenar archivos o contenido existente en FileField

Hemos visto cómo almacenar un archivo cargado en un campo de imagen; simplemente asígnalo al campo de esta manera:

```python
m.file_field = request.FILES["file_upload"]
```

Pero, ¿cómo podemos establecer el valor del campo en el de un archivo existente que ya tengamos en el disco? Podrías pensar que puedes usar un objeto de archivo estándar de Python, pero esto no funcionará:

```python
# No hagas esto
m.file_field = open("/path/to/file.txt", "rb")
```

También puedes intentar configurar el archivo usando algún contenido:

```python
# No hagas esto
m.file_field = "new file content"
```

Esto tampoco funcionará.

En su lugar, debes utilizar el método `save` de `FileField`, que acepta una instancia de un objeto `File` de Django o un objeto `ContentFile` (las rutas completas de estas clases son `django.core.files.File` y `django.core.files.base.ContentFile`, respectivamente). Analizaremos brevemente el método `save` y sus argumentos y luego volveremos a estas clases.

El método `save` de `FileField` toma tres argumentos:
- **name**: Este es el nombre del archivo que estás guardando y es el nombre que tendrá el archivo cuando se guarde en el motor de almacenamiento (en nuestro caso, en el disco, dentro de `MEDIA_ROOT`).
- **content**: Esta es una instancia de `File` o `ContentFile` que acabamos de ver; nuevamente, hablaremos de ellas pronto.
- **save**: Este argumento es opcional y su valor predeterminado es `True`. Esto indica si se debe guardar o no la instancia del modelo en la base de datos después de guardar el archivo. Si se establece en `False` (es decir, el modelo no se guarda), el archivo se escribirá en el motor de almacenamiento (en el disco), pero la asociación no se almacenará en el modelo. La ruta del archivo anterior (o ningún archivo si no se configuró uno) aún se almacenará en la base de datos hasta que se llame manualmente al método `save` de la instancia del modelo. Solo debes establecer este argumento en `False` si tienes la intención de realizar otros cambios en la instancia del modelo y luego guardarla manualmente.

Volviendo a `File` y `ContentFile`: cuál usar depende de lo que desees almacenar en `FileField`.

`File` se usa como un contenedor alrededor de un objeto de archivo de Python, y debes usarlo si tienes un archivo existente o un objeto similar a un archivo que deseas guardar. Los objetos similares a archivos incluyen instancias de `io.BytesIO` o `io.StringIO`. Para instanciar una instancia de `File`, simplemente pasa el objeto de archivo nativo al constructor, como en este ejemplo:

```python
f = open("/path/to/file.txt", "rb")
file_wrapper = File(f)
f.close()
```

Usa `ContentFile` cuando ya tengas algunos datos cargados, ya sea un objeto `str` o un objeto `bytes`. Pasa los datos al constructor `ContentFile`:

```python
string_content = ContentFile("A string value")
bytes_content = ContentField(b"A bytes value")
```

Ahora que tienes una instancia de `File` o una instancia de `ContentFile`, guardar los datos en `FileField` es fácil, usando el método `save`:

```python
m = ExampleModel.objects.first()
with open("/path/to/file.txt") as f:
    file_wrapper = File(f)
    m.file_field.save("file.txt", f)
```

Dado que no pasamos un valor para `save` al método `save`, el valor predeterminado será `True`, por lo que la instancia del modelo se conserva automáticamente en la base de datos.

A continuación, veremos cómo almacenar una imagen que ha sido manipulada con PIL en un campo de imagen.

#### Escribir imágenes de PIL en ImageField

En el Ejercicio 8.05 – Carga de imágenes mediante formularios de Django, usaste PIL para redimensionar una imagen y guardarla en el disco. Al trabajar con un modelo, es posible que desees realizar una operación similar pero hacer que Django maneje el almacenamiento de archivos usando `ImageField` para que no tengas que hacerlo manualmente. Al igual que en el ejercicio, podrías guardar la imagen en el disco y luego usar la clase `File` para envolver la ruta almacenada, algo como esto:

```python
image = Image.open(request.FILES["image_field"])
image.thumbnail((150, 150))
# guardar miniatura en ubicación temporal
image.save("/tmp/thumbnail.jpg")
with open("/tmp/thumbnail.jpg", "rb") as f:
    image_wrapper = File(f)
    m.image_field.save("thumbnail.jpg", image_wrapper, save=True)
# limpiar archivo temporal
os.unlink("/tmp/thumbnail.jpg")
```

En este ejemplo, estamos almacenando la imagen de PIL en una ubicación temporal con el método `Image.save()` y luego volviendo a abrir el archivo.

Este método funciona pero no es el ideal, ya que implica escribir el archivo en el disco y luego volver a leerlo, lo que a veces puede resultar lento. En su lugar, podemos realizar todo este proceso en la memoria.

`io.BytesIO` e `io.StringIO` son objetos útiles. Se comportan como archivos pero existen solo en la memoria. `BytesIO` se usa para almacenar bytes sin procesar y `StringIO` acepta las cadenas Unicode nativas de Python 3. Puedes usar `read`, `write` y `seek`, igual que con un archivo normal. Sin embargo, a diferencia de un archivo normal, no se escriben en el disco y, en cambio, desaparecerán cuando finalice tu programa o cuando salgan del ámbito y sean recolectados por el recolector de basura. Son muy útiles si una función quiere escribir en algo como un archivo, pero deseas acceder a los datos de inmediato.

Primero, guardaremos los datos de la imagen en un objeto `io.BytesIO`. Luego, envolveremos el objeto `BytesIO` en una instancia de `django.core.files.images.ImageFile` (una subclase de `File` que es específicamente para imágenes y proporciona los atributos `width` y `height`). Una vez que tengamos esta instancia de `ImageFile`, podemos usarla en el método `save` de `ImageField`.

`ImageFile` es un contenedor de archivos o similar a un archivo, al igual que `File`. Proporciona dos atributos adicionales: `width` y `height`. `ImageFile` no genera ningún error si lo usas para envolver algo que no sea una imagen. Por ejemplo, podrías usar `open()` para abrir un archivo de texto y pasar el identificador de archivo al constructor `ImageFile` sin problemas. Puedes comprobar si el archivo de imagen que pasaste era válido intentando acceder al atributo `width` o `height`; si estos son `None`, entonces PIL no pudo decodificar los datos de la imagen. Puedes verificar la validez de estos valores tú mismo y lanzar una excepción si son `None`.

Veamos esto en la práctica en una vista:

```python
from io import BytesIO
from PIL import Image
from django.core.files.images import ImageFile

def index(request, pk):
    # omitir la lógica para verificar si el método es POST
    # obtener una instancia de modelo o crear una nueva
    m = ExampleModel.objects.get(pk=pk)
    # almacenar la imagen cargada en una variable para abreviar
    # el código
    uploaded_image = request.FILES["image_field"]
    # cargar una instancia de imagen de PIL desde el archivo subido
    image = Image.open(uploaded)
    # realizar el cambio de tamaño de la imagen
    image.thumbnail((150, 150))
    # Crear un objeto similar a un archivo BytesIO para almacenar
    image_data = BytesIO()
    # Escribir los datos de la imagen de nuevo en el objeto BytesIO
    # Conservar el formato existente de la imagen cargada
    image.save(fp=image_data, uploaded_image.format)
    # Envolver el BytesIO que contiene los datos de la imagen
    image_file = ImageFile(image_data)
    # Guardar los datos del archivo de imagen envueltos con el nombre
    # original
    m.image_field.save(uploaded_image.name, image_file)
    # esto también guarda la instancia del modelo
    return redirect("/success-url/")
```

Puedes ver que esto requiere un poco más de código, pero ahorra la escritura de datos en el disco. Puedes optar por utilizar cualquiera de los dos métodos (u otro que se te ocurra) según tus necesidades.

#### Hacer referencia a archivos multimedia en plantillas

Una vez que hemos subido un archivo, queremos poder hacer referencia a él en una plantilla. Para una imagen subida, como la portada de un libro, querremos mostrar la imagen en la página. Vimos en el Ejercicio 8.02 – Configuración de plantillas y uso de `MEDIA_URL` en plantillas, cómo crear una URL usando `MEDIA_URL` en una plantilla. Al trabajar con `FileField` o `ImageField` en una instancia de modelo, no es necesario hacer esto, ya que Django proporciona esta funcionalidad por ti.

El atributo `url` de `FileField` generará automáticamente la URL completa al archivo multimedia según el `MEDIA_URL` en tus configuraciones.

Ten en cuenta que las referencias que hacemos a `FileField` en esta sección también se aplican a `ImageField`, ya que es una subclase de `FileField`.

Esto se puede usar en cualquier lugar donde tengas acceso a la instancia y al campo, como en una vista o una plantilla. El siguiente ejemplo está en una vista:

```python
instance = ExampleModel.objects.first()
url = instance.file_field.url  # Obtener la URL
```

El siguiente ejemplo está en una plantilla (asumiendo que `instance` se ha pasado al contexto de la plantilla):

```django
<img src="{{ instance.file_field.url }}">
```

En el siguiente ejercicio, crearemos un nuevo modelo con `FileField` e `ImageField` y luego mostraremos cómo Django puede guardarlos automáticamente. También demostraremos cómo recuperar la URL de un archivo subido.

#### Ejercicio 8.06 – FileField e ImageField en modelos

En este ejercicio, crearemos un modelo con `FileField` e `ImageField`. Después de hacer esto, tendremos que generar una migración y aplicarla. Luego cambiaremos el `UploadForm` que hemos estado usando para que tenga tanto `FileField` como `ImageField`. La vista `media_example` se actualizará para almacenar los archivos subidos en la instancia del modelo. Finalmente, agregaremos `<img>` en la plantilla de ejemplo para mostrar la imagen previamente cargada:

1. En PyCharm, abre el archivo `models.py` de la aplicación `media_example`. Crea un nuevo modelo llamado `ExampleModel` con dos campos: un `ImageField` llamado `image_field` y un `FileField` llamado `file_field`. `ImageField` debe tener `upload_to` establecido en `images/`, y `FileField` debe tener `upload_to` establecido en `files/`. El modelo terminado debería verse así:
   ```python
   class ExampleModel(models.Model):
       image_field = models.ImageField(upload_to="images/")
       file_field = models.FileField(upload_to="files/")
   ```
2. Abre una terminal y navega hasta el directorio del proyecto `media_project`. Asegúrate de que tu entorno virtual `djangoenv` esté activo. Ejecuta el comando de administración `makemigrations` para generar las migraciones para este nuevo modelo:
   ```bash
   python manage.py makemigrations
   ```
   Para aprender a crear y activar un entorno virtual, consulta la sección Prefacio del libro.
   La salida debería ser similar a la siguiente:
   ```text
   % python3 manage.py makemigrations
   Migrations for 'media_example':
     media_example/migrations/0001_initial.py
       - Create model ExampleModel
   ```
3. Aplica la migración ejecutando el comando de administración `migrate`:
   ```bash
   python3 manage.py migrate
   ```
   La salida será la siguiente:
   ```text
   % python3 manage.py migrate
   Operations to perform:
     Apply all migrations: admin, auth, contenttypes, reviews, sessions
   Running migrations:
     # salida recortada por brevedad
     Applying media_example.0001_initial... OK
   ```
   Ten en cuenta que también se aplicarán todas las migraciones iniciales de Django, ya que no las aplicamos después de crear el proyecto.
4. Vuelve a PyCharm y abre el archivo `forms.py` de la aplicación `media_example`. Cambia el nombre del `ImageField` existente de `file_upload` a `image_upload`. Luego, agrega un nuevo `FileField` llamado `file_upload`. Después de realizar estos cambios, el código de tu `UploadForm` debería verse así:
   ```python
   class UploadForm(forms.Form):
       image_upload = forms.ImageField()
       file_upload = forms.FileField()
   ```
   Puedes guardar y cerrar el archivo.
5. Abre el archivo `views.py` de la aplicación `media_example`. Primero, importa `ExampleModel` en el archivo. Para hacer esto, agrega esta línea en la parte superior del archivo después de las declaraciones de importación existentes:
   ```python
   from .models import ExampleModel
   ```
6. Algunas importaciones ya no serán necesarias, por lo que puedes eliminar estas líneas:
   ```python
   from PIL import Image
   from django.conf import settings
   ```
7. En la vista `media_example`, establece un valor predeterminado para la instancia que renderizarás en caso de que no se cree una. Después de la definición de la función, define una variable llamada `instance` y establécela en `None`:
   ```python
   def media_example(request):
       instance = None
   ```
8. Puedes eliminar por completo el contenido de la rama `form.is_valid()`, ya que ya no necesitas guardar manualmente el archivo. En su lugar, se guardará automáticamente cuando se guarde la instancia de `ExampleModel`. Instanciarás una instancia de `ExampleModel` y establecerás los campos de archivo e imagen desde el formulario cargado. Agrega este código debajo de la línea `if form.is_valid():`:
   ```python
   instance = ExampleModel()
   instance.image_field = form.cleaned_data["image_upload"]
   instance.file_field = form.cleaned_data["file_upload"]
   instance.save()
   ```
9. Pasa la instancia a la plantilla en el diccionario de contexto que se pasa a `render`. Usa la clave `instance`:
   ```python
   return render(request, "media-example.html", {"form": form, "instance": instance})
   ```
   Ahora puedes guardar y cerrar este archivo.
10. Abre la plantilla `media-example.html`. Agrega un elemento `<img>` que muestre la última imagen subida. Debajo de la etiqueta de cierre `</form>`, agrega una etiqueta de plantilla `if` que verifique si se ha proporcionado `instance`. Si es así, muestra `<img>` con un atributo `src` de `instance.image_field.url`:
    ```django
    {% if instance %}
        <img src="{{ instance.image_field.url }}">
    {% endif %}
    ```
11. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego navega a `http://127.0.0.1:8000/media-example/`. Deberías ver el formulario renderizado con dos campos:
    *Figura 8.19: UploadForm con dos campos*
12. Selecciona un archivo para cada campo: para `ImageField`, debes seleccionar una imagen, pero se permite cualquier tipo de archivo para `FileField`. Consulta la Figura 8.20, que muestra los campos con los archivos seleccionados:
    *Figura 8.20: ImageField y FileField con archivos seleccionados*
13. Luego, envía el formulario. Si el envío es exitoso, la página se recargará y se mostrará la última imagen que subimos (Figura 8.21):
    *Figura 8.21: Se muestra la última imagen que se subió*
14. Puedes ver cómo almacena Django los archivos mirando en el directorio `MEDIA_ROOT`. La Figura 8.22 muestra la disposición de los directorios en PyCharm:
    *Figura 8.22: Archivos subidos que Django ha creado*
    Puedes ver que Django ha creado los directorios `files` e `images`. Estos fueron los que configuraste en los argumentos `upload_to` en `ImageField` y `FileField` del modelo.

También podríamos verificar estas cargas intentando descargarlas, por ejemplo, en `http://127.0.0.1:8000/media/files/sample.txt` o `http://127.0.0.1:8000/media/images/cover.jpg`.

En este ejercicio, creamos `ExampleModel` con `FileField` e `ImageField` y vimos cómo almacenar archivos subidos en él. Vimos cómo generar una URL a un archivo subido para usarlo en una plantilla. Intentamos subir algunos archivos y vimos que Django creó automáticamente los directorios de `upload_to` (`media/files` y `media/images`) y luego almacenó los archivos dentro de ellos.

En la siguiente sección, veremos cómo podemos simplificar el proceso aún más usando `ModelForm` para generar el formulario y guardar el modelo sin tener que configurar manualmente los archivos en la vista.

---

### Sección: ModelForm y carga de archivos

Hemos visto cómo el uso de `forms.ImageField` en un formulario puede evitar que se carguen elementos que no sean imágenes. También hemos visto cómo `models.ImageField` facilita el almacenamiento de una imagen para un modelo. Pero debemos tener en cuenta que Django no nos impide asignar un archivo que no sea de imagen a `ImageField`. Por ejemplo, considera un formulario que tiene tanto `FileField` como `ImageField`:

```python
class ExampleForm(forms.Form):
    uploaded_file = forms.FileField()
    uploaded_image = forms.ImageField()
```

En la siguiente vista, el formulario no se validaría si el campo `uploaded_image` en el formulario no fuera una imagen, por lo que se garantiza cierta validez de los datos cargados:

```python
def view(request):
    form = ExampleForm(request.POST, request.FILES)
    if form.is_valid():
        m = ExampleModel()
        m.file_field = form.cleaned_data["uploaded_file"]
        m.image_field = forms.cleaned_data["uploaded_image"]
        m.save()
        return render(request, "template.html")
```

Dado que estamos seguros de que el formulario es válido, sabemos que `forms.cleaned_data["uploaded_image"]` debe contener una imagen. Por lo tanto, nunca asignaríamos algo que no fuera una imagen al `image_field` de la instancia del modelo.

Sin embargo, supongamos que cometimos un error en nuestro código y escribimos algo como lo siguiente:

```python
m.image_field = forms.cleaned_data["uploaded_file"]
```

Es decir, si accidentalmente hacemos referencia a `FileField` por error, Django no valida que se esté asignando un archivo que potencialmente no sea una imagen a `ImageField`, por lo que no arroja una excepción ni genera ningún tipo de error. Podemos mitigar la posibilidad de que surjan problemas como este mediante el uso de `ModelForm`.

Introdujimos `ModelForm` en el Capítulo 7 (*Validación Avanzada de Formularios y Formularios de Modelos*). Este es un tipo de formulario cuyos campos se definen automáticamente a partir de un modelo. Vimos que `ModelForm` tiene un método `save` que crea o actualiza automáticamente los datos del modelo en la base de datos. Cuando se usa con un modelo que tiene `FileField` o `ImageField`, el método `save` de `ModelForm` también guardará los archivos cargados.

He aquí un ejemplo de uso de `ModelForm` para guardar una nueva instancia de modelo en una vista. Aquí, solo nos aseguramos de pasar `request.FILES` al constructor de `ModelForm`:

```python
class ExampleModelForm(forms.Model):
    class Meta:
        # La misma clase ExampleModel que hemos visto anteriormente
        model = ExampleModel
        fields = "__all__"

def view(request):
    if request.method == "POST":
        form = ExampleModelForm(request.POST, request.FILES)
        form.save()
        return redirect("/success-page")
    else:
        form = ExampleModelForm()
        return (request, "template.html", {"form": form})
```

Al igual que con cualquier `ModelForm`, se puede llamar al método `save` con el argumento `commit` establecido en `False`. Entonces, la instancia del modelo no se guardará en la base de datos y los archivos `FileField` e `ImageField` no se guardarán en el disco. El método `save` debe llamarse en la propia instancia del modelo; esto confirmará los cambios en la base de datos y guardará los archivos. En este siguiente ejemplo breve, establecemos un valor en la instancia del modelo antes de guardarla:

```python
def view(request):
    if request.method == "POST":
        form = ExampleModelForm(request.POST, request.FILES)
        m = form.save(False)
        # Establecer un valor arbitrario en la instancia del modelo
        # antes de guardar
        m.attribute = "value"
        # guardar la instancia del modelo, también escribir los
        # archivos en el disco
        m.save()
        return redirect("/success-page/")
    else:
        form = ExampleModelForm()
        return (request, "template.html", {"form": form})
```

Llamar al método `save` en la instancia del modelo guarda tanto los datos del modelo en la base de datos como los archivos cargados en el disco. En el siguiente ejercicio, crearemos `ModelForm` a partir del `ExampleModel` que creamos en el Ejercicio 8.06 – `FileField` e `ImageField` en modelos, y luego probaremos cargar archivos con él.

#### Ejercicio 8.07 – Carga de archivos e imágenes usando ModelForm

En este ejercicio, actualizarás `UploadForm` para que sea una subclase de `ModelForm` y se cree automáticamente a partir de `ExampleModel`. Luego cambiarás la vista `media_example` para guardar la instancia automáticamente desde el formulario, de modo que puedas ver cómo se puede reducir la cantidad de código:

1. En PyCharm, abre el archivo `forms.py` de la aplicación `media_example`. Necesitas usar `ExampleModel` en este capítulo, así que usa `import` para importarlo en la parte superior del archivo después de la declaración `from django import forms`. Inserta esta línea:
   ```python
   from .models import ExampleModel
   ```
2. Cambia `UploadForm` para que sea una subclase de `forms.ModelForm`. Elimina el cuerpo de la clase y reemplázalo con una definición de `class Meta`. Su configuración `model` debe ser `ExampleModel`. Establece el atributo `fields` en `"__all__"`. Después de completar este paso, `UploadForm` debería verse así:
   ```python
   class UploadForm(forms.ModelForm):
       class Meta:
           model = ExampleModel
           fields = "__all__"
   ```
   Guarda y cierra el archivo.
3. Abre el archivo `views.py` de la aplicación `media_example`. Como ya no necesitas hacer referencia a `ExampleModel` directamente, puedes eliminar su importación en la parte superior del archivo. Elimina la siguiente línea:
   ```python
   from .models import ExampleModel
   ```
4. En la vista `media_example`, elimina la totalidad de la rama `form.is_valid()` y reemplázala con una sola línea:
   ```python
   instance = form.save()
   ```
   El método `save` del formulario se encargará de persistir la instancia en la base de datos y guardar los archivos. Devolverá una instancia de `ExampleModel`, igual que el otro `ModelForm` con el que trabajamos en el Capítulo 7 (*Validación Avanzada de Formularios y Formularios de Modelos*).
5. Guarda y cierra `views.py`.
6. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego navega a `http://127.0.0.1:8000/media-example/`. Deberías ver el formulario renderizado con dos campos, Image field y File field (Figura 8.23):
   *Figura 8.23: UploadForm como ModelForm renderizado en el navegador*
   Ten en cuenta que los nombres de estos campos ahora coinciden con los del modelo en lugar de los del formulario, ya que el formulario simplemente usa los campos del modelo.
7. Explora y selecciona una imagen y un archivo (Figura 8.24), luego envía el formulario:
   *Figura 8.24: Imagen y archivo seleccionados*
8. La página se recargará y, como en el Ejercicio 8.06 – `FileField` e `ImageField` en modelos, verás la imagen previamente cargada (Figura 8.25):
   *Figura 8.25: Imagen que se muestra después de la carga*
9. Finalmente, examina el contenido del directorio `media`. Deberías ver que la disposición de directorios coincide con la del Ejercicio 8.06 – `FileField` e `ImageField` en modelos, con imágenes dentro del directorio `images` y archivos dentro del directorio `files`:
   *Figura 8.26: El directorio de archivos cargados coincide con el Ejercicio 6*

En este ejercicio, cambiamos `UploadForm` a una subclase de `ModelForm`, lo que nos permitió generar automáticamente los campos de carga. Pudimos reemplazar el código que almacenaba los archivos cargados en los modelos con una llamada al método `save` del formulario.

Antes de pasar a las actividades de este capítulo, debemos analizar brevemente cómo manejar el guardado de archivos cuando se usa una instancia.

#### Manejo del guardado de archivos

Como sabes por trabajar con `ModelForms` en el Capítulo 7 (*Validación Avanzada de Formularios y Formularios de Modelos*), puedes pasar un argumento `instance` a `ModelForm` para hacer que actualice un modelo existente en lugar de crear uno nuevo al guardar.

Por ejemplo, haz lo siguiente para actualizar un modelo de ejemplo en una vista:

```python
def update_view(request, pk):
    instance = ExampleModel.objects.get(pk=pk)
    form = UploadForm(request.POST, request.FILES, instance=instance)
    if form.valid():
        form.save()
```

Al acceder al atributo `cleaned_data` del formulario, recuperaremos los datos contenidos en `request.POST`. Si no había ningún valor para esa clave, obtendremos el valor de la instancia en su lugar.

Esto normalmente no causa ningún problema, pero cuando se trabaja con `ImageField` o `FileField`, el tipo de datos que se devuelven diferirá de la solicitud o de la instancia.

Si se subió un archivo, los datos limpios serán `InMemoryUploadedFile` o `TemporaryUploadedFile` (según su tamaño; solo los archivos pequeños cabrán en la memoria). Si no se subió ningún archivo, el tipo será `FieldFile` para `FileField` o `ImageFieldFile` para `ImageField`.

Para explicarlo, considera este código:

```python
def update_view(request, pk):
    instance = ExampleModel.objects.get(pk=pk)
    form = UploadForm(request.POST, request.FILES, instance=instance)
    if form.is_valid():
        form_file = form.cleaned_data["file_field"]
        form_image = form.cleaned_data["image_field"]
```

En el caso de que se hayan cargado datos (es decir, el usuario seleccionó una imagen y un archivo en el formulario), `form_file` y `form_image` serán `InMemoryUploadedFile` o `TemporaryUploadedFile`. Sin embargo, si el usuario no cargó nada para los campos (suponiendo que los campos no sean obligatorios en el formulario), entonces `form_file` será una instancia de `FieldFile` y `form_image` será una instancia de `ImageFieldFile`.

Para diferenciarlos, hay algunas comprobaciones diferentes que puedes hacer. Uno de los métodos más sencillos es comprobar la existencia de un atributo `path`. Dado que los archivos subidos aún no se han guardado en el disco, no tienen ruta:

```python
if hasattr(form_file, "path"):
    print("This is a FieldFile")
else:
    print("This is an uploaded file")
```

A veces necesitarás verificar con qué tipo de archivo estás trabajando antes de realizar alguna acción, y quizás omitir esa acción si el usuario no ha subido un archivo nuevo. Por ejemplo, no querríamos redimensionar una imagen que ya estaba asignada a un modelo. Solo haríamos el cambio de tamaño de la imagen en imágenes recién subidas:

```python
if hasattr(image_file, "path"):
    print("This image was on the instances so no need to resize")
else:
    perform_resize(image_file)
```

Deberás realizar una comprobación como esta en la primera actividad de este capítulo.

Ahora hemos cubierto todo lo que necesitas para comenzar a mejorar Bookr con la carga de archivos. En la actividad de este capítulo, agregaremos soporte para cargar una imagen de portada y un documento de muestra (PDF, archivo de texto y más) para un libro. La portada del libro se redimensionará utilizando PIL antes de guardarse.

---

### Sección: Actividad 8.01 – Carga de imágenes y PDF para libros

En esta actividad, comenzarás limpiando (eliminando) las vistas de ejemplo, plantillas, formularios, modelos y mapas de URL que estuvimos usando a lo largo de los ejercicios de este capítulo. Luego deberás generar y aplicar una migración para eliminar `ExampleModel` de la base de datos.

Luego puedes comenzar a agregar las mejoras de Bookr, primero agregando `ImageField` y `FileField` al modelo `Book` para almacenar la portada y la muestra del libro. Luego crearás una migración y la aplicarás para agregar estos campos a la base de datos. Luego puedes crear un formulario que mostrará solo estos nuevos campos. Agregarás una vista que usa este formulario para guardar la instancia del modelo con los archivos cargados después de cambiar primero el tamaño de la imagen a un tamaño de miniatura. Podrás reutilizar la plantilla `instance-form.html` del Capítulo 7 (*Validación Avanzada de Formularios y Formularios de Modelos*), con un cambio menor para permitir la carga de archivos.

Estos pasos te ayudarán a completar la actividad:

1. Actualiza la configuración de Django para agregar las configuraciones `MEDIA_ROOT` y `MEDIA_URL`.
2. El mapeo de URL `/media/` debe agregarse al archivo principal `urls.py`. Usa la vista `static` y utiliza `MEDIA_ROOT` y `MEDIA_URL` de la configuración de Django. Recuerda, este mapeo solo debe agregarse si `DEBUG` es `True`.
3. Agrega `ImageField` (llamado `cover`) y `FileField` (llamado `sample`) al modelo `Book`. Los campos deben cargarse en `book_covers/` y `book_samples/`, respectivamente. Ambos deben permitir los valores `null` y `blank`.
4. Ejecuta `makemigrations` y `migrate` nuevamente para aplicar los cambios del modelo `Book` a la base de datos.
5. Crea `BookMediaForm` como una subclase de `ModelForm`. Su modelo debe ser `Book`, y los campos solo deben ser los campos que agregaste en el paso 3.
6. Agrega una vista `book_media`. Esto no te permitirá crear un `Book`; en su lugar, solo te permitirá agregar medios a un `Book` existente (por lo que debe tomar `pk` como un argumento obligatorio).
7. La vista `book_media` debe validar el formulario y usar `save` para guardarlo, pero sin confirmar la instancia (`commit=False`). La portada cargada primero debe redimensionarse utilizando el método `thumbnail`, como se demostró en la sección *Escribir imágenes de PIL en ImageField*. El tamaño máximo debe ser de 300 px por 300 px. Luego debe almacenarse en la instancia y la instancia debe guardarse. Recuerda que el campo `cover` no es obligatorio, por lo que debes verificar esto antes de intentar manipular la imagen, y no necesitarás cambiar el tamaño de la imagen si ya hay una en la instancia. En un POST exitoso, registra un mensaje de éxito indicando que el `Book` se actualizó, luego redirige a la vista `book_detail`.
8. Renderiza `instance-form.html`, pasando un diccionario de contexto que contenga `form`, `model_type` e `instance`, como hiciste en el Capítulo 6 (*Formularios*). Además, pasa otro elemento, `is_file_upload`, establecido en `True`. Esta variable se utilizará en el siguiente paso.
9. En la plantilla `instance-form.html`, usa la variable `is_file_upload` para agregar el atributo `enctype` correcto al formulario. Esto te permitirá cambiar los modos del formulario para habilitar la carga de archivos cuando sea necesario.
10. Finalmente, agrega un mapa de URL que asigne `/books/<pk>/media/` a la vista `book_media`.

*Figura 8.27: BookMediaForm en el navegador*

Selecciona una imagen de portada y un archivo de muestra para el libro. Puedes usar la imagen en `Chapter08/Activity8.01/bookr/media/book_covers/machine-learning-for-algorithmic-trading.png` y el PDF en `Chapter08/Activity8.01/bookr/media/book_samples/machine-learning-for-trading.pdf` (o puedes usar cualquier otra imagen/PDF de tu elección).

*Figura 8.28: Portada del libro y muestra seleccionadas*

Después de enviar el formulario, serás redirigido a la vista de detalles del libro (*Book Details*) y verás el mensaje de éxito (Figura 8.29):

*Figura 8.29: Mensaje de éxito en la página Book Details*

Si vuelves a la página de medios del mismo libro, deberías ver que los campos ahora están completos con una opción para borrar los datos de ellos:

*Figura 8.30: BookMediaForm con valores existentes*

En la Actividad 8.02 – Mostrar la portada y el enlace de muestra, agregarás estos archivos subidos a la vista de detalles del libro, pero por ahora, si deseas verificar que las cargas hayan funcionado, puedes mirar dentro del directorio `media` en el proyecto Bookr:

*Figura 8.31: Archivos multimedia de Book*

Deberías ver los directorios que se crearon y los archivos cargados, según la Figura 8.31. Abre una imagen cargada y deberías ver que su dimensión máxima es de 300 px.

---

### Sección: Actividad 8.02 – Mostrar la portada y el enlace de muestra

En esta actividad, actualizarás la plantilla `book_detail.html` para mostrar la portada de `Book` (si se ha configurado una). También agregarás un enlace para descargar la muestra nuevamente, solo si se ha configurado una. Utilizarás los atributos `url` de `FileField` e `ImageField` para generar las URLs a los archivos multimedia.

Estos pasos te ayudarán a completar esta actividad:

1. Dentro de la visualización de detalles del libro en la vista `book_detail.html`, agrega un elemento `<img>` si el libro tiene una imagen de portada. Luego, muestra la portada del libro dentro de él. Usa `<br>` después de la etiqueta `<img>` para que la imagen quede en su propia línea.
2. Después de la visualización de *Publication Date*, agrega un enlace al archivo de muestra. Solo debe mostrarse si se ha cargado un archivo de muestra. Asegúrate de agregar otra etiqueta `<br>` para que se muestre correctamente.
3. En la sección que tiene un enlace para agregar una reseña, agrega otro enlace que vaya a la página de medios del libro. Sigue el mismo estilo que el enlace *Add Review*.

Cuando hayas completado estos pasos, deberías poder cargar una página de detalles del libro. Si el libro no tiene portada ni muestra, la página debería verse muy similar a como se veía antes, excepto que deberías ver el nuevo enlace a la página Media en la parte inferior (Figura 8.32):

*Figura 8.32: El nuevo botón Media visible en la página Book Details*

Una vez que hayas subido una portada y/o una muestra para un libro, se deben mostrar la imagen de portada y el enlace de muestra (Figura 8.33):

*Figura 8.33: La portada del libro y el enlace Sample mostrados*

---

### Sección: Resumen

En este capítulo, agregamos las configuraciones `MEDIA_ROOT` y `MEDIA_URL` y un mapa de URL especial para servir archivos multimedia. Luego creamos un formulario y una vista para cargar archivos y guardarlos en el directorio `media`. Vimos cómo agregar el procesador de contexto `media` para tener acceso automáticamente a la configuración `MEDIA_URL` en todas nuestras plantillas. Luego mejoramos y simplificamos el código de nuestro formulario utilizando un formulario de Django con `FileField` o `ImageField`, en lugar de definir uno manualmente en HTML.

Examinamos algunas de las mejoras que ofrece Django para imágenes con `ImageField` y cómo interactuar con una imagen mediante Pillow. Mostramos una vista de ejemplo que podría servir archivos que requerían autenticación mediante la clase `FileResponse`. Luego, vimos cómo almacenar archivos en modelos usando `FileField` e `ImageField` y hacer referencia a ellos en una plantilla usando el atributo `FileField.url`. Pudimos reducir la cantidad de código que tuvimos que escribir al construir automáticamente `ModelForm` a partir de una instancia de modelo. Finalmente, realizamos dos actividades para mejorar Bookr agregando una imagen de portada y un archivo de muestra al modelo `Book`.

En el Capítulo 9 (*Sesiones y Autenticación*), aprenderemos cómo agregar autenticación a una aplicación Django para protegerla de usuarios no autorizados.
