# Parte 1: Primeros pasos con Django

## Capítulo 3: Vistas, Configuración de URLs y Plantillas en Django

En el Capítulo 2, nos introdujimos en las bases de datos y aprendimos a almacenar, recuperar, actualizar y eliminar registros de una base de datos. También aprendimos a crear modelos de Django y aplicar migraciones de bases de datos.

Sin embargo, estas operaciones de base de datos por sí solas no pueden mostrar los datos de la aplicación a un usuario. Necesitamos una forma de mostrar toda la información almacenada de manera significativa al usuario; por ejemplo, mostrar todos los libros presentes en la base de datos de nuestra aplicación Bookr, en un navegador y en un formato presentable. Aquí es donde entran en juego las vistas de Django, la configuración de URLs y las plantillas. Las vistas son una de las partes más importantes de una aplicación Django, donde se escribe la lógica de la aplicación. Esta lógica de aplicación controla las interacciones con la base de datos, como crear, leer, actualizar o eliminar registros de la base de datos. Las solicitudes del servidor se enrutan a las vistas mediante el mapeo de URLs. Las vistas también controlan cómo se pueden mostrar los datos al usuario. Esto se hace con la ayuda de las plantillas HTML de Django.

Comenzarás explorando los dos tipos principales de vistas en Django: vistas basadas en funciones (*function-based views*) y vistas basadas en clases (*class-based views*). A continuación, aprenderás los conceptos básicos del lenguaje de plantillas de Django y la herencia de plantillas. Con estos conceptos, crearás una página para mostrar la lista de todos los libros en la aplicación Bookr. También crearás otra página para mostrar los detalles, comentarios de reseñas y calificaciones de los libros.

En este capítulo, estudiaremos los siguientes temas:
- Trabajo con vistas de Django
- Configuración de URLs
- Trabajo con plantillas de Django

Al utilizar y aprender sobre estos temas, al final de este capítulo habrás implementado una forma de enumerar todos los libros y mostrar los detalles de cualquier libro dado en la aplicación Bookr.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter03` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos descritos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Trabajo con vistas de Django

Las vistas son la parte de una aplicación Django que recibe una solicitud web, la procesa y proporciona una respuesta web. Por ejemplo, un usuario hace clic en el enlace de detalles de un producto:
1. El navegador envía una solicitud a una URL específica (por ejemplo, `/products/12/`), que Django mapea a una vista.
2. La vista va a la base de datos para obtener el producto 12 y prepara los datos de ese objeto para utilizarlos en la plantilla de la vista. Estos datos se denominan **datos de contexto** (*context data*) y se utilizan junto con la plantilla para crear el HTML para la respuesta.
3. La respuesta toma la plantilla renderizada para mostrar el HTML en el navegador del usuario. Las vistas de Django se pueden clasificar ampliamente en dos tipos: vistas basadas en funciones y vistas basadas en clases. Echemos un vistazo a cada una en detalle.

En este capítulo, aprenderemos más sobre las vistas basadas en funciones en Django. Las vistas basadas en clases, que son un tema más avanzado, se tratarán en detalle en el Capítulo 11. Las vistas asíncronas se discutirán en el Capítulo 17.

#### Comprensión de las vistas basadas en funciones

Como su nombre lo indica, las vistas basadas en funciones se implementan como funciones de Python. Para comprender cómo funcionan, considera el siguiente fragmento, que muestra una función de vista simple llamada `home_page`:

```python
from django.http import HttpResponse

def home_page(request):
    message = "<html><h1>Welcome to my Website</h1></html>"
    return HttpResponse(message)
```

La función de vista definida aquí, llamada `home_page`, toma un objeto `request` como argumento y devuelve un objeto `HttpResponse`, que contiene el mensaje `Welcome to my Website`. La ventaja de utilizar vistas basadas en funciones es que, dado que se implementan como funciones simples de Python, son más fáciles de aprender y también fácilmente legibles para otros programadores. La principal desventaja de las vistas basadas en funciones es que el código no se puede reutilizar ni hacer tan conciso como las vistas basadas en clases para casos de uso genéricos. La siguiente sección es una breve introducción a las vistas basadas en clases.

#### Comprensión de las vistas basadas en clases

Como su nombre lo indica, las vistas basadas en clases se implementan como clases de Python. Utilizando los principios de herencia de clases, estas clases se implementan como subclases de las clases de vistas genéricas de Django. A diferencia de las vistas basadas en funciones, donde toda la lógica de la vista se expresa explícitamente en una función, las clases de vista genéricas de Django vienen con varias propiedades y métodos preconstruidos que pueden proporcionar atajos para escribir vistas limpias y reutilizables. Esta propiedad resulta útil con bastante frecuencia durante el desarrollo web; por ejemplo, los desarrolladores a menudo necesitan renderizar una página HTML sin necesidad de insertar datos de la base de datos o cualquier personalización específica para el usuario. En este caso, es posible simplemente heredar de `TemplateView` de Django y especificar la ruta del archivo HTML. El siguiente es un ejemplo de una vista basada en clases que puede mostrar el mismo mensaje que en el ejemplo de vista basada en funciones:

```python
from django.views.generic import TemplateView

