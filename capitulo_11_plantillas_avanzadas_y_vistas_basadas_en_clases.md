# Parte 3: Características avanzadas de Django

## Capítulo 11: Plantillas avanzadas y vistas basadas en clases

En el Capítulo 3 (*Mapeo de URLs, Vistas y Plantillas*), aprendimos cómo crear vistas y plantillas en Django. Luego, aprendimos a usar esas vistas para renderizar las plantillas que creamos. En este capítulo, ampliaremos nuestro conocimiento sobre el desarrollo de vistas mediante el uso de vistas basadas en clases (*class-based views* o CBVs), lo que nos permitirá escribir vistas que pueden agrupar métodos lógicos en una sola entidad. Esta habilidad resulta útil cuando se desarrolla una vista que se asigna a múltiples métodos de solicitud HTTP para el mismo punto de conexión de la interfaz de programación de aplicaciones (API). Con las vistas basadas en métodos o funciones, podemos terminar usando muchas condiciones `if-else` para manejar con éxito los diferentes tipos de métodos de solicitud HTTP. Por el contrario, las vistas basadas en clases nos permiten definir métodos separados para cada método de solicitud HTTP que queramos manejar. Luego, según el tipo de solicitud recibida, Django se encarga de llamar al método correcto en la vista basada en clases.

Más allá de la capacidad de crear vistas basadas en diferentes técnicas de desarrollo, Django también incluye un potente motor de plantillas. Este motor permite a los desarrolladores crear plantillas reutilizables para sus aplicaciones web. Esta reutilización del motor de plantillas se mejora aún más mediante el uso de etiquetas de plantilla (*template tags*), filtros (*template filters*) y parciales de plantilla (*template partials*), que ayudan a implementar fácilmente funciones de uso común dentro de las plantillas, como iterar sobre listas de datos, formatear datos con un estilo determinado, extraer un fragmento de texto de una variable para mostrarlo y anular el contenido en un bloque específico de una plantilla. Todas estas características también amplían la reutilización de una plantilla de Django.

A lo largo de este capítulo, veremos cómo podemos expandir el conjunto predeterminado de filtros y etiquetas de plantilla proporcionados por Django aprovechando la capacidad de Django para definir nuestras propias etiquetas de plantilla personalizadas, filtros y parciales de plantilla. Estas etiquetas, filtros y parciales personalizados se pueden usar luego para implementar algunas características comunes de manera reutilizable en toda nuestra aplicación web. Por ejemplo, al crear una insignia de perfil de usuario que se puede mostrar en varios lugares dentro de una aplicación web, es mejor aprovechar la capacidad de escribir una etiqueta de inclusión de plantilla personalizada que simplemente inserte la plantilla de la insignia en cualquiera de las vistas que deseemos, en lugar de reescribir todo el código para la plantilla de la insignia o introducir complejidad adicional en las plantillas.

En este capítulo, cubriremos los siguientes temas:
- Filtros de plantillas
- Filtros de plantillas personalizados
- Filtros de cadenas
- Etiquetas de plantillas
- Parciales de plantillas
- Vistas de Django
- Vistas basadas en clases

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter11` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Filtros de plantillas

Al desarrollar plantillas, los desarrolladores a menudo solo quieren cambiar el valor de una variable de plantilla antes de mostrársela al usuario. Por ejemplo, considera que estamos creando una página de perfil para un usuario de Bookr. Allí, queremos mostrar la cantidad de libros que el usuario ha leído. Debajo de eso, también queremos mostrar una tabla con la lista de los libros que ha leído.

Para lograr esto, podemos pasar dos variables separadas de nuestra vista a la plantilla HTML. Una puede llamarse `books_read`, que denota la cantidad de libros leídos por el usuario. La otra puede ser `book_list`, que contiene la lista de nombres de los libros leídos por el usuario, por ejemplo:

```html
<span class="books_read">You have read {{ books_read }} books</span>
<ul>
    {% for book in book_list %}
        <li>{{ book }} </li>
    {% endfor %}
</ul>
```

Los filtros de plantilla en Django son funciones simples basadas en Python que aceptan una variable como argumento (y cualquier dato adicional en el contexto de la variable), cambian su valor según nuestros requisitos y luego renderizan el valor modificado.

Ahora, el mismo resultado de escribir el fragmento anterior también se puede obtener sin el uso de dos variables separadas mediante el uso de filtros de plantilla en Django, de la siguiente manera:

```html
<span class="books_read">You have read {{ book_list|length }}</span>
<ul>
    {% for book in book_list %}
        <li>{{ book }}</li>
    {% endfor %}
</ul>
```

Aquí, usamos el filtro integrado `length` proporcionado por Django. El uso de este filtro hace que se evalúe y devuelva la longitud de la variable `book_list`, que luego se inserta en nuestra plantilla HTML durante el renderizado.

Al igual que `length`, hay muchos otros filtros de plantilla que vienen preempaquetados con Django y que están listos para usarse. Por ejemplo, el filtro `lower` convierte el texto a minúsculas, el filtro `last` se puede usar para devolver el último elemento de la lista y el filtro `json_script` se puede usar para generar un objeto de Python pasado a la plantilla como un valor JSON envuelto en una etiqueta `<script>` en tu plantilla.

Puedes consultar la documentación oficial de Django para obtener la lista completa de filtros de plantilla que ofrece Django aquí: [https://docs.djangoproject.com/en/5.2/ref/templates/builtins/](https://docs.djangoproject.com/en/5.2/ref/templates/builtins/).

Ahora, con el conocimiento de cómo usar filtros de plantilla, pasemos a comprender cómo podemos escribir nuestros propios filtros personalizados.

---

### Sección: Filtros de plantillas personalizados

Django proporciona muchos filtros útiles que podemos usar en nuestras plantillas mientras trabajamos en nuestros proyectos. Pero, ¿qué pasa si alguien quiere formatear un fragmento específico de texto y renderizarlo con diferentes fuentes? O, digamos, alguien quiere traducir un código de error a un mensaje de error fácil de usar basado en el mapeo del código de error en el backend. En estos casos, los filtros predefinidos no son suficientes y nos gustaría escribir nuestro propio filtro que podamos reutilizar en todo el proyecto.

Afortunadamente, Django proporciona una API fácil de usar que podemos utilizar para escribir filtros personalizados. Esta API proporciona a los desarrolladores algunas funciones de decorador útiles que se pueden usar para registrar rápidamente una función de Python como un filtro de plantilla personalizado. Una vez que una función de Python se registra como un filtro personalizado, un desarrollador puede comenzar a usar la función en plantillas.

Se requiere una instancia de este método de biblioteca de plantillas para acceder a los filtros. Esta instancia se puede crear instanciando la clase `Library()` en Django desde el módulo `template` de Django, como se muestra aquí:

```python
from django import template
register = template.Library()
```

Una vez creada la instancia, podemos usar el decorador `filter` de la instancia de la biblioteca de plantillas para registrar nuestros filtros.

#### Creación de filtros de plantillas personalizados

Hay un par de pasos que debemos seguir para crear filtros de plantilla personalizados. Intentemos comprender cuáles son estos pasos y cómo nos ayudan con la creación de un filtro de plantilla personalizado en la siguiente subsección.

##### Configuración del directorio para almacenar filtros de plantillas
Es importante tener en cuenta que al crear un filtro de plantilla personalizado o una etiqueta de plantilla, debemos colocarlos en el directorio `templatetags` debajo del directorio de la aplicación. Este requisito surge porque Django está configurado internamente para buscar etiquetas y filtros de plantilla personalizados al cargar una aplicación web. Si no nombras el directorio `templatetags`, Django no cargará los filtros y etiquetas de plantilla personalizados que creamos.

Para crear este directorio, primero navega a la carpeta de la aplicación dentro de la cual deseas crear filtros de plantilla personalizados y luego ejecuta el siguiente comando en la terminal:

```bash
mkdir templatetags
```

Una vez creado el directorio, el siguiente paso es crear un nuevo archivo dentro del directorio `templatetags` para almacenar el código de nuestros filtros personalizados. Esto se puede hacer ejecutando el siguiente comando dentro del directorio `templatetags`:

```bash
touch custom_filter.py
```

El comando antes mencionado no funcionará en Windows. Sin embargo, puedes navegar al directorio deseado y crear un nuevo archivo usando el Explorador de Windows o la interfaz de PyCharm.

##### Configuración de la biblioteca de plantillas
Una vez creado el archivo para almacenar el código del filtro personalizado, podemos comenzar a trabajar en la implementación de nuestro código de filtro personalizado. Para que los filtros personalizados funcionen en Django, deben registrarse en la biblioteca de plantillas de Django antes de que puedan usarse dentro de las plantillas. Para ello, el primer paso es configurar una instancia de la biblioteca de plantillas, que se utilizará para registrar nuestros filtros personalizados. Para esto, dentro del archivo `custom_filters.py` que creamos en la sección anterior, primero debemos importar el módulo `template` del proyecto Django:

```python
from django import template
```

Una vez resuelta la importación, el siguiente paso es crear una instancia de la biblioteca de plantillas agregando la siguiente línea de código:

```python
register = template.Library()
```

La clase `Library` del módulo `template` de Django se implementa como una clase singleton que devuelve el mismo objeto que solo se inicializa una vez al inicio de la aplicación.

Una vez configurada la instancia de la biblioteca de plantillas, podemos proceder a implementar nuestro filtro personalizado.

##### Implementación de la función de filtro personalizado
Los filtros personalizados dentro de Django no son más que funciones simples de Python que esencialmente toman los siguientes parámetros:
- El valor sobre el cual se aplica el filtro (obligatorio).
- Cualquier parámetro adicional (cero o más) que deba pasarse al filtro (opcional).

Estas funciones deben estar decoradas con el atributo `filter` de la instancia de la biblioteca de plantillas de Django para comportarse como filtros de plantilla. Por ejemplo, la implementación genérica de un filtro personalizado se verá así:

```python
@register.filter
def my_filter(value, arg):
    # Implementation logic of the filter
