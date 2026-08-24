# Parte 3: Características avanzadas de Django

## Capítulo 12: Creación de una API REST

En el capítulo anterior, aprendimos sobre plantillas y vistas basadas en clases. Estos conceptos ayudan enormemente a expandir la gama de funcionalidades que podemos proporcionar al usuario en el frontend (en su navegador web). Sin embargo, eso no es suficiente para construir una aplicación web moderna. Las aplicaciones web suelen tener el frontend construido con una biblioteca completamente separada, como ReactJS o AngularJS. Estas bibliotecas proporcionan herramientas poderosas para crear interfaces de usuario dinámicas, pero no se comunican directamente con nuestro código Django de backend ni con la base de datos. El código frontend simplemente se ejecuta en el navegador web y no tiene acceso directo a ningún dato en nuestro servidor backend. Por lo tanto, necesitamos crear una forma para que estas aplicaciones "hablen" con nuestro código de backend. Una de las mejores formas de hacer esto en Django es mediante el uso de APIs REST.

Las interfaces de programación de aplicaciones (APIs) se utilizan para facilitar la interacción entre diferentes piezas de software y se comunican mediante el Protocolo de Transferencia de Hipertexto (HTTP). Este es el protocolo estándar para la comunicación entre servidores y clientes y es fundamental para la transferencia de información en la web. Las APIs reciben solicitudes y envían respuestas en formato HTTP.

Este capítulo presenta las APIs REST y Django REST framework (DRF). Comenzarás implementando una API simple para el proyecto `bookr`. A continuación, aprenderás sobre la serialización de instancias de modelos, que es un paso clave en la entrega de datos al lado frontend de las aplicaciones Django. Finalmente, explorarás diferentes tipos de vistas de API, incluidos los tipos basados en funciones y en clases.

En nuestro caso de uso en este capítulo, una API ayudará a facilitar la interacción entre nuestro backend de Django y nuestro código JS de frontend. Por ejemplo, imagina que queremos crear una aplicación frontend que permita a los usuarios agregar nuevos libros a la base de datos de Bookr. El navegador web del usuario enviaría un mensaje (una solicitud HTTP) a nuestra API para decir que desea crear una entrada para un libro nuevo y tal vez incluir algunos detalles sobre el libro en ese mensaje. Nuestro servidor enviaría una respuesta para informar si el libro se agregó correctamente o no. El navegador web podría entonces mostrar el resultado de su acción al usuario.

En este capítulo, cubriremos los siguientes temas:
- Comprensión de las APIs REST
- DRF (Django REST framework)
- Comprensión de los serializadores
- Simplificación del código mediante ViewSets
- Implementación de la autenticación

Al final de este capítulo, podrás implementar puntos de conexión de API personalizados, incluida la autenticación basada en tokens.

---

### Sección: Requisitos técnicos

Todos los archivos de código de este capítulo se pueden encontrar en la carpeta `Chapter12` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Comprensión de las APIs REST

La mayoría de las APIs web modernas se pueden clasificar como APIs de transferencia de estado representacional (*Representational State Transfer*, REST). Las APIs REST son simplemente un tipo de API que se enfoca en comunicar y sincronizar el estado de los objetos entre el servidor de base de datos y el cliente frontend.

Por ejemplo, imagina que estás actualizando tus datos en un sitio web en el que has iniciado sesión en tu cuenta. Cuando vas a la página de detalles de la cuenta, el servidor web le informa a tu navegador sobre los diversos detalles adjuntos a tu cuenta. Cuando cambias los valores en esa página, el navegador envía los datos actualizados al servidor web y le indica que actualice estos detalles en la base de datos. Si la acción tiene éxito, el sitio web te mostrará un mensaje de confirmación.

Este es un ejemplo muy simple de lo que se conoce como arquitectura desacoplada (*decoupled architecture*) entre los sistemas de frontend y backend. El desacoplamiento permite una mayor flexibilidad y facilita la actualización o el cambio de componentes en tu arquitectura. Por lo tanto, supongamos que deseas crear un nuevo sitio web de frontend. En tal caso, no tienes que cambiar el código de backend en absoluto, siempre que tu nuevo frontend esté diseñado para realizar las mismas solicitudes de API que el anterior.

Las APIs REST no tienen estado (*stateless*), lo que significa que ni el cliente ni el servidor almacenan ningún estado intermedio para realizar la comunicación. Cada vez que se realiza una solicitud, los datos se procesan y se devuelve una respuesta sin que el protocolo tenga que almacenar ningún dato intermedio. Lo que esto significa es que la API procesa cada solicitud de forma aislada. No necesita almacenar información sobre la sesión en sí. Esto contrasta con un protocolo con estado (*stateful*, como el Protocolo de Control de Transmisión o TCP), que mantiene información sobre la sesión durante su vida útil.

Por lo tanto, un servicio web RESTful, como su nombre indica, es una colección de APIs REST utilizadas para llevar a cabo un conjunto de tareas. Por ejemplo, si desarrollamos un conjunto de APIs REST para que la aplicación `bookr` lleve a cabo un determinado conjunto de tareas, podemos llamarlo un servicio web RESTful. A continuación, comenzaremos a trabajar con DRF.

---

### Sección: DRF

