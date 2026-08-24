# Parte 2: Creación de aplicaciones web con Django

## Capítulo 6: Formularios

Este capítulo presenta los formularios web, un método para enviar información desde el navegador a un servidor web. Comenzaremos introduciendo los formularios en general y analizando cómo se codifican los datos para enviarlos al servidor.

Hasta ahora, las vistas que hemos estado construyendo para Django han sido únicamente unidireccionales. Nuestro navegador recupera datos de las vistas que hemos escrito, pero no les envía ningún dato de vuelta. En el Capítulo 4 (*Introducción al Administrador de Django*), creamos instancias de modelos utilizando el administrador de Django y enviando formularios, pero aquellos utilizaban vistas integradas en Django, no creadas por nosotros. En este capítulo, utilizaremos la biblioteca Django Forms para comenzar a aceptar datos enviados por los usuarios. Los datos se proporcionarán mediante solicitudes GET en los parámetros de la URL y/o solicitudes POST en el cuerpo de la solicitud. Pero antes de entrar en detalles, primero comprendamos qué son los formularios en Django. Aprenderás sobre las diferencias entre enviar datos de formularios en una solicitud HTTP GET y enviarlos en una solicitud HTTP POST, y cómo elegir cuál utilizar.

En este capítulo, cubriremos los siguientes temas:
- Fundamentos de los formularios
- Seguridad en formularios
- La biblioteca Django Forms
- Validación de formularios y obtención de valores de Python
- Actividad 6.01 – Búsqueda de libros

Al final de este capítulo, habrás aprendido cómo se utiliza la biblioteca Forms de Django para construir y validar formularios automáticamente y cómo reduce la cantidad de HTML manual que debes escribir.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter06` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Fundamentos de los formularios

Los formularios de Django actúan como un asistente que realiza tres tareas: construyen los campos HTML, verifican que la entrada del usuario sea válida y proporcionan datos limpios para usar en una vista. Son como un guardián entre lo que escribe un usuario y lo que acepta la aplicación.

Cuando trabajamos con una aplicación web interactiva, no solo queremos proporcionar datos a los usuarios, sino también aceptar datos de ellos, ya sea para personalizar las respuestas que estamos generando o para permitirles enviar datos al sitio. Un formulario se compone de entradas (*inputs*) que definen pares clave-valor de datos para enviar al servidor. Por ejemplo, al iniciar sesión en un sitio web, los datos que se envían tendrían las claves `username` y `password`, con los valores de tu nombre de usuario y tu contraseña, respectivamente. Cada entrada en el formulario tiene un nombre (`name`), y así es como se identifican sus datos en el lado del servidor (en una vista de Django). Puede haber múltiples entradas con el mismo nombre, cuyos datos están disponibles en una lista que contiene todos los valores enviados con este nombre; por ejemplo, una lista de casillas de verificación (*checkboxes*) con permisos para aplicar a los usuarios. Cada casilla de verificación tendría el mismo nombre pero un valor diferente. El formulario tiene atributos que especifican a qué URL debe enviar los datos el navegador y qué método debe utilizar para enviarlos (los navegadores solo admiten GET o POST).

#### El elemento form

Todas las entradas utilizadas durante el envío del formulario deben estar contenidas dentro de un elemento `<form>`. Hay tres atributos HTML que debes utilizar para modificar el comportamiento del formulario:
- **Method**: Este es el método HTTP para enviar el formulario; es GET o POST. Si se omite, el valor predeterminado es `get` (porque este es el método predeterminado al escribir una URL en el navegador y presionar Enter).
- **Action**: Se refiere a la URL (o ruta) a la que se enviarán los datos del formulario. Si se omite, los datos se devuelven a la página actual.
- **Enctype**: Establece el tipo de codificación del formulario. Solo necesitas cambiar esto si estás utilizando el formulario para subir archivos. Los valores más comunes son `application/x-www-form-urlencoded` (el valor predeterminado si se omite este valor) o `multipart/form-data` (establécelo si estás subiendo archivos). Ten en cuenta que no tienes que preocuparte por el tipo de codificación en tu vista: Django maneja los diferentes tipos automáticamente.

He aquí un ejemplo de un formulario sin ninguno de sus atributos establecidos:

```html
<form> <!-- Input elements go here --> </form>
```

Enviará sus datos mediante una solicitud GET a la URL actual en la que se muestra el formulario, utilizando el tipo de codificación `application/x-www-form-urlencoded`.

En el siguiente ejemplo, estableceremos los tres atributos en un formulario:

```html
<form method="post" action="/form-submit" enctype="multipart/form-data"> <!-- Input elements go here --> </form>
```

Este formulario enviará sus datos con una solicitud POST a la ruta `/form-submit`, codificando los datos como `multipart/form-data`.

¿En qué se diferencian las solicitudes GET y POST en la forma en que se envían los datos? Recuerda que en el Capítulo 1 (*Introducción a Django*) analizamos cómo son las solicitudes y respuestas HTTP subyacentes que envía tu navegador. En los dos ejemplos siguientes, enviaremos el mismo formulario dos veces: la primera vez usando GET y la segunda vez usando POST. El formulario tendrá dos entradas: un nombre (`first_name`) y un apellido (`last_name`).

Un formulario enviado mediante GET envía sus datos en la URL de esta manera:

```http
GET /form-submit?first_name=Joe&last_name=Bloggs HTTP/1.1
Host: www.example.com
```

Sin embargo, un formulario enviado mediante POST envía sus datos en el cuerpo de la solicitud, de esta manera:

```http
POST /form-submit HTTP/1.1
Host: www.example.com
Content-Length: 31
Content-Type: application/x-www-form-urlencoded