```

Con esto, hemos aprendido a implementar un filtro personalizado que se puede utilizar dentro de las plantillas. En la siguiente sección, aprenderemos a usar estos filtros personalizados.

##### Uso de filtros personalizados dentro de las plantillas
Una vez creado el filtro, es sencillo empezar a usarlo dentro de nuestras plantillas. Para hacer eso, el filtro primero debe importarse a la plantilla. Esto se puede hacer fácilmente agregando la siguiente línea en la parte superior del archivo de plantilla:

```django
{% load custom_filter %}
```

Cuando el motor de plantillas de Django analiza los archivos de plantilla, Django resuelve automáticamente la línea anterior para encontrar el módulo correcto especificado en el directorio `templatetags`. Como consecuencia, todos los filtros mencionados dentro del módulo `custom_filter` están disponibles automáticamente dentro de la plantilla.

Usar nuestro filtro personalizado dentro de la plantilla es tan simple como agregar la siguiente línea:

```django
{{ some_value|generic_filter:"arg" }}
```

Es un proceso simple crear un filtro personalizado dentro de Django y luego usarlo en nuestras plantillas. En la siguiente sección, echemos un vistazo a otro tipo de filtro, a saber, los filtros de cadenas (*string filters*), que funcionan únicamente con valores de tipo cadena.

---

### Sección: Filtros de cadenas

En el caso general, al crear un filtro personalizado, este puede tomar cualquier tipo de argumento variable y devolver cualquier tipo. Por ejemplo, el filtro `length` toma un iterable y devuelve un valor entero. Pero, ¿qué pasa si quisiéramos restringir nuestro filtro para que funcione solo con cadenas y no con ningún otro tipo de valores, como números enteros?

Podemos usar el decorador `stringfilter` proporcionado por la biblioteca de plantillas de Django para desarrollar filtros que funcionen solo en cadenas. Cuando se utiliza el decorador `stringfilter` para registrar un método de Python como un filtro en Django, el framework se asegura de que el valor que se pasa al filtro se convierta en una cadena antes de que se ejecute el filtro. Esto reduce cualquier problema potencial que pueda surgir cuando se pasan valores que no son de cadena a nuestro filtro.

Los pasos para implementar un filtro de cadenas son similares a los que seguimos para crear un filtro personalizado con algunos cambios menores.

¿Recuerdas el archivo `custom_filter.py` que creamos en la sección *Configuración del directorio para almacenar filtros de plantillas*? Agreguemos ahora una nueva función de Python dentro de él que actuará como nuestro filtro de cadenas.

Antes de que podamos implementar un filtro de cadenas, primero debemos importar el decorador `stringfilter`, que delimita una función de filtro personalizada como un filtro de cadenas. Puedes agregar este decorador agregando la siguiente declaración de importación dentro del archivo `custom_filters.py`:

```python
from django.template.defaultfilters import stringfilter
```

Ahora, para implementar nuestro filtro de cadenas personalizado, se puede utilizar la siguiente sintaxis:

```python
@register.filter
@stringfilter
def generic_string_filter(value, arg):
    # Logic for string filter implementation