DRF es una biblioteca de Python de código abierto que se puede utilizar para desarrollar APIs REST para un proyecto de Django. DRF tiene la mayor parte de la funcionalidad necesaria incorporada para ayudar a desarrollar APIs para cualquier proyecto de Django. A lo largo de este capítulo, lo usaremos para desarrollar APIs para nuestro proyecto `bookr`. En la siguiente sección, instalaremos y configuraremos DRF.

#### Instalación y configuración

Para instalar `djangorestframework` en la configuración de entorno virtual junto con PyCharm, sigue estos pasos:

1. Ingresa el siguiente código en tu aplicación de terminal o símbolo del sistema para hacer esto:
   ```bash
   pip install djangorestframework
   ```
2. A continuación, abre el archivo `settings.py` y agrega `rest_framework` a `INSTALLED_APPS` como se muestra en el siguiente fragmento:
   ```python
   INSTALLED_APPS = [
       'reviews.adminconfig.ReviewsAdminConfig',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'rest_framework',
       'reviews',
   ]
   ```

Ahora estás listo para comenzar a usar DRF para crear tu primera API simple.

#### Vistas de API funcionales

En el Capítulo 3 (*Mapeo de URLs, Vistas y Plantillas*), aprendimos sobre vistas funcionales simples que toman una solicitud y devuelven una respuesta. Podemos escribir vistas funcionales similares usando DRF. Sin embargo, ten en cuenta que las vistas basadas en clases se usan más comúnmente y se cubrirán a continuación. Una vista funcional se crea simplemente agregando el siguiente decorador a una vista normal, de la siguiente manera:

```python
from rest_framework.decorators import api_view

@api_view
def my_view(request):
    ...
```

Este decorador toma la vista funcional y la convierte en una subclase de `APIView` de DRF. Es una forma rápida de incluir una vista existente como parte de tu API. Utilizando lo que hemos aprendido hasta ahora, crearemos una API REST simple usando DRF en la siguiente sección.

#### Ejercicio 12.01 – Creación de una API REST simple

En este ejercicio, crearás tu primera API REST usando DRF e implementarás un punto de conexión usando una vista funcional. Crearás este punto de conexión para ver el número total de libros en la base de datos:

1. Deberás tener instalado DRF en tu sistema para continuar con este ejercicio. Si aún no lo has instalado, asegúrate de consultar la sección *Instalación y configuración* anteriormente en este capítulo.
2. Crea `api_views.py` en la carpeta `bookr/reviews`. Las vistas de la API REST funcionan como las vistas convencionales de Django. Podríamos haber agregado las vistas de API, junto con las otras vistas, en el archivo `views.py`. Sin embargo, tener nuestras vistas de API REST en un archivo separado nos ayudará a mantener una base de código más limpia.
3. Agrega el siguiente código en `api_views.py`:
   `bookr/reviews/api_views.py`:
   ```python
   from rest_framework.decorators import api_view
   from rest_framework.response import Response
   from .models import Book

   @api_view(['GET'])
   def first_api_view(request):
       num_books = Book.objects.count()
       return Response({"num_books": num_books})
   ```
   La primera línea importa el decorador `api_view`, que convertirá nuestra vista funcional en una que se pueda usar con DRF, y la segunda línea importa `Response`, que se usará para devolver una respuesta.
   La función de vista devuelve un objeto `Response` que contiene un diccionario con la cantidad de libros en nuestra base de datos.
4. Abre `bookr/reviews/urls.py` e importa el módulo `api_views`. Luego, agrega una nueva ruta al módulo `api_views` en los patrones de URL que hemos desarrollado a lo largo de este curso, de la siguiente manera:
   `bookr/reviews/urls.py`:
   ```python
   from . import views, api_views

   urlpatterns = [
       path('api/first_api_view/', api_views.first_api_view),
       …
   ]
   ```
5. Inicia el servicio de Django con el comando `python manage.py runserver` y ve a `http://127.0.0.1:8000/api/first_api_view/` para realizar tu primera solicitud de API. Tu pantalla debería aparecer como en la Figura 12.1:
   *Figura 12.1: Primera vista de API con el número de libros*
   Llamar a este punto de conexión de URL realizó una solicitud GET predeterminada al punto de conexión de API, que devolvió un par clave-valor de notación de objetos de JavaScript (JSON) (`"num_books": 0`). Además, observa cómo DRF proporciona una interfaz agradable para ver e interactuar con las APIs.
6. También podríamos usar el comando `curl` (*client URL*) de Linux para enviar una solicitud HTTP de la siguiente manera:
   ```bash
   curl http://127.0.0.1:8000/api/first_api_view/
   ```
   Salida:
   ```json
   {"num_books":18}
   ```
   Alternativamente, si estás usando Windows 10, puedes realizar una solicitud HTTP equivalente con `curl.exe` desde el símbolo del sistema de la siguiente manera:
   ```cmd
   curl.exe http://127.0.0.1:8000/api/first_api_view/
   ```

En este ejercicio, aprendimos a crear una vista de API utilizando DRF y una vista funcional simple. Ahora veremos una forma más elegante de convertir entre la información almacenada en la base de datos y lo que devuelve nuestra API mediante serializadores.

---

### Sección: Comprensión de los serializadores