first_name=Joe&last_name=Bloggs
```

Notarás que los datos del formulario se codifican de la misma manera en ambos casos; simplemente se ubican de manera diferente para las solicitudes GET y POST. En la sección *Elección entre GET y POST*, analizaremos cómo elegir entre estos dos tipos de solicitudes.

---

### Sección: Seguridad en formularios

Dado que los formularios HTML son una fuente de datos externos que llegan a una aplicación, debemos considerar las implicaciones de seguridad que plantean.

Analizaremos brevemente las principales consideraciones de seguridad que los formularios HTML introducen en una aplicación y las abordaremos en este y en capítulos posteriores. Esta no es de ninguna manera una lista completa de consideraciones de seguridad, ya que surgen nuevas amenazas con el tiempo.

Para obtener más información sobre seguridad, consulta *Attacking and Exploiting Modern Web Applications*, de Simone y Donato Onofri (Packt Publishing, 2023, ISBN: 9781801816298): [https://www.packtpub.com/en-us/product/attacking-and-exploiting-modern-web-applications-9781801816298](https://www.packtpub.com/en-us/product/attacking-and-exploiting-modern-web-applications-9781801816298).

El Proyecto Abierto de Seguridad de Aplicaciones Web (*Open Worldwide Application Security Project* u OWASP) proporciona excelentes recursos en línea para la educación en seguridad de aplicaciones, incluidas las diez principales vulnerabilidades de OWASP: [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/).

#### Validación de entrada (Input validation)

Estamos acostumbrados a ver la validación de formularios del lado del cliente, como las comprobaciones de JavaScript, dentro de la página HTML ejecutada por el navegador, pero también es crucial llevar a cabo estas comprobaciones en el lado del servidor. Si la validación del formulario solo se realiza en el lado del cliente, esto se puede eludir desactivando JavaScript en el navegador o evitando el navegador enviando los datos del formulario mediante un script o una herramienta de desarrollo.

Tradicionalmente, este enfoque seguro ha implicado una lógica duplicada al incluir la validación del frontend así como la validación del backend. Afortunadamente, en Django, la clase `django.forms.Form` proporciona una fuente única de verdad para definir la lógica de validación, sobre la cual aprenderemos en la sección *La biblioteca Django Forms*.

#### Protección contra la falsificación de solicitudes en sitios cruzados (CSRF)

La protección contra la falsificación de solicitudes en sitios cruzados (*Cross-Site Request Forgery* o CSRF) evita que una aplicación web reciba publicaciones de formularios desde otro sitio.

Históricamente, el atributo `action` del formulario podía especificar cualquier recurso web como una URL, y era posible crear un formulario remoto que incluso pudiera hacerse pasar por otro propósito, particularmente incluyendo contenido malicioso en campos ocultos. De hecho, el ataque CSRF podría incluso implementarse sin un formulario, siendo una solicitud HTTP POST implementada en JavaScript en la página remota.

La protección CSRF requiere que se incluya un token en el formulario que se espera en el lado del servidor. El token CSRF se almacena como una cookie en el navegador y es único para cada usuario y por sesión.

Como veremos en el Ejercicio 6.01, las aplicaciones de Django están configuradas de forma predeterminada con la protección CSRF habilitada.

Django también nos brinda un mecanismo para incluir el token CSRF en el formulario.

El token CSRF debe agregarse al HTML de cada formulario que se envíe, y esto se hace con la etiqueta de plantilla `{% csrf_token %}`. Cuando se renderiza la plantilla, esta etiqueta se interpolará de forma similar a la siguiente:

```html
<input type="hidden" name="csrfmiddlewaretoken" value="tETZjLDUXev1tiYqGCSbMQkhWiesHCnutxpt6mutHI6YH64F0nin5k2JW3B68IeJ">
```

Dado que este es un campo oculto, el formulario en la página no se ve diferente de cómo se veía antes.

El token CSRF es único para cada visitante del sitio. Si un atacante copiara el HTML de nuestro sitio, obtendría su propio token CSRF, que no coincidiría con el de ningún otro usuario, por lo que Django rechazaría el formulario cuando fuera publicado por otra persona.

Los tokens CSRF también cambian periódicamente. Esto limita el tiempo que tendría el atacante para aprovechar una combinación particular de usuario y token. Incluso si pudieran obtener el token CSRF de un usuario al que intentaban explotar, tendrían una ventana corta de tiempo para usarlo.

Implementaremos un token CSRF en un formulario en el Ejercicio 6.02.

#### Prevención de inyección SQL

Los ataques de inyección SQL fueron una debilidad de seguridad frecuente en los primeros días de la web. Un usuario podía inyectar SQL en un formulario creando cuidadosamente entradas que agregarían sentencias SQL adicionales a la llamada a la base de datos que se realiza en el lado del servidor durante la respuesta HTTP.

Este enfoque se basaba en que el código del lado del servidor estuviera mal escrito, de modo que la entrada del usuario se enviara a la base de datos sin que se escapara ningún carácter de control.

Al desarrollar nuestro código con un ORM u otra herramienta de base de datos, el programador se aleja de la construcción de SQL sin formato a partir de la entrada del usuario, y podemos eliminar este riesgo.

Vimos cómo construir consultas utilizando el ORM de Django en el Capítulo 2 y Capítulo 3.

#### Política de seguridad de contenido (CSP) y ataques XSS

Los ataques de secuencias de comandos en sitios cruzados (*Cross-Site Scripting* o XSS) son un amplio grupo de actos maliciosos que implican incluir contenido de JavaScript con etiquetas HTML en campos de entrada que luego se pueden renderizar en las páginas de visualización del sitio con fines maliciosos.

Considera un campo de texto en un formulario en un sitio web que es vulnerable a ataques XSS. Un usuario malicioso podría ingresar lo siguiente:

```html
"><script>malicious_function()</script><p class="
```

Si el sitio vulnerable no escapa la sintaxis de las etiquetas HTML, esto se renderizará de la siguiente manera:

```html
<input type="text" name="fname" value=""> <script>malicious_function()</script><p class="">
```

Afortunadamente, la biblioteca Django Forms ha sido diseñada para proteger contra este tipo de ataque. Esta entrada se escaparía de la siguiente manera, y los elementos HTML en la entrada se escaparían para que no se inyecten etiquetas de script en la página web:

```html
"><script>malicious_function()</script><p class="
```

Sin embargo, sería posible anular este diseño de seguridad si un desarrollador desarrollara una plantilla HTML personalizada que permitiera específicamente contenido etiquetado no seguro sin escaparlo:

```html
<input type="text" name="fname" value="UNTRUSTED DATA ">
```

#### HTTPS y cabeceras de seguridad

Alojar una aplicación sobre HTTP la hace vulnerable a ataques de intermediario (*man-in-the-middle*), donde un tercero observa el tráfico entre el cliente y el servidor y compromete la seguridad al capturar esos datos y reproducirlos en el servidor.

Al alojar nuestra aplicación web sobre HTTPS, evitamos esta vulnerabilidad. Nuestro entorno de desarrollo utiliza HTTP, pero aprenderemos más sobre cómo desplegar nuestra aplicación en HTTPS en el Capítulo 18.

Un proyecto de Django viene preconfigurado con el middleware que admite las mejores prácticas de seguridad.

El paquete de middleware de seguridad principal es `django.middleware.security.SecurityMiddleware`, que se encarga de configurar varios encabezados de seguridad importantes en las respuestas HTTP. Los encabezados se pueden configurar con variables en el archivo `settings.py` del proyecto.

Implementa la Seguridad de Transporte Estricta HTTP (*HTTP Strict Transport Security* o HSTS) para restringir las solicitudes únicamente a HTTPS. Cuando está en modo de desarrollo, `SECURE_HSTS_SECONDS` se establece en `0`, pero tras el despliegue en producción, si HTTPS está configurado, debe establecerse en un valor pequeño, como `60`.

El análisis de tipo de contenido (*content type sniffing*) es una técnica para determinar el tipo de contenido de un archivo examinando una parte de los datos que contiene. Puede ser una técnica útil en algunas situaciones de carga de archivos, pero también está asociada con algunas vulnerabilidades de seguridad. De forma predeterminada, `SECURE_CONTENT_TYPE_NOSNIFF` se establece en `True`. Esta es la opción más segura.

Django también incluye `django.middleware.clickjacking.XFrameOptionsMiddleware`. En su modo predeterminado, agregará este encabezado a las respuestas HTTP:

```http
X-Frame-Options: DENY
```

Esta configuración es necesaria para evitar ataques de *clickjacking* en un sitio web.

El *clickjacking* es un ataque que implica configurar un sitio web externo que envuelve el sitio web de destino con contenido HTML discreto y a menudo invisible. El sitio de destino se carga como un iframe y el sitio envoltorio de *clickjack* superpone opciones maliciosas sobre la pantalla del sitio de destino, de modo que el usuario no se da cuenta de que está haciendo clic en un sitio superpuesto externo.

Puedes leer más sobre los encabezados de seguridad en la documentación de Django en [https://docs.djangoproject.com/en/5.1/topics/security/](https://docs.djangoproject.com/en/5.1/topics/security/).

Con esta breve introducción a la seguridad de formularios completa, comenzaremos un ejercicio para construir un proyecto de Django con un formulario que podamos usar para demostrar algunos de los conceptos que hemos discutido.

#### Ejercicio 6.01 – Construcción de un formulario en HTML

Para los primeros ejercicios de este capítulo, necesitaremos un formulario HTML para realizar pruebas. En este ejercicio, agregaremos un formulario HTML a un proyecto de Django. Esto también te permitirá experimentar con cómo se validan y envían los diferentes campos. Esto se hará en un nuevo proyecto de Django para no interferir con Bookr. Puedes consultar el Capítulo 1 para refrescar tu memoria sobre la creación de un proyecto de Django:

1. Comenzaremos creando el nuevo proyecto de Django. Puedes reutilizar el entorno virtual de Bookr que ya tiene Django instalado. Abre una nueva terminal y activa el entorno virtual. Luego, usa `django-admin` para iniciar un proyecto de Django llamado `form_project`. Para hacer esto, ejecuta el siguiente comando:
   ```bash
   django-admin startproject form_project
   ```
   Esto estructurará el proyecto Django en un directorio llamado `form_project`.
2. Crea una nueva aplicación Django en este proyecto mediante el comando de gestión `startapp`. La aplicación debe llamarse `form_example`. Para hacer esto, entra al directorio `form_project`, luego ejecuta lo siguiente:
   ```bash
   python manage.py startapp form_example
   ```
   Esto creará el directorio de la aplicación `form_example` dentro del directorio `form_project`.
3. Inicia PyCharm, luego abre el directorio `form_project`. Si ya tienes un proyecto abierto, puedes hacerlo seleccionando **File | Open**; de lo contrario, simplemente haz clic en **Open** en la ventana Welcome to PyCharm. Navega hasta el directorio `form_project`, selecciónalo y luego haz clic en Open. Si se te solicita, asegúrate de elegir **Trust Project**.
   La ventana de `form_project` debería mostrarse así:
   *Figura 6.1: El proyecto form_project abierto*
4. Crea una nueva configuración de ejecución para ejecutar `manage.py runserver` para el proyecto seleccionando la opción **Edit Configuration** en el menú Current File en la parte superior de la ventana. Puedes reutilizar el entorno virtual que ya has creado. La ventana Run/Debug Configurations debería verse similar a la siguiente figura cuando hayas terminado:
   *Figura 6.2: Run/Debug Configurations para runserver*
   Puedes probar que la configuración se haya establecido correctamente haciendo clic en el botón **Run** y luego visitando `http://127.0.0.1:8000/` en tu navegador. Deberías ver la pantalla de bienvenida de Django. Si el servidor de depuración no se inicia o ves la página principal de Bookr, probablemente todavía tengas el proyecto Bookr en ejecución. Intenta detener el proceso `runserver` de Bookr y luego iniciar el nuevo que acabas de configurar.
5. Abre `settings.py` en el directorio `form_project` y agrega `'form_example'` a la configuración `INSTALLED_APPS`.
6. El último paso para configurar este nuevo proyecto es crear un directorio `templates` para la aplicación `form_example`. Haz clic con el botón derecho en el directorio `form_example`, luego selecciona **New | Directory**. Nómbralo `templates`.
7. Necesitamos una plantilla HTML para mostrar nuestro formulario. Podemos copiar el HTML para el ejemplo de formulario del repositorio Git visitando esta URL y guardando el archivo en el directorio `form_example` en la carpeta `Chapter06` en el repositorio de GitHub de este libro.
   El código de esta página HTML aparece en la Figura 6.3. Su atributo `method` está establecido en `post`. El atributo `action` se omite, lo que significa que el formulario se enviará de vuelta a la misma URL en la que se cargó:
   ```html
   <form method="post">
   ```
   Dentro de la etiqueta `form` hay varias etiquetas `<p>`, que envuelven las entradas del formulario y crean un espacio para darle al formulario un diseño visualmente agradable:
   *Figura 6.3: form-example.html*
8. Como ocurre con cualquier plantilla, no podemos verla a menos que tengamos una vista para renderizarla. Abre el archivo `views.py` de la aplicación `form_example` y agrega una nueva vista llamada `form_example`. Debe renderizar y devolver la plantilla que acabas de crear, de la siguiente manera:
   ```python
   def form_example(request):
       return render(request, "form-example.html")
   ```
   Ahora puedes guardar y cerrar `views.py`.
9. Ya deberías estar familiarizado con el siguiente paso, que es agregar un mapeo de URL a la vista. Abre el archivo `urls.py` en el directorio del paquete `form_project`. Agrega un mapeo para la ruta `form-example/` a tu vista `form_example`, a la variable `urlpatterns`. Debería verse así:
   ```python
   path('form-example/', form_example.views.form_example)
   ```
   Asegúrate de agregar también una importación de `form_example.views`. Guarda y cierra `urls.py`.
10. Inicia el servidor de desarrollo de Django (si aún no se está ejecutando), luego carga tu nueva vista en tu navegador web; la dirección es `http://127.0.0.1:8000/form-example/`. Tu página debería verse así:
    *Figura 6.4: Página de entradas de ejemplo*
11. Todavía no hemos configurado todo para el envío del formulario, por lo que si corriges todos los errores en el formulario e intentas enviarlo (haciendo clic en cualquiera de los botones de envío), recibirás un error que indica `CSRF verification failed`, como podemos ver en la siguiente figura. Esto demuestra la protección CSRF de Django, que discutimos en la sección anterior. Cuando recibas este error, simplemente regresa en tu navegador para volver a la página de ejemplo de entrada.
    *Figura 6.5: Error de verificación CSRF*

En este ejercicio, creaste una página de ejemplo que muestra muchas entradas HTML, luego creaste una vista para renderizarla y una URL para mapearla. Cargaste la página en tu navegador y experimentaste cambiando datos e intentando enviar el formulario cuando contenía errores.

Acabamos de ver un ejemplo de la protección CSRF de Django. En el siguiente ejercicio, solucionaremos el error del token CSRF y aprenderemos cómo maneja Django los envíos de formularios. Antes de llegar a eso, tengamos un breve repaso sobre cómo acceder a los datos en una solicitud mediante objetos `QueryDict`.

#### Acceso a los datos en la vista

Como comentamos en el Capítulo 1, Django proporciona dos objetos `QueryDict` en las instancias de `HTTPRequest` que se pasan a la función de vista. Estos son `request.GET`, que contiene los parámetros pasados en la URL, y `request.POST`, que contiene los parámetros en el cuerpo de la solicitud HTTP. Aunque `request.GET` tiene `GET` en su nombre; esta variable se llena incluso para solicitudes HTTP que no son GET. Esto se debe a que los datos que contiene se analizan a partir de la URL. Dado que todas las solicitudes HTTP tienen una URL, todas las solicitudes HTTP pueden contener datos GET, incluso si son POST o PUT, y así sucesivamente.

En el siguiente ejercicio, agregaremos código a nuestra vista para leer y mostrar los datos POST.

#### Ejercicio 6.02 – Trabajo con datos POST en una vista

Ahora agregaremos algo de código a nuestra vista de ejemplo para imprimir los datos POST recibidos en la consola. También insertaremos el método HTTP que se utilizó para generar la página en la salida HTML. Esto nos permitirá estar seguros de qué método se utilizó para generar la página (GET o POST) y ver cómo difiere el formulario para cada tipo:

1. Primero, en PyCharm, abre el archivo `views.py` de la aplicación `form_example`. Modifica la vista `form_example` para que imprima cada valor en el POST en la consola agregando este código dentro de la función:
   ```python
   for name in request.POST:
       print(f"{name}: {request.POST.getlist(name)}")
   ```
   Este código itera sobre cada clave en el objeto `QueryDict` de los datos de la solicitud POST e imprime la clave y la lista de valores en la consola. Ya sabemos que cada objeto `QueryDict` puede tener múltiples valores para una clave, por lo que usamos la función `getlist` para obtenerlos todos.