```

Con este enfoque, podemos crear tantos filtros de cadenas como queramos y utilizarlos como cualquier otro filtro.

Equipados con este conocimiento, creemos ahora nuestro primer filtro de cadenas personalizado.

#### Ejercicio 11.01 – Creación de un filtro de plantilla personalizado

En este ejercicio, escribiremos un filtro personalizado llamado `link_worldcat`, que devuelve una URL al sitio de WorldCat cuando se le proporciona una cadena de ISBN o título junto con un argumento explícito para indicar si la cadena es un ISBN o un título.

He aquí un ejemplo:

```python
value = "9781727518603"
```

Aplicarás el siguiente filtro a esta cadena:

```django
{{ value|link_worldcat:"isbn" }}
```

La salida después de aplicar este filtro debería ser la siguiente:

```text
"https://www.worldcat.org/isbn/9781727518603"
```

Alternativamente, si el valor fuera el título de un libro:

```python
value = "The Iliad"
```

El filtro se puede utilizar para construir una consulta de título de libro en worldcat.org de tal manera que:

```django
{{ value|link_worldcat:"title" }}
```

produciría:

```text
"https://www.worldcat.org/search?q=The+Iliad"
```

Comencemos con los pasos:

1. Ahora, crea un nuevo directorio llamado `templatetags` dentro del directorio de tu aplicación `reviews` para almacenar el código de tus filtros de plantilla personalizados. Para crear el directorio, ejecuta el siguiente comando desde dentro del directorio `filter_demo` desde la aplicación de terminal o el símbolo del sistema:
   ```bash
   mkdir templatetags
   ```
2. Una vez creado el directorio, crea un nuevo archivo llamado `link_worldcat_filter.py` dentro del directorio `reviews/templatetags`, agregándole las siguientes líneas:
   ```python
   import urllib.parse
   from django import template
   from django.template.defaultfilters import stringfilter

   register = template.Library()
   ```
   El código anterior crea una instancia de la biblioteca de Django que se puede usar para registrar nuestro filtro personalizado con Django.
3. Agrega el siguiente código para implementar el filtro `link_worldcat`:
   ```python
   @register.filter
   @stringfilter
   def link_worldcat (value, identifier):
       if not value:
           return ""
       if identifier=='title':
           link = urllib.parse.quote_plus(value)
           return f"https://www.worldcat.org/search?q={link}"
       else:
           return f"https://www.worldcat.org/isbn/{value}"
   ```
   El filtro `link_worldcat` toma dos argumentos: uno es `value`, sobre el cual se utilizó el filtro, y el segundo es `identifier`, pasado desde la plantilla al filtro. El filtro creará un enlace apropiado dependiendo de si `identifier` se especifica como `isbn` o `title`.
4. Con el filtro personalizado listo, podemos revisar la plantilla de detalle de libro existente donde se puede aplicar este filtro. Agrega la directiva `load link_worldcat_filter` al comienzo del archivo después de la directiva `extends` y antes de la declaración de bloque. Luego agrega nuevos elementos `span` que mostrarán el ISBN y usarán el filtro `link_worldcat` para proporcionar un enlace a la entrada de WorldCat del libro.
   `reviews/templates/reviews/book_detail.html`:
   ```django
   {% extends 'reviews/base.html' %}
   {% load link_worldcat_filter %}

   {% block content %}
       …
       <span class="text-info">Publication Date: </span> <span>{{ book.publication_date }}</span> <br>
       <span class="text-info">ISBN: </span> <span><a href="{{ book.isbn|link_worldcat:"isbn" }}"> {{ book.isbn }}</a></span> <br>
       …
   {% endblock %}
   ```
5. Del mismo modo, podemos agregar elementos `span` similares para el ISBN y el enlace de WorldCat en la plantilla de lista de libros:
   `reviews/templates/reviews/book_list.html`:
   ```django
   {% extends 'reviews/base.html' %}
   {% load link_worldcat_filter %}

   {% block content %}
   <ul class="list-group">
       {% for item in book_list %}
           <li class="list-group-item">
               …
               <span class="text-info">ISBN: </span> <span><a href="{{ item.book.isbn|link_worldcat:"isbn" }}"> {{ item.book.isbn }}</a></span> <br>
               …
           </li>
       {% endfor %}
   </ul>
   {% endblock %}
   ```
   Después de realizar los cambios, el código debe parecerse al del directorio del proyecto `bookr` en la carpeta `Chapter11` en el repositorio de GitHub de este libro.
6. Para ver si el filtro personalizado funciona, ejecuta el siguiente comando:
   ```bash
   python manage.py runserver localhost:8000
   ```
7. Ahora, navega a la siguiente página en el navegador: [http://localhost:8000/books/2](http://localhost:8000/books/2).
   Esta página debería aparecer como se muestra en la Figura 11.1:
   *Figura 11.1 – Página de detalles del libro con enlace ISBN utilizando el filtro link_worldcat*

---

### Sección: Etiquetas de plantillas

Las etiquetas de plantilla (*template tags*) son una poderosa característica del motor de plantillas de Django. Permiten a los desarrolladores crear plantillas potentes generando HTML mediante la evaluación de ciertas condiciones y ayudan a evitar la escritura repetitiva de código común.

Un ejemplo en el que podemos usar etiquetas de plantilla son las opciones de registro/inicio de sesión en la barra de navegación de un sitio web. En este caso, podemos usar etiquetas de plantilla para evaluar si el visitante en la página actual ha iniciado sesión. En función de eso, podemos representar un banner de perfil o un banner de registro/inicio de sesión.

Las etiquetas también son una ocurrencia común durante el desarrollo de plantillas. Por ejemplo, considera la siguiente línea de código, que usamos para importar los filtros personalizados dentro de nuestras plantillas en la sección anterior:

```django
{% load explode_filter %}
```

Esto usa una etiqueta de plantilla conocida como `load`, responsable de cargar el filtro `explode` en la plantilla. Las etiquetas de plantilla son mucho más potentes en comparación con los filtros. Mientras que los filtros tienen acceso solo a los valores sobre los que operan, las etiquetas de plantilla tienen acceso al contexto de toda la plantilla y, por lo tanto, se pueden usar para crear muchas funcionalidades complejas dentro de una plantilla.

Veamos ahora los diferentes tipos de etiquetas de plantilla admitidos por Django y cómo podemos crear nuestras propias etiquetas de plantilla personalizadas.

#### Los tipos de etiquetas de plantillas

Django admite principalmente dos tipos de etiquetas de plantilla:
- **Etiquetas simples** (*Simple tags*): Estas son las etiquetas que operan sobre los datos de las variables proporcionadas (y cualquier variable adicional a ellas) y se renderizan en la misma plantilla en la que se han invocado. Por ejemplo, un caso de uso de este tipo puede incluir renderizar un mensaje de bienvenida personalizado para el usuario según su nombre de usuario o mostrar la hora del último inicio de sesión del usuario según su nombre de usuario.
- **Etiquetas de inclusión** (*Inclusion tags*): Estas etiquetas toman las variables de datos proporcionadas y generan una salida renderizando otra plantilla. Por ejemplo, la etiqueta puede tomar una lista de objetos e iterar sobre ellos para generar una lista HTML.

En las siguientes secciones, veremos cómo podemos crear estos diferentes tipos de etiquetas y utilizarlas en nuestra aplicación.

#### Etiquetas simples

Las etiquetas simples proporcionan una forma para que los desarrolladores creen etiquetas de plantilla que toman una o más variables de la plantilla, las procesan y devuelven una respuesta. La respuesta devuelta por la etiqueta de plantilla reemplaza la definición de la etiqueta de plantilla proporcionada dentro de la plantilla HTML. Este tipo de etiquetas se pueden utilizar para crear varias funcionalidades útiles, por ejemplo, el análisis de fechas o la visualización de alertas activas, si las hay, que queramos mostrar al usuario.

Las etiquetas simples se pueden crear fácilmente usando el decorador `simple_tag` provisto por la biblioteca de plantillas, decorando el método de Python que debería actuar como una etiqueta de plantilla. Ahora, veamos cómo podemos implementar una etiqueta simple personalizada utilizando la biblioteca de plantillas de Django.

##### Creación de una etiqueta de plantilla simple
La creación de etiquetas de plantilla simples sigue las mismas convenciones que discutimos en la sección *Filtros de plantillas personalizados*, con algunas diferencias sutiles. Primero, repasemos cómo se pueden crear etiquetas de plantilla para usar en nuestras plantillas de Django.

##### Configuración del directorio
Al igual que los filtros personalizados, las etiquetas de plantilla personalizadas también deben crearse dentro del mismo directorio `templatetags` para que el motor de plantillas de Django las pueda descubrir. El directorio se puede crear directamente usando la GUI de PyCharm o ejecutando el siguiente comando dentro del directorio de la aplicación donde queremos crear nuestras etiquetas personalizadas:

```bash
mkdir templatetags
```

Una vez hecho esto, ahora podemos crear un nuevo archivo que almacenará el código para nuestras etiquetas de plantilla personalizadas mediante el siguiente comando:

```bash
touch custom_tags.py
```

El comando antes mencionado no funcionará en Windows. Sin embargo, puedes crear un nuevo archivo usando el Explorador de Windows.

##### Configuración de la biblioteca de plantillas
Una vez que la estructura del directorio esté configurada y tengamos un archivo para guardar el código de nuestras etiquetas de plantilla personalizadas, ahora podemos comenzar a crear nuestras etiquetas de plantilla. Pero antes de eso, necesitamos configurar una instancia de la biblioteca de plantillas de Django como lo hicimos anteriormente. Esto se puede hacer agregando las siguientes líneas de código a nuestro archivo `custom_tag.py`:

```python
from django import template
register = template.Library()
```

Al igual que con los filtros personalizados, la instancia de la biblioteca de plantillas se utiliza aquí para registrar las etiquetas de plantilla personalizadas para su uso dentro de las plantillas de Django.

##### Implementación de una etiqueta de plantilla simple
Las etiquetas de plantilla simples dentro de Django son funciones de Python que pueden tomar cualquier cantidad de argumentos según lo deseemos. Estas funciones de Python deben decorarse con el decorador `simple_tag` de la biblioteca de plantillas para registrar esas funciones como etiquetas de plantilla simples. El siguiente fragmento de código muestra cómo se implementa una etiqueta de plantilla simple:

```python
@register.simple_tag
def generic_simple_tag(arg1, arg2):
    # Logic to implement a generic simple tag