A estas alturas, estamos bien versados en la forma en que Django trabaja con los datos en nuestra aplicación. En términos generales, las columnas de una tabla de base de datos se definen en una clase en `models.py`, y cuando accedemos a una fila de la tabla, estamos trabajando con una instancia de esa clase. Idealmente, a menudo solo queremos pasar este objeto a nuestra aplicación frontend. Por ejemplo, si quisiéramos crear un sitio web que mostrara una lista de libros en nuestra aplicación Bookr, querríamos llamar a la propiedad `title` de cada instancia de libro para saber qué cadena mostrar al usuario. Sin embargo, nuestra aplicación frontend no sabe nada sobre Python y necesita recuperar estos datos a través de una solicitud HTTP, que simplemente devuelve una cadena en un formato específico.

Esto significa que cualquier información traducida entre Django y el frontend (a través de nuestra API) debe realizarse representando la información en formato JSON. Los objetos JSON se parecen a un diccionario de Python, excepto que hay algunas reglas adicionales que restringen la sintaxis exacta. En nuestro ejemplo anterior en el Ejercicio 12.01, la API devolvió el siguiente objeto JSON que contenía la cantidad de libros en nuestra base de datos:

```json
{"num_books": 0}
```

¿Pero qué pasa si quisiéramos devolver los detalles completos sobre un libro real en nuestra base de datos con nuestra API? La clase de serializador de DRF ayuda a convertir objetos complejos de Python a formatos como JSON o XML para que puedan transmitirse a través de la web mediante el protocolo HTTP. La parte de DRF que realiza esta conversión se denomina serializador (*serializer*). Los serializadores también realizan la deserialización, que se refiere a convertir los datos serializados nuevamente en objetos de Python para que los datos se puedan procesar en la aplicación. En la siguiente sección, implementaremos una forma de mostrar una lista de libros.

#### Ejercicio 12.02 – Creación de una vista de API para mostrar una lista de libros

En este ejercicio, utilizarás serializadores para crear una API que devuelva una lista de todos los libros presentes en la aplicación Bookr:

1. Crea un archivo llamado `serializers.py` en la carpeta `bookr/reviews`. Este es el archivo donde colocaremos todo el código de serializadores para las APIs.
2. Agrega el siguiente código a `serializers.py`:
   `bookr/reviews/serializers.py`:
   ```python
   from rest_framework import serializers

   class PublisherSerializer(serializers.Serializer):
       name = serializers.CharField()
       website = serializers.URLField()
       email = serializers.EmailField()

   class BookSerializer(serializers.Serializer):
       title = serializers.CharField()
       publication_date = serializers.DateField()
       isbn = serializers.CharField()
       publisher = PublisherSerializer()
   ```
   Aquí, la primera línea importa los `serializers` del módulo `rest_framework`.
   Siguiendo las importaciones, hemos definido dos clases: `PublisherSerializer` y `BookSerializer`. Como sugieren los nombres, son serializadores para los modelos `Publisher` y `Book`, respectivamente. Ambos serializadores son subclases de `serializers.Serializer`, y hemos definido tipos de campos para cada serializador, como `CharField`, `URLField`, `EmailField`, etc.
   Mira el modelo `Publisher` en el archivo `bookr/reviews/models.py`. El modelo `Publisher` tiene los atributos `name`, `website` y `email`. Entonces, para serializar un objeto `Publisher`, necesitamos los atributos `name`, `website` y `email` en la clase serializadora, que hemos definido en consecuencia en `PublisherSerializer`.
   Del mismo modo, para el modelo `Book`, hemos definido `title`, `publication_date`, `isbn` y `publisher` como los atributos deseados en `BookSerializer`. Dado que `publisher` es una clave foránea para el modelo `Book`, hemos utilizado `PublisherSerializer` como serializador para el atributo `publisher`.
3. Abre `bookr/reviews/api_views.py`, elimina cualquier código preexistente y agrega el siguiente código:
   `bookr/reviews/api_views.py`:
   ```python
   from rest_framework.decorators import api_view
   from rest_framework.response import Response
   from .models import Book
   from .serializers import BookSerializer

   @api_view(['GET'])
   def all_books(request):
       books = Book.objects.all()
       book_serializer = BookSerializer(books, many=True)
       return Response(book_serializer.data)
   ```
   En la segunda línea, hemos importado la clase `BookSerializer` recién creada desde el módulo `serializers`.
   Luego agregamos una vista funcional, `all_books` (como en el ejercicio anterior). Esta vista toma un `QuerySet` que contiene todos los libros y luego los serializa usando `BookSerializer`. La clase serializadora también toma un argumento, `many=True`, que indica que el objeto `books` es un `QuerySet` o una lista de muchos objetos. Recuerda que la serialización toma objetos de Python y los devuelve en un formato serializable JSON, de la siguiente manera:
   ```text
   [OrderedDict([('title', 'Advanced Deep Learning with Keras'), ('publication_date', '2018-10-31'), ('isbn', '9781788629416'), ('publisher', OrderedDict([('name', 'Packt Publishing'), ('website', 'https://www.packtpub.com/'), ('email', 'info@packtpub.com')]))]), OrderedDict([('title', 'Hands-On Machine Learning for Algorithmic Trading'), ('publication_date', '2018-12-31'), ('isbn', '9781789346411'), ('publisher', OrderedDict([('name', 'Packt Publishing'), ('website', 'https://www.packtpub.com/'), ('email', 'info@packtpub.com')]))]) …
   ```