2. Pasa el `request.method` de la plantilla a una variable de contexto llamada `method`. Haz esto actualizando la llamada a `render` en la vista para que quede así:
   ```python
   return render(request, "form-example.html", {"method": request.method})
   ```
3. Ahora mostraremos la variable `method` en la plantilla. Abre la plantilla `form-example.html` y usa una etiqueta `<h4>` para mostrar la variable `method`. Coloca esto justo después de la etiqueta de apertura `<body>`, de la siguiente manera:
   ```html
   <body>
       <h4>Method: {{ method }}</h4>
   ```
   Ten en cuenta que podríamos acceder al método directamente dentro de la plantilla sin pasarlo en un diccionario de contexto, mediante el uso de la variable `request.method`. Sabemos por el Capítulo 3 (*Mapeo de URLs, Vistas y Plantillas*) que al usar la función de acceso directo `render`, la solicitud siempre está disponible en la plantilla. Solo demostramos cómo acceder al método en la vista aquí porque, más adelante, cambiaremos el comportamiento de la página según el método.
4. También debemos agregar el token CSRF al HTML del formulario. Hacemos esto colocando la etiqueta de plantilla `{% csrf_token %}` después de la etiqueta de apertura `<form>`. El inicio del formulario debería verse así:
   ```django
   <form method="post">
       {% csrf_token %}
   ```
   Ahora, guarda el archivo.
5. Inicia el servidor de desarrollo de Django si aún no se está ejecutando. Carga la página de ejemplo (`http://127.0.0.1:8000/form-example/`) en tu navegador; deberías ver que ahora muestra el método en la parte superior de la página (GET):
   *Figura 6.6: Método en la parte superior de la página*
6. Ingresa algo de texto o datos en cada una de las entradas y envía el formulario haciendo clic en el botón **Submit Input**:
   *Figura 6.7: Hacer clic en el botón Submit Input para enviar el formulario*
7. Deberías ver que la página se vuelve a cargar y la visualización del método cambia a POST:
   *Figura 6.8: Método actualizado a POST después de enviar el formulario*
8. Vuelve a PyCharm y mira en la consola de ejecución en la parte inferior de la ventana. Si no es visible, haz clic en el botón **Run** en la parte inferior de la ventana para mostrarlo:
   *Figura 6.9: Hacer clic en el botón Run en la parte inferior de la ventana para mostrar la consola*
   Si estás ejecutando el servidor de desarrollo en modo de depuración, haz clic en **Debug** en su lugar.
9. Dentro de la consola Run, debería mostrarse la lista de valores que se enviaron al servidor:
   *Figura 6.10: Valores de entrada mostrados en la consola Run*
   Aquí hay algunas cosas que debes tener en cuenta:
   - Todos los valores se envían como texto, incluso las entradas de número y fecha.
   - Para las entradas de selección (`select`), se envían los atributos de valor seleccionados de las opciones seleccionadas, no el contenido de texto de la etiqueta `option`.
   - Si seleccionas múltiples opciones para `books_you_own`, verás múltiples valores en la solicitud. Esta es la razón por la que usamos el método `getlist`, ya que se envían múltiples valores para el mismo nombre de entrada.
   - Si la casilla de verificación estaba marcada, tendrás una entrada `checkbox_on` en la salida de depuración. Si no estaba marcada, la clave no existirá en absoluto (es decir, no hay clave en lugar de que la clave exista con una cadena vacía o un valor `None`).
   - Tenemos un valor para el nombre `submit_input`, que es el texto `Submit Input`. Enviaste el formulario haciendo clic en el botón Submit Input, por lo que recibimos su valor. Observa que no se establece ningún valor para la entrada `button_element`, ya que no se hizo clic en ese botón.
10. Experimentaremos con otras dos formas de enviar el formulario: primero, escribiendo Enter cuando el cursor esté en una entrada similar a texto (Texto, Contraseña, Fecha o Correo electrónico, pero no Área de texto, ya que escribir Enter agregará una nueva línea). Si envías un formulario de esta manera, el formulario actuará como si hubieras hecho clic en el primer botón Enviar del formulario, por lo que se incluirá el valor de entrada `submit_input`. La salida que ves debe coincidir con la de la figura anterior.
11. La otra forma de enviar el formulario es haciendo clic en la entrada de envío Button Element, donde intentaremos hacer clic en este botón para enviar el formulario. Deberías ver que `submit_button` ya no está en la lista de valores publicados, mientras que `button_element` ahora está presente:
    *Figura 6.11: submit_button ahora no está en las entradas y se ha agregado button_element*
    Puedes usar esta técnica de envíos múltiples para alterar cómo se comporta tu vista, dependiendo de en qué botón se haya hecho clic. Incluso puedes tener múltiples botones de envío con el mismo atributo `name` para hacer que la lógica sea más fácil de escribir.

En este ejercicio, agregaste un token CSRF a tu elemento de formulario mediante el uso de la etiqueta de plantilla `{% csrf_token %}`. Esto significa que tu formulario se puede enviar a Django con éxito sin que genere una respuesta HTTP Permission Denied. Luego agregamos algo de código para generar los valores que contenía nuestro formulario cuando se envió. Intentamos enviar el formulario con varios valores para ver cómo se analizan en variables de Python en la parte `request.POST` de `QueryDict`. Ahora discutiremos algo más de la teoría sobre la diferencia entre las solicitudes GET y POST, y luego pasaremos a la biblioteca Django Forms, que facilita el diseño y la validación de formularios.

#### Elección entre GET y POST

Elegir cuándo usar una solicitud GET o POST requiere que consideres varios factores. Lo más importante es decidir si la solicitud debe ser idempotente. Se puede decir que una solicitud es **idempotente** si se puede repetir y producir el mismo resultado cada vez. En otras palabras, si ejecutarla varias veces siempre debe dejar las cosas igual, usa GET. Si repetirla pudiera cambiar datos o crear duplicados, usa POST. Veamos algunos ejemplos.

Si escribes cualquier dirección web en tu navegador (como cualquiera de las páginas de Bookr que hemos creado hasta ahora), realizará una solicitud GET para obtener la información. Puedes actualizar la página y, sin importar cuántas veces hagas clic en Actualizar, obtendrás los mismos datos de vuelta. La solicitud que estás realizando no afecta el contenido en el servidor. Dirías que estas solicitudes son idempotentes.

Ahora, recuerda cuando agregaste datos a través de la interfaz de administración de Django (en el Capítulo 4). Escribiste la información para el nuevo libro en un formulario y luego hiciste clic en Guardar. Tu navegador realizó una solicitud POST para crear un nuevo libro en el servidor. Si repitieras esa solicitud POST, el servidor crearía otro libro y lo haría cada vez que repitieras la solicitud. Dado que la solicitud está actualizando información, no es idempotente. Tu navegador te advertirá sobre esto. Si alguna vez intentaste actualizar una página a la que te enviaron después de enviar un formulario, es posible que hayas recibido un mensaje preguntándote si deseas "reenviar los datos del formulario" (o algo más detallado, como se muestra en la siguiente figura). Esta es una advertencia de que estás enviando los datos del formulario nuevamente, lo que podría hacer que la acción que acabas de emprender se repita:

*Figura 6.12: Firefox confirmando si se debe reenviar la información*

Esto no sugiere que todas las solicitudes GET sean idempotentes y todas las solicitudes POST no lo sean: tu aplicación backend se puede diseñar de la forma que desees. Aunque no es una buena práctica, un desarrollador podría haber decidido hacer que los datos se actualicen durante una solicitud GET en su aplicación web. Cuando estés creando tus aplicaciones, debes intentar asegurarte de que las solicitudes GET sean idempotentes y dejar la alteración de datos únicamente a las solicitudes POST. Cíñete a estos principios a menos que tengas una buena razón para no hacerlo.

Otro punto a considerar es que Django solo aplica la protección CSRF a las solicitudes POST. Se puede acceder a cualquier solicitud GET, incluida una que altere datos, sin un token CSRF.

A veces, puede ser difícil decidir si una solicitud es idempotente o no; por ejemplo, un formulario de inicio de sesión. Antes de enviar tu nombre de usuario y contraseña, no habías iniciado sesión, y después, el servidor consideró que habías iniciado sesión, por lo que ¿podríamos considerar que no es idempotente, ya que cambió tu estado de autenticación con el servidor? Por otro lado, una vez que hayas iniciado sesión, si pudieras enviar tus credenciales nuevamente, seguirías conectado. Esto implica que la solicitud es idempotente y repetible. Entonces, ¿la solicitud debería ser GET o POST? Esto nos lleva al segundo punto a considerar al elegir qué método usar: la visibilidad de los datos del formulario.

Si envías datos de formulario con una solicitud GET, los parámetros del formulario serán visibles en la URL. Por ejemplo, si creáramos un formulario de inicio de sesión mediante una solicitud GET, la URL de inicio de sesión podría ser `https://www.example.com/login?username=user&password=password1`. El nombre de usuario y, lo que es peor, la contraseña, son visibles en la barra de direcciones del navegador web. También se almacenaría en el historial del navegador, por lo que cualquiera que usara el navegador después del usuario real podría iniciar sesión en el sitio. La URL a menudo también se almacena en los archivos de registro del servidor web, lo que significa que las credenciales también estarían visibles allí. En resumen, independientemente de la idempotencia de una solicitud, no pases datos confidenciales a través de parámetros de URL.

A veces, saber que el parámetro será visible en la URL puede ser algo que desees. Por ejemplo, al realizar búsquedas con un motor de búsqueda, el parámetro de búsqueda será visible en la URL. Para ver esto en acción, intenta visitar `https://www.google.com` y buscar algo. Notarás que la página con los resultados tiene tu término de búsqueda como el parámetro `q`. Por ejemplo, una búsqueda de Django te llevará a la URL `https://www.google.com/search?q=Django`. Esto te permite compartir los resultados de la búsqueda con otra persona enviándole esta URL. En la Actividad 6.01, agregarás un formulario de búsqueda que pasa un parámetro de manera similar.

Otra consideración es que la longitud máxima de una URL permitida por un navegador puede ser corta en comparación con el tamaño de un cuerpo POST: a veces, solo alrededor de 2000 caracteres (o aproximadamente 2 KB) en comparación con muchos megabytes o gigabytes que puede tener un cuerpo POST (asumiendo que tu servidor esté configurado para permitir solicitudes de este tamaño).

Como mencionamos anteriormente, los parámetros de URL están disponibles en `request.GET`, independientemente del tipo de solicitud que se realice (GET, POST, PUT, etc.). Puede que te resulte útil enviar algunos datos en los parámetros de la URL y otros en el cuerpo de la solicitud (disponibles en `request.POST`). Por ejemplo, podrías especificar un argumento de formato en la URL que establezca a qué formato se transformarán algunos datos de salida, pero los datos de entrada se proporcionan en el cuerpo del POST.

Establecer parámetros en la cadena de consulta GET puede parecer redundante cuando los valores se pueden pasar como parte de la ruta de la URL y luego ser analizados por Django para enrutar la URL. A continuación, veremos algunas razones por las que preferirías usar la cadena de consulta (*query string*).

#### ¿Por qué usar GET cuando podemos poner parámetros en los mapeos de URL?

Django nos permite definir fácilmente mapas de URL que contienen variables. Podríamos, por ejemplo, configurar un mapeo de URL para una vista de búsqueda como esta:

```python
path("/search/<str:search>/", reviews.views.search)
```

Esto probablemente parece un buen enfoque al principio, pero cuando queremos comenzar a personalizar la vista de resultados con argumentos, puede complicarse rápidamente. Por ejemplo, es posible que deseemos poder pasar de una página de resultados a la siguiente, por lo que agregaremos un argumento `page`:

```python
path("/search/<str:search>/<int:page>", reviews.views.search)
```

Y luego, es posible que también queramos ordenar los resultados de la búsqueda por una categoría específica, como el nombre del autor o la fecha de publicación, por lo que agregaremos otro argumento para eso:

```python
path("/search/<str:search>/<int:page>/<str:order >", reviews.views.search)
```

Es posible que puedas ver el problema con este enfoque: no podemos ordenar los resultados sin proporcionar una página. Si quisiéramos agregar también un argumento `results_per_page`, no podríamos usarlo sin establecer una clave de página y de orden.

Contrasta esto con el uso de parámetros de consulta: todos ellos son opcionales, por lo que podrías buscar de esta manera:

```text
?search=search+term:
```

O podrías establecer una página como esta:

```text
?search=search+term&page=2
```

O simplemente establecer el orden de los resultados como este:

```text
?search=search+term&order=author
```

O podrías combinarlos todos:

```text
?search=search+term&page=2&order=author
```

Otra razón para usar parámetros de consulta de URL es que, al enviar un formulario, el navegador siempre envía los valores de entrada de esta manera: no se puede cambiar para que los parámetros se envíen como componentes de ruta en la URL. Por lo tanto, al enviar un formulario mediante GET, los parámetros de consulta de URL deben usarse como datos de entrada.

Ahora que hemos presentado los fundamentos de los formularios en solicitudes HTTP y HTML, veamos la biblioteca Django Forms, que hace que trabajar con formularios y solicitudes sea mucho más fácil.

---

### Sección: La biblioteca Django Forms

Hemos visto cómo escribir formularios manualmente en HTML y cómo acceder a los datos en el objeto de solicitud mediante `QueryDict`. Vimos que el navegador nos proporciona cierta validación para ciertos tipos de campos (como correo electrónico o números), pero no hemos intentado validar los datos en la vista de Python. Deberíamos validar el formulario en la vista de Python por dos razones:
1. No es seguro confiar únicamente en la validación basada en el navegador de los datos de entrada. Es posible que un navegador no implemente ciertas funciones de validación, lo que significa que el usuario podría enviar cualquier tipo de datos. Por ejemplo, los navegadores más antiguos no validan los campos numéricos, por lo que un usuario puede escribir un número fuera del rango que esperamos. Además, un usuario malicioso podría intentar enviar datos dañinos sin utilizar un navegador en absoluto. La validación del navegador debe considerarse una cortesía para el usuario, y nada más.
2. El navegador no nos permite realizar una validación entre campos cruzados (*cross-field validation*). Por ejemplo, podemos usar el atributo `required` para las entradas que son obligatorias. A menudo, sin embargo, queremos establecer el atributo `required` en función del valor de otra entrada. Por ejemplo, la entrada de la dirección de correo electrónico solo debe establecerse como obligatoria si el usuario ha marcado la casilla de verificación Registrar mi correo electrónico.

La biblioteca Django Forms te permite definir rápidamente un formulario mediante una clase de Python. Esto se hace creando una subclase de la clase base `django.forms.Form`. Luego puedes usar una instancia de esta clase para renderizar el formulario en tu plantilla y validar los datos de entrada. Nos referimos a nuestras clases como formularios, similar a cómo subclasificamos los modelos de Django para crear clases `Model`. Los formularios contienen uno o más campos de un tipo determinado (como campos de texto, campos numéricos o campos de correo electrónico). Notarás que esto suena como los modelos de Django, y los formularios son similares a los modelos pero usan diferentes clases de campo. Incluso puedes crear automáticamente un formulario a partir de un modelo; cubriremos esto en el Capítulo 7.

#### Definición de un formulario

Crear un formulario de Django es similar a crear un modelo de Django: defines una clase que hereda de la clase `django.forms.Form`. La clase tiene atributos que son instancias de diferentes subclases de `django.forms.Field`. Cuando se renderiza, el nombre del atributo en la clase corresponde a su nombre de entrada (`name`) en HTML. Para darte una idea rápida de qué campos existen, algunos ejemplos son `CharField`, `IntegerField`, `BooleanField`, `ChoiceField` y `DateField`. Cada campo generalmente corresponde a una entrada cuando se renderiza en HTML, pero no siempre hay un mapeo uno a uno entre una clase de campo de formulario y un tipo de entrada. Los campos de formulario están más acoplados al tipo de datos que recopilan que a cómo se muestran.

Para ilustrar esto, considera una entrada de texto y una entrada de contraseña. Ambos aceptan algunos datos de texto escritos, pero la principal diferencia entre ellos es que el texto se muestra en una entrada de texto, mientras que en una entrada de contraseña, el texto se oculta. En un formulario de Django, ambos campos se representan mediante `CharField`. La diferencia en cómo se muestran se establece cambiando el widget que utiliza el campo.

Si no estás familiarizado con la palabra **widget**, es un término para describir la entrada real con la que se interactúa y cómo se muestra. Las entradas de texto, las entradas de contraseña, los menús de selección, las casillas de verificación y los botones son ejemplos de diferentes widgets. Las entradas que hemos visto en HTML se corresponden uno a uno con los widgets. En Django, este no es el caso, y el mismo tipo de clase `Field` se puede renderizar de múltiples maneras, dependiendo del widget que se especifique.

Django define varias clases de widgets que definen cómo se debe renderizar un campo como HTML. Heredan de `django.forms.widgets.Widget`. Se puede pasar un widget al constructor de `Field` para cambiar la forma en que se renderiza. Por ejemplo, `CharField` se renderiza como `<input type="text">` de forma predeterminada. Si usamos el widget `PasswordInput`, se renderizará como `<input type="password">`. Los otros widgets que usaremos son los siguientes:
- `RadioSelect`, que renderiza `ChoiceField` como botones de opción (*radio buttons*) en lugar de un menú `<select>`
- `Textarea`, que renderiza `CharField` como `<textarea>`
- `HiddenInput`, que renderiza un campo como una entrada oculta

Veremos un formulario de ejemplo y agregaremos campos y características uno por uno. Primero, creemos un archivo `forms.py` que contenga un formulario con una entrada de texto y una entrada de contraseña:

```python
from django import forms

class ExampleForm(forms.Form):
    text_input = forms.CharField()
    password_input = forms.CharField(
        widget=forms.PasswordInput
    )
```

El argumento `widget` puede ser simplemente una subclase de widget, lo cual puede estar bien muchas veces. Si deseas personalizar aún más la visualización de la entrada y sus atributos, puedes establecer el argumento `widget` en una instancia de la clase de widget. Pronto veremos cómo personalizar aún más las pantallas de los widgets. En este caso, estamos utilizando solo la clase `PasswordInput`, ya que no estamos personalizando más allá de cambiar el tipo de entrada que se muestra.

Para proporcionar un formulario al usuario, debe ponerse a disposición de la plantilla a través del contexto de una vista. He aquí un ejemplo:

```python
from django.shortcuts import render
from .forms import ExampleForm

def example_view(request):
    form = ExampleForm()  # formulario vacío para GET
    return render(request, "example.html", {"form": form})
```

Cuando el formulario se renderiza en una plantilla, se ve así:

*Figura 6.13: Formulario de Django renderizado en el navegador*

Ten en cuenta que las entradas no contienen ningún contenido cuando se carga la página; el texto se ha ingresado para ilustrar los diferentes tipos de entrada.

Si examinamos el código fuente de la página, veremos el HTML que genera Django. Para los dos primeros campos, se ve así (se ha agregado algo de espaciado para facilitar la lectura):

```html
<p>
    <label for="id_text_input">Text input:</label>
    <input type="text" name="text_input" required id="id_text_input">
</p>
<p>
    <label for="id_password_input">Password input:</label>
    <input type="password" name="password_input" required id="id_password_input">
</p>
```

Observa que Django ha generado automáticamente una etiqueta (`<label>`) con texto derivado del nombre del campo. Los atributos `name` e `id` se han establecido automáticamente. Django también agrega automáticamente el atributo `required` a la entrada. Al igual que los campos del modelo, los constructores de campos de formulario también aceptan un argumento `required`; este valor predeterminado es `True`. Establecer esto en `False` elimina el atributo `required` del HTML generado.

A continuación, veremos cómo se agrega una casilla de verificación al formulario.

Una casilla de verificación se representa con `BooleanField`, ya que solo puede tener dos valores: marcada o desmarcada. Se agrega al formulario de la misma manera que el otro campo:

```python
class ExampleForm(forms.Form):
    …
    checkbox_on = forms.BooleanField()
```

El HTML que genera Django para este nuevo campo es similar a los dos campos anteriores:

```html
<label for="id_checkbox_on">Checkbox on:</label>
<input type="checkbox" name="checkbox_on" required id="id_checkbox_on">
```

A continuación se muestran las entradas de selección:
Debemos proporcionar una lista de opciones para mostrar en el menú desplegable `<select>`.
El constructor de la clase de campo toma un argumento `choices`. Las opciones se proporcionan como una tupla de tuplas de dos elementos. El primer elemento de cada subtupla es el valor de la opción, mientras que el segundo elemento es el texto o la descripción de la opción. Por ejemplo, las opciones podrían definirse de la siguiente manera:

```python
BOOK_CHOICES = (
    ("1", "Deep Learning with Keras"),
    ("2", "Web Development with Django"),
    ("3", "Brave New World"),
    ("4", "The Great Gatsby")
)
```

Ten en cuenta que puedes usar listas en lugar de tuplas si lo deseas (o una combinación de las dos). Esto puede ser útil si deseas que tus opciones sean mutables:

```python
BOOK_CHOICES = [
    ["1", "Deep Learning with Keras"],
    ["2", "Web Development with Django"],
    ["3", "Brave New World"],
    ["4", "The Great Gatsby"]
]
```

Para implementar `optgroup`, podemos anidar las opciones. Para implementar las opciones como lo hicimos en nuestros ejemplos anteriores, podemos usar una estructura como esta:

```python
BOOK_CHOICES = (
    (
        "Non-Fiction",
        (
            ("1", "Deep Learning with Keras"),
            ("2", "Web Development with Django")
        )
    ),
    (
        "Fiction",
        (
            ("3", "Brave New World"),
            ("4", "The Great Gatsby")
        )
    )
)
```

El elemento de selección se agrega al formulario mediante `ChoiceField`. El widget tiene como valor predeterminado una entrada de selección, por lo que no es necesaria ninguna configuración aparte de configurar `choices`:

```python
class ExampleForm(forms.Form):
    …
    favorite_book = forms.ChoiceField(choices=BOOK_CHOICES)
```

Este es el HTML que se genera:

```html
<label for="id_favorite_book">Favorite book:</label>
<select name="favorite_book" id="id_favorite_book">
    <optgroup label="Non-Fiction">
        <option value="1">Deep Learning with Keras </option>
        <option value="2">Web Development with Django </option>
    </optgroup>
    <optgroup label="Fiction">
        <option value="3">Brave New World</option>
        <option value="4">The Great Gatsby</option>
    </optgroup>
</select>
```