class HomePage(TemplateView):
    template_name = 'home_page.html'
```

En el fragmento de código anterior, `HomePage` es una vista basada en clases que hereda `TemplateView` de Django del módulo `django.views.generic`. El atributo de clase `template_name` define la plantilla que se renderizará cuando se invoque la vista. Para la plantilla, agregamos un archivo HTML a nuestra carpeta `templates` con el siguiente contenido:

```html
<html><h1>Welcome to my Website</h1></html>
```

Este es un ejemplo muy básico de vistas basadas en clases, que se explorará más a fondo en el Capítulo 11. La principal ventaja de usar vistas basadas en clases es que se deben usar menos líneas de código para implementar la misma funcionalidad en comparación con las vistas basadas en funciones. Además, al heredar las vistas genéricas de Django, podemos mantener el código conciso y evitar la duplicación de código. Sin embargo, una desventaja de las vistas basadas en clases es que el código a menudo es menos legible para alguien nuevo en Django, lo que significa que aprender sobre ellas suele ser un proceso más largo en comparación con las vistas basadas en funciones.

En la siguiente sección, aprenderemos sobre la configuración de URLs.

---

### Sección: Configuración de URLs

Las vistas de Django no pueden funcionar por sí solas en una aplicación web. Cuando se realiza una solicitud web a la aplicación, la configuración de URLs de Django se encarga de enrutar la solicitud a la función de vista adecuada para procesarla. Una configuración de URLs típica en el archivo `urls.py` en Django se ve así:

```python
from . import views

urlpatterns = [
    path('url-path/', views.my_view, name='my-view'),
]
```

Aquí, `urlpatterns` es la variable que define la lista de rutas de URL, y `'url-path/'` define la ruta que debe coincidir.
`views.my_view` es la función de vista que se invocará cuando haya una coincidencia de URL, y `name='my-view'` es el nombre de la función de vista utilizado para hacer referencia a la vista. Puede haber una situación en la que, en otra parte de la aplicación, queramos obtener la URL de esta vista. No querríamos codificar el valor de forma rígida (*hardcode*), ya que luego tendría que especificarse dos veces en la base de código. En su lugar, podemos acceder a la URL utilizando el nombre de la vista, de la siguiente manera:

```python
from django.urls import reverse

url = reverse('my-view')
```

Si es necesario, también podemos usar una expresión regular en una ruta de URL para hacer coincidir patrones de cadena usando `re_path()`:

```python
urlpatterns = [
    re_path(r'^url-path/(?P<name>pattern)/$', views.my_view, name='my-view')
]
```

Aquí, `name` se refiere al nombre del patrón, que puede ser cualquier patrón de expresión regular de Python, y este debe coincidir antes de llamar a la función de vista definida. También puedes pasar parámetros desde la URL a la vista misma, como se muestra en este ejemplo:

```python
urlpatterns = [
    path('url-path/<int:id>/', views.my_view, name='my-view')
]
```

En el ejemplo anterior, `<int:id>` le dice a Django que busque URLs que contengan un número entero en esta posición de la cadena y que asigne el valor de ese número entero al argumento `id`. Esto significa que si el usuario navega a `/url-path/14/`, el argumento de palabra clave `id=14` se pasa a la vista. Esto suele ser útil cuando una vista necesita buscar un objeto específico en la base de datos y devolver los datos correspondientes. Por ejemplo, supongamos que tenemos un modelo `User` y queremos que la vista muestre el nombre del usuario. La vista podría escribirse de la siguiente manera:

```python
def my_view(request, id):
    user = User.objects.get(id=id)
    return HttpResponse(f"This user's name is { user.first_name } { user.last_name }")
```

Cuando el usuario accede a `/url-path/14/`, se llama a `my_view` y el argumento `id=14` se pasa a la función.

Este es el flujo de trabajo típico cuando se invoca una URL como `http://0.0.0.0:8000/url-path/` mediante un navegador web:
1. Se realiza una solicitud HTTP a la aplicación en ejecución para la ruta URL. Al recibir la solicitud, el servidor busca la configuración `ROOT_URLCONF` presente en el archivo `settings.py`:
   ```python
   ROOT_URLCONF = 'project_name.urls'
   ```
   Esto determina el archivo de configuración de URLs que se utilizará primero. En este caso, es el archivo de URL presente en el directorio del proyecto, `project_name/urls.py`.
2. A continuación, Django recorre la lista llamada `urlpatterns` y, una vez que coincide `url-path/` con la ruta presente en la URL, `http://0.0.0.0:8000/url-path/`, invoca la función de vista correspondiente.

La configuración de URLs a veces también se denomina *URL conf* o mapeo de URLs, y estos términos se suelen utilizar indistintamente. En general, podemos decir que la configuración de URLs está presente para enrutar la solicitud de URL enviada en un navegador a una función de vista adecuada. Para comprender mejor las vistas y la configuración de URLs, comencemos con un ejercicio simple.