```

A continuación, usemos estas etiquetas simples dentro de las plantillas.

##### Uso de etiquetas simples dentro de las plantillas
Usar etiquetas simples dentro de las plantillas de Django es bastante fácil. Dentro del archivo de plantilla, primero debemos asegurarnos de tener la etiqueta importada dentro de la plantilla agregando lo siguiente en la parte superior del archivo de plantilla:

```django
{% load custom_tag %}
```

La declaración anterior cargará todas las etiquetas del archivo `custom_tag.py` que definimos anteriormente y las pondrá a disposición dentro de nuestra plantilla. Luego podemos usar nuestra etiqueta simple personalizada agregando el siguiente comando:

```django
{% custom_simple_tag "argument1" "argument2" %}
```

Ahora, pongamos en práctica este conocimiento y creemos nuestra primera etiqueta simple personalizada.

#### Ejercicio 11.02 – Creación de una etiqueta simple personalizada

Gran parte del código de plantilla existente está algo desordenado y es difícil de leer porque contiene código de control y condicional. Considera esta sección de la plantilla de resultados de búsqueda:

`reviews/templates/reviews/search-results.html`:
```django
{% for contributor in book.contributors.all %}
    {{ contributor.first_names }} {{ contributor.last_names }}
    {% if not forloop.last %}, {% endif %}
{% endfor %}
```

Tiene sentido utilizar etiquetas simples como una forma de mantener este código condicional y de control en su propio método. Ahora el código en la plantilla se puede reemplazar con dos llamadas:

```django
{% load list_items %}
{% list_items book.contributors.all 'first_names' 'last_names' %}
```

La etiqueta simple `list_items` se puede reutilizar en nuestras plantillas cuando necesitemos representar otra lista como una cadena delimitada por comas.

En este ejercicio, crearemos una etiqueta simple que toma múltiples argumentos: el primero será un iterable (como una lista de libros o colaboradores), y los argumentos siguientes serán atributos que pertenecen al elemento de la lista. La etiqueta devolverá una representación en cadena de los atributos especificados separados por comas.

1. Primero, crea un nuevo archivo llamado `simple_tag.py` en el directorio `reviews/templatetags`. Dentro de este archivo, agrega el siguiente código:
   ```python
   from django import template

   register = template.Library()

   @register.simple_tag
   def list_items(iterable, *fields):
       return ', '.join(
           [' '.join([getattr(item, field, '') for field in fields]) for item in iterable])
   ```
   Aquí, has creado un nuevo método de Python, `list_items`, que toma un argumento iterable seguido de múltiples argumentos para los campos a los que se hará referencia. Este método luego se decora con `@register.simple_tag`, lo que indica que este método es una etiqueta simple y se puede usar como una etiqueta de plantilla en las plantillas.
2. Ahora, edita la plantilla de resultados de búsqueda que usará tu etiqueta simple:
   `reviews/templates/reviews/search-results.html`:
   ```django
   {% extends 'base.html' %}
   {% load simple_tag %}

   {% block title %}
   …
   <ul class="list-group">
       {% for book in books %}
           <li class="list-group-item">
               …
               <span class="text-info">Contributors: </span>
               {% list_items book.contributors.all 'first_names' 'last_names' %}
           </li>
           …
       </ul>
   {% endif %}
   {% endblock %}
   ```
   En el fragmento de código anterior, acabamos de crear una página HTML básica que utilizará tu etiqueta simple personalizada. La semántica de cargar una etiqueta de plantilla personalizada es similar a la de cargar un filtro de plantilla personalizado y requiere el uso de una etiqueta `{% load %}` en la plantilla. El proceso buscará el módulo `simple_tag.py` en el directorio `templatetags` y, si lo encuentra, cargará las etiquetas que se hayan definido en el módulo.
   El conjunto final de cambios debe parecerse a los de la carpeta `Chapter11` en el repositorio de GitHub de este libro.
3. Para ver la etiqueta personalizada en acción, inicia tu servidor web ejecutando el siguiente comando:
   ```bash
   python manage.py runserver localhost:8000
   ```
   Al usar el formulario de búsqueda, los colaboradores se seguirán renderizando como antes, pero la etiqueta simple simplifica enormemente la plantilla de Django.
   *Figura 11.2 – Resultados del formulario de búsqueda*

Con esto, creamos nuestra primera etiqueta de plantilla personalizada y la usamos con éxito para renderizar nuestra plantilla, como se muestra en la Figura 11.2. Ahora, veamos otro aspecto importante de las etiquetas simples, que está asociado con pasar las variables de contexto disponibles en la plantilla a la etiqueta de plantilla.

#### Paso del contexto de la plantilla en una etiqueta de plantilla personalizada

En el ejercicio anterior, creamos una etiqueta simple a la que pasamos dos argumentos: el mensaje de saludo y el nombre de usuario. Pero, ¿qué pasa si quisiéramos pasar una gran cantidad de variables a la etiqueta? O, simplemente, ¿qué pasa si no quisiéramos pasar el nombre de usuario explícitamente a la etiqueta?

Hay ocasiones en las que los desarrolladores desearían tener acceso a todas las variables y datos que están presentes en la plantilla para que estén disponibles dentro de la etiqueta personalizada. Afortunadamente para nosotros, esto es fácil de implementar.

Creemos una nueva etiqueta llamada `contextual_greet_user` y veamos cómo podemos pasar los datos disponibles en la plantilla directamente a la etiqueta en lugar de pasarlos manualmente como argumento.

Configura la etiqueta de la siguiente manera:

```python
@register.simple_tag(takes_context=True)
def contextual_greet_user(context, message):
    username = context['username']
    return f"{message}, {username}"
```

Con esto, le decimos a Django que cuando se use nuestra etiqueta `contextual_greet_user`, Django también debe pasarle el contexto de la plantilla, que tiene todos los datos pasados de la vista a la plantilla. Acepta el contexto agregado como argumento y renderiza un mensaje de saludo al usuario actual.

En el ejemplo de código anterior, podemos ver cómo se modificó el método `contextual_greet_user()` para aceptar el contexto pasado como primer argumento, seguido del mensaje de saludo pasado por el usuario.

Para usar esto, necesitamos que la siguiente llamada a la etiqueta `contextual_greet_user` dentro de `simple_tag_template.html` bajo `filter_demo` se vea así:

```django
{% contextual_greet_user "Hey" %}
```

Luego, cuando recarguemos nuestra aplicación web Django, la salida en `http://localhost:8000/filter_demo/greet` debería verse igual a como se mostró en el paso 5 del Ejercicio 11.02.

Con esto, aprendimos cómo podemos crear una etiqueta simple y manejar el paso del contexto de la plantilla a la etiqueta. Ahora, echemos un vistazo a cómo podemos crear una etiqueta de inclusión que se pueda usar para renderizar datos en un formato determinado, según lo describe otra plantilla.

#### Etiquetas de inclusión

Las etiquetas simples nos permiten crear etiquetas que aceptan una o más variables de entrada, realizan algún procesamiento en ellas y devuelven una salida. Esta salida luego se inserta donde se usó la etiqueta simple.

Pero, ¿qué pasa si quisiéramos crear etiquetas que, en lugar de devolver una salida de texto, devuelvan una plantilla HTML, que luego se pueda usar para renderizar partes de la página? Por ejemplo, muchas aplicaciones web permiten a los usuarios agregar widgets personalizados a sus perfiles. Estos widgets individuales se pueden crear como una etiqueta de inclusión y renderizarse de forma independiente. Este tipo de enfoque mantiene el código para la plantilla de la página base y las plantillas individuales por separado y, por lo tanto, permite una fácil reutilización y refactorización.

Desarrollar etiquetas de inclusión personalizadas es un proceso similar a cómo desarrollamos nuestras etiquetas simples. Esto implica el uso del decorador `inclusion_tag` proporcionado por la biblioteca de plantillas. Entonces, veamos cómo podemos hacerlo.

##### Implementación de etiquetas de inclusión
Las etiquetas de inclusión son aquellas etiquetas que se utilizan para renderizar una plantilla como respuesta a su uso dentro de una plantilla. Estas etiquetas se pueden implementar de manera similar a cómo se implementan otras etiquetas de plantilla personalizadas, con algunas modificaciones menores.

Las etiquetas de inclusión también son funciones simples de Python que pueden tomar múltiples parámetros, donde cada parámetro se asigna a un argumento pasado desde la plantilla donde se llamó a la etiqueta. Estas etiquetas se decoran usando el decorador `inclusion_tag` de la biblioteca de plantillas de Django. El decorador `inclusion_tag` toma un único parámetro, el nombre de la plantilla, que debe renderizarse como respuesta al procesamiento de la etiqueta de inclusión.

Una implementación genérica de una etiqueta de inclusión se verá como la que se muestra en el siguiente fragmento de código:

```python
@register.inclusion_tag('template_file.html')
def my_inclusion_tag(arg):
    # logic for processing
    return {'key1': 'value1'}
```

Observa el valor de retorno en este caso. Se supone que una etiqueta de inclusión devuelve un diccionario de valores que se utilizará para renderizar el archivo `template_file.html` especificado como argumento en el decorador `inclusion_tag`.

##### Uso de una etiqueta de inclusión dentro de una plantilla
Una etiqueta de inclusión se puede utilizar fácilmente dentro de un archivo de plantilla. Esto se puede hacer importando primero la etiqueta de la siguiente manera:

```django
{% load custom_tags %}
```

Luego, usando la etiqueta como cualquier otra etiqueta:

```django
{% my_inclusion_tag "argument1" %}
```

La respuesta del renderizado de esta etiqueta será una subplantilla que se renderizará dentro de nuestra plantilla principal donde se utilizó la etiqueta de inclusión.

#### Ejercicio 11.03 – Creación de una etiqueta de inclusión personalizada

En este ejercicio, crearemos una etiqueta de inclusión personalizada llamada `book_item`, que mostrará los detalles de un libro, simplificando el código en la plantilla de lista de libros existente de la aplicación `reviews`.

Para este ejercicio, continuarás usando las mismas carpetas de demostración que en los ejercicios anteriores:

1. Primero, crea un nuevo archivo llamado `inclusion_tag.py` en el directorio `filter_demo/templatetags` y escribe el siguiente código dentro de él:
   ```python
   from django import template

   register = template.Library()

   @register.inclusion_tag('book_list_item.html')
   def book_list_item(book):
       return {'item': book}
   ```
   El decorador `@register.inclusion_tag` se utiliza para marcar el método como una etiqueta de inclusión personalizada. Este decorador toma el nombre de la plantilla como un argumento que debe usarse para renderizar los datos devueltos por la función de la etiqueta.