4. Abre `bookr/reviews/urls.py`, elimina la ruta de ejemplo anterior para `first_api_view` y agrega la ruta `all_books`, como se muestra en el siguiente código:
   `bookr/reviews/urls.py`:
   ```python
   from django.urls import path
   from . import views, api_views

   urlpatterns = [
       path(
           'api/all_books/',
           api_views.all_books,
           name='all_books'
       ),
       …
   ]
   ```
   Esta ruta recién agregada llama a la función de vista `all_books` cuando encuentra la ruta `api/all_books/` en la URL.
5. Una vez agregado todo el código, ejecuta el servidor Django con el comando `python manage.py runserver` y navega a `http://127.0.0.1:8000/api/all_books/`. Deberías ver algo similar a la Figura 12.2:
   *Figura 12.2: Lista de libros mostrada en el punto de conexión all_books*

La captura de pantalla anterior muestra que se devuelve una lista de todos los libros al llamar al punto de conexión `/api/all_books`. Con eso, has utilizado con éxito un serializador para devolver datos de manera eficiente en tu base de datos con la ayuda de una API REST.

Hasta ahora, nos hemos centrado en las vistas funcionales. Sin embargo, ahora aprenderás que las vistas basadas en clases se usan más comúnmente en DRF y te harán la vida mucho más fácil.

#### Vistas de API basadas en clases y vistas genéricas

De manera similar a lo que aprendimos en el Capítulo 11 (*Plantillas Avanzadas y Vistas Basadas en Clases*), también podemos escribir vistas basadas en clases para APIs REST. Las vistas basadas en clases son la forma preferida de escribir vistas entre los desarrolladores, ya que se puede lograr mucho escribiendo muy poco código.

Al igual que con las vistas convencionales, DRF ofrece un conjunto de vistas genéricas que hacen que escribir vistas basadas en clases sea aún más simple. Las vistas genéricas están diseñadas considerando algunas de las operaciones más comunes necesarias al crear APIs. Algunas de las vistas genéricas que ofrece DRF son `ListAPIView`, `RetrieveAPIView`, etc. En el Ejercicio 12.02, nuestra vista funcional era responsable de crear un `QuerySet` de los objetos y luego llamar al serializador. De manera equivalente, podríamos usar `ListAPIView` para hacer lo mismo:

```python
class AllBooks(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

Aquí, se define un `QuerySet` de objetos como un atributo de clase. Pasar `queryset` al serializador es manejado por métodos en `ListAPIView`.

#### Serializadores de modelos

En el Ejercicio 12.02, nuestro serializador se definió de la siguiente manera:

```python
class BookSerializer(serializers.Serializer):
    title = serializers.CharField()
    publication_date = serializers.DateField()
    isbn = serializers.CharField()
    publisher = PublisherSerializer()
```

Sin embargo, nuestro modelo para `Book` se ve así (observa qué tan similares parecen ser las definiciones del modelo y del serializador):

```python
class Book(models.Model):
    """A published book."""
    title = models.CharField(
        max_length=70,
        help_text="The title of the book.")
    publication_date = models.DateField(
        verbose_name="Date the book was published.")
    isbn = models.CharField(
        max_length=20,
        verbose_name="ISBN number of the book.")
    …
    def __str__(self):
        return f"{self.title} ({self.isbn})"
```

Preferiríamos no especificar que el título debe ser `serializers.CharField()`. Sería más fácil si el serializador simplemente mirara cómo se definió el título en el modelo y pudiera determinar qué campo de serializador usar.

Aquí es donde entran los serializadores de modelos (*model serializers*). Proporcionan atajos para crear serializadores utilizando la definición de los campos en el modelo. En lugar de especificar que el título debe serializarse mediante `CharField`, simplemente le decimos al serializador de modelo que queremos incluir `title`, y utiliza el serializador `CharField` porque el campo `title` en el modelo también es `CharField`.

Por ejemplo, supongamos que quisiéramos crear un serializador para el modelo `Contributor` en `models.py`. En lugar de especificar los tipos de serializadores que deben usarse para cada campo, podemos darle una lista de nombres de campos y dejar que descubra el resto:

```python
from rest_framework import serializers
from .models import Contributor

class ContributorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Contributor
        fields = ['first_names', 'last_names', 'email']