#### Ejercicio 3.01 – Implementación de una vista simple basada en funciones

En este ejercicio, escribiremos una vista basada en funciones muy básica y utilizaremos la configuración de URLs asociada para mostrar el mensaje `Welcome to Bookr!` en un navegador web. También le diremos al usuario cuántos libros tenemos en la base de datos. Comencemos con los pasos:

1. Primero, asegúrate de que `ROOT_URLCONF` en `bookr/settings.py` apunte al archivo de URL del proyecto agregando el siguiente comando:
   ```python
   ROOT_URLCONF = 'bookr.urls'
   ```

2. Abre el archivo `bookr/reviews/views.py` y agrega el siguiente fragmento de código:
   ```python
   from django.http import HttpResponse
   from .models import Book

   def home(request):
       message = f"""<html><h1>Welcome to Bookr!</h1> <p>{Book.objects.count()} books and counting!</p></html>"""
       return HttpResponse(message)
   ```
   En el fragmento anterior, primero importamos la clase `HttpResponse` del módulo `django.http`. A continuación, definimos la función `home`, que puede mostrar el mensaje `Welcome to Bookr!` en un navegador web. El objeto `request` es un parámetro de función que transporta el objeto de solicitud HTTP. La siguiente línea define la variable `message`, que contiene HTML que muestra el encabezado, seguido de una línea que cuenta la cantidad de libros disponibles en la base de datos.
   En la última línea, devolvemos un objeto `HttpResponse` con la cadena asociada con la variable `message`. Cuando se llame a la función de vista `home`, mostrará el mensaje `Welcome to Bookr! 2 Books and counting!` en el navegador web.

3. Ahora, crea la configuración de URLs para llamar a la función de vista recién creada. Abre el archivo de URL del proyecto, `bookr/urls.py`, y agrega la lista de `urlpatterns` de la siguiente manera:
   ```python
   from django.contrib import admin
   from django.urls import include, path

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', include('reviews.urls'))
   ]
   ```
   La primera línea en la lista de `urlpatterns`, es decir, `path('admin/', admin.site.urls)`, enruta a las URLs de administración si `admin/` está presente en la ruta de la URL (por ejemplo, `http://0.0.0.0:8000/admin`).
   De manera similar, considera la segunda línea, `path('', include('reviews.urls'))`. Aquí, la ruta mencionada es una cadena vacía, `''`. Si la URL no tiene ninguna ruta específica después de `http://hostname:port-number/` (por ejemplo, `http://0.0.0.0:8000/`), incluye `urlpatterns` presentes en `reviews.urls`.
   La función `include` es un atajo que te permite combinar configuraciones de URLs. Es una práctica común mantener una configuración de URLs por aplicación en tu proyecto de Django. Aquí, hemos creado una configuración de URLs separada para la aplicación `reviews` y la hemos agregado a nuestra configuración de URLs a nivel de proyecto.

4. Dado que aún no tenemos el módulo de URL `reviews.urls`, crea un archivo llamado `bookr/reviews/urls.py` y agrega las siguientes líneas de código:
   ```python
   from django.contrib import admin
   from django.urls import path
   from . import views

   urlpatterns = [
       path('', views.home, name='home'),
   ]
   ```
   Aquí, hemos vuelto a utilizar una cadena vacía para la ruta de la URL. Por lo tanto, cuando se invoque `http://0.0.0.0:8000/`, después de ser enrutado desde `bookr/urls.py` hacia `bookr/reviews/urls.py`, este patrón invocará la función de vista `home`.

5. Después de realizar cambios en los dos archivos, tenemos la configuración de URLs necesaria lista para llamar a la vista `home`. Ahora, inicia el servidor de Django con `./manage.py runserver` y escribe `http://0.0.0.0:8000` o `http://127.0.0.1:8000` en tu navegador web. Deberías poder ver el mensaje `Welcome to Bookr!`:
   *Figura 3.1 – Mostrando "Welcome to Bookr!" y el número de libros en la página de inicio*

Si no hay coincidencia de URL, Django invoca el manejo de errores, como mostrar un mensaje `404 Page not found` o algo similar.

En el desarrollo, con la opción `DEBUG` habilitada, la página de error de Django proporcionará información para ayudar a comprender el error y ayudar a solucionarlo. En el ejemplo de un error 404, mostrará los patrones de URL con los que ha intentado coincidir para encontrar una ruta a una vista para la solicitud.

En este ejercicio, aprendimos cómo escribir una función de vista básica y realizar la configuración de URLs asociada. Hemos creado una página web que muestra un mensaje simple al usuario e informa cuántos libros hay actualmente en nuestra base de datos.

Sin embargo, el lector astuto habrá notado que no se ve muy bien tener código HTML dentro de nuestra función de Python como en el ejemplo anterior. A medida que nuestras vistas se hagan más grandes, esto se volverá aún más insostenible. Por lo tanto, en la siguiente sección, dirigiremos nuestra atención a dónde se supone que debe estar nuestro código HTML: dentro de las plantillas.