2. Después del decorador, definimos una función que implementa la lógica de nuestra etiqueta de inclusión personalizada. Esta función toma un único argumento llamado `book`. Este argumento se pasará desde el archivo de plantilla y contendrá un objeto `Book`.
   Este objeto luego se devuelve como contexto para la plantilla `book_list_item.html`:
   ```python
   return {'item': book}
   ```
   El valor devuelto por este método será pasado por Django a la plantilla `book_list_item.html`, y los contenidos luego se renderizarán.
3. A continuación, crea la plantilla real, que contendrá la estructura de representación para la etiqueta de plantilla. Para esto, crea un nuevo archivo de plantilla, `book_list_item.html`, en el directorio `reviews/templates`, y agrégale el siguiente contenido:
   `reviews/templates/book_list_item.html`:
   ```django
   {% load link_worldcat_filter %}
   <li class="list-group-item">
       <span class="text-info">Title: </span> <span>{{ item.book.title }}</span> <br>
       <span class="text-info">Publisher: </span> <span>{{ item.book.publisher }}</span> <br>
       <span class="text-info">Publication Date: </span> <span>{{ item.book.publication_date }}</span> <br>
       <span class="text-info">ISBN: </span> <span> <a href="{{ item.book.isbn|link_worldcat:"isbn" }}"> {{ item.book.isbn }}</a></span> <br>
       <span class="text-info">Rating: </span> <span class="badge badge-primary badge-pill"> {{ item.book_rating }}</span> <br>
       <span class="text-info">Number of reviews: </span> <span>{{ item.number_of_reviews }}</span> <br>
       {% if not item.book_rating %}
           <span class="text-secondary"> Provide a rating and write the first review for this book.</span> <br>
       {% endif %}
       <a class="btn btn-primary btn-sm active" role="button" aria-pressed="true" href="/books/{{ item.book.id }}/">Reviews</a>
   </li>
   ```
   Aquí, en el nuevo archivo de plantilla, usamos el contenido de la etiqueta `<li>` de la plantilla `reviews/templates/reviews/book_list.html`. Ten en cuenta que también incluimos la directiva `load link_worldcat_filter`, ya que este filtro ahora está incluido dentro de la etiqueta de inclusión.
4. Con la plantilla definida para la etiqueta `book_list_item`, modifica la plantilla `book_list.html` existente para que esta etiqueta esté disponible dentro de ella y reemplaza el contenido de la etiqueta `<li>` con una llamada a `book_list_item`:
   `reviews/templates/reviews/book_list.html`:
   ```django
   {% extends 'reviews/base.html' %}
   {% load inclusion_tag %}

   {% block content %}
   <ul class="list-group">
       {% for item in book_list %}
           {% book_list_item item %}
       {% endfor %}
   </ul>
   {% endblock %}
   ```
   En este fragmento, lo primero que hiciste fue cargar el módulo `inclusion_tag` escribiendo lo siguiente:
   ```django
   {% load inclusion_tag %}
   ```
   Una vez cargada la etiqueta, puedes usarla en cualquier lugar de la plantilla. Para usarla, agregaste la etiqueta `book_list_item` en el siguiente formato:
   ```django
   {% book_list_item item %}
   ```
   Esta etiqueta toma un solo argumento, un objeto `Book`.
   Una vez que se hayan realizado los cambios, los archivos deberían parecerse a los alojados en la carpeta `Chapter11` en el repositorio de GitHub de este libro.
5. Con los cambios anteriores implementados, es hora de renderizar la plantilla modificada. Para hacer esto, reinicia el servidor de tu aplicación Django ejecutando el siguiente comando:
   ```bash
   python manage.py runserver localhost:8080
   ```
6. Navega a `http://localhost:8080/books`, que ahora debería renderizar una página similar a la siguiente captura de pantalla:
   *Figura 11.3 – Lista de libros después de refactorizar el código de la plantilla con inclusion_tag*

El punto de conexión de libros todavía se renderiza igual, pero hemos simplificado la plantilla `book_list.html` mediante el uso de una etiqueta de inclusión. Al modularizar el código de la plantilla con etiquetas de inclusión, facilitamos la reutilización de componentes de páginas sin una repetición masiva de bloques de código comunes.

#### Actividad 11.01 – Representación de detalles en la página de perfil de usuario mediante etiquetas de inclusión

En esta actividad, crearás una etiqueta de inclusión personalizada que ayude a desarrollar una página de perfil de usuario que no solo muestre los detalles de los usuarios sino que también enumere los libros que han leído.

Los siguientes pasos te ayudarán a completar esta actividad con éxito:

1. Crea un nuevo directorio `templatetags` en la aplicación `reviews` dentro del proyecto `bookr` para proporcionar un lugar donde puedas crear tus etiquetas de plantilla personalizadas.
2. En el archivo `inclusion_tag.py`, que creaste en el Ejercicio 11.03, agrega la siguiente funcionalidad:
   - Importa el modelo `Review` de la aplicación `reviews` para recuperar las reseñas escritas por un usuario. Esto se utilizará para filtrar las reseñas del usuario actual para representarlas en la página de perfil del usuario.
   - A continuación, crea una nueva función de Python llamada `review_list`, que contendrá la lógica para tu etiqueta de inclusión. Esta función solo debe tomar un único parámetro, el nombre de usuario del usuario que ha iniciado sesión actualmente.
   - Dentro del cuerpo de la función `review_list`, agrega la lógica para recuperar las reseñas para el usuario dado y extraer los IDs de reseña y los títulos de los libros que este usuario ha reseñado.
   - Decora esta función `review_list` con el decorador `inclusion_tag` y proporciónale un nombre de plantilla: `review_list.html`.
3. Crea un nuevo archivo de plantilla llamado `review_list.html`, que se especificó para el decorador de etiquetas de inclusión en el paso 2. Dentro de este archivo, agrega código para representar una lista de libros con hipervínculos a las reseñas del usuario. Esto se puede hacer usando una construcción de bucle `for` y renderizando etiquetas HTML de anclaje y salto de línea (`<a>`, `<br>`) para cada elemento dentro de la lista proporcionada. El bucle `for` contendrá una cláusula `empty` que mostrará: `"No reviews found"`.
4. Modifica el archivo `profile.html` existente en el directorio `templates`, que se usará para representar el perfil de usuario. Dentro de este archivo de plantilla, carga la biblioteca `inclusion_tag` y usa `review_list` para representar la lista de libros revisados por el usuario. El código se envolverá dentro de un nuevo `div` de clase `infocell`.

Una vez que completes todos estos pasos, iniciar el servidor de la aplicación y visitar la página de perfil de usuario debería renderizar una página similar a la que se muestra en la Figura 11.10:

*Figura 11.10 – La página de perfil de usuario con la lista de libros revisados por el usuario*

---

### Sección: Parciales de plantillas

A medida que codificamos nuestras plantillas en una aplicación Django, tendemos a ver mucha repetición en las características que hemos codificado. El uso de la herencia de plantillas base es una buena manera de reutilizar elementos comunes como encabezados, menús y pies de página, pero a veces queremos reutilizar componentes dentro de una página. Los parciales de plantilla (*template partials*) son similares a funciones que se definen dentro de una plantilla para definir un bloque de plantilla reutilizable.

Para crear un parcial de plantilla, primero definimos el contenido del parcial de plantilla entre las etiquetas de plantilla `partialdef` y `endpartialdef`. Podemos hacer referencia a variables de contexto en el parcial de plantilla como lo hacemos en el resto de una plantilla:

```django
{% partialdef "chapter" %}
    <h2> <a href="{{chapter.url}}" >{{chapter.number}}: {{chapter.title}}</a></h2>
{% endpartialdef %}
```

El parcial de plantilla se invoca utilizando la etiqueta de plantilla `partial`:

```django
{% for chapter in chapters %}
    {% partial "chapter" %}
{% endfor %}
```

Supongamos que la vista de Django analiza una variable de contexto para capítulos como esta:

```python
context = dict(chapters=(
    dict(url='/chap/1', number=1, title='Introduction'),
    dict(url='/chap/2', number=2, title='Historical Background'),
    dict(url='/chap/3', number=3, title='Technical Information'),
    dict(url='/chap/4', number=4, title='Conclusion'),
))
```

La plantilla se renderizará de la siguiente manera:

```html
<h2><a href="/chap/1">1: Introduction</a></h2>
<h2><a href="/chap/2">2: Historical Background</a></h2>
<h2><a href="/chap/3">3: Technical Information</a></h2>
<h2><a href="/chap/4">4: Conclusion</a></h2>
```

Este enfoque ofrece una manera más simple de reutilizar el código de plantilla que usar una etiqueta de inclusión, ya que el parcial de plantilla se define en el archivo de plantilla y no requiere la sobrecarga de una función adicional en el archivo `views.py`. Sin embargo, hay algunos casos, como la etiqueta de inclusión `review_list` en la Actividad 11.01, donde tiene sentido encapsular la base de datos o la lógica de control en la etiqueta de inclusión para que se pueda compartir entre múltiples vistas. En la siguiente actividad, reemplazaremos la etiqueta de inclusión `book_list_item` del Ejercicio 11.03 con un parcial de plantilla.