```

En el siguiente ejercicio, veremos cómo podemos usar un serializador de modelo para evitar la duplicación de código en las clases anteriores.

#### Ejercicio 12.03 – Creación de vistas de API basadas en clases y serializadores de modelos

En este ejercicio, crearás vistas basadas en clases para mostrar una lista de todos los libros mientras utilizas serializadores de modelos:

1. Abre el archivo `bookr/reviews/serializers.py`, elimina cualquier código preexistente y reemplázalo con el siguiente código:
   `bookr/reviews/serializers.py`:
   ```python
   from rest_framework import serializers
   from .models import Book, Publisher

   class PublisherSerializer(
       serializers.ModelSerializer):
       class Meta:
           model = Publisher
           fields = ['name', 'website', 'email']

   class BookSerializer(serializers.ModelSerializer):
       publisher = PublisherSerializer()

       class Meta:
           model = Book
           fields = ['title', 'publication_date', 'isbn', 'publisher']
   ```
   Aquí, hemos incluido dos clases de serializadores de modelos: `PublisherSerializer` y `BookSerializer`. Ambas clases heredan la clase principal `serializers.ModelSerializer`. No necesitamos especificar cómo se serializa cada campo; en su lugar, simplemente podemos pasar una lista de nombres de campos, y los tipos de campo se infieren de la definición en `models.py`.
   Aunque mencionar el campo dentro de `fields` es suficiente para el serializador de modelo, en ciertos casos especiales, como este, es posible que tengamos que personalizar el campo ya que el campo `publisher` es una clave foránea. Por lo tanto, debemos usar `PublisherSerializer` para serializar el campo `publisher`.
2. A continuación, abre `bookr/reviews/api_views.py`, elimina cualquier código preexistente y agrega el siguiente código:
   `bookr/reviews/api_views.py`:
   ```python
   from rest_framework import generics
   from .models import Book
   from .serializers import BookSerializer

   class AllBooks(generics.ListAPIView):
       queryset = Book.objects.all()
       serializer_class = BookSerializer
   ```
   Aquí, usamos la vista `ListAPIView` basada en clases de DRF en lugar de una vista funcional. Esto significa que la lista de libros se define como un atributo de clase y no tenemos que escribir una función que maneje directamente la solicitud y llame al serializador. El serializador de libros del paso anterior también se importa y se asigna como un atributo de esta clase.
3. Abre el archivo `bookr/reviews/urls.py` y modifica la ruta de API `/api/all_books` para incluir la nueva vista basada en clases de la siguiente manera:
   `bookr/reviews/urls.py`:
   ```python
   urlpatterns = [
       path(
           'api/all_books/',
           api_views.AllBooks.as_view(),
           name='all_books'
       ),
       …
   ]
   ```
   Como estamos usando una vista basada en clases, tenemos que usar el nombre de la clase junto con el método `as_view()`.
4. Una vez completadas todas las modificaciones anteriores, espera hasta que se reinicie el servicio de Django o inicia el servidor con el comando `python manage.py runserver`, y luego abre la API en `http://127.0.0.1:8000/api/all_books/` en el navegador web. Deberías ver algo como la Figura 12.3:
   *Figura 12.3: Lista de libros mostrada en el punto de conexión all_books*

Similar a lo que vimos en el Ejercicio 12.02, esta es una lista de todos los libros presentes en la aplicación de reseñas de libros. En este ejercicio, utilizamos serializadores de modelos para simplificar nuestro código y la vista genérica basada en clases `ListAPIView` para devolver una lista de los libros en nuestra base de datos. En la siguiente sección, implementarás un punto de conexión de API para devolver una lista de los principales colaboradores.

#### Actividad 12.01 – Creación de un punto de conexión de API para una página de principales colaboradores

Imagina que tu equipo decide crear una página web que muestre los principales colaboradores (autores, coautores y editores) en tu base de datos. Deciden contratar los servicios de un desarrollador externo para crear una aplicación en ReactJS. Para integrarse con el backend de Django, el desarrollador necesitará un punto de conexión que proporcione lo siguiente:
- Una lista de todos los colaboradores en la base de datos.
- Para cada colaborador, una lista de todos los libros a los que contribuyó.
- Para cada colaborador, la cantidad de libros a los que contribuyó.
- Para cada libro al que contribuyó, su rol en el libro.

La vista final de la API debería verse así:

*Figura 12.4: El punto de conexión de los principales colaboradores*

Para realizar esta tarea, ejecuta los siguientes pasos:
1. Agrega un método a la clase `Contributor` que devuelva el número de contribuciones realizadas.
2. Agrega `ContributionSerializer`, que serializa el modelo `BookContribution`.
3. Agrega `ContributorSerializer`, que serializa el modelo `Contributor`.
4. Agrega `ContributorView`, que utiliza `ContributorSerializer`.
5. Agrega un patrón a `urls.py` para permitir el acceso a `ContributorView`.

---

### Sección: Simplificación del código mediante ViewSets

Hemos visto cómo podemos optimizar nuestro código y hacerlo más conciso utilizando vistas genéricas basadas en clases. Los `ViewSets` y los enrutadores (*routers*) nos ayudan a simplificar aún más nuestro código. Como indica el nombre, los `ViewSets` son un conjunto de vistas representadas en una sola clase. Por ejemplo, usamos la vista `AllBooks` para devolver una lista de todos los libros en la aplicación y la vista `BookDetail` para devolver los detalles de un solo libro. Usando `ViewSets`, podríamos combinar ambas clases en una sola clase.

DRF también proporciona una clase llamada `ModelViewSet`. Esta clase no solo combina las dos vistas mencionadas en la discusión anterior (lista y detalle), sino que también te permite crear, actualizar y eliminar instancias de modelos. El código necesario para implementar toda esta funcionalidad podría ser tan simple como especificar el serializador y el `QuerySet`. Por ejemplo, una vista que te permita administrar todas estas acciones para tu modelo de usuario podría definirse de manera tan concisa como la siguiente:

```python
class UserViewSet(viewsets.ModelViewSet):
    serializer_class = UserSerializer
    queryset = User.objects.all()
```

Por último, DRF proporciona una clase `ReadOnlyModelViewSet`. Esta es una versión más simple de la clase `ModelViewSet` anterior. Es idéntica, excepto que solo te permite listar y recuperar usuarios específicos. No puedes crear, actualizar ni eliminar registros. En la siguiente sección, aprenderemos sobre los enrutadores.

#### Configuración de URLs mediante enrutadores y ViewSets