---

### Sección: Trabajo con plantillas de Django

En el Ejercicio 3.01, vimos cómo crear una vista, configurar la URL y mostrar un mensaje en el navegador. Pero si recuerdas, codificamos de forma fija el mensaje HTML `Welcome to Bookr!` en la propia función de vista y devolvimos un objeto `HttpResponse`, de la siguiente manera:

```python
message = f"""<html><h1>Welcome to Bookr!</h1> <p>{Book.objects.count()} books and counting!</p></html>"""
return HttpResponse(message)
```

La codificación fija de HTML dentro de los módulos de Python no es una buena práctica, porque a medida que aumenta el contenido que se va a procesar en una página web, también lo hace la cantidad de código HTML que debemos escribir para ella. Tener una gran cantidad de código HTML entre el código de Python puede hacer que el código sea difícil de leer y mantener a largo plazo.

Por esta razón, las plantillas de Django nos brindan una mejor manera de escribir y administrar plantillas HTML. Las plantillas de Django no solo funcionan con contenido HTML estático, sino también con plantillas HTML dinámicas.

La configuración de las plantillas de Django se realiza en la variable `TEMPLATES` presente en el archivo `settings.py`. Así es como se ve la configuración predeterminada:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Revisemos cada palabra clave presente en el fragmento anterior:
- `'BACKEND': 'django.template.backends.django.DjangoTemplates'`: Esto se refiere al motor de plantillas que se utilizará. Un motor de plantillas es una API utilizada por Django para trabajar con plantillas HTML. Django está construido con Jinja2 y el motor `DjangoTemplates`. La configuración predeterminada es el motor `DjangoTemplates` y el lenguaje de plantillas de Django. Sin embargo, esto se puede cambiar para utilizar uno diferente si es necesario, como Jinja2 o cualquier otro motor de plantillas de terceros. Para nuestra aplicación Bookr, sin embargo, dejaremos esta configuración como está.
- `'DIRS': []`: Esto se refiere a la lista de directorios donde Django busca las plantillas en el orden dado.
- `'APP_DIRS': True`: Esto le indica al motor de plantillas de Django si debe buscar plantillas en las aplicaciones instaladas definidas en `INSTALLED_APPS` en el archivo `settings.py`. La opción predeterminada para esto es `True`.
- `'OPTIONS'`: Este es un diccionario que contiene configuraciones específicas del motor de plantillas. Dentro de este diccionario, hay una lista predeterminada de procesadores de contexto (*context processors*), que ayuda a que el código de Python interactúe con las plantillas para crear y representar plantillas HTML dinámicas.

La configuración predeterminada actual está bien para nuestros propósitos. Sin embargo, en el próximo ejercicio, crearemos un nuevo directorio para nuestras plantillas y necesitaremos especificar la ubicación de esta carpeta. Por ejemplo, si tenemos un directorio llamado `my_templates`, debemos especificar su ubicación agregándolo a la configuración `TEMPLATES` de la siguiente manera:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'my_templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Al definir los directorios para recopilar plantillas desde aquí, se utiliza el módulo `os` de Python, así que recuerda usar `import os` en tu archivo de configuración si utilizas este método.

`BASE_DIR` es la ruta del directorio a la carpeta del proyecto. Esto se define en el archivo `settings.py`. El método `os.path.join()` une el directorio del proyecto con el directorio de plantillas, devolviendo la ruta completa para el directorio de plantillas. En el siguiente ejercicio, usaremos una plantilla para mostrar un mensaje al usuario.

#### Ejercicio 3.02 – Uso de plantillas para mostrar un mensaje de bienvenida

En este ejercicio, crearemos nuestra primera plantilla de Django y, tal como lo hicimos en el ejercicio anterior, mostraremos el mensaje `Welcome to Bookr!` utilizando plantillas. Comencemos con los pasos:

1. Crea un directorio llamado `templates` en el directorio del proyecto `bookr` y, dentro de él, crea un archivo llamado `base.html`. Si ya tienes un directorio `templates`, considera eliminarlo o adaptar `base.html` si tienes uno. La estructura del directorio debería verse como la siguiente figura:
   *Figura 3.2 – Estructura de directorios para bookr*
   Cuando se utiliza la configuración predeterminada (es decir, cuando `DIRS` es una lista vacía), Django busca las plantillas presentes solo en el directorio de plantillas de las carpetas de las aplicaciones (la carpeta `reviews/templates` en el caso de una aplicación de reseñas de libros). Dado que incluimos el nuevo directorio de plantillas en el directorio principal del proyecto, el motor de plantillas de Django no podría encontrar el directorio a menos que esté incluido en la lista `'DIRS'`.

2. Agrega la carpeta a la configuración `TEMPLATES`:
   ```python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           'DIRS': [BASE_DIR / 'templates'],
           'APP_DIRS': True,
           'OPTIONS': {
               'context_processors': [
                   'django.template.context_processors.debug',
                   'django.template.context_processors.request',
                   'django.contrib.auth.context_processors.auth',
                   'django.contrib.messages.context_processors.messages',
               ],
           },
       },
   ]
   ```