Hacer una selección múltiple requiere el uso de `MultipleChoiceField`. Toma un argumento `choices` en el mismo formato que el `ChoiceField` normal para selecciones individuales:

```python
class ExampleForm(forms.Form):
    …
    books_you_own = forms.MultipleChoiceField(choices=BOOK_CHOICES)
```

Y su HTML es similar al de la selección simple, excepto que tiene agregado el atributo `multiple`:

```html
<label for="id_books_you_own">Books you own:</label>
<select name="books_you_own" required id="id_books_you_own" multiple>
    <optgroup label="Non-Fiction">
        <option value="1">Deep Learning with Keras </option>
        <option value="2">Web Development with Django </option>
    </optgroup>
    <optgroup label="Fiction">
        <option value="3">Brave New World</option>
        <option value="4">The Great Gatsby</option>
    </optgroup>
</select>
```

Las opciones también se pueden configurar después de que se haya creado una instancia del formulario. Es posible que desees generar la opción de lista/tupla dentro de tu vista dinámicamente y luego asignarla al atributo `choices` del campo, de la siguiente manera:

```python
form = ExampleForm()
form.fields["books_you_own"].choices = [("1", "Deep Learning with Keras"), …]
```

A continuación se muestran las entradas de radio, que son similares a las de selección:
Al igual que las selecciones, las entradas de radio usan `ChoiceField`, ya que proporcionan una opción única entre múltiples alternativas.
Las opciones entre las cuales elegir se pasan al constructor del campo con el argumento `choices`.
Las opciones se proporcionan como una tupla de tuplas de dos elementos, también como las selecciones:

```python
choices = (
    ("1", "Option One"),
    ("2", "Option Two"),
    ("3", "Option Three")
)
```

`ChoiceField` se muestra de forma predeterminada como una entrada de selección, por lo que el widget debe establecerse en `RadioSelect` para que se represente como botones de opción. Poniendo la configuración de opciones junto con esto, podemos agregar botones de opción al formulario de esta manera:

```python
RADIO_CHOICES = (
    ("Value One", "Value One"),
    ("Value Two", "Value Two"),
    ("Value Three", "Value Three")
)

class ExampleForm(forms.Form):
    …
    radio_input = forms.ChoiceField(choices=RADIO_CHOICES, widget=forms.RadioSelect)
```

Este es el HTML que se genera:

```html
<label for="id_radio_input_0">Radio input:</label>
<ul id="id_radio_input">
    <li>
        <label for="id_radio_input_0">
            <input type="radio" name="radio_input" value="Value One" required id="id_radio_input_0">
            Value One
        </label>
    </li>
    <li>
        <label for="id_radio_input_1">
            <input type="radio" name="radio_input" value="Value Two" required id="id_radio_input_1">
            Value Two
        </label>
    </li>
    <li>
        <label for="id_radio_input_2">
            <input type="radio" name="radio_input" value="Value Three" required id="id_radio_input_2">
            Value Three
        </label>
    </li>
</ul>
```

Django genera automáticamente una etiqueta y un ID únicos para cada uno de los tres botones de opción.

Para crear un área de texto, usa `CharField` con un widget `Textarea`:

```python
class ExampleForm(forms.Form):
    …
    text_area = forms.CharField(widget=forms.Textarea)
```

Es posible que notes que esta área de texto es mucho más grande que las anteriores que hemos visto (consulta la siguiente figura):

*Figura 6.14: Área de texto normal (arriba) versus área de texto predeterminada de Django (abajo)*

Esto se debe a que Django agrega automáticamente los atributos `cols` y `rows`. Estos establecen la cantidad de columnas y filas que muestra el campo de texto, respectivamente:

```html
<label for="id_text_area">Text area:</label>
<textarea name="text_area" cols="40" rows="10" required id="id_text_area"></textarea>
```

Ten en cuenta que las configuraciones de `cols` y `rows` no afectan la cantidad de texto que se puede ingresar en un campo, solo la cantidad que se muestra a la vez. Además, ten en cuenta que el tamaño de `textarea` se puede establecer mediante CSS (por ejemplo, las propiedades `height` y `width`). Esto anulará las configuraciones de `cols` y `rows`.

Para crear entradas numéricas, podrías esperar que Django tenga un tipo `NumberField`, pero no es así.
Recuerda que los campos de formulario de Django están centrados en los datos y no en la visualización, por lo que Django proporciona diferentes clases de `Field`, según el tipo de datos numéricos que desees almacenar:
- Para números enteros, usa `IntegerField`.
- Para números de coma flotante, usa `FloatField` o `DecimalField`. Estos dos últimos difieren en cómo convierten sus datos en un valor de Python. `FloatField` se convertirá en un `float`, mientras que `DecimalField` es un `Decimal`.

Los valores decimales ofrecen una mayor precisión al representar números que los valores flotantes, pero es posible que no se integren bien con tu código Python existente.

Agregaremos los tres campos al formulario a la vez:

```python
class ExampleForm(forms.Form):
    …
    integer_input = forms.IntegerField()
    float_input = forms.FloatField()
    decimal_input = forms.DecimalField()
```

Aquí está el HTML para los tres:

```html
<p>
    <label for="id_integer_input">Integer input:</label>
    <input type="number" name="integer_input" required id="id_integer_input">
</p>
<p>
    <label for="id_float_input">Float input:</label>
    <input type="number" name="float_input" step="any" required id="id_float_input">
</p>
<p>
    <label for="id_decimal_input">Decimal input:</label>
    <input type="number" name="decimal_input" step="any" required id="id_decimal_input">
</p>
```

Al HTML generado de `IntegerField` le falta el atributo `step` que tienen los otros dos, lo que significa que el widget solo aceptará valores enteros.

Los otros dos campos (`FloatField` y `DecimalField`) generan un HTML muy similar; su comportamiento es el mismo en el navegador y solo difieren cuando sus valores se utilizan en el código de Django.

Como habrás adivinado, se puede crear una entrada de correo electrónico con `EmailField`:

```python
class ExampleForm(forms.Form):
    …
    email_input = forms.EmailField()
```

Su HTML es similar a la entrada de correo electrónico que creamos manualmente:

```html
<label for="id_email_input">Email input:</label>
<input type="email" name="email_input" required id="id_email_input">
```

Continuando con nuestro formulario creado manualmente, el siguiente campo que veremos es `DateField`:
De forma predeterminada, Django representará `DateField` como una entrada de texto y el navegador no mostrará una ventana emergente de calendario cuando se haga clic en el campo. Podemos agregar `DateField` al formulario sin argumentos, de esta manera:

```python
class ExampleForm(forms.Form):
    …
    date_input = forms.DateField()
```

Cuando se renderiza, parece una entrada de texto normal:

*Figura 6.15: Visualización predeterminada de DateField en el formulario*

Este es el HTML generado de forma predeterminada:

```html
<label for="id_date_input">Date input:</label>
<input type="text" name="date_input" required id="id_date_input">
```

La razón para usar una entrada de texto es que permite al usuario ingresar la fecha en varios formatos diferentes. Por ejemplo, de forma predeterminada, el usuario puede escribir la fecha en formatos Año-Mes-Día (separados por guiones) o Mes/Día/Año (separados por barras).

Los formatos aceptados se pueden especificar pasando una lista de formatos al constructor `DateField` mediante el argumento `input_formats`. Por ejemplo, podríamos aceptar fechas en formato Día/Mes/Año o Día/Mes/Año-con-siglo, de la siguiente manera:

```python
DateField(input_formats=["%d/m/%y", "%d/%m/%Y"])
```

Podemos anular cualquier atributo en el widget de un campo pasando el argumento `attrs` al constructor del widget. Esto acepta un diccionario de claves/valores de atributos que se renderizarán en el HTML de la entrada.

Aún no hemos usado esto, pero lo veremos nuevamente en el próximo capítulo cuando personalicemos más el renderizado del campo. Por ahora, solo estableceremos un atributo, `type`, que sobrescribirá el tipo de entrada predeterminado:

```python
class ExampleForm(forms.Form):
    …
    date_input = forms.DateField(
        widget=forms.DateInput(
            attrs={"type": "date"}))
```

Cuando se renderiza, ahora se parece al campo de fecha que teníamos antes, y hacer clic en él abre el selector de fecha del calendario:

*Figura 6.16: DateField con entrada de fecha*

Al examinar el HTML generado ahora, podemos ver que usa el tipo `date`:

```html
<label for="id_date_input">Date input:</label>
<input type="date" name="date_input" required id="id_date_input">
```

La última entrada que nos falta es la entrada oculta:
Una vez más, debido a la naturaleza centrada en los datos de los formularios de Django, no existe `HiddenField`.
En su lugar, elegimos el tipo de campo que debe ocultarse y configuramos su widget en `HiddenInput`. Luego podemos establecer el valor del campo utilizando el argumento `initial` del constructor del campo:

```python
class ExampleForm(forms.Form):
    …
    hidden_input = forms.CharField(
        widget=forms.HiddenInput,
        initial="Hidden Value")
```

Este es el HTML generado:

```html
<input type="hidden" name="hidden_input" value="Hidden Value" id="id_hidden_input">
```

Ten en cuenta que, como se trata de una entrada oculta, Django no genera una etiqueta ni elementos `<p>` circundantes.