Los enrutadores (*routers*), cuando se usan junto con un `ViewSet`, crean automáticamente los puntos de conexión de URL requeridos para el `ViewSet`. Esto se debe a que se accede a un único `ViewSet` en diferentes URLs. Por ejemplo, en la clase `UserViewSet` anterior, accederías a una lista de usuarios en la URL `/api/users/`, y a un registro de usuario específico en la URL `/api/users/123`, donde 123 es la clave primaria de ese registro de usuario. He aquí un ejemplo simple de cómo podrías usar un enrutador en el contexto de la clase `UserViewSet` definida previamente:

```python
from rest_framework import routers

router = routers.SimpleRouter()
router.register(r'users', UserViewSet)
urlpatterns = router.urls
```

Ahora, intentemos combinar los conceptos de enrutadores y `ViewSets` en un ejercicio simple.

#### Ejercicio 12.04 – Uso de ViewSets y enrutadores

En este ejercicio, combinaremos las vistas existentes para crear un `ViewSet` y crearemos el enrutamiento requerido para el `ViewSet`:

1. Abre el archivo `bookr/reviews/serializers.py`, elimina el código preexistente y agrega el siguiente fragmento de código:
   `bookr/reviews/serializers.py`:
   ```python
   from django.contrib.auth.models import User
   from django.utils import timezone
   from rest_framework import serializers
   from rest_framework.exceptions import NotAuthenticated, PermissionDenied
   from .models import Book, Publisher, Review
   from .utils import average_rating

   class PublisherSerializer(serializers.ModelSerializer):
   ```
   Puedes encontrar el fragmento de código completo en la carpeta `Chapter12` en el repositorio de GitHub de este libro.
   Aquí, agregamos dos campos nuevos a `BookSerializer`, a saber, `reviews` y `rating`. Lo interesante de estos campos es que la lógica detrás de ellos se define como un método en el propio serializador. Es por eso que usamos el tipo `serializers.SerializerMethodField` para establecer los atributos de la clase serializadora.
2. Abre el archivo `bookr/reviews/api_views.py`, elimina el código preexistente y agrega lo siguiente:
   `bookr/reviews/api_views.py`:
   ```python
   from rest_framework import viewsets
   from rest_framework.pagination import (LimitOffsetPagination)
   from .models import Book, Review
   from .serializers import (BookSerializer, ReviewSerializer)

   class BookViewSet(viewsets.ReadOnlyModelViewSet):
       queryset = Book.objects.all()
       serializer_class = BookSerializer

   class ReviewViewSet(viewsets.ModelViewSet):
       queryset = Review.objects.order_by(
           '-date_created')
       serializer_class = ReviewSerializer
       pagination_class = LimitOffsetPagination
       authentication_classes = []
   ```
   Aquí, hemos eliminado las vistas `AllBook` y `BookDetail` y las hemos reemplazado con `BookViewSet` y `ReviewViewSet`. En la primera línea, importamos el módulo `viewsets` de `rest_framework`. La clase `BookViewSet` es una subclase de `ReadOnlyModelViewSet`, lo que garantiza que las vistas solo se utilicen para la operación GET.
3. A continuación, abre el archivo `bookr/reviews/urls.py`, elimina los dos primeros patrones de URL que comienzan con `api/` y luego agrega el siguiente código:
   `bookr/reviews/urls.py`:
   ```python
   from django.urls import path, include
   from rest_framework.routers import DefaultRouter
   from . import views, api_views

   router = DefaultRouter()
   router.register(r'books', api_views.BookViewSet)
   router.register(r'reviews', api_views.ReviewViewSet)

   urlpatterns = [
       path(
           'api/',
           include((router.urls, 'api'))
       ),
       path(
           'books/',
           views.book_list,
           name='book_list'),
       …
       path(
           'publishers/new/',
           views.publisher_edit,
           name='publisher_create'
       )
   ]
   ```
   Aquí, hemos combinado las rutas `all_books` y `book_detail` en una sola ruta llamada `books`. También hemos agregado un nuevo punto de conexión bajo la ruta `reviews`, que necesitaremos en un capítulo posterior.
   Comenzamos importando la clase `DefaultRouter` de `rest_framework.routers`. Luego, creamos un objeto enrutador usando la clase `DefaultRouter` y luego registramos las clases `BookViewSet` y `ReviewViewSet` recién creadas. Esto asegura que `BookViewSet` se invoque siempre que la API tenga la ruta `/api/books`.
4. Guarda todos los archivos y, una vez que se reinicie el servicio de Django (o lo inicies manualmente con el comando `python manage.py runserver`), ve a la URL `http://127.0.0.1:8000/api/books/` para obtener una lista de todos los libros. Deberías ver la siguiente vista en el explorador de la API:
   *Figura 12.5: Lista de libros en la ruta /api/books*
5. También puedes acceder a los detalles de un libro específico utilizando la URL `http://127.0.0.1:8000/api/books/1/`. En este caso, devolverá detalles para el libro con una clave primaria de 1 (si existe en tu base de datos):
   *Figura 12.6: Detalles del libro para "Advanced Deep Learning with Keras"*