3. Agrega las siguientes líneas de código al archivo `base.html`:
   ```html
   <!doctype html>
   <html lang="en">
   <head>
       <meta charset="utf-8">
       <title>Home Page</title>
   </head>
   <body>
       <h1>Welcome to Bookr!</h1>
   </body>
   </html>
   ```
   Este es un HTML simple que muestra el mensaje `Welcome to Bookr!` en el encabezado.

4. Modifica el código dentro de `bookr/reviews/views.py` para que se vea de la siguiente manera:
   ```python
   from django.shortcuts import render

   def home(request):
       return render(request, 'base.html')
   ```
   Dado que ya hemos configurado el directorio `'templates'` en la configuración de `TEMPLATES`, `base.html` está disponible para su uso con el motor de plantillas. El código procesa el archivo `base.html` utilizando el método `render` importado del módulo `django.shortcuts`.

5. Guarda los archivos, ejecuta `./manage.py runserver` y abre la URL `http://0.0.0.0:8000/` o `http://127.0.0.1:8000/` para comprobar la carga de la plantilla recién agregada en el navegador:
   *Figura 3.3 – Mostrando "Welcome to Bookr!" en la página de inicio*

En este ejercicio, creamos una plantilla HTML y usamos plantillas y vistas de Django para devolver el mensaje `Welcome to Bookr!`.

En general, podemos decir que las plantillas de Django nos ayudan a trabajar con plantillas HTML y presentar información al usuario en el formato deseado.

A continuación, aprenderemos sobre el lenguaje de plantillas de Django, que se puede utilizar para renderizar los datos de la aplicación junto con las plantillas HTML.

---

### Sección: Lenguaje de plantillas de Django (Django Template Language)

Las vistas de Django no solo devuelven plantillas HTML estáticas, sino que también pueden agregar datos dinámicos de la aplicación al generar las plantillas. Junto con los datos, también podemos incluir algunos elementos de programación en las plantillas. Todo esto en conjunto forma los conceptos básicos del lenguaje de plantillas de Django. En las siguientes secciones se analizan algunas de las partes básicas del lenguaje de plantillas de Django, como variables de plantilla, etiquetas de plantilla, comentarios y filtros.

#### Variables de plantilla (Template variables)

Una variable de plantilla se representa entre dos llaves, como se muestra aquí:

```django
{{ variable }}
```

Cuando esto está presente en la plantilla, el valor transportado por las variables será reemplazado en la plantilla. Las variables de plantilla ayudan a agregar los datos de la aplicación en las plantillas. Se pueden agregar definiendo datos de contexto para una vista, modificando el código dentro de `bookr/reviews/views.py` para que se vea de la siguiente manera:

```python
from django.shortcuts import render

def home(request):
    context_data = {
        "template_variable": "I am a template variable."
    }
    return render(request, 'base.html', context=context_data)
```

Luego, en `base.html`, la variable se incluiría de la siguiente manera:

```html
<body>
    {{ template_variable }}
</body>
```

#### Etiquetas de plantilla (Template tags)

Una etiqueta es similar a un flujo de control programático, como una condición `if` o un bucle `for`. Una etiqueta se representa entre dos llaves y signos de porcentaje, como se muestra. He aquí un ejemplo de un bucle `for` que itera sobre una lista usando etiquetas de plantilla:

```django
{% for element in element_list %}
{% endfor %}
```

A diferencia de la programación en Python, también agregamos el final del flujo de control agregando la etiqueta de finalización, como `{% endfor %}`. Esto se puede utilizar junto con variables de plantilla para mostrar los elementos de la lista, como se muestra aquí:

```html
<ul>
    {% for element in element_list %}
        <li>{{ element.title }}</li>
    {% endfor %}
</ul>
```

#### Comentarios

Los comentarios en el lenguaje de plantillas de Django se pueden escribir como se muestra aquí; todo lo que esté entre `{% comment %}` y `{% endcomment %}` se comentará:

```django
{% comment %}
    <p>This text has been commented out</p>
{% endcomment %}
```

#### Filtros (Filters)

Los filtros se pueden usar para modificar una variable para representarla en un formato diferente. La sintaxis para un filtro es una variable separada del nombre del filtro mediante un símbolo de tubería (`|`):

```django
{{ variable|filter }}
```

Aquí hay algunos ejemplos de filtros integrados:
- `{{ variable|lower }}`: Convierte la cadena de la variable a minúsculas.
- `{{ variable|title }}`: Convierte la primera letra de cada palabra a mayúsculas.

Usemos los conceptos que hemos aprendido hasta ahora para desarrollar la aplicación de reseñas de libros.

#### Ejercicio 3.03 – Visualización de una lista de libros y reseñas