Hay otros campos de formulario que proporciona Django que funcionan de manera similar. Estos van desde `DateTimeField` (para capturar una fecha y una hora) hasta `GenericIPAddressField` (para direcciones IPv4 o IPv6) y `URLField` (para URLs). Una lista completa de campos está disponible en [https://docs.djangoproject.com/en/5.1/ref/forms/fields/](https://docs.djangoproject.com/en/5.1/ref/forms/fields/).

#### Renderizado de un formulario en una plantilla

Ahora hemos visto cómo crear un formulario y agregar campos, y hemos visto cómo se ve el formulario y qué HTML se genera. Pero, ¿cómo se renderiza el formulario en la plantilla? Simplemente creamos una instancia de la clase de formulario y la pasamos a la función `render` en una vista, utilizando el contexto, al igual que cualquier otra variable.

Por ejemplo, así es como podemos pasar `ExampleForm` a una plantilla:

```python
def view_function(request):
    form = ExampleForm()
    return render(request, "template.html", {"form": form})
```

Django no agrega el elemento `<form>` ni los botones de envío al renderizar la plantilla; debes agregarlos alrededor de donde se coloca el formulario en la plantilla. El formulario se puede renderizar como cualquier otra variable.

Mencionamos brevemente antes que el formulario se estaba renderizando en la plantilla utilizando el método `as_p`. Se eligió este método de diseño porque se asemeja más al formulario de ejemplo que creamos manualmente. Django ofrece cuatro métodos de diseño que se pueden utilizar:

- **as_table**: El formulario se representa como filas de tabla, con cada entrada en su propia fila. Django no genera el elemento de tabla circundante, por lo que debes envolver el formulario tú mismo. He aquí un ejemplo:
  ```django
  <form method="post">
      <table>
          {{ form.as_table }}
      </table>
  </form>
  ```
  `as_table` es el método de representación predeterminado, por lo que `{{ form.as_table }}` y `{{ form }}` son equivalentes.
  Cuando se renderiza, el formulario se ve así:
  *Figura 6.17: Formulario renderizado como una tabla*
  Aquí hay una pequeña muestra del HTML que se genera:
  ```html
  <tr>
      <th>
          <label for="id_text_input">Text input:</label>
      </th>
      <td>
          <input type="text" name="text_input" required id="id_text_input">
      </td>
  </tr>
  <tr>
      <th>
          <label for="id_password_input">Password input:</label>
      </th>
      <td>
          <input type="password" name="password_input" required id="id_password_input">
      </td>
  </tr>
  ```

- **as_ul**: Esto renderiza los campos del formulario como elementos de lista (`<li>`) dentro de un elemento `<ul>` u `<ol>`. Al igual que con `as_table`, el elemento contenedor (`<ul>` u `<ol>`) no lo crea Django y debes agregarlo tú:
  ```django
  <form method="post">
      <ul>
          {{ form.as_ul }}
      </ul>
  </form>
  ```
  Así es como se renderiza el formulario usando `as_ul`:
  *Figura 6.18: Formulario renderizado usando as_ul*
  Y aquí hay una muestra del HTML generado:
  ```html
  <li>
      <label for="id_text_input">Text input:</label>
      <input type="text" name="text_input" required id="id_text_input">
  </li>
  <li>
      <label for="id_password_input">Password input:</label>
      <input type="password" name="password_input" required id="id_password_input">
  </li>
  ```

- **as_p**: Luego está el método `as_p`, que usamos en nuestros ejemplos anteriores. Cada entrada se envuelve dentro de etiquetas `<p>`, lo que significa que no tienes que envolver el formulario manualmente (en `<table>` o `<ul>`) como lo hiciste con los métodos anteriores:
  ```django
  <form method="post">
      {{ form.as_p }}
  </form>
  ```
  Así es como se ve el formulario renderizado:
  *Figura 6.19: Formulario renderizado usando as_p*
  Ya has visto esto antes, pero una vez más, aquí hay una muestra del HTML generado:
  ```html
  <p>
      <label for="id_text_input">Text input:</label>
      <input type="text" name="text_input" required id="id_text_input">
  </p>
  <p>
      <label for="id_password_input">Password input:</label>
      <input type="password" name="password_input" required id="id_password_input">
  </p>
  ```

- **as_field_group**: Django también ofrece un control de diseño más detallado para que los elementos de formulario individuales se puedan disponer en un diseño personalizado sin restringirlos a las estructuras `table`, `ul` o `p`. `as_field_group` es un método del objeto de campo en lugar del objeto de formulario. Se puede llamar desde los campos individuales en el formulario de la siguiente manera:
  ```django
  <form method="post">
      {{ form.field.text_input.as_field_group }}
      <br />
      {{ form.field.password_input.as_field_group }}
      <br />
  </form>
  ```
  Esto se renderizará de la siguiente manera:
  ```html
  <form method="post">
      <label for="id_text_input">Text input:</label>
      <input type="text" name="text_input" required id="id_text_input">
      <br />
      <label for="id_password_input">Password input:</label>
      <input type="password" name="password_input" required id="id_password_input">
      <br />
  </form>
  ```
  Si deseas generar un formulario con campos separados por un elemento `div`, puedes iterar a través de los campos del formulario de esta manera en la plantilla:
  ```django
  <form method="post">
      {% csrf_token %}
      {% for field in form %}
          <div class="field_wrapper" id="field_group_{{field.name}}">
              {{ field.as_field_group }}
          </div>
      {% endfor %}
  </form>
  ```

Depende de ti decidir qué método deseas utilizar para renderizar tu formulario, según cuál se adapte mejor a tu aplicación. En términos de su comportamiento y uso en tu vista, todos son idénticos. En el Capítulo 15, también presentaremos un método de representación de formularios que utilizará las clases CSS de Bootstrap.

Ahora que te has familiarizado con los formularios de Django, podemos actualizar nuestra página de formulario de ejemplo para usar un formulario de Django en lugar de escribir manualmente todo el HTML. Haremos esto en el próximo ejercicio reemplazando el HTML escrito a mano con un formulario de Django.

#### Ejercicio 6.03 – Construcción y renderizado de un formulario de Django

En este ejercicio, crearás un formulario de Django utilizando todos los campos que hemos visto. El formulario y la vista se comportarán de manera similar al formulario creado manualmente; sin embargo, podrás ver cuánto menos código se requiere al escribir formularios utilizando Django. Tu formulario también obtendrá automáticamente la validación de campos y, si hacemos cambios en el formulario, no tendremos que hacer cambios en el HTML, ya que se actualizará dinámicamente según la definición del formulario:

1. En PyCharm, crea un nuevo archivo llamado `forms.py` dentro del directorio de la aplicación `form_example`.
2. Importa la biblioteca de formularios de Django en la parte superior de tu archivo `forms.py`:
   ```python
   from django import forms
   ```
3. Define las opciones para los botones de opción creando una variable `RADIO_CHOICES`. Llégala de la siguiente manera:
   ```python
   RADIO_CHOICES = (
       ("Value One", "Value One Display"),
       ("Value Two", "Text For Value Two"),
       ("Value Three", "Value Three's Display Text"),
   )
   ```
   Utilizarás esto pronto cuando crees un `ChoiceField` llamado `radio_input`.
4. Define las opciones anidadas para el libro y las entradas de selección creando una variable `BOOK_CHOICES`. Llégala de la siguiente manera:
   ```python
   BOOK_CHOICES = (
       (
           "Non-Fiction",
           (
               ("1", "Deep Learning with Keras"),
               ("2", "Web Development with Django")
           )
       ),
       (
           "Fiction",
           (
               ("3", "Brave New World"),
               ("4", "The Great Gatsby")
           )
       )
   )
   ```
5. Crea una clase llamada `ExampleForm`, que hereda de la clase `forms.Form`:
   ```python
   class ExampleForm(forms.Form):
   ```
6. Agrega los siguientes campos como atributos a la clase:
   ```python
   text_input = forms.CharField()
   password_input = forms.CharField(
       widget=forms.PasswordInput)
   checkbox_on = forms.BooleanField()
   radio_input = forms.ChoiceField(
       choices=RADIO_CHOICES,
       widget=forms.RadioSelect)
   favorite_book = forms.ChoiceField(
       choices=BOOK_CHOICES)
   books_you_own = forms.MultipleChoiceField(
       choices=BOOK_CHOICES)
   text_area = forms.CharField(widget=forms.Textarea)
   integer_input = forms.IntegerField()
   float_input = forms.FloatField()
   decimal_input = forms.DecimalField()
   email_input = forms.EmailField()
   date_input = forms.DateField(
       widget=forms.DateInput(
           attrs={"type": "date"}))
   hidden_input = forms.CharField(
       widget=forms.HiddenInput,
       initial="Hidden Value")
   ```
7. Guarda el archivo.
8. Abre el archivo `views.py` de tu aplicación `form_example`. En la parte superior del archivo, agrega una línea para importar `ExampleForm` desde tu archivo `forms.py`:
   ```python
   from .forms import ExampleForm
   ```
9. Dentro de la vista `form_example`, crea una instancia de la clase `ExampleForm` y asígnala a la variable `form`:
   ```python
   form = ExampleForm()
   ```
10. Agrega la variable `form` al diccionario de contexto usando la clave `form`. La línea `return` debería verse así:
    ```python
    return render(request, "form-example.html", {"method": request.method, "form": form})
    ```
11. Guarda el archivo. Asegúrate de no haber eliminado el código que imprime los datos que ha enviado el formulario, ya que lo usaremos nuevamente más adelante en este ejercicio.
12. Abre el archivo `form-example.html` dentro del directorio `templates` de la aplicación `form_example`. Puedes eliminar casi todo el contenido del elemento `form`, excepto la etiqueta de plantilla `{% csrf_token %}` y los botones de envío. Agrega una representación de la variable `form` utilizando el método `as_p` después de la etiqueta de plantilla `{% csrf_token %}`. Todo el elemento `form` ahora debería verse así:
    ```django
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <p>
            <input type="submit" name="submit_input" value="Submit Input">
        </p>
        <p>
            <button type="submit" name="button_element" value="Button Element">
                Button With <strong>Styled</strong> Text
            </button>
        </p>
    </form>
    ```
13. Inicia el servidor de desarrollo de Django si aún no se está ejecutando, luego visita la página de ejemplo del formulario en tu navegador, en `http://127.0.0.1:8000/form-example/`. Debería verse de la siguiente manera:
    *Figura 6.20: Formulario de ejemplo de Django renderizado en el navegador*
14. Ingresa algunos datos en el formulario; dado que Django marca todos los campos como obligatorios, deberás ingresar texto o seleccionar valores para todos los campos, asegurándote también de que la casilla de verificación esté marcada. Envía el formulario.
15. Vuelve a PyCharm y mira en la consola de depuración en la parte inferior de la ventana. Deberías ver que todos los valores enviados por el formulario se imprimen en la consola, de manera similar al Ejercicio 6.02:
    *Figura 6.21: Valores enviados por el formulario de Django*
    Puedes ver que los valores siguen siendo cadenas y los nombres coinciden con los de los atributos de la clase `ExampleForm`. Observa que se incluye el botón de envío en el que hiciste clic, así como el token CSRF. El formulario que envías puede ser una combinación de campos de formulario de Django y campos arbitrarios que agregues; ambos estarán contenidos en el objeto `QueryDict` de `request.POST`.

En este ejercicio, creaste un formulario de Django con muchos tipos diferentes de campos de formulario. Creaste una instancia en una variable en tu vista y luego la pasaste a `form-example.html`, donde se renderizó como HTML. Finalmente, enviaste el formulario y examinaste los valores que publicó. Observa que la cantidad de código que tuvimos que escribir para generar el mismo formulario se redujo considerablemente. No tuvimos que codificar manualmente ningún HTML y ahora tenemos un solo lugar que define cómo se mostrará el formulario y cómo se validará.

En la siguiente sección, examinaremos cómo los formularios de Django pueden validar automáticamente los datos enviados, así como cómo se convierten los datos de cadenas a objetos de Python.

---

### Sección: Validación de formularios y obtención de valores de Python

Hasta ahora, hemos visto cómo los formularios de Django hacen que sea mucho más sencillo definir un formulario utilizando código Python y que se renderice automáticamente. Ahora veremos la otra parte de lo que hace que los formularios de Django sean útiles: su capacidad para validar automáticamente el formulario y luego recuperar objetos y valores nativos de Python a partir de ellos.

En Django, un formulario puede ser **no vinculado** (*unbound*) o **vinculado** (*bound*). Estos términos describen si el formulario ha recibido o no los datos POST enviados para su validación. Hasta ahora, solo hemos visto formularios no vinculados: se instancian sin argumentos, de esta manera:

```python
form = ExampleForm()
```

Un formulario está vinculado si se llama con algunos datos para ser utilizados para la validación, como los datos POST. Se puede crear un formulario vinculado de esta manera:

```python
form = ExampleForm(request.POST)
```

Un formulario vinculado nos permite comenzar a usar herramientas integradas relacionadas con la validación en la instancia del formulario. Primero, está el método `is_valid`, que verifica la validez del formulario, y luego el atributo `cleaned_data`, que contiene los valores convertidos de cadenas a objetos de Python. El atributo `cleaned_data` solo está disponible después de que el formulario se haya limpiado, que es el proceso de limpiar los datos y convertirlos de cadenas a objetos de Python. El proceso de limpieza se ejecuta durante la llamada a `is_valid`. Se generará un `AttributeError` si intentas acceder a `cleaned_data` antes de llamar a `is_valid`.

Un breve ejemplo de cómo acceder a los datos limpios de `ExampleForm` se ve así:

```python
form = ExampleForm(request.POST)
if form.is_valid():
    # cleaned_data solo se llena si el formulario es válido
    if form.cleaned_data["integer_input"] > 5:
        do_something()
```

En este ejemplo, `form.cleaned_data["integer_input"]` es el valor entero, `10`, por lo que se puede comparar con el número `5`. Compara esto con el valor que se envió, que es la cadena `'10'`. El proceso de limpieza realiza esta conversión por nosotros. Otros campos, como fechas o booleanos, se convierten en consecuencia.

El proceso de limpieza también establece cualquier error en el formulario y en los campos, que se mostrarán cuando el formulario se vuelva a renderizar. Veamos todo esto en acción. Los navegadores modernos proporcionan una gran cantidad de validación del lado del cliente, por lo que evitan que se envíen formularios a menos que se cumplan sus reglas básicas de validación. Es posible que ya hayas visto esto si intentaste enviar el formulario en el ejercicio anterior con campos vacíos:

*Figura 6.22: Envío de formulario bloqueado por el navegador*

La Figura 6.22 muestra cómo el navegador impide el envío del formulario. Dado que el navegador impide el envío, Django nunca tiene la oportunidad de validar el formulario por sí mismo. Para permitir que se envíe el formulario, debemos agregar una validación más avanzada que el navegador no pueda validar por sí mismo. Analizaremos los diferentes tipos de validaciones que se pueden aplicar a los campos del formulario en la siguiente sección, pero por ahora, solo agregaremos una configuración `max_digits` de 3 a `decimal_input` de `ExampleForm`. Esto significa que el usuario no debe ingresar más de tres dígitos en el formulario.

¿Por qué debería Django validar el formulario si el navegador ya lo está haciendo y evitando el envío? Una aplicación del lado del servidor nunca debe confiar en la entrada del usuario: el usuario podría estar usando un navegador más antiguo u otros clientes HTTP para enviar la solicitud, sin recibir ningún error de su navegador. Además, como acabamos de mencionar, existen tipos de validación que el navegador no comprende, por lo que Django debe validarlos por su parte.

`ExampleForm` se puede actualizar de la siguiente manera:

```python
class ExampleForm(forms.Form):
    …
    decimal_input = forms.DecimalField(max_digits=3)
    …
```

Ahora, la vista debe actualizarse para pasar `request.POST` a la clase de formulario cuando el método sea POST, por ejemplo, de esta manera:

```python
if request.method == "POST":
    form = ExampleForm(request.POST)
else:
    form = ExampleForm()
```

Si pasas `request.POST` al constructor del formulario cuando el método no es POST, el formulario siempre contendrá errores cuando se renderice por primera vez, ya que `request.POST` estará vacío.

Ahora, el navegador nos permitirá enviar el formulario, pero se nos mostrará un error si `decimal_input` contiene más de tres dígitos:

*Figura 6.23: Se muestra un error cuando un campo no es válido*

Django renderiza automáticamente el formulario de manera diferente en la plantilla cuando tiene errores. Pero, ¿cómo podemos hacer que la vista se comporte de manera diferente según la validez del formulario? Como mencionamos anteriormente, debemos usar el método `is_valid` del formulario. Una vista que use esta verificación podría contener código como este:

```python
form = ExampleForm(request.POST)
if form.is_valid():
    # realizar operaciones con los datos de
    # form.cleaned_data
    return redirect("/success-page")  # redirigir a una página de éxito
```

En este ejemplo, estamos redirigiendo a una página de éxito si el formulario es válido. De lo contrario, asumimos que el flujo de ejecución continúa como antes y pasamos el formulario no válido de vuelta a la función `render` para que se muestre al usuario con errores.

¿Por qué devolvemos una redirección en caso de éxito? Por dos razones: primero, un retorno anticipado evita la ejecución del resto de la vista (es decir, la rama de fallo); segundo, evita el mensaje sobre el reenvío de los datos del formulario si el usuario luego recarga la página.

En el siguiente ejercicio, veremos la validación del formulario en acción y cambiaremos el flujo de ejecución de la vista según la validez del formulario.

#### Ejercicio 6.04 – Validación de formularios en una vista

En este ejercicio, actualizaremos la vista de ejemplo para crear una instancia del formulario de manera diferente según el método HTTP. También cambiaremos el formulario para que imprima los datos limpios en lugar de los datos POST sin procesar, pero solo si el formulario es válido:

1. En PyCharm, abre el archivo `forms.py` dentro del directorio de la aplicación `form_example`. Agrega un argumento `max_digits=3` a `decimal_input` de `ExampleForm`:
   ```python
   class ExampleForm(forms.Form):
       …
       decimal_input = forms.DecimalField(max_digits=3)
   ```
   Una vez que se haya agregado este argumento, podemos enviar el formulario ya que el navegador no sabe cómo validar esta regla, pero Django sí.
2. Abre el archivo `views.py` de la aplicación `reviews` (o `form_example`). Debemos actualizar la vista `form_example` para que si el método de la solicitud es POST, `ExampleForm` se instancie con los datos POST; de lo contrario, se crea una instancia sin argumentos. Reemplaza la inicialización del formulario actual con este código:
   ```python
   def form_example(request):
       if request.method == "POST":
           form = ExampleForm(request.POST)
       else:
           form = ExampleForm()
   ```
3. A continuación, también para el método de solicitud POST, verificaremos si el formulario es válido mediante el método `is_valid`. Si el formulario es válido, imprimiremos todos los datos limpios. Agrega una condición después de la creación de la instancia de `ExampleForm` para verificar `form.is_valid()`, luego mueve el bucle de impresión de depuración dentro de esta condición. Tu rama POST debería verse así:
   ```python
   if request.method == "POST":
       form = ExampleForm(request.POST)
       if form.is_valid():
           for name in request.POST:
               print(f"{name}: {request.POST.getlist(name)}")
   ```
4. En lugar de iterar sobre el `QueryDict` de `request.POST` sin procesar (en el que todos los datos son cadenas), iteraremos sobre el `cleaned_data` del formulario. Este es un diccionario normal y contiene los valores convertidos en objetos de Python. Reemplaza la línea `for` y la línea `print` con estas dos:
   ```python
   for name, value in form.cleaned_data.items():
       print(f"{name}: ({type(value)}) {value}")
   ```
   Ya no necesitamos usar `getlist()` ya que `cleaned_data` ya ha convertido los campos con múltiples valores en listas.
5. Inicia el servidor de desarrollo de Django, si aún no se está ejecutando. Cambia a tu navegador y navega a la página de formulario de ejemplo en `http://127.0.0.1:8000/form-example/`. El formulario debería verse como antes. Llena todos los campos, pero asegúrate de ingresar cuatro o más números en el campo Decimal input para que el formulario no sea válido. Envía el formulario; deberías ver aparecer el mensaje de error para el campo Decimal input cuando se actualice la página:
   *Figura 6.24: Error de entrada decimal mostrado después de enviar el formulario*
6. Corrige los errores del formulario asegurándote de que solo haya tres dígitos en el campo Decimal input, luego envía el formulario nuevamente. Vuelve a PyCharm y revisa la consola de depuración. Deberías ver que todos los datos limpios se han impreso:
   *Figura 6.25: Los datos limpios del formulario se han impreso*
   Observa las conversiones que han tenido lugar: `CharField` se ha convertido en `str`, `BooleanField` en `bool`, y `IntegerField`, `FloatField` y `DecimalField` en `int`, `float` y `Decimal`, respectivamente. `DateField` se convierte en un `datetime.date`, y los campos de opción conservarán los valores de cadena de sus valores de opción iniciales. Observa que `books_you_own` se convierte automáticamente en una lista de `str`.
   Además, ten en cuenta que, a diferencia de cuando iteramos sobre todos los datos POST, `cleaned_data` solo contiene campos de formulario. Los otros datos (como el token CSRF y el botón de envío en el que se hizo clic) están presentes en el objeto `QueryDict` de la solicitud POST pero no están incluidos, ya que no son campos de formulario.

En este ejercicio, actualizaste `ExampleForm` para que el navegador permitiera enviarlo a pesar de que Django lo consideraría no válido. Esto permitió a Django realizar su validación en el formulario. Luego actualizaste la vista `form_example` para instanciar la clase `ExampleForm` de manera diferente, según el método HTTP, pasando los datos POST de la solicitud para una solicitud POST. La vista también actualizó su código de salida de depuración para imprimir el diccionario `cleaned_data`. Finalmente, probaste enviar datos de formulario válidos e inválidos para ver las diferentes rutas de ejecución y los tipos de datos que generaba el formulario. Vimos que Django convirtió automáticamente los datos POST de cadenas en tipos de Python según la clase de campo.

A continuación, veremos cómo agregar más opciones de validación a los campos, lo que nos permitirá controlar más estrictamente los valores que se pueden ingresar.

#### Validación de campos integrada

Aún no hemos analizado los argumentos de validación estándar que se pueden usar en los campos. Aunque ya mencionamos el argumento `required` (que es `True` de forma predeterminada), se pueden usar muchos otros para controlar los datos que se ingresan en un campo de manera más estricta. Estos son algunos muy útiles:
- **max_length**: Establece el número máximo de caracteres que se pueden ingresar en el campo, disponible en `CharField` (y `FileField`, que cubriremos en el Capítulo 8).
- **min_length**: Establece el número mínimo de caracteres que se deben ingresar en el campo, disponible en `CharField` (y `FileField`).
- **max_value**: Establece el valor máximo que se puede ingresar en un campo numérico, disponible en `IntegerField`, `FloatField` y `DecimalField`.
- **min_value**: Establece el valor mínimo que se puede ingresar en un campo numérico, disponible en `IntegerField`, `FloatField` y `DecimalField`.
- **max_digits**: Establece el número máximo de dígitos que se pueden ingresar. Esto incluye antes y después de un punto decimal (si existe uno). Por ejemplo, el número `12.34` tiene cuatro dígitos, mientras que el número `56.7` tiene tres. Esto se usa en `DecimalField`.
- **decimal_places**: Establece el número máximo de dígitos que pueden estar después del punto decimal. Esto se usa junto con `max_digits`, y el número de lugares decimales siempre contará para el número de dígitos, incluso si ese número de decimales no se ha ingresado después del lugar decimal. Por ejemplo, usando un valor de `max_digits` de `4` y un `decimal_places` de `3`: si se ingresó el número `12.34`, se interpretaría como `12.340`, es decir, se agregan ceros hasta que el número de dígitos después del punto decimal sea igual a la configuración de `decimal_places`. Como establecimos `3` como el valor para `decimal_places`, el número total de dígitos termina siendo `5`, lo que excede la configuración de `max_digits` de `4`. El número `1.2` sería válido ya que incluso después de expandirse a `1.200`, el número total de dígitos es solo `4`.

Puedes mezclar y combinar las reglas de validación (siempre que los campos las admitan). `CharField` puede tener un valor de `max_length` y un valor de `min_length`; los campos numéricos pueden tener tanto `min_value` como `max_value`, y así sucesivamente.

Si necesitas más opciones de validación, puedes escribir validadores personalizados, que cubriremos en el próximo capítulo. En este momento, agregaremos algunos validadores a `ExampleForm` para verlos en acción.

#### Ejercicio 6.05 – Adición de validación de campos adicional

En este ejercicio, agregaremos y modificaremos las reglas de validación en los campos de `ExampleForm`. Luego veremos cómo estos cambios afectan el comportamiento del formulario, tanto en el navegador como cuando Django valida el formulario:

1. En PyCharm, abre el archivo `forms.py` dentro del directorio de la aplicación `form_example`.
2. Haremos que `text_input` requiera como máximo tres caracteres. Agrega un argumento `max_length=3` al constructor de `CharField`:
   ```python
   text_input = forms.CharField(max_length=3)
   ```
3. Haz que `password_input` sea más seguro requiriendo un mínimo de ocho caracteres. Agrega un argumento `min_length=8` al constructor de `CharField`:
   ```python
   password_input = forms.CharField(min_length=8, widget=forms.PasswordInput)
   ```
4. El usuario puede no tener libros, por lo que el campo `books_you_own` no debería ser obligatorio. Agrega un argumento `required=False` al constructor de `MultipleChoiceField`:
   ```python
   books_you_own = forms.MultipleChoiceField(
       required=False,
       choices=BOOK_CHOICES)
   ```
5. El usuario solo debería poder ingresar un valor entre 1 y 10 en `integer_input`. Agrega los argumentos `min_value=1` y `max_value=10` al constructor de `IntegerField`:
   ```python
   integer_input = forms.IntegerField(
       min_value=1,
       max_value=10)
   ```
6. Finalmente, agrega `max_digits=5` y `decimal_places=3` al constructor de `DecimalField`:
   ```python
   decimal_input = forms.DecimalField(
       max_digits=5,
       decimal_places=3)
   ```
7. Guarda el archivo.
8. Inicia el servidor de desarrollo de Django si no se está ejecutando. No tenemos que realizar cambios en ningún otro archivo para obtener estas nuevas reglas de validación, ya que Django actualiza automáticamente la lógica de generación y validación de HTML. Este es un gran beneficio que se obtiene al usar formularios de Django. Simplemente visita o actualiza `http://127.0.0.1:8000/form-example/` en tu navegador, y la nueva validación se agregará automáticamente. El formulario no debería verse diferente hasta que intentes enviarlo con valores incorrectos, en cuyo caso tu navegador puede mostrar errores automáticamente. Algunas cosas para probar son las siguientes:
   - Ingresar más de tres caracteres en el campo Text input es algo que no podrás hacer.
   - Escribir menos de ocho caracteres en el campo Password y luego hacer clic fuera de él puede hacer que tu navegador muestre un error que indique que no es válido (solo algunos navegadores admiten esto).
   - No seleccionar ningún valor para el campo Books you own ya no impedirá que envíes el formulario.
   - Usa los botones de incremento/decremento en Integer input. Solo podrás ingresar un valor entre 1 y 10. Si escribes un valor fuera de este rango, tu navegador debería mostrar un error.
   - Decimal input es el único campo que no valida las reglas de Django en el navegador. Deberás ingresar un valor no válido (como `123.456`) y enviar el formulario antes de que se muestre un error (generado por Django).

La siguiente figura muestra algunos de los campos que el navegador puede utilizar para validarse a sí mismo:

*Figura 6.26: Algunos navegadores resaltan campos no válidos*

La Figura 6.27 muestra una forma alternativa en que el navegador puede presentar errores de validación, colocando un mensaje de error junto a un campo particular al enviar el formulario:

*Figura 6.27: Los navegadores pueden mostrar mensajes junto a los campos al enviar*

La Figura 6.28 muestra un error que solo puede ser generado por Django, ya que el navegador no comprende las reglas de validación de `DecimalField`:

*Figura 6.28: El navegador considera válido el formulario, pero Django no*

En este ejercicio, implementamos algunas reglas de validación básicas en nuestros campos de formulario. Luego cargamos la página de ejemplo del formulario en el navegador sin tener que realizar cambios en nuestra plantilla o vista. Intentamos enviar el formulario con diferentes valores para verificar cómo el navegador puede validar el formulario en comparación con Django.

En la actividad de este capítulo, implementaremos la vista de búsqueda de libros utilizando un formulario de Django.

---

### Sección: Actividad 6.01 – Búsqueda de libros

En esta actividad, finalizarás la vista de búsqueda de libros que se inició en el Capítulo 1. Crearás un `SearchForm` que envía y acepta una cadena de búsqueda de `request.GET`. Tendrá un campo de selección para elegir buscar por un título o un colaborador. Luego buscará todos los libros que contengan el texto dado en su título o en los nombres o apellidos del colaborador. Luego renderizarás esta lista de libros en la plantilla `search-results.html`. El término de búsqueda no debe ser obligatorio, pero si existe, debe tener una longitud de más de tres caracteres. Dado que la vista buscará incluso cuando se use el método GET, el formulario siempre tendrá su validación comprobada. Si hiciéramos el campo obligatorio, siempre mostraría un error cada vez que se cargue la página.

Habrá dos formas de realizar la búsqueda:
1. La primera es enviando el formulario de búsqueda que se encuentra en la plantilla `base.html` y, por lo tanto, en la esquina superior derecha de cada página. Esto solo buscará a través de títulos de libros.
2. El otro método es enviando un `SearchForm` que se renderiza en la página `search-results.html`. Este formulario mostrará un `ChoiceField` para seleccionar entre una búsqueda de título o de colaborador.

Estos pasos te ayudarán a completar esta actividad:

1. Crea un archivo `forms.py`, importa la biblioteca de formularios de Django y luego crea una clase `SearchForm`.
2. `SearchForm` debe tener dos campos. El primero es `CharField` con el nombre `search`. Este campo no debe ser obligatorio, pero debe tener una longitud mínima de 3.
3. El segundo campo en `SearchForm` es `ChoiceField` con el nombre `search_in`. Esto te permitirá seleccionar entre título y colaborador (con etiquetas de `Title` y `Contributor`, respectivamente). No debe ser obligatorio.
4. Actualiza la vista `book_search` para que cree una instancia de un `SearchForm` utilizando datos de `request.GET`.
5. Agrega código para buscar modelos `Book` usando `title__icontains` (para búsquedas que no distingan entre mayúsculas y minúsculas). Esto se debe hacer si se busca por título. La búsqueda solo debe realizarse si el formulario es válido y contiene algún texto de búsqueda. El valor de `search_in` debe recuperarse de `cleaned_data` mediante el método `get` ya que podría no existir, al no ser obligatorio. Establécelo de forma predeterminada en `title`.
6. Al buscar colaboradores, usa `first_names__icontains` o `last_names__icontains`, luego itera sobre los colaboradores y recupera los libros para cada colaborador. Esto se debe hacer si estás buscando por colaborador. La búsqueda solo debe realizarse si el formulario es válido y contiene algún texto de búsqueda. Hay muchas formas de combinar los resultados de búsqueda para nombres o apellidos. El método más sencillo utilizando las técnicas que se te han presentado hasta ahora es realizar dos consultas, una para nombres que coincidan y luego para apellidos, e iterarlas por separado.
7. Actualiza la llamada a `render` para que incluya la variable `form` y los libros (`books`) que se recuperaron en el contexto (así como `search_text`, que ya se estaba pasando). La ubicación de la plantilla se cambió en el Capítulo 3, así que actualiza el segundo argumento de `render` en consecuencia.
8. La plantilla `search-results.html` que creamos en el Capítulo 1 es esencialmente redundante ahora, por lo que puedes borrar su contenido. Actualiza `search-results.html` para que se extienda de `base.html` en lugar de ser un archivo de plantilla independiente.
9. Agrega un bloque `title` que mostrará `Search Results for <search_text>` si el formulario es válido y se estableció `search_text`; de lo contrario, simplemente muestra `Book Search`. Este bloque también se agregará a `base.html` más adelante en esta actividad.
10. Agrega un bloque `content`, que debe mostrar un encabezado `<h2>` con el texto `Search for Books`; debajo del `<h2>`, renderiza el formulario. El elemento `<form>` no puede tener atributos y se establecerá de forma predeterminada en realizar una solicitud GET a la misma URL en la que se encuentra. Agrega un botón de envío, como hemos usado en actividades anteriores, con la clase `btn btn-primary`.
11. Debajo del formulario, muestra un mensaje que diga `Search results for <search_text>` si el formulario es válido y se ingresó texto de búsqueda; de lo contrario, no muestres ningún mensaje. Esto debe mostrarse en un encabezado `<h3>` y el texto de búsqueda debe estar envuelto en un `<em>`.
12. Itera sobre los resultados de la búsqueda y renderiza cada uno. Muestra el título del libro y el nombre y apellido del colaborador (o colaboradores). El título del libro debe vincularse a la página `book_detail`. Si la lista de libros está vacía, muestra el texto `No results found`. Debes envolver los resultados en una lista `<ul>` con una clase llamada `list-group`; cada resultado debe ser un elemento `<li>` con una clase de `list-group-item`. Esto será como la página `book_list`, pero no mostraremos tanta información (solo el título y los colaboradores).
13. Actualiza el archivo `base.html` a nivel de proyecto para que incluya un atributo `action` en la etiqueta `<form>` de búsqueda. Usa la etiqueta de plantilla `url` para generar la URL para este atributo.
14. Establece el atributo `name` del campo de búsqueda en `search`, y el atributo `value` en el `search_text` ingresado. Además, asegúrate de que la longitud mínima del campo sea 3.
15. Mientras estás en `base.html`, agrega un bloque `title` a la etiqueta `title` que fue anulada por otras plantillas (como en el Paso 9). Agrega una etiqueta de plantilla `block` dentro del elemento HTML `<title>`. Debe contener el contenido `Bookr`.

Después de completar esta actividad, deberías poder abrir la página de búsqueda de libros en `http://127.0.0.1:8000/book-search/`; se verá como la Figura 6.29:

*Figura 6.29: La página de búsqueda de libros sin una búsqueda*

Al buscar algo usando solo dos caracteres, tu navegador debería impedir que envíes cualquiera de los campos de búsqueda.

Si buscas algo que no arroja resultados, verás un mensaje que indica que no hubo resultados. La búsqueda por título (esto se puede hacer con cualquiera de los dos campos) mostrará los resultados coincidentes:

*Figura 6.30: Búsqueda por título*

De manera similar, al buscar por colaborador (aunque esto solo se puede hacer en el formulario inferior), verás los resultados coincidentes:

*Figura 6.31: Una búsqueda por colaborador*

---

### Sección: Resumen

Este capítulo proporcionó una introducción a los formularios en Django. Presentamos varias entradas HTML para ingresar datos en una página web. Hablamos sobre cómo se envían los datos a una aplicación web y cuándo usar solicitudes GET y POST. Luego vimos cómo las clases de formulario de Django pueden simplificar la generación del HTML del formulario, así como permitirnos construir formularios automáticamente utilizando modelos. Finalmente, mejoramos Bookr aún más al crear la funcionalidad de búsqueda de libros (*Book Search*).

En el próximo capítulo, profundizaremos en los formularios y aprenderemos cómo personalizar la visualización de los campos del formulario, cómo agregar una validación más avanzada a un formulario y cómo guardar automáticamente instancias de modelos mediante el uso de la clase `ModelForm`.