#### Actividad 11.02 – Renderizado de un elemento de lista de libros con parciales de plantilla

En esta actividad, convertiremos la etiqueta de inclusión definida en el Ejercicio 11.03 en un parcial de plantilla:

1. Utiliza el código en `reviews/templates/book_list_item.html` para formar el cuerpo de una definición parcial de plantilla en `reviews/templates/reviews/book_list.html`. Llama al parcial de plantilla `book_list_item` e incluye la directiva `load link_worldcat_filter` y el contenido de la etiqueta `li`.
2. Ahora, en el bloque de contenido en `reviews/templates/reviews/book_list.html`, reemplaza la llamada a la etiqueta de inclusión `book_list_item` con una llamada al parcial de plantilla `book_list_item` utilizando una directiva de plantilla de Django: `{% partial "book_list_item" %}`.
3. Ahora podemos eliminar el archivo `reviews/templates/book_list_item.html`, así como la etiqueta de inclusión `book_list_item` que está definida en `reviews/templatetags/inclusion_tag.py`.

La revisión de la lista de libros continuará renderizándose como lo hizo en el Ejercicio 11.03, pero hemos simplificado la codificación de la plantilla.

Con esto, ahora tenemos las bases sobre las cuales podemos crear filtros de plantilla altamente complejos, etiquetas personalizadas y parciales de plantilla que pueden ser útiles en el desarrollo de los proyectos en los que deseamos trabajar.

Ahora, echemos un vistazo a las vistas de Django y profundicemos en un nuevo territorio de vistas basadas en clases proporcionadas por Django para ayudarnos a aprovechar el poder de la programación orientada a objetos que permite la reutilización de código para el renderizado de una vista.

---

### Sección: Vistas de Django

Para recordar, una vista en Django es un fragmento de código de Python que permite recibir una solicitud, realiza una acción basada en la solicitud y luego devuelve una respuesta al usuario, formando así una parte importante de las aplicaciones de Django.

Dentro de Django, tenemos la opción de crear nuestras vistas siguiendo dos metodologías diferentes, una de las cuales ya hemos visto en los ejemplos anteriores y se conoce como vistas basadas en funciones (*function-based views*), mientras que la otra, que cubriremos en breve, se conoce como vistas basadas en clases (*class-based views*):
- **Vistas basadas en funciones (FBVs)**: Las FBVs dentro de Django no son más que funciones genéricas de Python que deben tomar un objeto de tipo `HTTPRequest` como su primer parámetro posicional y devolver un objeto de tipo `HTTPResponse`, que corresponde a la acción que la vista desea realizar una vez que la solicitud es procesada por ella. En el ejercicio anterior, `index()` y `greeting_view()` fueron ejemplos de FBVs.
- **Vistas basadas en clases (CBVs)**: Las CBVs son vistas que se adhieren estrechamente a los principios orientados a objetos de Python y permiten la asignación de llamadas a vistas en una representación basada en clases. Estas vistas son de naturaleza especializada y un tipo determinado de CBV realiza una operación específica. Los beneficios que brindan las CBVs incluyen una fácil extensibilidad de la vista y la reutilización de código, lo que puede resultar una tarea compleja con las FBVs.

Ahora, con las definiciones básicas claras y con el conocimiento de las FBVs ya en nuestro arsenal, veamos las CBVs y descubramos qué tienen reservado para nosotros.

---

### Sección: Vistas basadas en clases

Django proporciona diferentes formas en que los desarrolladores pueden escribir vistas para sus aplicaciones. Una forma es mapear una función de Python para que actúe como una función de vista para crear FBVs. Otra forma de crear vistas es usar instancias de objetos de Python (basadas en clases de Python). Estas se conocen como CBVs. Una pregunta importante que surge es: ¿cuál es la necesidad de una CBV cuando ya podemos crear vistas usando el enfoque FBV?

Al crear FBVs, la idea es que, a veces, podemos estar replicando la misma lógica una y otra vez, por ejemplo, el procesamiento de ciertos campos o la lógica para manejar ciertos tipos de solicitudes. Aunque es completamente posible crear funciones lógicamente separadas que manejen un fragmento particular de lógica, la tarea se vuelve difícil de administrar a medida que aumenta la complejidad de la aplicación.

Aquí es donde las CBVs resultan útiles, ya que abstraen la implementación del código repetitivo común que necesitamos escribir para manejar ciertas tareas, como la representación de plantillas, al mismo tiempo que facilitan la reutilización de fragmentos de código mediante el uso de herencia y mixins. Por ejemplo, el siguiente fragmento de código muestra la implementación de una CBV:

```python
from django.http import HttpResponse
from django.views import View

class IndexView(View):
    def get(self, request):
        return HttpResponse("Hey there!")
```

En el ejemplo anterior, creamos una CBV simple heredando de la clase integrada `View` que proporciona Django.

El uso de estas CBVs también es bastante fácil. Por ejemplo, digamos que queríamos asignar `IndexView` a un punto de conexión de URL en nuestra aplicación. En este caso, todo lo que tenemos que hacer es agregar la siguiente línea a nuestra lista `urlpatterns` dentro del archivo `urls.py` de la aplicación:

```python
urlpatterns = [
    path('my_path', IndexView.as_view(), name='index_view')
]
```

En esto, como podemos observar, usamos el método `as_view()` de la CBV que creamos. Cada CBV implementa el método `as_view()`, que permite asignar la clase `View` a un punto de conexión de URL devolviendo la instancia del controlador de vista desde la clase de vista.

Django proporciona un par de CBVs integradas que proporcionan la implementación de muchas tareas comunes, como cómo renderizar una plantilla o cómo procesar una solicitud en particular. Las CBVs integradas ayudan a evitar reescribir código desde cero al manejar la funcionalidad básica, lo que permite la reutilización de código. Algunas de estas vistas integradas incluyen las siguientes:
- **View**: Esta es la clase base para todas las CBVs disponibles en Django que permite escribir una CBV personalizada con todas las características proporcionadas y modificables. Un usuario puede implementar sus propias definiciones para diferentes métodos de solicitud HTTP, como GET, POST, PUT y DELETE, y la vista delegará automáticamente la llamada al método responsable de manejar la solicitud según el tipo de solicitud recibida.
- **TemplateView**: Esta es una vista que se puede utilizar para renderizar una plantilla según los parámetros para los datos de la plantilla proporcionados en la URL de la llamada. Esto permite a los desarrolladores renderizar fácilmente una plantilla sin escribir ninguna lógica relacionada con cómo se debe manejar el renderizado.
- **RedirectView**: Esta es una vista que puede redirigir automáticamente a un usuario al recurso correcto según la solicitud que haya realizado.
- **DetailView**: Esta es una vista que se asigna a un modelo de Django y se puede utilizar para representar los datos obtenidos del modelo mediante una plantilla de elección.

Las vistas anteriores son solo algunas de las vistas integradas que Django proporciona de forma predeterminada, y cubriremos más a medida que avancemos en el capítulo.

Ahora, para comprender mejor cómo funcionan las CBVs dentro de Django, intentemos crear nuestra primera CBV.

#### Ejercicio 11.04 – Creación de un catálogo de libros mediante una CBV

En este ejercicio, crearemos una vista de formulario basada en clases que ayudará a crear un catálogo de libros. Este catálogo constará del nombre del libro y el nombre del autor del libro:

1. Para comenzar, crea una nueva aplicación dentro del proyecto `bookr` y nómbrala `book_management`. Esto se puede hacer simplemente ejecutando el siguiente comando:
   ```bash
   python manage.py startapp book_management
   ```
2. Ahora, antes de crear el catálogo de libros, primero debes definir un modelo de Django que te ayudará a almacenar los registros dentro de la base de datos. Para hacer esto, abre el archivo `models.py` debajo de la aplicación `book_management` que acabas de crear y define un nuevo modelo llamado `Book`, como se muestra aquí:
   `book_management/models.py`:
   ```python
   from django.db import models

   class Book(models.Model):
       name = models.CharField(max_length=255)
       author = models.CharField(max_length=50)
   ```
   El modelo contiene dos campos: el nombre del libro (`name`) y el nombre del autor (`author`).