En este ejercicio, crearemos una página web que pueda mostrar una lista de todos los libros, sus calificaciones y la cantidad de reseñas presentes en la aplicación de reseñas de libros. Para esto, utilizaremos algunas características del lenguaje de plantillas de Django, como variables y etiquetas de plantilla, para pasar los datos de la aplicación de reseñas de libros a las plantillas y mostrar datos significativos en la página web:

1. Crea un archivo llamado `utils.py` en `bookr/reviews/utils.py` y agrega el siguiente código:
   ```python
   def average_rating(rating_list):
       if not rating_list:
           return 0
       return round(sum(rating_list) / len(rating_list))
   ```
   `average_rating` es un método auxiliar que se utilizará para calcular la calificación promedio de un libro.

2. Incluye estas declaraciones de importación al principio de `bookr/reviews/views.py`:
   ```python
   from django.shortcuts import render
   from .models import Book
   from .utils import average_rating
   ```
   Esto sirve para importar la función `render` de Django, la clase de modelo `Book` y el método auxiliar que acabamos de agregar.

3. Agrega esta función a `bookr/reviews/views.py`:
   ```python
   def book_list(request):
       books = Book.objects.all()
       book_list = []
       for book in books:
           reviews = book.review_set.all()
           if reviews:
               book_rating = average_rating([
                   review.rating for review in reviews])
               number_of_reviews = len(reviews)
           else:
               book_rating = None
               number_of_reviews = 0
           book_list.append({
               'book': book,
               'book_rating': book_rating,
               'number_of_reviews': number_of_reviews})
       context = {
           'book_list': book_list
       }
       return render(request, 'reviews/book_list.html', context)
   ```
   Esta es una vista para mostrar la lista de libros para la aplicación Bookr. Este ejemplo incluye una carpeta `reviews` en el nombre de la plantilla para la función `render`. Esta carpeta no se creó en la configuración inicial del directorio `templates` con la plantilla `base.html`, por lo que es probable que el código arroje un error `TemplateDoesNotExist`. Es una buena práctica crear una carpeta con el mismo nombre que la aplicación para sus plantillas. Estas plantillas específicas de la aplicación pueden heredar de la misma plantilla base, que cubre la configuración básica para todo el proyecto, como hojas de estilo y bibliotecas de JavaScript comunes a todas las plantillas.
   Django leerá todos los directorios de plantillas y recopilará los archivos, por lo que sin una estructura de carpetas, no hay diferenciación entre dos plantillas con el mismo nombre. Aquí, `book_list` es la función de vista. En esta función, comenzamos consultando la lista de todos los libros. A continuación, para cada libro, calculamos la calificación promedio y el número de reseñas publicadas. Toda esta información para cada libro se agrega a una lista llamada `book_list` como una lista de diccionarios. Luego, esta lista se agrega a un diccionario llamado `context` y se pasa a la función `render`.
   La función `render` toma tres parámetros: el primero es el objeto `request` que se pasó a la vista, el segundo es la plantilla HTML `book_list.html`, que mostrará la lista de libros, y el tercero es `context`, que pasamos a la plantilla.
   Dado que hemos pasado `book_list` como parte del contexto, la plantilla utilizará esto para representar la lista de libros mediante etiquetas y variables de plantilla.

4. Crea el archivo `book_list.html` en la ruta `bookr/reviews/templates/reviews/book_list.html` y agrega el siguiente código HTML al archivo:
   ```html
   <!doctype html>
   <html lang="en">
   <head>
       <meta charset="utf-8">
       <title>Bookr</title>
   </head>
   <body>
       <h1>Book Review application</h1>
       <hr>
   ```
   Puedes encontrar el código completo en la carpeta `Chapter03` en el repositorio de GitHub de este libro.
   Esta es una plantilla HTML simple con etiquetas y variables de plantilla que iteran sobre `book_list` para mostrar la lista de libros.

5. En `bookr/reviews/urls.py`, agrega a `urlpatterns` el siguiente patrón de URL para invocar la vista `book_list`:
   ```python
   path('books/', views.book_list, name='book_list'),
   ```
   Esto configura la URL para la función de vista `book_list`.

6. Guarda todos los archivos modificados y espera a que se reinicie el servicio de Django. Abre `http://0.0.0.0:8000/books/` en el navegador y deberías ver algo similar a la siguiente figura:
   *Figura 3.4 – Lista de libros presentes en la aplicación de reseñas de libros*

En este ejercicio, creamos una función de vista, creamos plantillas y agregamos una configuración de URLs que puede mostrar una lista de todos los libros presentes en la aplicación. Aunque pudimos mostrar una lista de libros utilizando una sola plantilla, a continuación, exploremos un poco sobre cómo trabajar con múltiples plantillas en una aplicación que tiene código común o similar.

---

### Sección: Herencia de plantillas (Template inheritance)

A medida que construyamos el proyecto, la cantidad de plantillas aumentará. Es muy probable que cuando diseñemos la aplicación, algunas de las páginas se vean similares y tengan código HTML común para ciertas funciones. Mediante la herencia de plantillas, podemos heredar el código HTML común en otros archivos HTML. Esto es similar a la herencia de clases en Python, donde la clase principal (*parent class*) tiene todo el código común y la clase secundaria (*child class*) tiene esos extras que son únicos para los requisitos secundarios.