En este ejercicio, vimos cómo podemos usar `ViewSets` y enrutadores para combinar vistas de lista y de detalle en un solo `ViewSet`. El uso de `ViewSets` hace que nuestro código sea más consistente e idiomático, lo que facilita la colaboración con otros desarrolladores. Esto se vuelve particularmente importante cuando se integra con una aplicación frontend separada. En la siguiente sección, aprenderemos brevemente sobre los diferentes tipos de autenticación e implementaremos la autenticación basada en tokens para la aplicación de reseñas de libros.

---

### Sección: Implementación de la autenticación

Como aprendimos en el Capítulo 9 (*Sesiones y Autenticación*), es importante autenticar a los usuarios de nuestra aplicación. Es una buena práctica permitir que solo aquellos usuarios que se hayan registrado en la aplicación inicien sesión y accedan a la información de la aplicación. Del mismo modo, para las APIs REST, también necesitamos diseñar una forma de autenticar y autorizar a los usuarios antes de que se transmita cualquier información. Por ejemplo, supongamos que el sitio web de Facebook realiza una solicitud de API para obtener una lista de todos los comentarios de una publicación. Si no tuvieran autenticación en este punto de conexión, podrías usarlo para obtener comentarios de cualquier publicación que desees mediante programación. Obviamente no quieren permitir esto, por lo que se debe implementar algún tipo de autenticación.

Existen diferentes esquemas de autenticación, como autenticación básica (*basic authentication*), autenticación de sesión (*session authentication*), autenticación de token (*token authentication*), autenticación de usuario remoto (*remote user authentication*) y varias soluciones de autenticación de terceros. Para el alcance de este capítulo y para nuestra aplicación Bookr, utilizaremos la autenticación por token.