3. Una vez completados todos los pasos anteriores, agrega tu aplicación `book_management` a la lista `INSTALLED_APPS` para que Django pueda descubrirla y puedas usar tu modelo correctamente. Para esto, abre el archivo `settings.py` bajo el directorio `bookr` y agrega el siguiente código en la posición final en la sección `INSTALLED_APPS`:
   ```python
   INSTALLED_APPS = [
       …,
       'book_management'
   ]
   ```
   Después de realizar los cambios, el archivo `settings.py` debería verse como en la carpeta `Chapter11` en el repositorio de GitHub de este libro.
4. Migra tu modelo a la base de datos ejecutando los siguientes dos comandos. Estos primero crearán un archivo de migraciones de Django y luego crearán una tabla en tu base de datos:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```
5. Ahora, con el modelo de base de datos en su lugar, creemos un nuevo formulario que usaremos para capturar información perteneciente a los libros, como el título del libro, el autor y el ISBN. Para esto, crea un nuevo archivo llamado `forms.py` en el directorio `book_management` y agrega el siguiente código dentro de él:
   `book_management/forms.py`:
   ```python
   from django import forms
   from .models import Book

   class BookForm(forms.ModelForm):
       class Meta:
           model = Book
           fields = ['name', 'author']
   ```
   En el fragmento de código anterior, primero importamos el módulo `forms` de Django, que nos permitirá crear formularios fácilmente y también proporcionar la capacidad de renderizado del formulario. La siguiente línea importa el modelo que almacenará los datos para el formulario:
   ```python
   from django import forms
   from .models import Book
   ```
   En la siguiente línea, creamos una nueva clase llamada `BookForm`, que hereda de `ModelForm`. Esto no es más que una clase que asigna los campos del modelo al formulario. Para lograr con éxito esta asignación entre el modelo y el formulario, definimos una nueva subclase llamada `Meta` bajo la clase `BookForm` y establecemos el atributo `model` para que apunte al modelo `Book` y el atributo `fields` a la lista de campos que deseas mostrar en el formulario:
   ```python
   class Meta:
       model = Book
       fields = ['name', 'author']
   ```
   Esto permite que `ModelForm` renderice el formulario HTML correcto cuando se espera que lo haga. La clase `ModelForm` proporciona un método integrado `Form.save()`, que, cuando se usa, escribe los datos del formulario en la base de datos y ayuda a evitar tener que escribir código redundante.
6. Ahora que tienes listos tanto tu modelo como el formulario, continúa e implementa una vista que renderizará el formulario y aceptará la entrada del usuario. Para esto, abre `views.py` en el directorio `book_management` y agrega las siguientes líneas de código al archivo:
   `book_management/views.py`:
   ```python
   from django.http import HttpResponse
   from django.views.generic.edit import FormView
   from django.views import View
   from .forms import BookForm

   class BookRecordFormView(FormView):
       template_name = 'book_form.html'
       form_class = BookForm
       success_url = '/book_management/entry_success'

       def form_valid(self, form):
           form.save()
           return FormView.form_valid(self, form)

   class FormSuccessView(View):
       def get(self, request, *args, **kwargs):
           return HttpResponse(
               "Book record saved successfully")
   ```
   En el fragmento de código anterior, creamos dos vistas principales: `BookRecordFormView`, que también es responsable de renderizar el formulario de entrada del catálogo de libros, y `FormSuccessView`, que usarás para renderizar el mensaje de éxito si los datos del formulario se guardan correctamente. Veamos ahora ambas vistas individualmente y comprendamos lo que estamos haciendo.
   Primero, creamos una nueva vista llamada `BookRecordFormView` CBV, que hereda de `FormView`:
   ```python
   class BookRecordFormView(FormView)
   ```
   La clase `FormView` nos permite crear fácilmente vistas que se ocupan de formularios. A esta clase, debemos proporcionarle ciertos parámetros, como el nombre de la plantilla que renderizará para mostrar el formulario, la clase de formulario que debe usar para renderizar el formulario y la URL de éxito a la que redirigir cuando el formulario se procese con éxito:
   ```python
   template_name = 'book_form.html'
   form_class = BookForm
   success_url = '/book_management/entry_success'
   ```
   La clase `FormView` también proporciona un método `form_valid()`, que se llama cuando el formulario finaliza con éxito la validación. Dentro del método `form_valid()`, podemos decidir qué queremos hacer. Para nuestro caso de uso, cuando la validación del formulario se completa con éxito, primero llamamos al método `form.save()`, que persiste los datos de nuestro formulario en la base de datos, y luego llamamos al método `form_valid()` de la clase base, que hará que la vista del formulario redirija a la URL exitosa en caso de que la validación del formulario haya sido un éxito:
   ```python
   def form_valid(self, form):
       form.save()
       return FormView.form_valid(self, form)
   ```
   El método `form_valid()` siempre debe devolver un objeto `HttpResponse`.
   Esto completa la implementación de `BookRecordFormView`. Lo siguiente que tenemos que hacer es crear una vista llamada `FormSuccessView`, que usaremos para renderizar el mensaje de éxito una vez que los datos se guarden correctamente para el formulario de registro de libro que acabamos de crear. Esto se hace creando una nueva clase de vista llamada `FormSuccessView`, que hereda de la clase base `View` de las CBVs de Django:
   ```python
   class FormSuccessView(View)
   ```
   Dentro de esta clase, anulamos el método `get()`, que se renderizará cuando el formulario se guarde correctamente. Dentro del método `get()`, representamos un mensaje de éxito simple devolviendo una nueva `HttpResponse`:
   ```python
   def get(self, request, *args, **kwargs):
       return HttpResponse("Book record saved successfully")
   ```
7. Ahora, crea una plantilla que se utilizará para representar el formulario. Para esto, crea una nueva carpeta `templates` en el directorio `book_management` y un nuevo archivo llamado `book_form.html`. Agrega las siguientes líneas de código dentro del archivo:
   `book_management/templates/book_form.html`:
   ```html
   <html>
   <head>
       <title>Book Record Insertion</title>
   </head>
   <body>
       <form method="POST">
           {% csrf_token %}
           {{ form.as_p }}
           <input type="submit" value="Save record" />
       </form>
   </body>
   </html>
   ```
8. Con las CBVs ya creadas, continúa y asígnalas a URLs para que puedas comenzar a usarlas para agregar nuevos registros de libros. Para hacer esto, crea un nuevo archivo llamado `urls.py` en el directorio `book_management` y agrégale el siguiente código:
   `book_management/urls.py`:
   ```python
   from django.urls import path
   from .views import BookRecordFormView, FormSuccessView

   urlpatterns = [
       path('new_book_record', BookRecordFormView.as_view(), name='book_record_form'),
       path('entry_success', FormSuccessView.as_view(), name='form_success'),
   ]
   ```
   La mayoría de las partes del fragmento anterior son similares a las que escribiste anteriormente, pero hay algo diferente en la forma en que asignamos nuestras CBVs bajo los patrones de URL. Al usar CBVs, en lugar de agregar directamente el nombre de la función, usamos el nombre de la clase y su método `as_view`, que asigna el objeto de clase a la vista. Por ejemplo, para asignar `BookRecordFormView` como una vista, usaremos `BookRecordFormView.as_view()`.
9. Con las URLs agregadas a nuestro archivo `urls.py`, lo siguiente es agregar la asignación de URL de la aplicación al proyecto `bookr`. Para hacer esto, abre el archivo `urls.py` bajo la aplicación `bookr` y agrega la siguiente línea a `urlpatterns`:
   `bookr/urls.py`:
   ```python
   urlpatterns = [
       path('book_management/', include('book_management.urls')),
       …
   ]
   ```
10. Ahora, inicia tu servidor de desarrollo ejecutando el siguiente comando:
    ```bash
    python manage.py runserver localhost:8080
    ```
11. Luego, visita `http://localhost:8080/book_management/new_book_record`. Si todo funciona bien, verás una página como la que se muestra aquí:
    *Figura 11.4 – La vista para agregar un nuevo libro a la base de datos*
12. Al hacer clic en **Save record**, tu registro se escribirá en la base de datos y aparecerá la siguiente página:
    *Figura 11.5 – La plantilla se renderiza cuando el registro se inserta con éxito*

Con esto, hemos creado nuestra propia CBV, que nos permite guardar registros para nuevos libros. Con nuestro conocimiento de las CBVs a cuestas, veamos ahora cómo podemos realizar operaciones de creación, lectura, actualización y eliminación (CRUD) con la ayuda de las CBVs.

#### Operaciones CRUD con CBVs

Al trabajar con modelos de Django, uno de los patrones más comunes con los que nos encontramos implica crear, leer, actualizar y eliminar objetos que están almacenados dentro de nuestra base de datos. La interfaz de administración de Django nos permite lograr estas operaciones CRUD fácilmente, pero ¿qué pasa si quisiéramos crear vistas personalizadas para darnos la misma capacidad?