Sería una buena práctica actualizar la vista `home` para usar un `home.html` que sea hijo de `base.html`. Tener la plantilla base como ancestro común de las plantillas permite que todo el código reutilizable (*boilerplate*), hojas de estilo, JavaScript, etc., se defina una vez y luego se comparta entre todas las plantillas.

Por ejemplo, consideremos que la siguiente es una plantilla principal llamada `base.html`:

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>{% block title %}Hello World{% endblock %}</title>
</head>
<body>
    <h1>Hello World using Django templates!</h1>
    {% block content %}
    {% endblock %}
</body>
</html>
```

El siguiente es un ejemplo de una plantilla secundaria:

```html
{% extends 'base.html' %}

{% block content %}
<h1>How are you doing?</h1>
{% endblock %}
```

En el fragmento anterior, la línea `{% extends 'base.html' %}` extiende la plantilla de `base.html`, que es la plantilla principal. Después de extender desde la plantilla principal, cualquier código HTML entre el bloque `content` se mostrará junto con la plantilla principal. Una vez que se renderiza la plantilla secundaria, así es como se ve en el navegador:

*Figura 3.5 – Mensaje de saludo después de extender la plantilla base.html*

En la siguiente sección, realizaremos algunos estilos de plantilla utilizando Bootstrap.

---

### Sección: Estilos de plantillas con Bootstrap

Hemos visto cómo mostrar todos los libros mediante vistas, configuración de URLs y plantillas. Aunque pudimos mostrar toda la información en el navegador, sería aún mejor si pudiéramos agregar algunos estilos y hacer que la página web se vea mejor. Para esto, podemos agregar algunos elementos de Bootstrap. Bootstrap es un framework de Hojas de Estilo en Cascada (*Cascading Style Sheets* o CSS) de código abierto que es particularmente bueno para diseñar páginas adaptables (*responsive*) que funcionan en navegadores móviles y de escritorio.

Usar Bootstrap es simple. Primero, debes agregar el CSS de Bootstrap a tu HTML. Puedes experimentar por ti mismo creando un nuevo archivo llamado `example.html`. Llena el archivo con el siguiente código y ábrelo en un navegador:

```html
<!doctype html>
<html lang="en">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
</head>
<body>
    Content goes here
</body>
</html>
```

El enlace CSS de Bootstrap en el código anterior agrega la biblioteca CSS de Bootstrap a tu página. Esto significa que ciertos tipos de elementos y clases HTML heredarán sus estilos de Bootstrap. Por ejemplo, si agregas la clase `btn-primary` a la clase de un botón, el botón se mostrará en azul con texto blanco. Intenta agregar lo siguiente entre `<body>` y `</body>`:

```html
<h1>Welcome to my Site</h1>
<button type="button" class="btn btn-primary">
    Checkout my Blog!</button>
```

Verás que el título y el botón tienen un estilo agradable, utilizando los estilos predeterminados de Bootstrap:

*Figura 3.6 – Visualización después de aplicar Bootstrap*

Esto se debe a que en el código CSS de Bootstrap, especifican el color de la clase `btn-primary` con el siguiente código:

```css
.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff
}
```

Puedes ver que el uso de bibliotecas CSS de terceros como Bootstrap te permite crear rápidamente componentes con un estilo agradable sin necesidad de escribir demasiado CSS. A continuación, utilizaremos la herencia de plantillas de una plantilla base y también agregaremos algunos elementos de estilo, como una barra de navegación.

Te recomendamos que explores Bootstrap más a fondo con su tutorial aquí: [https://getbootstrap.com/docs/5.3/getting-started/introduction/](https://getbootstrap.com/docs/5.3/getting-started/introduction/).

#### Ejercicio 3.04 – Adición de herencia de plantillas y una barra de navegación Bootstrap

En este ejercicio, utilizaremos la herencia de plantillas para heredar los elementos de plantilla de una plantilla base y reutilizarlos en la plantilla `book_list` para mostrar la lista de libros. También utilizaremos ciertos elementos de Bootstrap en el archivo HTML base para agregar una barra de navegación en la parte superior de nuestra página. El código de Bootstrap para `base.html` se tomó de [https://getbootstrap.com/docs/5.3/getting-started/introduction/](https://getbootstrap.com/docs/5.3/getting-started/introduction/) y [https://getbootstrap.com/docs/5.3/components/navbar/](https://getbootstrap.com/docs/5.3/components/navbar/). Comencemos con los pasos:

1. Abre el archivo `base.html` desde la ubicación `bookr/templates/base.html`. Elimina cualquier código existente y reemplázalo con el siguiente código:
   ```html
   <!doctype html>
   {% load static %}
   <html lang="en">
   <head>
       <!-- Required meta tags -->
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <!-- Bootstrap CSS -->
   ```
   Puedes ver el código completo de este archivo en la carpeta `Chapter03` en el repositorio de GitHub de este libro. Este es un archivo `base.html` con todos los elementos de Bootstrap para el estilo y la barra de navegación.

2. A continuación, abre la plantilla en `bookr/reviews/templates/reviews/book_list.html`, elimina todo el código existente y reemplázalo con el siguiente código:
   ```html
   {% extends 'base.html' %}
   {% block content %}
   <ul class="list-group">
       {% for item in book_list %}
           <li class="list-group-item">
               <span class="text-info">Title: </span>
               <span>{{ item.book.title }}</span>
               <br>
               <span class="text-info">Publisher: </span><span>{{ item.book.publisher }}</span>
   ```
   Puedes ver el código completo de este archivo en la carpeta `Chapter03` en el repositorio de GitHub de este libro. Esta plantilla se ha configurado para heredar el archivo `base.html`, y también se le han agregado algunos elementos de estilo para mostrar la lista de libros. La parte de la plantilla que ayuda a heredar el archivo `base.html` es la siguiente:
   ```html
   {% extends 'base.html' %}
   {% block content %}
   {% endblock %}
   ```

3. Después de agregar las dos nuevas plantillas, abre cualquiera de las URLs (`http://0.0.0.0:8000/books/` o `http://127.0.0.1:8000/books/`) en tu navegador web para ver la página de lista de libros, que ahora debería verse con un formato ordenado:
   *Figura 3.7 – Página de lista de libros con formato ordenado*