Para leer más sobre todos los esquemas de autenticación, consulta la documentación oficial en [https://www.django-rest-framework.org/api-guide/authentication](https://www.django-rest-framework.org/api-guide/authentication).

#### Autenticación basada en tokens

La autenticación basada en tokens funciona generando un token único para un usuario a cambio de su nombre de usuario y contraseña. Una vez que se genera el token, se almacenará en la base de datos para referencia futura y se devolverá al usuario en cada inicio de sesión.

Este token es exclusivo del usuario y luego puede usarlo para autorizar cada solicitud de API que realice. La autenticación basada en tokens elimina la necesidad de pasar el nombre de usuario y la contraseña en cada solicitud. Es mucho más segura y se adapta mejor a la comunicación cliente-servidor, como un cliente web basado en JavaScript que interactúa con la aplicación backend a través de APIs REST.

Un ejemplo de esto sería una aplicación ReactJS o AngularJS que interactúa con un backend de Django a través de APIs REST.

La misma arquitectura se puede utilizar si estás desarrollando una aplicación móvil para interactuar con el servidor backend a través de APIs REST; por ejemplo, una aplicación de Android o iOS que interactúa con un backend de Django a través de APIs REST.

#### Ejercicio 12.05 – Implementación de autenticación basada en tokens para las APIs de Bookr

En este ejercicio, implementarás la autenticación basada en tokens para las APIs de la aplicación Bookr:

1. Abre el archivo `bookr/settings.py` y agrega `rest_framework.authtoken` a `INSTALLED_APPS`:
   `bookr/settings.py`:
   ```python
   INSTALLED_APPS = [
       …
       'rest_framework',
       'rest_framework.authtoken',
       'reviews'
   ]
   ```
2. Dado que la aplicación `authtoken` tiene cambios asociados en la base de datos, ejecuta el comando `migrate` en la línea de comandos/terminal de la siguiente manera:
   ```bash
   python manage.py migrate
   ```
3. Abre el archivo `bookr/reviews/api_views.py`, elimina cualquier código preexistente y reemplázalo con lo siguiente:
   `bookr/reviews/api_views.py`:
   ```python
   from django.contrib.auth import authenticate
   from rest_framework import viewsets
   from rest_framework.authentication import (
       TokenAuthentication)
   from rest_framework.authtoken.models import Token
   from rest_framework.pagination import (
       LimitOffsetPagination)
   from rest_framework.permissions import IsAuthenticated
   from rest_framework.response import Response
   from rest_framework.status import (
       HTTP_404_NOT_FOUND, HTTP_200_OK)
   from rest_framework.views import APIView
   ```
   Puedes encontrar el fragmento de código completo en la carpeta `Chapter12` en el repositorio de GitHub de este libro.
   Aquí, hemos definido una vista llamada `Login`. El propósito de esta vista es permitir que un usuario obtenga (o cree si aún no existe) un token que pueda usar para autenticarse con la API.
   Anulamos el método `post` de esta vista porque queremos personalizar el comportamiento cuando un usuario nos envía datos (es decir, sus datos de inicio de sesión). Primero, usamos el método `authenticate` de la biblioteca `auth` de Django para verificar si el nombre de usuario y la contraseña son correctos. Si son correctos, tendremos un objeto `user`. Si no, devolvemos un error HTTP 404. Si tenemos un objeto `user` válido, simplemente obtenemos o creamos un token y se lo devolvemos al usuario.
4. A continuación, agreguemos una clase de autenticación a `BookViewSet`. Esto significa que cuando un usuario intente acceder a este `ViewSet`, requerirá que se autentique mediante la autenticación basada en tokens. Ten en cuenta que es posible incluir una lista de diferentes métodos de autenticación aceptados, no solo uno. También agregamos el atributo `permission_classes`, que simplemente usa la clase incorporada de DRF que verifica si el usuario dado tiene permiso para ver los datos en este modelo:
   ```python
   class BookViewSet(viewsets.ReadOnlyModelViewSet):
       queryset = Book.objects.all()
       serializer_class = BookSerializer
       authentication_classes = [TokenAuthentication]
       permission_classes = [IsAuthenticated]
   ```
5. Abre el archivo `bookr/reviews/urls.py` y agrega la siguiente ruta en los patrones de URL:
   `bookr/reviews/urls.py`:
   ```python
   path(
       'api/login',
       api_views.Login.as_view(),
       name='login'
   )
   ```
6. Guarda el archivo y espera a que la aplicación se reinicie, o inicia el servidor manualmente con el comando `python manage.py runserver`. Luego, accede a la aplicación mediante la URL `http://127.0.0.1:8000/api/login`. Tu pantalla debería aparecer de la siguiente manera:
   *Figura 12.7: Página de inicio de sesión*
   La API en `/api/login` es un mensaje de solo POST; por lo tanto, se muestra *Method GET not allowed*.
7. A continuación, ingresa el siguiente fragmento en el campo Content y haz clic en POST:
   ```json
   {
       "username": "Peter",
       "password": "testuserpassword"
   }
   ```
   Deberás reemplazar esto con un nombre de usuario y contraseña reales para tu cuenta en la base de datos. Ahora, puedes ver el token generado para el usuario. Este es el token que debemos usar para acceder a `BookSerializer`:
   *Figura 12.8: El token generado para el usuario*
8. Intenta acceder a la lista de libros mediante la API que creamos anteriormente en `http://127.0.0.1:8000/api/books/`. Ten en cuenta que ahora no tienes permiso para acceder a ella. Esto se debe a que este `ViewSet` ahora requiere que uses tu token para autenticarte. Se puede acceder a la misma API usando `curl` en la línea de comandos:
   ```bash
   curl -X GET http://127.0.0.1:8000/api/books/
   ```
   Salida:
   ```json
   {"detail":"Authentication credentials were not provided."}
   ```
   Dado que no se proporcionó el token, se muestra el mensaje *Authentication credentials were not provided*:
   *Figura 12.9: Mensaje que indica que no se proporcionaron los detalles de autenticación*
   Ten en cuenta que si estás usando Windows 10, reemplaza `curl` en el comando anterior con `curl.exe` y ejecútalo desde el símbolo del sistema.
9. Para pasar el token de autorización (obtenido en el paso 7) como un encabezado, puedes usar el siguiente comando (los usuarios de Windows pueden reemplazar `curl` con `curl.exe`):
   ```bash
   curl -X GET http://127.0.0.1:8000/api/books/ -H "Authorization: Token cd5dafa1d4361fd1502652d266eed3dcdb55c64f1"
   ```
   Antes de pegar este comando, asegúrate de haber reemplazado el token con el que obtuviste cuando ejecutaste el paso 7 de este ejercicio. Será diferente al que hemos mostrado aquí.
   El comando anterior ahora debería devolver una lista de libros:
   ```json
   [{"title":"Advanced Deep Learning with Keras","publication_date":"2018-10-31","isbn":"9781788629416","publisher":{"name":"Packt Publishing","website":"https://www.packtpub.com/","email":"info@packtpub.com"},"rating":4,"reviews":[{"content":"A must read for all","date_created":… (truncated)
   ```
   Esta operación aseguró que solo un usuario existente de la aplicación pudiera acceder y obtener la colección de todos los libros.
10. Antes de continuar, establece las clases de autenticación y permisos en `BookViewSet` en una lista vacía. Los capítulos futuros no utilizarán estos métodos de autenticación y asumiremos, en aras de la simplicidad, que un usuario no autenticado puede acceder a nuestra API:
    ```python
    class BookViewSet(viewsets.ReadOnlyModelViewSet):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
        authentication_classes = []
        permission_classes = []
    ```

En este ejercicio, implementamos la autenticación basada en tokens en nuestra aplicación Bookr. Creamos una vista de inicio de sesión que nos permite recuperar el token para un usuario autenticado determinado. Esto nos permitió realizar solicitudes de API desde la línea de comandos pasando el token como un encabezado en la solicitud.

En general, en esta sección, aprendimos sobre los diferentes tipos de autenticación y luego aprendimos en detalle sobre la autenticación y autorización de tokens mientras trabajamos con APIs REST.

---

### Sección: Resumen

Este capítulo presentó las APIs REST, un componente fundamental en la mayoría de las aplicaciones web del mundo real. Estas APIs facilitan la comunicación entre el servidor backend y el navegador web, por lo que son fundamentales para tu crecimiento como desarrollador web de Django. Aprendimos a serializar datos en nuestra base de datos para que puedan transmitirse a través de una solicitud HTTP. A continuación, aprendimos sobre las diversas opciones que nos brinda DRF para simplificar el código que escribimos, aprovechando las definiciones existentes de los propios modelos. También cubrimos `ViewSets` y enrutadores y vimos cómo se pueden usar para condensar aún más el código al combinar la funcionalidad de múltiples vistas. Finalmente, aprendimos sobre autenticación y autorización e implementamos la autenticación basada en tokens para la aplicación de reseñas de libros.

En el próximo capítulo, ampliaremos la funcionalidad de Bookr para sus usuarios aprendiendo a escribir pruebas unitarias exhaustivas para garantizar la calidad y robustez de nuestro código.