Resulta que las CBVs de Django nos permiten lograr esto con bastante facilidad. Todo lo que tenemos que hacer es escribir nuestras CBVs personalizadas y heredarlas de las clases base integradas proporcionadas por Django. Sobre la base de nuestro ejemplo existente de gestión de registros de libros, veamos cómo podemos crear vistas basadas en CRUD en Django.

##### La vista Create (Crear)
Para crear una vista que ayude en la creación de objetos, necesitaremos abrir el archivo `views.py` bajo el directorio `book_management` y agregarle las siguientes líneas de código:

```python
from django.views.generic.edit import CreateView
from .models import Book

class BookCreateView(CreateView):
    model = Book
    fields = ['name', 'author']
    template_name = 'book_form.html'
    success_url = '/book_management/entry_success'
```

Con esto, creamos `CreateView` para el recurso de libros. Antes de que podamos usarlo, necesitaremos asignarlo a una URL. Para hacer esto, podemos abrir el archivo `urls.py` y agregar la siguiente entrada debajo de la lista `urlpatterns`:

```python
urlpatterns = [
    …,
    path('book_record_create', BookCreateView.as_view(), name='book_create')
]
```

Ahora, cuando visitemos `http://localhost:8080/book_management/book_record_create`, seremos recibidos con la siguiente página:

*Figura 11.6 – Una vista para insertar un nuevo registro de libro basado en la vista Create*

Esto se parece a lo que obtuvimos al usar la vista de formulario. Al completar los datos y hacer clic en **Save record**, Django guardará los datos en la base de datos.

##### La vista Read (Leer / Detalle)
En esta vista, queremos ver una lista de los registros que almacena nuestra base de datos para los libros. Para lograr esto, crearemos una vista llamada `DetailView`, que renderizará detalles sobre el libro que estamos solicitando. Para crear esta vista, podemos agregar las siguientes líneas de código a nuestro archivo `views.py` bajo el directorio `book_management`:

```python
from django.views.generic import DetailView

class BookRecordDetailView(DetailView):
    model = Book
    template_name = 'book_detail.html'
```

En el fragmento de código anterior, creamos `DetailView`, que nos ayudará a representar los detalles pertenecientes al ID de libro que estamos solicitando. `DetailView` consulta internamente nuestro modelo de base de datos con el ID de libro que le proporcionamos y, si se encuentra un registro, renderiza la plantilla con los datos almacenados dentro del registro pasándolo como una variable de objeto (`object`) dentro del contexto de la plantilla.

Una vez hecho esto, el siguiente paso es crear la plantilla para los detalles de nuestro libro. Para esto, necesitaremos crear un nuevo archivo de plantilla llamado `book_detail.html` en nuestro directorio `templates` dentro de la aplicación `book_management` con los siguientes contenidos:

`book_management/templates/book_detail.html`:
```html
<html>
<head>
    <title>Book List</title>
</head>
<body>
    <span>Book Name: {{ object.name }}</span><br />
    <span>Author: {{ object.author }}</span>
</body>
</html>
```

Ahora, con la plantilla creada, lo último que debemos hacer es agregar una asignación de URL para la vista de detalle. Esto se puede hacer agregando lo siguiente a la lista `urlpatterns` bajo el archivo `urls.py` de la aplicación `book_management`:

```python
path('book_record_detail/<int:pk>', BookRecordDetailView.as_view(), name='book_detail')
```

Ahora, con todo esto configurado, si abrimos `http://localhost:8080/book_management/book_record_detail/2`, podremos ver los detalles sobre nuestro libro, como se muestra aquí:

*Figura 11.7 – La vista renderizada cuando intentamos acceder a un registro de libro almacenado previamente*

Con los ejemplos anteriores, acabamos de habilitar las operaciones CRUD para nuestro modelo `Book`, todo mientras usamos CBVs.

##### La vista Update (Actualizar)
En esta vista, queremos actualizar los datos de un registro determinado. Para hacer esto, debemos abrir el archivo `views.py` en el directorio `book_management` y agregarle las siguientes líneas de código:

```python
from django.views.generic.edit import UpdateView
from .models import Book

class BookUpdateView(UpdateView):
    model = Book
    fields = ['name', 'author']
    template_name = 'book_form.html'
    success_url = '/book_management/entry_success'
```

En el fragmento de código anterior, utilizamos la plantilla integrada `UpdateView`, que nos permite actualizar los registros almacenados. Por lo tanto, el atributo `fields` aquí debe tomar el nombre de los campos que nos gustaría permitir que el usuario actualice.

Una vez creada la vista, el siguiente paso es agregar la asignación de URL. Para hacer esto, abre el archivo `urls.py` en el directorio `book_management` y agrega las siguientes líneas de código:

```python
urlpatterns = [
    path('book_record_update/<int:pk>', BookUpdateView.as_view(), name='book_update')
]
```

En este ejemplo, agregamos `<int:pk>` al campo de URL. Esto significa la entrada de campo que tendremos que recuperar para el registro. Dentro de los modelos de Django, Django inserta una clave primaria de tipo entero, que identifica de forma única los registros. Dentro de la asignación de URL, este es el campo que solicitamos insertar.

Ahora, cuando intentemos abrir `http://localhost:8080/book_management/book_record_update/1`, debería mostrarnos el primer registro que insertamos en nuestra base de datos y permitirnos editarlo:

*Figura 11.8 – La vista que muestra la plantilla de actualización de registro de libro basada en la vista Update*

##### La vista Delete (Eliminar)
La vista Delete, como su nombre lo indica, es una vista que elimina el registro de nuestra base de datos. Para implementar dicha vista para nuestro modelo `Book`, necesitaremos abrir el archivo `views.py` en el directorio `book_management` y agregarle el siguiente fragmento de código:

```python
from django.views.generic.edit import DeleteView
from .models import Book

class BookDeleteView(DeleteView):
    model = Book
    template_name = 'book_delete_form.html'
    success_url = '/book_management/delete_success'
```

Con esto, acabamos de crear una vista de eliminación para nuestros registros de libros. Como podemos ver, esta vista usa una plantilla diferente, donde todo lo que nos gustaría confirmar con el usuario es si realmente desea eliminar el registro o no. Para lograr esto, puedes crear un nuevo archivo de plantilla, `book_delete_form.html`, y agregarle el siguiente código:

`book_management/templates/book_delete_form.html`:
```html
<html>
<head>
    <title>Delete Book Record</title>
</head>
<body>
    <p>Delete Book Record</p>
    <form method="POST">
        {% csrf_token %}
        Do you want to delete the book record?
        <input type="submit" value="Delete record" />
    </form>
</body>
</html>
```

Para el código relacionado con el punto de conexión `/delete_success`, echa un vistazo a los archivos de código en el directorio `Chapter11` en el repositorio de GitHub que acompaña al libro.

Luego, podemos agregar una asignación para nuestra vista de eliminación modificando la lista `urlpatterns` dentro del archivo `urls.py` en el directorio `book_management` de la siguiente manera:

```python
urlpatterns = [
    …,
    path('book_record_delete/<int:pk>', BookDeleteView.as_view(), name='book_delete')
]
```

Ahora, al visitar `http://localhost:8080/book_management/book_record_delete/1`, deberíamos ser recibidos con la siguiente página:

*Figura 11.9 – La vista Delete Book Record basada en la clase DeleteView*

Al hacer clic en el botón **Delete record**, el registro se eliminará de la base de datos y se renderizará la página de éxito de eliminación (*Deletion success*).

---

### Sección: Resumen

En este capítulo, aprendimos sobre los conceptos avanzados de plantillas en Django y entendimos cómo podemos crear etiquetas de plantilla, filtros y parciales personalizados para adaptarnos a una gran variedad de casos de uso y respaldar la reutilización de componentes en toda la aplicación. Luego examinamos cómo Django nos brinda la flexibilidad de implementar FBVs y CBVs para renderizar nuestras respuestas.

Al explorar las CBVs, aprendimos cómo pueden ayudarnos a evitar la duplicación de código y aprovechar las CBVs integradas para representar formularios que guardan datos, nos ayudan a actualizar registros existentes e implementan operaciones CRUD en nuestros recursos de base de datos.

A medida que avanzamos hacia el próximo capítulo, ahora utilizaremos nuestro conocimiento sobre la creación de CBVs para trabajar en la implementación de APIs REST en Django. Esto nos permitirá realizar operaciones HTTP bien definidas sobre nuestros datos dentro de nuestra aplicación Bookr sin mantener ningún estado dentro de la aplicación.