En este ejercicio, agregamos algo de estilo a la aplicación mediante Bootstrap y también usamos la herencia de plantillas mientras mostramos la lista de libros de la aplicación de reseñas de libros. Hasta ahora, hemos trabajado exhaustivamente en mostrar todos los libros presentes en la aplicación. En la siguiente actividad, mostrarás los detalles y las reseñas de un libro individual.

---

### Sección: Actividad 3.01 – Implementación de la vista de detalles del libro

En esta actividad, implementarás una nueva vista, plantilla y configuración de URLs para mostrar estos detalles de un libro: título, editorial, fecha de publicación y calificación general. Además de estos detalles, la página también debe mostrar todos los comentarios de las reseñas, especificando el nombre del autor del comentario y las fechas en que se escribieron y (si corresponde) modificaron los comentarios. Los siguientes pasos te ayudarán a completar esta actividad:

1. Crea un *endpoint* de detalles del libro que extienda la plantilla base.
2. Crea una vista de detalles del libro que tome la clave primaria de un libro específico como argumento y devuelva una página HTML que enumere los detalles del libro y las reseñas asociadas.
3. Agrega la configuración de URLs requerida en `urls.py`. La URL de la vista de detalles del libro debe ser `http://0.0.0.0:8000/books/1/` (donde 1 representará el ID del libro al que se accede). Puedes utilizar el método `get_object_or_404` para recuperar el libro con la clave primaria dada.
   La función `get_object_or_404` es un atajo útil para recuperar una instancia basada en su clave primaria. También podrías hacer esto usando el método `.get()` descrito en el Capítulo 2: `Book.objects.get(pk=pk)`. Sin embargo, `get_object_or_404` tiene la ventaja adicional de devolver una respuesta HTTP `404 Not Found` si el objeto no existe. Si simplemente usamos `get()` y alguien intenta acceder a un objeto que no existe, nuestro código Python generará una excepción y devolverá una respuesta HTTP `500 Server Error`. Esto no es deseable porque parece como si nuestro servidor no hubiera podido manejar la solicitud correctamente.

Al final de la actividad, deberías poder hacer clic en el botón *Reviews* en la página de lista de libros y obtener una vista detallada del libro. La vista detallada debe tener todos los detalles mostrados, como se muestra en la siguiente captura de pantalla:

*Figura 3.8 – Página que muestra los detalles del libro*

Hasta ahora en esta sección, hemos aprendido a usar plantillas de Django para presentar información estática al usuario, el lenguaje de plantillas para mostrar los datos dinámicos de la aplicación y Bootstrap para agregar estilo a la información y hacerla presentable a los usuarios.

---

### Sección: Resumen

Este capítulo cubrió la infraestructura central requerida para manejar una solicitud HTTP a nuestro sitio web. La solicitud se asigna primero a través de patrones de URL a una vista adecuada. Los parámetros de la URL también se pasan a la vista para especificar el objeto que se muestra en la página.

La vista es responsable de recopilar cualquier información necesaria para mostrarla en el sitio web y luego pasa este diccionario a una plantilla, que representa la información como código HTML que se puede devolver como respuesta al usuario. Cubrimos tanto las vistas basadas en clases como las basadas en funciones y aprendimos sobre el lenguaje de plantillas de Django y la herencia de plantillas. Creamos dos páginas nuevas para la aplicación de reseñas de libros, una que muestra todos los libros presentes y la otra es la página de vista de detalles del libro.

En el próximo capítulo, aprenderemos sobre el administrador de Django y el superusuario, el registro de modelos y la realización de operaciones de creación, lectura, actualización y eliminación (CRUD) utilizando el sitio de administración.
