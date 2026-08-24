# Parte 1: Primeros pasos con Django

## Capítulo 1: Introducción a Django

"El framework web para perfeccionistas con fechas límite." Es un eslogan que describe acertadamente a Django, un framework que ha existido durante casi 20 años. Ha sido probado en batalla y es ampliamente utilizado, con más y más personas usándolo cada día. Todo esto podría hacerte pensar que Django es viejo y ya no es relevante. Por el contrario, su longevidad ha demostrado que su API es confiable y consistente, e incluso aquellos que aprendieron Django v1.0 en 2007 pueden escribir en su mayoría el mismo código para Django 6 hoy en día. Django todavía está en desarrollo activo, con correcciones de errores y parches de seguridad que se publican regularmente.

Al igual que Python, el lenguaje en el que está escrito, Django es fácil de aprender, pero lo suficientemente potente y flexible para crecer según tus necesidades. Es un framework "con baterías incluidas" – en otras palabras, no tienes que buscar e instalar muchas otras librerías o componentes para poner en marcha tu aplicación. Otros frameworks, como Flask o Pylons, requieren instalar manualmente frameworks de terceros para conexiones a bases de datos o renderizado de plantillas. En su lugar, Django cuenta con soporte integrado para consultas a bases de datos, mapeo de URLs y renderizado de plantillas (pronto entraremos en detalle sobre lo que esto significa). Sin embargo, el hecho de que Django sea fácil de usar no significa que sea limitado. Django es utilizado por muchos sitios grandes, incluidos Disqus, Instagram, Mozilla, Spotify, Open edX, OpenStack y National Geographic.

¿Dónde encaja Django en la web? Al hablar de frameworks web, podrías pensar en frameworks de frontend en JavaScript como ReactJS, Angular o Vue. Estos frameworks se utilizan para mejorar o añadir interactividad a páginas web ya generadas. Django se sitúa en la capa inferior a estas herramientas y, en su lugar, se encarga de enrutar una URL, obtener datos de bases de datos, renderizar plantillas y procesar la entrada de formularios de los usuarios. Sin embargo, esto no significa que debas elegir uno u otro; los frameworks de JavaScript se pueden utilizar para mejorar la salida de Django o interactuar con una API REST generada por Django.

En este libro, construiremos un proyecto de Django utilizando los mismos métodos que los desarrolladores profesionales de Django utilizan todos los días. El proyecto se llama **Bookr** y te permite explorar y añadir libros y reseñas de libros. Este libro está dividido en cuatro partes. En la primera parte, comenzaremos con los aspectos básicos de la estructuración (*scaffolding*) de una aplicación Django, construiremos rápidamente algunas páginas y las serviremos con el servidor de desarrollo de Django. Podrás añadir datos a la base de datos utilizando el sitio de administración de Django.

Este capítulo te introduce a Django y su papel en el desarrollo web. Comenzarás entendiendo cómo funciona el paradigma modelo-vista-plantilla (*Model-View-Template*) y cómo Django procesa las solicitudes y respuestas HTTP. Equipado con los conceptos básicos, crearás tu primer proyecto de Django, llamado **Bookr**, una aplicación para añadir, ver y gestionar reseñas de libros. Es una aplicación que seguirás mejorando y a la que añadirás funciones a lo largo de este libro. Luego aprenderás sobre el comando `manage.py` (utilizado para orquestar las acciones de Django). Utilizarás este comando para iniciar el servidor de desarrollo de Django y comprobar si el código que has escrito hasta ahora funciona como se espera. También aprenderás a trabajar con PyCharm, un popular IDE de Python, que utilizarás a lo largo de este libro. Lo utilizarás para escribir código que devuelve una respuesta a tu navegador web. Finalmente, aprenderás a utilizar el depurador (*debugger*) de PyCharm para solucionar problemas en tu código. Al final de este capítulo, tendrás las habilidades necesarias para comenzar a crear proyectos utilizando Django.

Cubriremos los siguientes temas en este capítulo:
- Estructuración inicial (*scaffolding*) de un proyecto y una aplicación de Django
- Comprensión del paradigma modelo-vista-plantilla
- Exploración de la estructura del proyecto Django
- Introducción a las vistas de Django
- Exploración en detalle del mapeo de URLs
- Exploración de la configuración de Django (*settings*)
- Localización de plantillas HTML en directorios de aplicaciones
- Depuración y gestión de errores
- Actividad 1.01 – Creación de una pantalla de bienvenida para el sitio
- Actividad 1.02 – Estructura inicial para la búsqueda de libros

Al final del libro, tendrás suficiente experiencia para diseñar y construir tu propio proyecto de Django de principio a fin.

---

### Sección: Requisitos técnicos

A lo largo de este libro, escribirás código. Si necesitas consultar el código completo para este capítulo, puedes encontrarlo en la carpeta `Chapter01` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos descritos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Tu compra incluye una copia gratuita en PDF + paquete de código

Tu compra incluye una copia en PDF sin DRM de este libro, el paquete de código y extras exclusivos adicionales. Consulta la sección *Free benefits with your book* en el Prefacio para desbloquearlos instantáneamente y maximizar tu aprendizaje.

---

### Sección: Creación de un entorno virtual con pyenv

Antes de profundizar en la teoría detrás de los paradigmas de Django y las solicitudes HTTP, te mostraremos lo fácil que es poner en marcha un proyecto de Django. Después de esta primera sección y ejercicio, habrás creado un proyecto de Django, realizado una solicitud a este con tu navegador y visto la respuesta.

Como desarrollador de Python, es posible que trabajes con muchas aplicaciones en tu sistema que tengan dependencias diferentes e incluso conflictivas. Los ejemplos de este libro utilizan **Python 3.12** y **Django 6.0**. Sin embargo, es posible que necesites mantener otras versiones de Python para tus otros proyectos que tal vez ni siquiera sean compatibles con Django 6.0.

Además, los sistemas operativos como macOS y Linux vienen con Python instalado, y los sistemas operativos lo utilizan con frecuencia como lenguaje de scripting. Por esta razón, la mejor práctica es desarrollar con una instalación de Python diferente que no interrumpa la funcionalidad del sistema operativo si se introducen conflictos.

Por estas razones, los entornos virtuales son una técnica ideal para desarrollar con múltiples entornos. Cuando instalas Python, Django y sus dependencias, quedan en cuarentena en su propio entorno específico de usuario y no afectan el comportamiento de otro software de Python que estés desarrollando y utilizando.

En este libro, utilizaremos `pyenv` para configurar entornos virtuales de Python.

#### Instalación de pyenv en Windows

`pyenv` para Windows está disponible en [https://github.com/pyenv-win/pyenv-win](https://github.com/pyenv-win/pyenv-win). Se puede instalar desde PowerShell con este comando:

```powershell
Invoke-WebRequest-UseBasicParsing-Uri"https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1"-OutFile"./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

#### Instalación de pyenv en macOS y Linux

Si utilizas el gestor de paquetes Homebrew en macOS, puedes instalar estos dos paquetes para obtener la funcionalidad requerida:

```bash
brew install pyenv pyenv-virtualenv
```

De lo contrario, en macOS y Linux, `pyenv` se puede instalar ejecutando lo siguiente en la línea de comandos:

```bash
curl https://pyenv.run | bash
```

En Linux, deberás incluir estas líneas en tu archivo `.bashrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

En macOS, incluye estas líneas en tu archivo `.zshrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

#### Creación de un entorno virtual con pyenv

Ahora que hemos instalado `pyenv`, podemos usarlo para crear un entorno para nuestro proyecto de Django. Dado que utilizamos Python 3.12 en este libro, comenzaremos instalando esta versión en nuestro entorno local de `pyenv`. Lo hacemos utilizando el comando `pyenv install`:

```bash
pyenv install 3.12
```

Podemos verificar la configuración utilizando el comando `pyenv versions`:

```bash
% pyenv versions
* system (set by /Users/chrisguest/.pyenv/version)
  3.12.0
```

Con nuestra versión deseada de Python instalada, podemos crear un entorno virtual. Lo llamaremos `djangoenv`:

```bash
pyenv virtualenv 3.12 djangoenv
```

Ahora, con nuestro entorno virtual creado, lo activaremos para que los comandos posteriores en la terminal se interpreten utilizando este entorno mediante el comando `pyenv local`:

```bash
pyenv local djangoenv
```

Con nuestro entorno creado, podemos instalar los módulos de Python que necesitaremos para el proyecto. En nuestro caso, como estamos desarrollando proyectos de Django utilizando Django 6.0, instalaremos esta versión de Django utilizando el comando `pip install`:

```bash
pip install django==6.0
```

La mayoría de los ejercicios y actividades de este libro tienen los módulos necesarios enumerados en un archivo `requirements.txt`. Por ejemplo, este capítulo tiene este archivo:

```text
Web-Development-with-Django-Third-Edition/Chapter01/requirements.txt
```

Para instalar los módulos desde un archivo `requirements.txt` específico, puedes usar este comando:

```bash
pip install -r requirements.txt
```

Si necesitas volver a la versión de Python del sistema, puedes usar este comando:

```bash
% pyenv global system
```

Con un entorno virtual creado y activado, estamos listos para comenzar a trabajar con Django.

---

### Sección: Estructuración de un proyecto y una aplicación de Django (Scaffolding)

Antes de profundizar en la teoría detrás de los paradigmas de Django y las solicitudes HTTP, te mostraremos lo fácil que es poner en marcha un proyecto de Django. Después de esta primera sección y ejercicio, habrás creado un proyecto de Django, realizado una solicitud a este con tu navegador y visto la respuesta.

Un proyecto de Django es un directorio que contiene todos los datos de tu proyecto: código, configuraciones, plantillas y recursos estáticos. Se crea y estructura ejecutando el comando `django-admin` en la línea de comandos con el argumento `startproject` y proporcionando el nombre del proyecto. Por ejemplo, para crear un proyecto de Django con el nombre `myproject`, el comando que se ejecuta es el siguiente:

```bash
django-admin startproject myproject
```

Esto creará el directorio `myproject`, que Django llena con los archivos necesarios para ejecutar el proyecto. Dentro del directorio `myproject` hay dos archivos (mostrados en la Figura 1.1):

*Figura 1.1 – El directorio del proyecto para myproject*

`manage.py` es un script de Python que se ejecuta en la línea de comandos para interactuar con tu proyecto. Lo utilizaremos para iniciar el servidor de desarrollo de Django, un servidor web de desarrollo que utilizarás para interactuar con tu proyecto de Django en tu computadora local. Al igual que `django-admin`, los comandos se pasan en la línea de comandos. A diferencia de `django-admin`, este script no está mapeado en la ruta de tu sistema (*PATH*), por lo que debemos ejecutarlo usando Python. Necesitaremos usar la línea de comandos para hacer eso. Por ejemplo, dentro del directorio del proyecto, ejecutamos el siguiente comando:

```bash
python manage.py runserver
```

Esto pasa el comando `runserver` al script `manage.py`, lo que inicia el servidor de desarrollo de Django. Examinaremos más comandos que acepta `manage.py` más adelante. Al interactuar con `manage.py` de esta manera, los llamamos **comandos de gestión** (*management commands*). Por ejemplo, podríamos decir que estamos "ejecutando el comando de gestión `runserver`".

El comando `startproject` también creó un directorio con el mismo nombre que el proyecto – en este caso, `myproject`. Este es un paquete de Python que contiene configuraciones y algunos otros archivos de configuración que tu proyecto necesita para ejecutarse. Examinaremos su contenido más adelante.

Después de iniciar el proyecto de Django, lo siguiente que se debe hacer es iniciar una **aplicación de Django** (*Django app*). Debemos intentar segregar nuestro proyecto de Django en diferentes aplicaciones, agrupadas por funcionalidad. Por ejemplo, con Bookr, tendremos una aplicación `reviews`. Esta contendrá todo el código, HTML, recursos y clases de base de datos específicos para trabajar con reseñas de libros. Si decidimos expandir Bookr para vender libros también, podríamos añadir una aplicación `store`, que contenga los archivos de la librería. Las aplicaciones se crean con el comando de gestión `startapp`, pasando el nombre de la aplicación. He aquí un ejemplo:

```bash
python manage.py startapp myapp
```

Esto crea el directorio de la aplicación (`myapp`) dentro del directorio del proyecto. Django llena automáticamente esto con archivos para la aplicación que están listos para completarse cuando comiences a desarrollar. Examinaremos estos archivos y discutiremos qué hace que una aplicación sea buena en la sección de aplicaciones de Django.

Ahora que hemos presentado los comandos básicos para estructurar un proyecto y una aplicación de Django, pongámoslos en práctica iniciando el proyecto `bookr` en el primer ejercicio de este libro.

#### Ejercicio 1.01 – Creación de un proyecto y una app, e inicio del servidor de desarrollo

Como se mencionó anteriormente, a lo largo de este libro construiremos un sitio web de reseñas de libros llamado **Bookr**. Te permitirá añadir campos para editoriales, colaboradores, libros y reseñas. Una editorial publicará uno o más libros, y cada libro tendrá uno o más colaboradores (autor, editor, coautor, etc.). Solo los usuarios administradores podrán modificar estos modelos. Una vez que un usuario se haya registrado para obtener una cuenta en el sitio, podrá comenzar a añadir reseñas de libros.

En este ejercicio, estructurarás el proyecto de Django `bookr`, comprobarás que Django funciona ejecutando el servidor de desarrollo y luego crearás la aplicación de Django `reviews`.

Ya deberías tener un entorno virtual configurado con Django instalado. Para saber cómo hacerlo, puedes consultar el Prefacio. Una vez que estés listo, comencemos creando el proyecto `bookr`:

1. Abre la terminal y ejecuta el comando para crear el directorio del proyecto `bookr` y las subcarpetas predeterminadas:
   ```bash
   django-admin startproject bookr
   ```
   Este comando no genera ninguna salida, pero creará una carpeta llamada `bookr` dentro del directorio en el que ejecutaste el comando. Puedes mirar dentro de este directorio y ver los elementos que describimos antes para el ejemplo `myproject`: el directorio del paquete `bookr` y el archivo `manage.py`.

2. Ahora podemos comprobar que el proyecto y Django están configurados correctamente ejecutando el servidor de desarrollo de Django. El inicio del servidor se realiza con el script `manage.py`. En tu terminal (o símbolo del sistema), cambia al directorio del proyecto `bookr` (usando el comando `cd`) y luego ejecuta el comando `manage.py runserver` de la siguiente manera:
   ```bash
   python manage.py runserver
   ```
   Este comando inicia el servidor de desarrollo de Django. Deberías obtener una salida similar a la siguiente:
   ```text
   Watching for file changes with StatReloader
   Performing system checks...

   System check identified no issues (0 silenced).
   You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
   Run 'python manage.py migrate' to apply them.
   December 06, 2025 - 13:10:57
   Django version 6.0, using settings 'bookr.settings'
   Starting development server at http://127.0.0.1:8000/
   Quit the server with CONTROL-C.
   ```
   Probablemente recibirás algunas advertencias sobre migraciones no aplicadas, pero eso está bien por ahora.

3. Abre un navegador web y ve a `http://127.0.0.1:8000/`, que te mostrará la pantalla de bienvenida de Django (Figura 1.2). Si ves esto, sabrás que tu proyecto de Django se creó correctamente y que todo funciona bien por ahora:
   *Figura 1.2 – La pantalla de bienvenida de Django*

4. Regresa a tu terminal y detén la ejecución del servidor de desarrollo presionando `Ctrl + C`.

5. Ahora crearemos la aplicación `reviews` para el proyecto `bookr`. En tu terminal, asegúrate de estar en el directorio del proyecto `bookr` y luego ejecuta el siguiente comando para crear la aplicación `reviews`:
   ```bash
   python manage.py startapp reviews
   ```
   Después de crear la aplicación `reviews`, los archivos en el directorio de tu proyecto `bookr` se verán como los de la carpeta `Chapter01` en el repositorio de GitHub de este libro.

   No hay salida si el comando tuvo éxito, pero se ha creado un directorio de la aplicación `reviews`. Puedes mirar dentro de este directorio para ver los archivos que se crearon: el directorio `migrations`, `admin.py`, `models.py`, etc. Los examinaremos en detalle en la sección de aplicaciones de Django.

En este ejercicio, creamos el proyecto `bookr`, comprobamos que el proyecto funcionaba iniciando el servidor de desarrollo de Django y luego creamos la aplicación `reviews` para el proyecto. Ahora que has tenido un tiempo práctico con un proyecto de Django, volveremos a parte de la teoría detrás del diseño de Django y las solicitudes y respuestas HTTP.

---

### Sección: Comprensión del paradigma Modelo-Vista-Plantilla (MVT)

Un patrón de diseño común en el diseño de aplicaciones es **Modelo Vista Controlador** (*Model View Controller* o MVC), donde el modelo de una aplicación (sus datos) se muestra en una o más vistas, y un controlador organiza la interacción entre el modelo y la vista. Django sigue un paradigma diferente, aunque similar, llamado **Modelo-Vista-Plantilla** (*Model-View-Template* o MVT).

Al igual que MVC, MVT también utiliza modelos para almacenar datos. Sin embargo, con MVT, una vista consultará un modelo y luego lo renderizará con una plantilla. Por lo general, con los lenguajes MVC, los tres componentes deben desarrollarse con el mismo lenguaje. Con MVT, la plantilla puede estar en un lenguaje diferente. En el caso de Django, los modelos y las vistas se escriben en Python, y la plantilla se escribe en HTML. Esto significa que un desarrollador de Python podría trabajar en los modelos y las vistas, mientras que un desarrollador especialista en HTML trabaja en el HTML. Primero explicaremos los modelos, las vistas y las plantillas con más detalle y luego veremos algunos escenarios de ejemplo donde se utilizan.

#### Modelos

Los modelos de Django definen los datos de tu aplicación y proporcionan una capa de abstracción para una base de datos SQL a la que se accede a través de un **Mapeador Objeto-Relacional** (*Object Relational Mapper* u ORM). Un ORM te permite definir tu esquema de datos (clases, campos y sus relaciones) mediante código Python, sin necesidad de comprender la base de datos subyacente. Básicamente, esto significa que puedes definir tu capa de base de datos en código Python y Django se encargará de generar consultas SQL por ti. El ORM se tratará en detalle en el Capítulo 2, *Modelos y Migraciones*.

SQL son las siglas de *Structured Query Language* (Lenguaje de Consulta Estructurado) y es una forma de describir un tipo de base de datos que almacena sus datos en tablas, donde cada tabla tiene varias filas. Piensa en cada tabla como si fuera una hoja de cálculo individual. Sin embargo, a diferencia de una hoja de cálculo, se pueden definir relaciones entre los datos de cada tabla. Puedes interactuar con los datos ejecutando consultas SQL (a menudo denominadas simplemente *queries* cuando se habla de bases de datos). Las consultas te permiten recuperar datos (`SELECT`), añadir o cambiar datos (`INSERT` y `UPDATE`, respectivamente) y eliminar datos (`DELETE`). Hay muchos servidores de bases de datos SQL para elegir, como SQLite, PostgreSQL, MySQL y Microsoft SQL Server. Gran parte de la sintaxis SQL es similar entre cada base de datos, pero puede haber algunas diferencias de dialecto. El ORM de Django se encarga de estas diferencias por ti: cuando comencemos a codificar, usaremos la base de datos SQLite para almacenar datos en disco, pero más adelante, cuando implementemos en un servidor, cambiaremos a PostgreSQL y no necesitaremos hacer ningún cambio en el código.

Normalmente, al consultar una base de datos, los resultados regresan como objetos primitivos de Python (por ejemplo, listas de cadenas, enteros, flotantes o bytes). Al usar el ORM, los resultados se convierten automáticamente en instancias de las clases de modelo que has definido. Usar un ORM significa que estás protegido automáticamente contra un tipo de vulnerabilidad conocida como ataque de inyección SQL.

Si estás más familiarizado con las bases de datos y SQL, siempre tienes la opción de escribir tus propias consultas también.

#### Vistas

Una vista de Django es donde se define la mayor parte de la lógica de tu aplicación. Cuando un usuario visita tu sitio, su navegador web enviará una solicitud para recuperar datos de tu sitio (entraremos en más detalle sobre qué es una solicitud HTTP y qué información contiene en la siguiente sección). Una vista es una función que escribes que recibirá esta solicitud en forma de un objeto Python (específicamente, un objeto `HttpRequest` de Django). Depende de tu vista decidir cómo debe responder a la solicitud y qué debe enviar de vuelta al usuario. Tu vista debe devolver un objeto `HttpResponse` que encapsule toda la información que se proporciona al cliente: contenido, estado HTTP y otros encabezados.

La vista también puede recibir opcionalmente información de la URL de la solicitud; por ejemplo, un número de ID. Un patrón de diseño común de una vista es consultar una base de datos a través del ORM de Django, utilizando un ID que se pasa a tu vista. Luego, la vista puede renderizar una plantilla (habrá más sobre esto en breve) proporcionándole datos del modelo recuperado de la base de datos. La plantilla renderizada se convierte en el contenido del objeto `HttpResponse` y se devuelve desde la función de vista. Django se encarga de la comunicación de los datos de vuelta al navegador.

#### Plantillas

Una plantilla es un archivo HTML (*HyperText Markup Language*, usualmente cualquier archivo de texto puede ser una plantilla) que contiene marcadores de posición especiales que son reemplazados por variables que proporciona tu aplicación. Por ejemplo, tu aplicación podría renderizar una lista de elementos en un diseño de galería o en un diseño de tabla. Tu vista obtendría los mismos modelos para cualquiera de los dos, pero podría renderizar un archivo HTML diferente con la misma información para presentar los datos de manera diferente. Django enfatiza la seguridad, por lo que se encargará de escapar automáticamente las variables por ti. Por ejemplo, los símbolos `<` y `>` (entre otros) son caracteres especiales en HTML. Si intentas usarlos en una variable, Django los codifica automáticamente para que se procesen correctamente en un navegador.

#### MVT en la práctica

Ahora veremos algunos ejemplos para ilustrar cómo funciona MVT en la práctica. En los ejemplos, tenemos un modelo `Book` que almacena información sobre diferentes libros y un modelo `Review` que almacena información sobre diferentes reseñas de libros.

En el primer ejemplo, queremos poder editar la información sobre `Book` o `Review`. Tomemos el primer escenario de editar los detalles de un libro. Tendríamos una vista para obtener los datos de `Book` de la base de datos y proporcionar el modelo `Book`. Luego, pasaríamos información de contexto que contiene el objeto `Book` (y otros datos) a una plantilla que mostraría un formulario para capturar la nueva información. El segundo escenario (editar una reseña) es similar: obtener un modelo `Review` de la base de datos y luego pasar el objeto `Review` y otros datos a una plantilla para mostrar un formulario de edición. Estos escenarios pueden ser tan similares que podemos reutilizar la misma plantilla para ambos, como se muestra en la Figura 1.3.

*Figura 1.3 – Edición de un solo libro o reseña*

Puedes ver aquí que usamos dos modelos, dos vistas y una plantilla. Cada vista obtiene una sola instancia de su modelo asociado, pero ambas pueden usar la misma plantilla, que es una página HTML genérica para mostrar un formulario. Las vistas pueden proporcionar datos de contexto adicionales para alterar ligeramente la visualización de la plantilla para cada tipo de modelo. También se ilustran en el diagrama las partes del código que están escritas en Python y las que están escritas en HTML.

En el segundo ejemplo, queremos poder mostrar a un usuario una lista de los libros o reseñas que están almacenados en la aplicación. Además, queremos permitir que el usuario busque libros y obtenga una lista de todos los que coincidan con sus criterios. Utilizaremos los mismos dos modelos que en el ejemplo anterior (`Book` y `Review`), pero crearemos nuevas vistas y plantillas. Dado que hay tres escenarios, utilizaremos tres vistas esta vez: la primera obtiene todos los libros, la segunda obtiene todas las reseñas y la última busca libros según algunos criterios de búsqueda. Una vez más, si escribimos bien una plantilla, podríamos usar solo una plantilla HTML nuevamente, como se muestra en la Figura 1.4.

*Figura 1.4 – Visualización de múltiples libros o reseñas*

Los modelos `Book` y `Review` permanecen sin cambios respecto al ejemplo anterior; las tres vistas obtendrán muchos (cero o más) libros o reseñas. Luego, cada vista puede usar la misma plantilla, que es un archivo HTML genérico que itera sobre una lista de objetos que se le proporciona y los renderiza. Una vez más, las vistas pueden enviar datos adicionales en el contexto para alterar el comportamiento de la plantilla, pero la mayor parte de la plantilla será lo más genérica posible.

En Django, no siempre es necesario utilizar un modelo para renderizar una plantilla HTML. Una vista puede generar los datos de contexto por sí misma y renderizar una plantilla con ellos, sin requerir ningún dato de modelo. La Figura 1.5 muestra una vista que envía datos directamente a una plantilla.

*Figura 1.5 – Una vista que envía datos a una plantilla sin un modelo*

En este ejemplo, hay una vista `Welcome View` para dar la bienvenida a un usuario al sitio. No necesita ninguna información de la base de datos, por lo que puede generar los datos de contexto por sí misma. Los datos de contexto dependen del tipo de información que desees mostrar; por ejemplo, podrías pasar la información del usuario para saludarlo por su nombre si ha iniciado sesión.

También es posible que una vista renderice una plantilla sin ningún dato de contexto. Esto puede resultar útil si tienes información estática en un archivo HTML que deseas servir.

Ahora que te has familiarizado con MVT en Django, podemos ver cómo Django procesa una solicitud HTTP y genera una respuesta HTTP. Sin embargo, primero debemos explicar con más detalle qué son las solicitudes y respuestas HTTP y qué información contienen. Veremos esto en la siguiente sección.

#### Introducción a HTTP

Supongamos que alguien quiere visitar tu página web. Escribe su URL o hace clic en un enlace a tu sitio desde una página en la que ya se encuentra. Su navegador web crea una solicitud HTTP (*HTTP request*), que se envía al servidor que aloja tu sitio web. Una vez que un servidor web recibe la solicitud HTTP de tu navegador, puede interpretarla y luego devolver una respuesta. La respuesta que envía el servidor puede ser simple, como simplemente leer un archivo HTML o de imagen del disco y enviarlo. Alternativamente, la respuesta puede ser más compleja, tal vez utilizando software del lado del servidor (como Django) para generar dinámicamente el contenido antes de enviarlo.

El siguiente diagrama muestra la dirección de la transmisión de solicitudes HTTP y respuestas HTTP entre un navegador y un servidor web.

*Figura 1.6 – Una solicitud HTTP y una respuesta HTTP*

La solicitud se compone de cuatro partes principales: el método, la ruta, los encabezados y el cuerpo. Algunos tipos de solicitudes no tienen cuerpo. Si simplemente visitas una página web, tu navegador no enviará un cuerpo, mientras que si estás enviando un formulario (por ejemplo, iniciando sesión en un sitio o realizando una búsqueda), tu solicitud tendrá un cuerpo que contendrá los datos que estás enviando. Veremos dos solicitudes de ejemplo ahora para ilustrar esto.

La primera solicitud será a una página de ejemplo con la siguiente URL: `https://www.example.com/page`. Cuando tu navegador visita esa página, detrás de escena, esto es lo que está enviando:

```http
GET /page HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0
Cookie: sessid=abc123def456
```

La primera línea contiene el método (`GET`) y la ruta (`/page`). También contiene la versión de HTTP – en este caso, 1.1, aunque no tienes que preocuparte por esto. Hay muchos métodos HTTP diferentes que se pueden utilizar, dependiendo de cómo desees interactuar con la página remota. Algunos comunes son `GET` (para recuperar la página remota), `POST` (para enviar datos a la página remota), `PUT` (para crear una página remota) y `DELETE` (para eliminar la página remota). Ten en cuenta que las descripciones de las acciones son algo simplificadas: el servidor remoto puede elegir cómo responde a los diferentes métodos, e incluso desarrolladores experimentados pueden no estar de acuerdo sobre el método correcto a implementar para una acción en particular. También es importante tener en cuenta que incluso si un servidor admite un método en particular, probablemente necesitarás los permisos correctos para realizar esa acción; no puedes simplemente eliminar una página web que no te gusta, por ejemplo.

Al escribir una aplicación web, la gran mayoría de las veces solo manejarás solicitudes `GET`. Cuando comiences a aceptar formularios, también tendrás que usar solicitudes `POST`. Solo cuando trabajes con funciones avanzadas, como la creación de APIs REST, tendrás que preocuparte por `PUT`, `DELETE` y otros métodos.

Haciendo referencia nuevamente a la solicitud de ejemplo, a partir de la línea 2 se muestran los encabezados (*headers*) de la solicitud. Los encabezados contienen metadatos adicionales sobre la solicitud. Cada encabezado está en su propia línea, con el nombre del encabezado y su valor separados por dos puntos. La mayoría son opcionales (excepto `Host`, sobre el cual habrá más información pronto). Los nombres de los encabezados no distinguen entre mayúsculas y minúsculas. A los efectos de este ejemplo, solo mostramos tres encabezados comunes aquí. Veamos los encabezados de ejemplo en orden:
- **Host**: Como se mencionó, este es el único encabezado que se requiere (para HTTP 1.1 o posterior). Es necesario para que el servidor web sepa qué sitio web o aplicación debe responder a la solicitud si hay varios sitios alojados en un solo servidor.
- **User-Agent**: Tu navegador suele enviar al servidor una cadena que identifica su versión y sistema operativo. La aplicación de tu servidor podría usar esto para servir diferentes páginas a diferentes dispositivos (por ejemplo, una página específica para teléfonos inteligentes).
- **Cookie**: Probablemente hayas visto un mensaje al visitar una página web que te informa que está almacenando una cookie en el navegador. Son pequeñas piezas de información que un sitio web puede almacenar en tu navegador y usar para identificarte o guardar configuraciones para cuando regreses al sitio. Si te preguntabas cómo envía tu navegador estas cookies de vuelta al servidor, es a través de este encabezado.

Hay muchos otros encabezados estándar definidos y ocuparía demasiado espacio enumerarlos todos. Se pueden usar para autenticarse en el servidor (`Authorization`), indicarle al servidor qué tipo de datos puedes recibir (`Accept`) o incluso qué idioma deseas para la página (`Accept-Language`, aunque esto solo funcionará si el creador de la página ha puesto el contenido a disposición en el idioma específico que solicitas). Incluso puedes definir tus propios encabezados a los que solo tu aplicación sepa responder.

Ahora, veamos una solicitud un poco más avanzada: una que envía cierta información a un servidor y, por lo tanto (a diferencia del ejemplo anterior), contiene un cuerpo. En este ejemplo, estamos iniciando sesión en una página web enviando un nombre de usuario y una contraseña; por ejemplo, visitas `https://www.example.com/login` y se muestra un formulario para ingresar un nombre de usuario y contraseña. Después de hacer clic en el botón Iniciar sesión (*Login*), esta es la solicitud que se envía al servidor:

```http
POST /login HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

username=user&password=password1
```

Como puedes ver, esto se parece al primer ejemplo, pero hay algunas diferencias. El método ahora es `POST` y se han introducido dos nuevos encabezados (puedes asumir que tu navegador también seguirá enviando los otros encabezados que estaban en el ejemplo anterior):
- **Content-Type**: Esto le indica al servidor el tipo de datos que se incluye en el cuerpo. En el caso de `application/x-www-form-urlencoded`, el cuerpo es un conjunto de pares clave-valor. Un cliente HTTP podría configurar este encabezado para indicarle al servidor si estuviera enviando otros tipos de datos, como JSON o XML.
- **Content-Length**: Para que el servidor sepa cuántos datos leer, el cliente debe indicarle cuántos datos se están enviando. El encabezado `Content-Length` contiene la longitud del cuerpo. Si cuentas la longitud del cuerpo en este ejemplo, verás que tiene 32 caracteres.

Los encabezados siempre están separados del cuerpo por una línea en blanco. Al mirar el ejemplo, deberías poder ver cómo se codifican los datos del formulario en el cuerpo: `username` tiene el valor `user1` y `password` tiene el valor `password1`.

Estas solicitudes fueron bastante simples, pero la mayoría de las solicitudes no se vuelven mucho más complicadas. Pueden tener diferentes métodos y encabezados, pero deben seguir el mismo formato. Ahora que hemos visto las solicitudes, echaremos un vistazo a las respuestas HTTP que regresan del servidor.

Una respuesta HTTP se parece a una solicitud y consta de tres partes principales: un estado, encabezados y un cuerpo. Al igual que una solicitud, dependiendo del tipo de respuesta, es posible que no tenga un cuerpo. El primer ejemplo de respuesta es una respuesta simple y exitosa:

```http
HTTP/1.1 200 OK
Server: nginx
Content-Length: 18132
Content-Type: text/html
Set-Cookie: sessid=abc123def46

<!DOCTYPE html><html><head>…
```

La primera línea contiene la versión de HTTP, un código de estado numérico (`200`) y luego una descripción de texto de lo que significa el código (`OK` – la solicitud fue exitosa). Mostraremos algunos estados más después del siguiente ejemplo. Las líneas 2 a 5 contienen encabezados, similares a una solicitud. Algunos encabezados que quizás hayas visto antes; los explicaremos todos en este contexto:
- **Server**: Esto es similar pero opuesto al encabezado `User-Agent`: este es el servidor que le indica al cliente qué software está ejecutando.
- **Content-Length**: El cliente utiliza este valor para determinar cuántos datos leer del servidor para obtener el cuerpo.
- **Content-Type**: El servidor utiliza este encabezado para indicar al cliente qué tipo de datos está enviando. El cliente puede entonces elegir cómo mostrará los datos; por ejemplo, una imagen debe mostrarse de manera diferente al HTML.
- **Set-Cookie**: Vimos en el primer ejemplo de solicitud cómo un cliente envía una cookie al servidor. Este es el encabezado correspondiente que envía un servidor para configurar esa cookie en el navegador.

Después de los encabezados hay una línea en blanco y luego el cuerpo de la respuesta. No lo hemos mostrado todo aquí, solo los primeros caracteres del HTML que se está recibiendo, de los 18,132 que ha enviado el servidor.

A continuación, mostraremos un ejemplo de una respuesta que se devuelve si no se encuentra la página solicitada:

```http
HTTP/1.1 404 Not Found
Server: nginx
Content-Length: 55
Content-Type: text/html

<!DOCTYPE html><html><body>Page Not Found</body></html>
```

Es similar al ejemplo anterior, pero el estado ahora es `404 Not Found`. Si alguna vez has estado navegando por Internet y recibiste un error 404, este es el tipo de respuesta que recibió tu navegador. Los distintos códigos de estado se agrupan según el tipo de éxito o fracaso que indican:
- **100–199**: El servidor envía códigos en este rango para indicar cambios de protocolo o que se requieren más datos. No tienes que preocuparte por estos códigos de estado.
- **200–299**: Un código de estado en este rango indica un manejo exitoso de una respuesta. Como vimos, el más común con el que lidiarás es `200 OK`.
- **300–399**: Un código de estado en este rango significa que la página que estás solicitando se ha movido a otra dirección. Un ejemplo de esto es un servicio de acortamiento de URLs que te redirigirá de la URL corta a la completa cuando la visites. Las respuestas comunes son `301 Moved Permanently` o `302 Found`. Al enviar una respuesta de redirección, el servidor también incluirá un encabezado `Location` que contiene la URL a la que debes ser redirigido.
- **400–499**: Un código de estado en este rango significa que la solicitud no se pudo manejar porque hubo un problema con lo que envió el cliente. Esto contrasta con una solicitud que no se puede manejar debido a un problema en el servidor (los discutiremos pronto). Ya hemos visto una respuesta `404 Not Found`; esto se debe a una solicitud incorrecta porque el cliente está solicitando un documento que no existe. Algunas otras respuestas comunes son `401 Unauthorized` (el cliente debe iniciar sesión) o `403 Forbidden` (el cliente no tiene permiso para acceder al recurso específico). Ambos problemas podrían evitarse haciendo que el cliente inicie sesión, por lo que se consideran problemas del lado del cliente (solicitud).
- **500–599**: Los códigos de estado en este rango indican un error del lado del servidor. El cliente no debe esperar poder ajustar una solicitud para solucionar el problema. Cuando trabajes con Django, el estado de error de servidor más común que verás es `500 Internal Server Error`. Esto se generará si tu código genera una excepción. Otro común es `504 Gateway Timeout`, que puede ocurrir si tu código tarda demasiado en ejecutarse. Las otras variantes que son comunes de ver son `502 Bad Gateway` y `503 Service Unavailable`, que generalmente significan que hay un problema con el alojamiento de tu aplicación de alguna manera.

Estos son solo algunos de los estados HTTP más comunes. Puedes encontrar una lista más completa en [https://developer.mozilla.org/en-US/docs/Web/HTTP/Status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status). Sin embargo, al igual que los encabezados HTTP, los estados son arbitrarios y una aplicación puede devolver estados personalizados. Depende del servidor y de los clientes decidir qué significan estos estados y códigos personalizados.

Si esta es la primera vez que te presentan el protocolo HTTP, hay bastante información que asimilar. Afortunadamente, Django hace todo el trabajo duro y encapsula los datos entrantes en un objeto `HttpRequest`. La mayoría de las veces no necesitas conocer la mayor parte de la información entrante, pero está disponible si la necesitas. Del mismo modo, al enviar una respuesta, Django encapsula tus datos en un objeto `HttpResponse`. Normalmente, solo configuras el contenido a devolver, pero también tienes la libertad de configurar códigos de estado y encabezados HTTP. Discutiremos cómo acceder y configurar la información en `HttpRequest` y `HttpResponse` más adelante en este capítulo. En la siguiente sección, veremos cómo Django recibe, analiza y responde a una solicitud HTTP.

#### Procesamiento de una solicitud

Esta es una línea de tiempo básica de los flujos de solicitud y respuesta para que puedas tener una idea de lo que hace el código que escribirás en cada etapa. En términos de escritura de código, la primera parte que escribirás es tu vista. La vista que crees realizará algunas acciones, como consultar una base de datos para obtener datos. Luego, la vista pasará estos datos a otra función para renderizar una plantilla, devolviendo finalmente el objeto `HttpResponse` que abarca los datos que deseas enviar de vuelta al cliente.

A continuación, Django necesita saber cómo mapear una URL específica a tu vista para que pueda cargar la vista correcta para la URL que recibe como parte de una solicitud. Escribirás este mapeo de URLs en un archivo de configuración de URLs en Python.

Cuando Django recibe una solicitud, analiza el archivo de configuración de URLs y luego encuentra la vista correspondiente. Llama a la vista, pasando el objeto `HttpRequest` que representa la solicitud. Tu vista devolverá su `HttpResponse`, y luego Django vuelve a tomar el control para enviar estos datos a su servidor web host y de vuelta al cliente que lo solicitó.

*Figura 1.7 – El flujo de solicitud y respuesta*

El flujo de solicitud y respuesta se ilustra en la Figura 1.7; las secciones indicadas como *Your Code* son para el código que tú escribes, y el primer y último paso son atendidos por Django. Django realiza la coincidencia de URLs por ti, llama al código de tu vista y luego se encarga de pasar la respuesta de vuelta al cliente.

En esta sección, aprendimos sobre la estructura de las solicitudes y respuestas HTTP, incluido para qué se utilizan los diferentes tipos de solicitudes. Vimos cómo se pueden utilizar los códigos de estado HTTP para indicar errores en las solicitudes o respuestas HTTP. También obtuvimos una descripción general de cómo Django puede procesar una solicitud HTTP. En la siguiente sección, exploraremos la estructura del proyecto Django y aprenderemos para qué se utilizan los distintos archivos y directorios.

---

### Sección: Exploración de la estructura del proyecto Django

Ya presentamos los proyectos de Django en la sección *Estructuración de un proyecto y una aplicación de Django (Scaffolding)*. Recordemos lo que sucede cuando ejecutamos `startproject` (para un proyecto llamado `myproject`): el comando crea un directorio `myproject` con un archivo llamado `manage.py`, y un directorio llamado `myproject` (esto coincide con el nombre del proyecto; en el Ejercicio 1.01, esta carpeta se llamó `bookr`, al igual que el proyecto). El diseño del directorio se muestra en la Figura 1.8. Ahora examinaremos el archivo `manage.py` y el contenido del paquete `myproject` con más detalle.

*Figura 1.8 – El directorio del proyecto para myproject*

Como sugiere el nombre, `manage.py` es un script que se utiliza para administrar tu proyecto de Django. La mayoría de los comandos que se utilizan para interactuar con tu proyecto se proporcionarán a este script en la línea de comandos. Los comandos se proporcionan como un argumento para este script; por ejemplo, si queremos ejecutar el comando `manage.py runserver`, significaría ejecutar el script `manage.py` de esta manera:

```bash
python manage.py runserver
```

Hay varios comandos útiles que proporciona `manage.py`. Se te presentarán con más detalle a lo largo del libro; algunos de los más comunes son los siguientes:
- `runserver`: Inicia el servidor HTTP de desarrollo de Django para servir la aplicación Django en tu computadora local.
- `startapp`: Crea una nueva aplicación Django en tu proyecto. Hablaremos sobre qué son las aplicaciones con más profundidad pronto.
- `shell`: Inicia un intérprete de Python con las configuraciones de Django precargadas. Esto es útil para interactuar con tu aplicación sin tener que cargar manualmente tus configuraciones de Django.
- `dbshell`: Inicia una consola interactiva conectada a tu base de datos, utilizando los parámetros predeterminados de tu configuración de Django. Puedes ejecutar consultas SQL manuales de esta manera.
- `makemigrations`: Genera instrucciones de cambio de base de datos a partir de las definiciones de tus modelos. Aprenderás qué significa esto y cómo usar este comando en el Capítulo 2.
- `migrate`: Aplica las migraciones generadas por el comando `makemigrations`. También usarás esto en el Capítulo 2.
- `test`: Ejecuta pruebas automatizadas que hayas escrito. Usarás este comando en el Capítulo 14.

Una lista completa de todos los comandos está disponible en [https://docs.djangoproject.com/en/3.0/ref/django-admin/](https://docs.djangoproject.com/en/3.0/ref/django-admin/).

En las siguientes secciones, exploraremos el contenido del directorio del proyecto, aprenderemos qué contienen los directorios de aplicaciones de Django y veremos cómo cargar tu proyecto en PyCharm.

#### El directorio myproject

Además del archivo `manage.py`, el otro elemento de archivo creado por `startproject` es el directorio `myproject`. Este es el paquete de Python real para tu proyecto. Contiene configuraciones para el proyecto, algunos archivos de configuración para tu servidor web y los mapas de URLs globales. Dentro del directorio `myproject` hay cinco archivos:
- `__init__.py`: Este es un archivo vacío que le permite a Python saber que el directorio `myproject` es un módulo de Python. Estarás familiarizado con estos archivos si has trabajado con Python antes.
- `settings.py`: Contiene todas las configuraciones de Django para tu aplicación. Explicaremos el contenido pronto.
- `urls.py`: Contiene los mapeos de URLs globales que Django utilizará inicialmente para ubicar vistas u otros mapeos de URLs secundarios. Pronto añadirás un mapa de URLs a este archivo.
- `asgi.py` y `wsgi.py`: Estos archivos son los que utilizan los servidores web ASGI o WSGI para comunicarse con tu aplicación Django cuando la implementas en un servidor web de producción. Normalmente no necesitas editarlos en absoluto y no se utilizan en el desarrollo diario. Su uso se discutirá más en el Capítulo 18.

#### Servidor de desarrollo de Django

Ya iniciaste el servidor de desarrollo de Django en el Ejercicio 1.01. Como mencionamos anteriormente, es un servidor web diseñado para ejecutarse únicamente en la máquina de un desarrollador durante el desarrollo. No está diseñado para su uso en producción.

De forma predeterminada, el servidor escucha en el puerto 8000 en `localhost` (`127.0.0.1`), pero esto se puede cambiar añadiendo un número de puerto, o dirección y número de puerto, después del argumento `runserver`:

```bash
python manage.py runserver 8001
```

Esto hará que el servidor escuche en el puerto 8001 en `localhost` (`127.0.0.1`).

También puedes hacer que escuche en una dirección específica si tu computadora tiene más de una, o `0.0.0.0` para todas las direcciones:

```bash
python manage.py runserver 0.0.0.0:8000
```

Esto hará que el servidor escuche en todas las direcciones de tu computadora en el puerto 8000, lo que puede ser útil si deseas probar tu aplicación desde otra computadora o tu teléfono inteligente.

El servidor de desarrollo observa el directorio de tu proyecto de Django y se reiniciará automáticamente cada vez que guardes un archivo, de modo que cualquier cambio de código que realices se recargue automáticamente en el servidor. Sin embargo, aún debes actualizar manualmente tu navegador para ver los cambios allí.

Cuando quieras detener el comando `runserver`, puedes hacerlo de la forma habitual para detener procesos en la terminal: presionando `Ctrl + C`.

#### Aplicaciones de Django (Django apps)

Ahora que hemos cubierto la mayor parte de la teoría sobre las aplicaciones, podemos ser más específicos sobre su propósito. El directorio de una aplicación contiene todos los modelos, vistas, plantillas (y más) que necesita para proporcionar la funcionalidad de la aplicación. Un proyecto de Django contendrá al menos una aplicación (a menos que haya sido muy personalizado para no depender de una gran cantidad de funciones de Django). Si está bien diseñada, una aplicación debería poder eliminarse de un proyecto y trasladarse a otro proyecto sin modificaciones. Por lo general, una aplicación contendrá modelos para un solo dominio de diseño, y esta puede ser una forma útil de determinar si tu aplicación debe dividirse en varias aplicaciones.

Tu aplicación puede tener cualquier nombre siempre que sea un nombre de módulo de Python válido (es decir, solo letras, números y guiones bajos) y no entre en conflicto con otros archivos en el directorio de tu proyecto. Por ejemplo, como hemos visto, ya hay un directorio llamado `myproject` en el directorio del proyecto (que contiene el archivo `settings.py`), por lo que no podrías tener una aplicación llamada `myproject`. Como vimos en el Ejercicio 1.01, la creación de una aplicación utiliza el comando `manage.py startapp appname`, como se muestra aquí:

```bash
python manage.py startapp myapp
```

El comando `startapp` crea un directorio dentro de tu proyecto con el nombre de la aplicación especificada. También estructura los archivos para la aplicación. Dentro del directorio de la aplicación hay varios archivos y una carpeta, como se muestra en la Figura 1.9:

*Figura 1.9 – El contenido del directorio de la aplicación myapp*

- `__init__.py`: Un archivo vacío que indica que este directorio es un módulo de Python.
- `admin.py`: Django tiene un sitio de administración incorporado para ver y editar datos con una GUI. En este archivo, definirás cómo se exponen los modelos de tu aplicación en el sitio de administración de Django. Cubriremos esto con más detalle en el Capítulo 4.
- `apps.py`: Contiene algunas configuraciones para los metadatos de tu aplicación. No necesitarás editar este archivo.
- `models.py`: Aquí es donde definirás los modelos para tu aplicación. Leerás sobre esto con más detalle en el Capítulo 2.
- `migrations`: Django utiliza archivos de migración para registrar automáticamente los cambios en tu base de datos subyacente a medida que cambian los modelos. Django los genera cuando ejecutas el comando `manage.py makemigrations` y se almacenan en este directorio. No se aplican a la base de datos hasta que ejecutas `manage.py migrate`. También se cubrirán en el Capítulo 2.
- `tests.py`: Para comprobar que tu código se comporta correctamente, Django admite la escritura de pruebas (unitarias, funcionales o de integración) y las buscará dentro de este archivo. Escribiremos algunas pruebas a lo largo de este libro y cubriremos las pruebas en detalle en el Capítulo 14.
- `views.py`: Tus vistas de Django (el código que responde a las solicitudes HTTP) irán aquí. Pronto crearás una vista básica y las vistas se cubrirán con más detalle en el Capítulo 3.

Examinaremos el contenido de estos archivos con más detalle más adelante, pero por ahora, pondremos en marcha PyCharm.

#### Configuración de PyCharm

Confirmamos en el Ejercicio 1.01 que el proyecto `bookr` se configuró correctamente (dado que el servidor de desarrollo se ejecuta correctamente), por lo que ahora podemos comenzar a usar PyCharm para ejecutar y editar nuestro proyecto. PyCharm es un IDE para el desarrollo de Python e incluye características como autocompletado de código, formato de estilo automático y un depurador integrado. Luego usaremos PyCharm para comenzar a escribir nuestros propios mapas de URLs, vistas y plantillas. También se utilizará para iniciar y detener el servidor de desarrollo, lo que permitirá depurar tu código estableciendo puntos de interrupción (*breakpoints*).

#### Ejercicio 1.02 – Configuración del proyecto en PyCharm

En este ejercicio, abriremos el proyecto `bookr` en PyCharm y configuraremos el intérprete del proyecto para que PyCharm pueda ejecutar y depurar el proyecto. Sigue estos pasos:

1. Abre PyCharm. La primera vez que abras PyCharm, se te mostrará la pantalla *Welcome to PyCharm*, que te pregunta qué deseas hacer:
   *Figura 1.10 – La pantalla de bienvenida de PyCharm*

2. Haz clic en **Open**, luego navega hasta el proyecto `bookr` que acabas de crear y ábrelo. Asegúrate de estar abriendo el directorio del proyecto `bookr` y no el directorio del paquete `bookr` que se encuentra dentro.

3. Se te preguntará si deseas confiar en el proyecto `bookr`. Como acabas de crearlo, es seguro, así que haz clic en **Trust Project**:
   *Figura 1.11 – El cuadro de diálogo Trust Project de PyCharm*

4. Si no has utilizado PyCharm antes, te preguntará sobre qué configuraciones y temas deseas usar, y una vez que hayas respondido a todas esas preguntas, verás la estructura de tu proyecto `bookr` abierta en el panel *Project* a la izquierda de la ventana:
   *Figura 1.12 – El panel Project de PyCharm*
   Tu panel *Project* debería verse como la Figura 1.13 y mostrar los directorios `bookr` y `reviews`, y el archivo `manage.py`. Si no los ves y en su lugar ves `asgi.py`, `settings.py`, `urls.py` y `wsgi.py`, entonces has abierto el directorio del paquete `bookr` en su lugar. Selecciona **File | Open** y luego navega y abre el directorio del proyecto `bookr`.

5. Antes de que PyCharm sepa cómo ejecutar tu proyecto para iniciar el servidor de desarrollo de Django, el intérprete debe configurarse con el binario de Python dentro de tu entorno virtual. Esto se hace primero agregando el intérprete a la configuración global del intérprete. Abre la ventana *Preferences* (macOS) o *Settings* (Windows/Linux) dentro de PyCharm:
   - macOS: Menú de PyCharm | Preferences
   - Windows y Linux: File | Settings

6. En el panel de lista de preferencias de la izquierda, abre el elemento **Project: bookr** y luego haz clic en **Python Interpreter**:
   *Figura 1.13 – La configuración de Python Interpreter*

7. A veces, PyCharm puede determinar automáticamente los entornos virtuales, por lo que en este caso, es posible que el intérprete del proyecto ya esté poblado con el intérprete correcto. Si es así y ves Django en la lista de paquetes, haz clic en **OK** para cerrar la ventana y completar este ejercicio.

8. En la mayoría de los casos, sin embargo, el intérprete de Python debe configurarse manualmente. Haz clic en el icono de engranaje junto al menú desplegable *Python Interpreter* y luego haz clic en **Add…**.

9. Ahora se muestra la ventana *Add Python Interpreter*. Selecciona el botón de radio **Existing Environment** y luego haz clic en los puntos suspensivos (`…`) junto a la selección *Interpreter*. Luego debes navegar y seleccionar el intérprete de Python para tu entorno virtual.
   *Figura 1.14 – La ventana Add Python Interpreter*
   En macOS (asumiendo que llamaste al entorno virtual `djangoenv`), la ruta suele ser `/Users/<yourusername>/.pyenv/djangoenv/bin/python`. De manera similar, en Linux, debería estar en `/home/<yourusername>/.pyenv/djangoenv/bin/python`.
   Si no estás seguro, puedes ejecutar el comando `which python` en la terminal donde ejecutaste previamente el comando `python manage.py`, y te indicará la ruta al intérprete de Python:
   ```bash
   which python
   /Users/chrisguest/.pyenv/djangoenv/bin/python
   ```
   En Windows, estará donde hayas creado tu entorno virtual con el comando `pyenv`.

10. Después de seleccionar el intérprete, tu ventana *Add Python Interpreter* debería verse como la Figura 1.15.

11. Haz clic en **OK** para cerrar la ventana *Add Python Interpreter*. Ahora deberías ver la ventana principal de preferencias, y Django (y otros paquetes en tu entorno virtual) aparecerán en la lista (ver Figura 1.16).
    *Figura 1.15 – Se enumeran los paquetes en el entorno virtual*

12. Haz clic en **OK** en la ventana principal de Preferencias para cerrarla. PyCharm ahora se tomará unos segundos para indexar tu entorno y las bibliotecas instaladas. Puedes ver el proceso en la barra de estado inferior derecha. Espera a que finalice este proceso y la barra de progreso desaparecerá.

13. Para ejecutar el servidor de desarrollo de Django, es necesario configurar Python con una configuración de ejecución (*run configuration*). La configuraremos ahora:
    - Haz clic en **Add Configuration…** en la parte superior derecha de la ventana del proyecto PyCharm para abrir la ventana *Run/Debug Configurations*:
      *Figura 1.16 – El botón Add Configuration… en la parte superior derecha de la ventana de PyCharm*
    - Haz clic en el botón `+` en la parte superior izquierda de esta ventana y selecciona **Python** en el menú desplegable.
      *Figura 1.17 – Adición de una nueva configuración de Python en la ventana Run/Debug Configurations*
    - Se mostrará un nuevo panel de configuración con campos sobre cómo ejecutar tu proyecto a la derecha de la ventana. Continúa y completa los campos de la siguiente manera:
      - El campo **Name** puede ser cualquier cosa, pero debe ser algo comprensible. Ingresa `Django Dev Server`.
      - **Script path** es la ruta a tu archivo `manage.py`. Si haces clic en el icono de carpeta en este campo, puedes explorar tu sistema de archivos para seleccionar el archivo `manage.py` dentro del directorio del proyecto `bookr`.
      - **Parameters** se refiere a los argumentos que vienen después del script `manage.py`, igual que si lo estuviéramos ejecutando desde la línea de comandos. Usaremos el mismo argumento aquí para iniciar el servidor, así que ingresa `runserver`.
      - Como se mencionó anteriormente, el comando `runserver` también puede aceptar un argumento para el puerto o la dirección a escuchar. Si lo deseas, puedes añadir este argumento después de `runserver` en el mismo campo *Parameters*.
      - La configuración de **Python Interpreter** debería haberse establecido automáticamente en la que se configuró en el Ejercicio 1.02. De lo contrario, haz clic en la flecha desplegable a la derecha para seleccionarla.
      - La configuración de **Working directory** debe ser el directorio del proyecto `bookr`. Es probable que esto ya se haya configurado correctamente.
      - Asegúrate de que las opciones **Add content roots to PYTHONPATH** y **Add source roots to PYTHONPATH** estén marcadas. Esto asegurará que PyCharm añada el directorio de tu proyecto `bookr` a `PYTHONPATH` (la lista de rutas que busca el intérprete de Python al cargar un módulo). Sin estas casillas marcadas, las importaciones de tu proyecto no funcionarán correctamente.
    *Figura 1.18 – Los ajustes de configuración*

14. Comprueba que tu ventana *Run/Debug Configurations* se vea similar a la Figura 1.19 y luego haz clic en **OK** para guardar la configuración.

15. Ahora, en lugar de iniciar el servidor de desarrollo de Django en una terminal, haz clic en el icono de **Play** en la parte superior derecha de la ventana del proyecto para iniciarlo (ver Figura 1.20).
    *Figura 1.19 – Configuración del servidor de desarrollo de Django con los botones Play, Debug y Stop*

16. Haz clic en el icono de Play para iniciar el servidor de desarrollo de Django. Asegúrate de detener cualquier otra instancia del servidor de desarrollo de Django que se esté ejecutando (como en una terminal); de lo contrario, el que estás iniciando no podrá conectarse al puerto 8000 y no se iniciará.

17. Se abrirá una consola en la parte inferior de la ventana de PyCharm, que mostrará una salida que indica que el servidor de desarrollo se ha iniciado (Figura 1.21).
    *Figura 1.20 – La consola con el servidor de desarrollo de Django en ejecución*

18. Abre un navegador web y navega hasta `http://127.0.0.1:8000`. Deberías ver la misma pantalla de ejemplo de Django que viste anteriormente, en el Ejercicio 1.01 (Figura 1.21), lo que confirmará que, una vez más, todo está configurado correctamente.

En este ejercicio, abrimos el proyecto Bookr en PyCharm y luego configuramos el intérprete de Python para nuestro proyecto. Luego añadimos una configuración de ejecución en PyCharm, lo que nos permite iniciar y detener el servidor de desarrollo de Django desde PyCharm. También podremos depurar nuestro proyecto más adelante ejecutándolo dentro del depurador de PyCharm.

En esta sección, exploramos la estructura de carpetas de Django y vimos el propósito de los directorios y carpetas estructurados. También vimos cómo crear una aplicación de Django y los archivos que contiene. Finalmente, pusimos en marcha un proyecto de Django en PyCharm.

---

### Sección: Introducción a las vistas de Django

Ahora tienes todo configurado para comenzar a escribir tus propias vistas de Django y configurar las URLs que se asignarán a ellas. Como vimos anteriormente en este capítulo, una vista es simplemente una función que toma una instancia de `HttpRequest` (construida por Django) y (opcionalmente) algunos parámetros de la URL. Luego realizará algunas operaciones, como obtener datos de una base de datos. Finalmente, devuelve `HttpResponse`.

Para usar nuestro proyecto `bookr` como ejemplo, podríamos tener una vista que recibe una solicitud de un libro determinado. Consulta la base de datos para este libro y luego devuelve una respuesta que contiene una página HTML que muestra información sobre el libro. Otra vista podría recibir una solicitud para enumerar todos los libros y luego devolver una respuesta con otra página HTML que contenga esta lista. Las vistas también pueden crear o modificar datos; otra vista podría recibir una solicitud para crear un nuevo libro, y luego agregaría el libro a la base de datos y devolvería una respuesta con HTML que muestra la información del nuevo libro.

En este capítulo, solo usaremos funciones como vistas, pero Django también admite **vistas basadas en clases** (*Class-Based Views*) que te permiten aprovechar los paradigmas orientados a objetos (como la herencia). Esto te permite simplificar el código utilizado en múltiples vistas que tienen la misma lógica de negocio. Por ejemplo, es posible que desees mostrar todos los libros o solo los libros de una editorial determinada. Ambas vistas necesitan consultar una lista de libros de la base de datos y representarlos en una plantilla de lista de libros. Una clase `View` podría heredar de otra e implementar la obtención de datos de manera diferente y dejar el resto de la funcionalidad (como el renderizado) idéntica. Las vistas basadas en clases pueden ser más potentes pero también más difíciles de aprender. Se presentarán más adelante, en el Capítulo 11, cuando tengas más experiencia con Django.

La instancia de `HttpRequest` que se pasa a la vista contiene todos los datos relacionados con la solicitud, con atributos como los siguientes:
- **method**: Una cadena que contiene el método HTTP que utilizó el navegador para solicitar la página, generalmente `GET`, pero será `POST` si un usuario ha enviado un formulario. Puedes usar esto para cambiar el flujo de la vista; por ejemplo, mostrar un formulario vacío en `GET`, o validar y procesar el envío de un formulario en `POST`.
- **GET**: Esta es una clase `QueryDict` que contiene los parámetros utilizados en la cadena de consulta de la URL (*query string*). Esta es la parte de la URL después de `?`, si contiene una. Profundizaremos en las clases `QueryDict` pronto. Ten en cuenta que este atributo siempre está disponible, incluso si la solicitud no fue `GET`.
- **POST**: Este es otro `QueryDict` que contiene los parámetros enviados a la vista en una solicitud `POST`, como el envío de un formulario. Por lo general, usarías esto junto con un formulario de Django, que se cubrirá en el Capítulo 6, *Formularios*.
- **headers**: Este es un diccionario con claves que no distinguen entre mayúsculas y minúsculas con los encabezados HTTP de la solicitud. Por ejemplo, podrías variar la respuesta con contenido diferente para diferentes navegadores según el encabezado `User-Agent`. Analizamos algunos encabezados HTTP que envía el cliente anteriormente en este capítulo.
- **path**: Esta es la ruta utilizada en la solicitud. Normalmente no necesitas examinar esto porque Django analizará automáticamente la ruta y la pasará a la función de vista como parámetros, pero puede ser útil en algunos casos.

Aún no utilizaremos todos estos atributos y hay otros que se presentarán más adelante, pero ahora puedes ver qué papel juega el argumento `HttpRequest` en tu vista.

A continuación, veremos cómo determina Django qué vista utilizar para una URL específica.

---

### Sección: Exploración en detalle del mapeo de URLs

Mencionamos brevemente los mapas de URLs anteriormente, en la sección *Procesamiento de una solicitud*. Django no sabe automáticamente qué función de vista debe ejecutarse cuando recibe una solicitud para una URL en particular. La función del mapeo de URLs es crear un vínculo entre una URL y una vista. Por ejemplo, en Bookr, es posible que desees asignar la URL `/books/` a una vista `books_list` que hayas creado.

El mapeo de URL a vista se define en el archivo que Django creó automáticamente, llamado `urls.py`, dentro del directorio del paquete `bookr` (aunque se puede configurar un archivo diferente en `settings.py`; habrá más información sobre eso más adelante).

Este archivo contiene una variable, `urlpatterns`, que es una lista de rutas que Django evalúa por turno hasta que encuentra una coincidencia para la URL que se solicita. La coincidencia se resolverá en una función de vista o en otro archivo `urls.py`, que también contiene una variable `urlpatterns`, que se resolverá de la misma manera. Los archivos de URL se pueden encadenar de esta manera todo el tiempo que desees. De esta manera, puedes dividir los mapas de URL en archivos separados (como uno o más por aplicación) para que no se vuelvan demasiado grandes. Una vez que se encuentra una vista, Django la llama con `HttpRequest` y cualquier parámetro analizado de la URL.

Las reglas se establecen llamando a la función `path`, que toma la ruta de la URL como primer argumento. La ruta puede contener parámetros con nombre que se pasarán a una vista como parámetros de función. Su segundo argumento es una vista u otro archivo que también contiene `urlpatterns`.

También existe la función `re_path`, que es similar a `path`, excepto que toma una expresión regular como primer argumento para una configuración más avanzada. Hay mucho más sobre el mapeo de URLs; se cubrirá en el Capítulo 3. Para ilustrar estos conceptos, la Figura 1.22 muestra el archivo `urls.py` predeterminado que genera Django:

*Figura 1.21 – El archivo urls.py predeterminado*

Puedes ver la variable `urlpatterns`, que enumera todas las URLs que están configuradas. Actualmente, solo hay una regla configurada, que asigna cualquier ruta que comience con `admin/` a los mapas de URL de administración (el módulo `admin.site.urls`). Este no es un mapeo a una vista; en su lugar, es un ejemplo de encadenamiento de mapas de URLs: el módulo `admin.site.urls` definirá el resto de las rutas (después de `admin/`) que se asignan a las vistas de administración. Cubriremos el sitio de administración de Django más adelante en el libro, a partir del Capítulo 4.

Ahora escribiremos una vista y configuraremos un mapa de URL hacia ella, para verlos en acción.

#### Ejercicio 1.03 – Creación de una vista y mapeo de una URL hacia ella

Nuestra primera vista será muy sencilla y simplemente devolverá texto estático. En este ejercicio, veremos cómo escribir una vista y configurar un mapa de URL para que se resuelva en una vista:

1. A medida que realizas cambios en los archivos de tu proyecto y los guardas, es posible que veas que el servidor de desarrollo de Django se reinicia automáticamente en la terminal o consola en la que se está ejecutando. Esto es normal: se reinicia automáticamente para cargar cualquier cambio de código que realices. Ten en cuenta también que no aplicará cambios automáticamente a la base de datos si editas modelos o migraciones; habrá más sobre esto en el Capítulo 2.

2. En PyCharm, expande la carpeta `reviews` en el explorador de proyectos a la izquierda y luego haz doble clic en el archivo `views.py` que se encuentra dentro para abrirlo. En el panel derecho (editor) de PyCharm, deberías ver el texto de marcador de posición generado automáticamente por Django:
   ```python
   from django.shortcuts import render

   # Create your views here.
   ```

3. Elimina este texto de marcador de posición de `views.py` y, en su lugar, inserta este contenido:
   ```python
   from django.http import HttpResponse

   def index(request):
       return HttpResponse("Hello, world!")
   ```
   Primero, la clase `HttpResponse` debe importarse desde `django.http`. Esto es lo que se utiliza para crear la respuesta que vuelve al navegador web. También puedes usarlo para controlar cosas como los encabezados HTTP o el código de estado. Por ahora, solo usará los encabezados predeterminados y el código de estado 200 Success. Su primer argumento es el contenido de la cadena que se enviará como cuerpo de la respuesta.
   Luego, la función de vista devuelve una instancia de `HttpResponse` con el contenido que definimos (`Hello, world!`).

4. Ahora configuraremos un patrón de URL que asigne la vista `index`. Esto será muy simple y no contendrá ningún parámetro. Expande el directorio `bookr` en el panel Proyecto y luego abre `urls.py`. Django ha generado automáticamente este archivo. Por ahora, solo agregaremos una URL simple para reemplazar el índice predeterminado que proporciona Django. Importa tus vistas en el archivo `urls.py` agregando esta línea después de las otras importaciones existentes:
   ```python
   import reviews.views
   ```

5. Añade un mapa a la vista `index` en la lista `urlpatterns` agregando una llamada a la función `path`, con una cadena vacía y una referencia a la función `index`:
   ```python
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', reviews.views.index),
   ]
   ```
   Asegúrate de no agregar paréntesis después de la función `index` (es decir, debe ser `reviews.views.index` y no `reviews.views.index()`), ya que estamos pasando una referencia a una función en lugar de llamarla.

6. Vuelve a tu navegador web y actualiza. La pantalla de bienvenida predeterminada de Django debería reemplazarse con el texto definido en la vista, `Hello, world!`.
   *Figura 1.22 – El navegador web ahora debería mostrar el mensaje Hello, world!*

Acabamos de ver cómo escribir una función de vista y asignarle una URL. Luego probamos la vista cargándola en un navegador web.

A continuación, veremos cómo trabajar con datos que se envían en una solicitud HTTP, pero no en la ruta de la URL.

---

### Sección: Trabajo con objetos GET, POST y QueryDict

Los datos pueden llegar a través de una solicitud HTTP como parámetros en una URL o dentro del cuerpo de una solicitud `POST`. Es posible que hayas notado parámetros en una URL al navegar por la web (el texto después de `?`), por ejemplo, `http://www.example.com/?parameter1=value1&parameter2=value2`. También vimos anteriormente en este capítulo un ejemplo de datos de formulario en una solicitud `POST` para iniciar sesión a un usuario (el cuerpo de la solicitud era `username=user&password=password1`).

Django analiza automáticamente estas cadenas de parámetros en objetos `QueryDict`. Luego, los datos están disponibles en el objeto `HttpRequest` que se pasa a tu vista, específicamente, en los atributos `HttpRequest.GET` y `HttpRequest.POST` para parámetros de URL y parámetros de cuerpo, respectivamente. Los objetos `QueryDict` se comportan en su mayoría como diccionarios, excepto que pueden contener múltiples valores para una clave.

Para mostrar diferentes métodos de acceso a elementos, utilizaremos un `QueryDict` simple (la variable `qd`) con solo una clave (`k`) como ejemplo. El elemento `k` tiene tres valores en una lista: las cadenas `a`, `b` y `c`. Los siguientes fragmentos de código muestran la salida de un intérprete de Python.

Primero, la variable `qd` de `QueryDict` se construye a partir de una cadena de parámetros:

```python
>>> from django.http.request import QueryDict
>>> qd = QueryDict("k=a&k=b&k=c")
```

Al acceder a elementos con la notación de corchetes o el método `get`, se devuelve el último valor de esa clave:

```python
>>> qd["k"]
'c'
>>> qd.get("k")
'c'
```

Para acceder a todos los valores de una clave, se debe utilizar `getlist`, de la siguiente manera:

```python
>>> qd.getlist("k")
['a', 'b', 'c']
```

`getlist` siempre devolverá una lista; estará vacía si la clave no existe:

```python
>>> qd.getlist("bad key")
[]
```

Si bien `getlist` no genera una excepción para las claves que no existen, acceder a una clave que no existe con la notación de corchetes generará `KeyError`, como un diccionario normal. Utiliza el método `get` para evitar este error.

Los objetos `QueryDict` para `GET` y `POST` son inmutables (no se pueden cambiar), por lo que se debe utilizar el método `copy` para obtener una copia mutable si necesitas cambiar sus valores, como se muestra a continuación:

```python
>>> qd["k"] = "d"
AttributeError: This QueryDict instance is immutable
>>> qd2 = qd.copy()
>>> qd2
<QueryDict: {'k': ['a', 'b', 'c']}>
>>> qd2["k"] = "d"
>>> qd2["k"]
"d"
```

Para dar un ejemplo de cómo se completa `QueryDict` a partir de una URL, imaginemos una URL de ejemplo: `http://127.0.0.1:8000?val1=a&val2=b&val2=c&val3`.

Detrás de escena, Django pasa la consulta de la URL (todo después de `?`) para crear una instancia de un objeto `QueryDict` y adjuntarlo a `request`, que se pasa a la función de vista. Es algo como esto:

```python
request.GET = QueryDict("val1=a&val2=b&val2=c&val3")
```

Recuerda, esto se hace con `request` antes de que lo recibas dentro de tu función de vista, por lo que no necesitas hacer esto.

En el caso de la URL del ejemplo anterior, podemos acceder a los parámetros dentro de la función de vista de la siguiente manera:

```python
request.GET["val1"]
```

Utilizando el acceso estándar al diccionario, devolvería el valor `"a"`:

```python
request.GET["val2"]
```

Nuevamente, utilizando el acceso estándar al diccionario, hay dos valores establecidos para la clave `val2`, por lo que devolvería el último valor, `"c"`:

```python
request.GET.getlist("val2")
```

Esto devolvería una lista de todos los valores de `val2`: `["b", "c"]`:

```python
request.GET["val3"]
```

Esta clave está en la cadena de consulta pero no tiene ningún valor establecido, por lo que devuelve una cadena vacía:

```python
request.GET["val4"]
```

Esta clave no está configurada, por lo que se generará `KeyError`. Utiliza `request.GET.get("val4")` en su lugar, que devolverá `None`:

```python
request.GET.getlist("val4")
```

Dado que esta clave no está configurada como una lista vacía, se devolverá `([])`.

Ahora veremos `QueryDict` en acción utilizando los parámetros `GET`. Examinaremos los parámetros `POST` más adelante en el Capítulo 6.

#### Ejercicio 1.04 – Exploración de valores GET y objetos QueryDict

Ahora realizaremos algunos cambios en nuestra vista `index` del ejercicio anterior para leer los valores de la URL en el atributo `GET`, y luego experimentaremos pasando diferentes parámetros para ver el resultado:

1. Abre el archivo `views.py` en PyCharm. Añade una nueva variable llamada `name`, que lee el nombre del usuario de los parámetros `GET`. Añade esta línea después de la definición de la función `index`:
   ```python
   name = request.GET.get("name") or "world"
   ```

2. Cambia el valor de retorno para que el nombre se utilice como parte del contenido que se devuelve:
   ```python
   return HttpResponse(f"Hello, {name}!")
   ```

3. Visita `http://127.0.0.1:8000` en tu navegador. Observa que la página todavía dice `Hello, world!`. Esto se debe a que no hemos proporcionado un parámetro `name`. Puedes añadir tu nombre a la URL; por ejemplo, `http://127.0.0.1:8000?name=Ben`. Esto se puede ver en la siguiente captura de pantalla:
   *Figura 1.23 – Configuración del nombre en la URL*

4. Intenta añadir dos nombres; por ejemplo, `http://127.0.0.1:8000?name=Ben&name=John`. Como mencionamos, el último valor del parámetro se recupera con la función `get`, por lo que deberías ver `Hello, John!`:
   *Figura 1.24 – Configuración de múltiples nombres en la URL*

5. Intenta no configurar ningún nombre, así: `http://127.0.0.1:8000?name=`. La página debería volver a mostrar `Hello, world!`:
   *Figura 1.25 – Sin nombre establecido en la URL*

Quizás te preguntes por qué configuramos `name` con el valor predeterminado `world` usando `or`, en lugar de pasar `'world'` como el valor predeterminado a `get`. Considera lo que sucedió en el paso 5, cuando pasamos un valor en blanco para el parámetro `name`. Si hubiéramos pasado `'world'` como valor predeterminado para `get`, la función `get` aún habría devuelto una cadena vacía. Esto se debe a que se establece un valor para `name`; es solo que está en blanco. Ten esto en cuenta al desarrollar tus vistas, ya que existe una diferencia entre no tener ningún valor establecido y tener un valor en blanco establecido. Dependiendo de tu caso de uso, puedes optar por pasar el valor predeterminado a `get`.

En este ejercicio, recuperamos valores de la URL en nuestra vista utilizando el atributo `GET` de la solicitud entrante. Vimos cómo configurar valores predeterminados y qué valor se recupera si se configuran múltiples valores para el mismo parámetro. La vista devolvió `HttpResponse`, que contenía el mensaje que queríamos enviar al navegador.

En la siguiente sección, aprenderemos sobre cómo actualizar y usar las configuraciones de Django.

---

### Sección: Exploración de la configuración de Django (Settings)

Todavía no hemos visto cómo almacena Django sus configuraciones. Ahora que hemos visto las diferentes partes de Django, es un buen momento para examinar el archivo `settings.py`. Hay muchas configuraciones que contiene Django que se pueden usar para personalizarlo. Se creó un archivo `settings.py` predeterminado cuando iniciaste el proyecto `bookr`. Discutiremos algunas de las configuraciones más importantes en el archivo ahora, y algunas otras que podrían ser útiles a medida que te vuelvas más fluido con Django. Debes abrir tu archivo `settings.py` en PyCharm y seguirlo para que puedas ver dónde y cuáles son los valores para tu proyecto.

Cada configuración en este archivo es simplemente una variable global del archivo. El orden en el que discutiremos las configuraciones es el mismo orden en el que aparecen en este archivo, aunque podemos omitir algunas; por ejemplo, existe la configuración `ALLOWED_HOSTS` entre `DEBUG` e `INSTALLED_APPS`, que no cubriremos en esta parte del libro (la verás en el Capítulo 18):

```python
SECRET_KEY = '…'
```

Este es un valor generado automáticamente que no debe compartirse con nadie. Se utiliza para hash, tokens y otras funciones criptográficas. Si tuvieras sesiones existentes en una cookie y cambiaras este valor, las sesiones ya no serían válidas.

```python
DEBUG = True
```

Con este valor establecido en `True`, Django mostrará automáticamente excepciones en el navegador para permitirte depurar cualquier problema que encuentres. Debe establecerse en `False` al implementar tu aplicación en producción.

```python
INSTALLED_APPS = […]
```

Cuando escribes tus propias aplicaciones de Django (como la aplicación `reviews`) o instalas aplicaciones de terceros (que se cubrirán en el Capítulo 15), se deben añadir a esta lista. Como hemos visto, no es estrictamente necesario añadirlas aquí (nuestra vista `index` funcionó sin que nuestra aplicación `reviews` estuviera en esta lista). Sin embargo, para que Django pueda encontrar automáticamente las plantillas, archivos estáticos, migraciones y otras configuraciones de la aplicación, debe aparecer aquí:

```python
ROOT_URLCONF = 'bookr.urls'
```

Este es el módulo de Python que Django cargará primero para encontrar URLs. Ten en cuenta que es el archivo al que agregamos nuestro mapa de URL de vista `index` anteriormente.

```python
TEMPLATES = […]
```

En este momento, no es demasiado importante entender todo en esta configuración, ya que no la cambiarás; la línea importante a señalar es esta:

```python
'APP_DIRS': True,
```

Esto le indica a Django que debe buscar en un directorio `templates` dentro de cada `INSTALLED_APP` al cargar una plantilla para renderizar. Todavía no tenemos un directorio `templates` para `reviews`, pero agregaremos uno en el próximo ejercicio.

Django tiene más configuraciones disponibles que no figuran en el archivo `settings.py`, por lo que usará sus valores predeterminados integrados en estos casos. También puedes usar el archivo para establecer configuraciones arbitrarias que inventes para tu aplicación. Es posible que las aplicaciones de terceros también deseen que se agreguen configuraciones aquí. En capítulos posteriores, agregaremos configuraciones aquí para otras aplicaciones. Puedes encontrar una lista de todas las configuraciones y sus valores predeterminados en [https://docs.djangoproject.com/en/5.1/ref/settings/](https://docs.djangoproject.com/en/5.1/ref/settings/).

#### Hacer referencia a la configuración de Django en tu código

A veces puede resultar útil hacer referencia a las configuraciones de `settings.py` en tu propio código, ya sean las configuraciones integradas de Django o las que hayas definido tú mismo. Podrías tener la tentación de escribir código como estas importaciones de configuración:

```python
from bookr import settings

# check if running in DEBUG mode
if settings.DEBUG:
    do_some_logging()
```

Este método es incorrecto por varias razones:
- Es posible ejecutar Django y especificar un archivo de configuración diferente para leer, en cuyo caso el código anterior causaría un error, ya que no podría encontrar ese archivo. Alternativamente, si el archivo existe, la importación se realizaría correctamente pero contendría las configuraciones incorrectas.
- Django tiene configuraciones que podrían no estar enumeradas en el archivo `settings.py`, y si no lo están, utilizará su propio valor predeterminado interno. Por ejemplo, si eliminaste la línea `DEBUG = True` de tu archivo `settings.py`, Django recurriría al uso de su valor interno para `DEBUG` (que es `False`). Sin embargo, obtendrías un error si intentaras acceder a él usando `settings.DEBUG` directamente.
- Las librerías de terceros pueden cambiar la forma en que se definen tus configuraciones, por lo que tu archivo `settings.py` se vería completamente diferente. Es posible que ninguna de las variables esperadas exista en absoluto. El comportamiento de todas estas aplicaciones está fuera del alcance de este libro, pero es algo a tener en cuenta.

La forma preferida es utilizar el módulo `django.conf` en su lugar, de la siguiente manera:

```python
# import settings from here instead
from django.conf import settings

if settings.DEBUG:
    do_some_logging()
```

Al importar configuraciones desde `django.conf`, Django mitiga los tres problemas que acabamos de discutir:
- Las configuraciones se leen desde cualquier archivo de configuración de Django que se haya especificado
- Se interpolan los valores de configuración predeterminados
- Django se encarga de analizar cualquier configuración definida por una librería de terceros

En nuestro nuevo fragmento de código de ejemplo más corto, incluso si falta `DEBUG` en el archivo `settings.py`, recurrirá al valor predeterminado que Django tiene internamente (que es `False`). Lo mismo ocurre con todas las demás configuraciones que define Django; sin embargo, si defines tus propias configuraciones personalizadas en este archivo, Django no tendrá valores internos para ellas, por lo que en tu código, debes tener alguna disposición para que no existan; cómo se comporta tu código es tu elección y está más allá del alcance de este libro.

A continuación, veremos cómo localiza Django los archivos de plantilla.

---

### Sección: Localización de plantillas HTML en directorios de aplicaciones

Hay muchas opciones disponibles para indicarle a Django cómo encontrar plantillas, que se pueden configurar en el ajuste `TEMPLATES` de `settings.py`, pero la más fácil (por ahora) es crear un directorio `templates` dentro del directorio `reviews`. Django buscará en este (y en los directorios `templates` de otras aplicaciones) debido a que `APP_DIRS` es `True` en el archivo `settings.py`, como vimos en la sección anterior. Sin embargo, para que Django sepa que el directorio `reviews` es una aplicación, debemos configurarlo en los ajustes. Haremos eso en el próximo ejercicio.

#### Ejercicio 1.05 – Creación de un directorio templates y una plantilla base

En este ejercicio, crearás un directorio `templates` para la aplicación `reviews`. Luego, agregarás un archivo de plantilla HTML que Django podrá representar en una respuesta HTTP.

Analizamos `settings.py` y su configuración `INSTALLED_APPS` en la sección *Exploración de la configuración de Django (Settings)*. Necesitamos agregar la aplicación `reviews` a `INSTALLED_APPS` para que Django pueda encontrar plantillas. Sigue estos pasos:

1. Abre `settings.py` en PyCharm. Actualiza la configuración `INSTALLED_APPS` y agrega `reviews` al final. Debería verse así:
   ```python
   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'reviews',
   ]
   ```

2. Guarda y cierra `settings.py`.

3. En el explorador del proyecto de PyCharm, haz clic con el botón derecho en el directorio `reviews` y selecciona **New | Directory**.
   *Figura 1.26 – Creación de un nuevo directorio dentro del directorio reviews*

4. Ingresa el nombre `templates` y presiona Enter/Return para crearlo.

5. Haz clic con el botón derecho en el directorio `templates` recién creado y selecciona **New | HTML File**.
   *Figura 1.27 – Creación de un nuevo archivo HTML en el directorio templates*

6. En la ventana que aparece, ingresa el nombre `base.html` y luego presiona Enter/Return para crear el archivo.

7. Después de que PyCharm crea el archivo, también lo abre automáticamente. Tendrá este contenido:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>

   </body>
   </html>
   ```

8. Entre las etiquetas `<body>` y `</body>`, añade un breve mensaje para validar que la plantilla se está renderizando: `Hello from a template!`.
   Tu código completo debería verse así:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
   Hello from a template!
   </body>
   </html>
   ```

En este ejercicio, creamos un directorio `templates` para la aplicación `reviews` y le agregamos una plantilla HTML. La plantilla HTML se renderizará una vez que implementemos el uso de la función `render` en nuestra vista.

#### Renderizado de una plantilla con la función render

Ahora tenemos una plantilla para usar, pero necesitamos actualizar nuestra vista `index` para que represente la plantilla, en lugar de devolver el texto `Hello (name)!` que muestra actualmente (consulta la Figura 1.30 para ver cómo se ve actualmente). Haremos esto utilizando la función `render` y proporcionando el nombre de la plantilla. `render` es una función abreviada (*shortcut function*) que devuelve `HttpResponse`. Hay otras formas de renderizar una plantilla que brindan más control sobre cómo se procesa, pero por ahora, esta función está bien para nuestras necesidades. `render` toma al menos dos argumentos: el primero es siempre la solicitud (`request`) que se pasó a la vista y el segundo es el nombre/ruta relativa de la plantilla que se está procesando. También la llamaremos con un tercer argumento, el contexto de renderizado (*render context*) que contiene todas las variables que estarán disponibles en la plantilla; habrá más sobre esto en el Ejercicio 1.07.

#### Ejercicio 1.06 – Renderizado de una plantilla en una vista

En este ejercicio, actualizarás tu función de vista `index` para renderizar la plantilla HTML que creaste en el Ejercicio 1.05. Harás uso de la función `render`, que carga tu plantilla desde el disco, la procesa y la envía al navegador. Esto reemplazará el texto estático que estás devolviendo actualmente desde la función de vista `index`:

1. En PyCharm, abre `views.py` en el directorio `reviews`.

2. Ya no creamos manualmente `HttpResponse`, así que elimina la línea de importación de `HttpResponse`:
   ```python
   from django.http import HttpResponse
   ```

3. Reemplázala con una importación de la función `render` desde `django.shortcuts`:
   ```python
   from django.shortcuts import render
   ```

4. Actualiza la función `index` para que, en lugar de devolver `HttpResponse`, devuelva una llamada a `render`, pasando `request` y el nombre de la plantilla.
   El archivo `views.py` ahora contiene este código:
   ```python
   from django.shortcuts import render

   def index(request):
       return render(request, "base.html")
   ```

5. Inicia el servidor de desarrollo si aún no se está ejecutando. Luego, abre tu navegador web y actualiza `http://127.0.0.1:8000`. Deberías ver el mensaje `Hello from a template!` renderizado, como se muestra en la Figura 1.38:
   *Figura 1.28 – Tu primera plantilla HTML renderizada*

#### Renderizado de variables en plantillas

Las plantillas no son solo HTML estático. La mayoría de las veces contendrán variables que se interpolan como parte del proceso de renderizado. Se pasan de una vista a una plantilla mediante un **contexto** (*context*): un diccionario (u objeto similar a un diccionario) que contiene los nombres de todas las variables que puede utilizar una plantilla. Tomemos nuevamente a Bookr como ejemplo. Sin variables en tu plantilla, necesitarías un archivo HTML diferente para cada libro que quisieras mostrar. En su lugar, usamos una variable como `book_name` dentro de la plantilla y luego la vista proporciona a la plantilla una variable `book_name`, configurada con el título del modelo de libro que ha cargado. Al mostrar un libro diferente, el HTML no necesita cambiar; la vista simplemente le pasa un libro diferente. Puedes ver cómo los modelos, las vistas y las plantillas se unen ahora.

A diferencia de otros lenguajes, como PHP, las variables deben pasarse explícitamente a una plantilla y las variables de una vista no están disponibles automáticamente para la plantilla. Esto es por razones de seguridad, así como para evitar contaminar accidentalmente el espacio de nombres de la plantilla (no queremos variables inesperadas en la plantilla).

Dentro de una plantilla, las variables se indican mediante llaves dobles: `{{ }}`. Si bien no es estrictamente un estándar, este estilo es bastante común y se utiliza en otras herramientas de plantillas, como Vue.js y Mustache. Symfony (un framework PHP) también utiliza llaves dobles en su lenguaje de plantillas Twig, por lo que es posible que las hayas visto utilizadas de manera similar allí.

Para representar una variable en una plantilla, simplemente envuélvela entre llaves: `{{ book_name }}`. Django escapará automáticamente el HTML en la salida para que puedas incluir caracteres especiales (como `<` o `>`) en tu variable sin preocuparte de que distorsione tu salida. Si no se pasa una variable a una plantilla, Django simplemente no representará nada en esa ubicación, en lugar de generar una excepción.

Hay muchas más formas de renderizar una variable de manera diferente usando filtros, pero estas se cubrirán en el Capítulo 3.

#### Ejercicio 1.07 – Uso de variables en plantillas

Pondremos una variable simple dentro del archivo `base.html` para demostrar cómo funciona la interpolación de variables de Django:

1. En PyCharm, abre el archivo `base.html`.

2. Actualiza el elemento `<body>` para que contenga un lugar para representar la variable `name`:
   ```html
   <body>
   Hello, {{ name }}!
   </body>
   ```

3. Regresa a tu navegador web y actualiza (aún deberías estar en `http://127.0.0.1:8000`). Verás que la página ahora muestra `Hello, !`. Esto se debe a que no hemos configurado la variable `name` en el contexto de renderizado.
   *Figura 1.29 – No se representa ningún valor en la plantilla porque no se estableció ningún contexto*

4. Abre `views.py` y añade una variable llamada `name`, establecida con el valor `"world"`, dentro de la función `index`:
   ```python
   def index(request):
       name = "world"
       return render(request, "base.html")
   ```

5. Actualiza tu navegador nuevamente. Ten en cuenta que nada ha cambiado: todo lo que queramos representar debe pasarse explícitamente a la función `render` como contexto. Este es el diccionario de variables que se ponen a disposición al renderizar.

6. Añade el diccionario de contexto como tercer argumento a la función `render`. Cambia tu línea de `render` a esto:
   ```python
   return render(request, "base.html", {"name": name})
   ```
   El archivo `views.py` completo contendrá este código:
   ```python
   from django.shortcuts import render

   def index(request):
       name = "world"
       return render(request, "base.html", {"name": name})
   ```

7. Actualiza tu navegador nuevamente y verás que ahora dice `Hello, world!`.
   *Figura 1.30 – Una plantilla renderizada con una variable*

En este ejercicio, combinamos la plantilla que creamos en el ejercicio anterior con la función `render` para renderizar una página HTML, con la variable `name` que se le pasó dentro de un diccionario de contexto.

En la siguiente sección, veremos algunas formas de depurar tu código y manejar errores.

---

### Sección: Depuración y gestión de errores

Al programar, a menos que seas el programador perfecto que nunca comete errores, probablemente tendrás que lidiar con errores o depurar tu código en algún momento. Cuando hay un error en tu programa, generalmente hay dos formas de saberlo: o tu código generará una excepción o obtendrás resultados o salidas inesperados al ver la página. Probablemente verás excepciones con más frecuencia, ya que hay muchas formas accidentales de causarlas. Si tu código genera resultados inesperados pero no genera ninguna excepción, probablemente querrás utilizar el depurador de PyCharm para averiguar por qué.

Comenzaremos con una descripción general de algunas excepciones de Python y cómo manejarlas, junto con un ejercicio que demuestra cómo muestra Django las excepciones. Luego, veremos cómo ejecutar Django dentro del depurador de PyCharm para que puedas observar el interior de tu programa mientras se ejecuta.

#### Excepciones

Si has trabajado con Python u otros lenguajes de programación antes, probablemente te hayas encontrado con excepciones. Si no, aquí tienes una breve introducción. Las excepciones se lanzan (*raised* o *thrown*) cuando ocurre un error. La ejecución de un programa se detiene en ese punto del código y la excepción viaja hacia arriba en la cadena de llamadas a funciones hasta que se detecta. Si no se detecta, el programa se bloqueará, a veces con un mensaje de error que describe la excepción y dónde ocurrió. Hay excepciones que genera el propio Python y tu código puede generar excepciones para detener rápidamente la ejecución en cualquier punto. Algunas excepciones comunes que puedes ver al programar en Python son las siguientes:
- **IndentationError**: Python generará esto si tu código no está sangrado correctamente o tiene una combinación de tabulaciones y espacios.
- **SyntaxError**: Python genera este error si tu código tiene una sintaxis no válida:
  ```python
  >>> a === 1
    File "<stdin>", line 1
      a === 1
         ^
  SyntaxError: invalid syntax
  ```
- **ImportError**: Se genera cuando falla una importación; por ejemplo, si se intenta importar desde un archivo que no existe o se intenta importar un nombre que no está establecido en un archivo:
  ```python
  >>> import missing_file
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ImportError: No module named missing_file
  ```
- **NameError**: Se genera al intentar acceder a una variable que aún no se ha establecido:
  ```python
  >>> a = b + 5
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  NameError: name 'b' is not defined
  ```
- **KeyError**: Se genera al acceder a una clave que no está establecida en un diccionario (o un objeto similar a un diccionario):
  ```python
  >>> d = {'a': 1}
  >>> d['b']
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  KeyError: 'b'
  ```
- **IndexError**: Se genera al acceder a un índice fuera de la longitud de una lista:
  ```python
  >>> l = ['a', 'b']
  >>> l[3]
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  IndexError: list index out of range
  ```
- **TypeError**: Se genera al intentar realizar una operación en un objeto que no la admite, o al usar dos objetos del tipo incorrecto; por ejemplo, intentar sumar una cadena a un entero:
  ```python
  >>> 1 + '1'
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: unsupported operand type(s) for +: 'int' and 'str'
  ```

Django también genera sus propias excepciones personalizadas y se te presentarán a lo largo del libro.

Al ejecutar el servidor de desarrollo de Django con `DEBUG = True` en tu archivo `settings.py`, Django capturará automáticamente las excepciones que ocurran en tu código (en lugar de bloquearse). Luego generará una respuesta HTTP que te mostrará un seguimiento de la pila (*stack trace*) y otra información para ayudarte a depurar el problema. Cuando se ejecuta en producción, `DEBUG` debe establecerse en `False`. Django devolverá entonces una página de error interno del servidor estándar, sin ninguna información confidencial. También tienes la opción de mostrar una página de error personalizada.

#### Ejercicio 1.08 – Generación y visualización de excepciones

Creemos una excepción simple en nuestra vista para que estés familiarizado con cómo las muestra Django. En este caso, intentaremos usar una variable que no existe, lo que generará `NameError`:

1. En PyCharm, abre `views.py`. En la función de vista `index`, cambia el contexto que se envía a la función `render` para que use una variable que no existe. Intentaremos enviar `invalid_name` en el diccionario de contexto, en lugar de `name`. No cambies la clave del diccionario de contexto, solo su valor:
   ```python
   return render(request, "base.html", {"name": invalid_name})
   ```

2. Vuelve a tu navegador y actualiza la página. Deberías ver una pantalla como la Figura 1.31:
   *Figura 1.31 – Una pantalla de excepción de Django*

3. Las dos primeras líneas de encabezado de la página te informan del error ocurrido:
   ```text
   NameError at /
   name 'invalid_name' is not defined
   ```

4. Debajo del encabezado hay un rastreo (*traceback*) de dónde ocurrió la excepción. Haz clic en las distintas líneas de código para expandirlas y ver el código circundante, o haz clic en *Local vars* para cada marco (*frame*) para expandirlos y ver cuáles son los valores de las variables:
   *Figura 1.32 – La línea que causó la excepción*
   En nuestro caso, podemos ver que la excepción se generó en la línea seis de nuestro archivo `views.py`, y al expandir su menú desplegable *Local vars*, vemos que `name` tiene el valor `world`, y la única otra variable es la solicitud entrante (Figura 1.43).

5. Regresa a `views.py` y corrige `NameError` cambiando el nombre de `invalid_name` a `name`.

6. Guarda el archivo y actualiza tu navegador. `Hello, world!` debería mostrarse nuevamente (como se muestra en la Figura 1.41).

En este ejercicio, hicimos que nuestro código Django generara una excepción (`NameError`) al intentar usar una variable que no se había establecido. Vimos que Django envió automáticamente detalles de esta excepción y un seguimiento de la pila al navegador para ayudarnos a encontrar la causa del error. Luego revertimos nuestro cambio de código para asegurarnos de que nuestra vista funcionara correctamente nuevamente.

#### Depuración (Debugging)

Cuando intentas encontrar problemas en tu código, puede resultar útil utilizar un depurador. Esta es una herramienta que te permite recorrer tu código línea por línea, en lugar de ejecutarlo todo a la vez. Cada vez que el depurador se detiene en una línea de código en particular, puedes ver los valores de todas las variables actuales. Esto es muy útil para encontrar errores en tu código que no generan excepciones.

Por ejemplo, en Bookr, hablamos de tener una vista que obtiene una lista de libros de la base de datos y los renderiza en una plantilla HTML. Si ves la página en el navegador, es posible que veas solo un libro cuando esperas varios. Podrías pausar la ejecución dentro de tu función de vista y ver qué valores se obtuvieron de la base de datos. Si tu vista solo recibe un libro de la base de datos, sabrás que hay un problema con tu consulta a la base de datos en alguna parte. Si tu vista obtiene con éxito varios libros pero solo se procesa uno, probablemente sea un problema con la plantilla. La depuración te ayuda a delimitar fallas como esta.

PyCharm tiene un depurador integrado para facilitar el paso por el código y ver lo que sucede en cada línea. Para indicarle al depurador dónde detener la ejecución del código, debes establecer un punto de interrupción (*breakpoint*) en una o más líneas de código. Se llaman así porque la ejecución del código se interrumpirá (se detendrá) en ese punto.

Para que los puntos de interrupción se activen, PyCharm debe configurarse para ejecutar tu proyecto en su depurador. Hay una pequeña penalización de rendimiento, pero generalmente no se nota, por lo que puedes optar por ejecutar siempre tu código dentro del depurador para que puedas establecer rápidamente un punto de interrupción, sin tener que detener y reiniciar el servidor de desarrollo de Django.

Ejecutar el servidor de desarrollo de Django dentro del depurador es tan simple como hacer clic en el icono de **Debug** en lugar del icono de Play (ver Figura 1.20) para iniciarlo.

#### Ejercicio 1.09 – Depuración de tu código

En este ejercicio, aprenderás los conceptos básicos del depurador de PyCharm. Ejecutarás el servidor de desarrollo de Django en el depurador y luego establecerás un punto de interrupción en tu función de vista para pausar la ejecución y poder examinar las variables:

1. Si el servidor de desarrollo de Django se está ejecutando, detenlo haciendo clic en el botón **Stop** en la esquina superior derecha de la ventana de PyCharm:
   *Figura 1.33 – El botón Stop en la esquina superior derecha de la ventana de PyCharm*

2. Inicia el servidor de desarrollo de Django nuevamente dentro del depurador haciendo clic en el icono de **Debug** justo a la izquierda del botón Stop (Figura 1.44). El servidor tardará unos segundos en iniciarse y luego deberías poder actualizar la página en tu navegador para asegurarte de que aún se esté cargando. No deberías notar ningún cambio; todo el código se ejecuta igual que antes.

3. Ahora, podemos establecer un punto de interrupción que hará que la ejecución se detenga para que podamos ver el estado del programa. En PyCharm, haz clic justo a la derecha de los números de línea, en la línea 5, en el margen izquierdo del panel del editor. Aparecerá un círculo rojo para indicar que el punto de interrupción ya está activo:
   *Figura 1.34 – Un punto de interrupción en la línea 5*

4. Regresa a tu navegador y actualiza la página. Tu navegador no mostrará ningún contenido; en su lugar, simplemente continuará intentando cargar la página. Dependiendo de tu sistema operativo, PyCharm debería activarse nuevamente; si no, tráelo al primer plano. Deberías ver que la línea 5 está resaltada y en la parte inferior de la ventana se muestra el depurador. Los marcos de pila (*stack frames*, la cadena de funciones a las que se llamó para llegar a la línea actual) están a la izquierda y las variables actuales de la función están a la derecha.
   *Figura 1.35 – El depurador se detuvo con la línea actual (5) resaltada*

5. Actualmente hay una variable en el alcance, `request`. Si haces clic en el triángulo de alternancia a la izquierda de su nombre, puedes mostrar u ocultar los atributos que tiene establecidos.
   *Figura 1.36 – Los atributos de la variable request*
   Por ejemplo, si te desplazas hacia abajo en la lista de atributos, puedes ver que `method` es `GET` y `path` es `/`.

6. La barra de acciones, que se muestra en la Figura 1.37, está encima de los marcos de pila y las variables. Sus botones (de izquierda a derecha) son los siguientes:
   *Figura 1.37 – La barra de acciones*
   - **Rerun**: Detiene el programa y lo reinicia.
   - **Stop**: Detiene el depurador.
   - **Resume Program**: Continúa ejecutándose hasta el siguiente punto de interrupción.
   - **Pause Program**: Interrumpe el programa en su punto de ejecución actual.
   - **Step Over**: Ejecuta la línea actual de código y continúa hasta la siguiente línea.
   - **Step Into**: Entra en la línea actual. Por ejemplo, si la línea contenía una función, continuaría con el depurador dentro de esta función.
   - **Step into My Code**: Entra en la línea que se está ejecutando pero continúa hasta que encuentra el código que has escrito. Por ejemplo, si estás entrando en código de una librería de terceros que luego llama a tu código, no te mostrará el código de terceros, sino que continuará hasta que regrese al código que has escrito.
   - **Step Out**: Regresa del código actual a la función o método que lo llamó. Es lo opuesto a la acción Step In.
   - **View Breakpoints**: Abre una ventana para ver todos los puntos de interrupción que has establecido.
   - **Mute Breakpoints**: Activa o desactiva todos los puntos de interrupción pero no los elimina.
   Ten en cuenta que no todos los botones son útiles todo el tiempo. Por ejemplo, puede ser fácil salir de tu vista y terminar confundiendo el código de la librería Django.

7. Haz clic en el botón **Step Over** una vez para ejecutar la línea 5. Puedes ver que la variable `name` se ha añadido a la lista de variables en la vista del depurador y su valor es `world`.
   *Figura 1.38 – La nueva variable name está ahora dentro del alcance, con el valor world*

8. Ahora estamos al final de nuestra función de vista `index`, y si fuéramos a pasar por encima (*step over*) de esta línea de código, saltaría al código de la librería Django, que no queremos ver. Para continuar ejecutando y enviar la respuesta de vuelta a tu navegador, haz clic en el botón **Resume Program**, el tercero desde la izquierda en la barra de herramientas del depurador (Figura 1.39). Deberías ver que tu navegador ahora ha cargado la página nuevamente:
   *Figura 1.39 – Acciones para controlar la ejecución: el icono de reproducción verde es el botón Resume Program*

9. Al hacer clic en el botón de puntos suspensivos verticales, `⋮`, se muestran más botones (consulte la Figura 1.50); desde arriba, son **Force Step Over** (pasa por alto la línea de código actual, ignorando cualquier llamada a métodos hasta la siguiente línea), **Smart Step Into** (permite la selección con el cursor de un método desde la línea de código actual), **Run to Cursor** (sigue ejecutando el código hasta la línea actual bajo el cursor, suspendiendo el programa en cualquier punto de interrupción), **Force Run to Cursor** (sigue ejecutando el código hasta la línea actual bajo el cursor, ignorando cualquier punto de interrupción), **Show Execution Point** (enfoca el punto de ejecución en la ventana del editor y el marco de pila en el panel), **Evaluate Expression…** (evalúa una expresión de la ventana del editor), **Debugger Settings** (menú para configuraciones configurables) y **Modify Run Configuration…** (abre la ventana de configuración de ejecución).

10. Por ahora, desactiva el punto de interrupción en PyCharm haciendo clic en él (el círculo rojo al lado de la línea 5).
    *Figura 1.40 – Al hacer clic en el punto de interrupción que estaba en la línea 5 se desactiva*

Esta es solo una breve introducción a cómo establecer puntos de interrupción en PyCharm. Si has utilizado funciones de depuración en otros IDEs, entonces deberías estar familiarizado con los conceptos: puedes recorrer el código paso a paso, entrar y salir de funciones o evaluar expresiones. Una vez que hayas establecido un punto de interrupción, puedes hacer clic derecho sobre él para cambiar las opciones. Por ejemplo, puedes hacer que el punto de interrupción sea condicional para que la ejecución solo se detenga bajo ciertas circunstancias. Todo esto está fuera del alcance de este libro, pero es útil conocerlo cuando se intenta resolver problemas en el código.

Terminaremos el capítulo con dos actividades. Primero, construirás una pantalla de bienvenida para Bookr y luego estructurarás la página de búsqueda de libros.

---

### Sección: Actividad 1.01 – Creación de una pantalla de bienvenida para el sitio

El sitio web Bookr que estamos construyendo necesita tener una página de inicio (*splash page*) que dé la bienvenida a los usuarios y les informe en qué sitio se encuentran. También contendrá enlaces a otras partes del sitio, pero estos se agregarán en capítulos posteriores. Por ahora, crearás una página con un mensaje de bienvenida.

Estos pasos te ayudarán a completar la actividad:
1. En tu vista `index`, renderiza la plantilla `base.html`.
2. Actualiza la plantilla `base.html` para que contenga el mensaje de bienvenida. Debe estar tanto en la etiqueta `<title>` dentro de `<head>` como en una nueva etiqueta `<h1>` en el cuerpo (`<body>`).

Después de completar la actividad, deberías poder ver algo como esto:

*Figura 1.41 – La página de bienvenida de Bookr*

En la siguiente actividad, crearás una página básica para mostrar los resultados de una búsqueda de libros en Bookr.

---

### Sección: Actividad 1.02 – Estructura inicial para la búsqueda de libros

Una característica útil de un sitio como Bookr es la capacidad de buscar entre datos para encontrar algo en el sitio rápidamente. Bookr implementará la búsqueda de libros para permitir a los usuarios encontrar un libro en particular por parte de su título. Si bien todavía no tenemos libros que encontrar, aún podemos implementar una página que muestre el texto que buscó un usuario. El usuario ingresa la cadena de búsqueda como parte de los parámetros de la URL. Implementaremos la búsqueda y un formulario para facilitar la entrada de texto en el Capítulo 6, *Formularios*.

Estos pasos te ayudarán a completar la actividad:
1. Crea una plantilla HTML de resultados de búsqueda. Debe incluir un marcador de posición de variable para mostrar la(s) palabra(s) de búsqueda que se pasaron a través del contexto de renderizado. Muestra la variable pasada en las etiquetas `<title>` y `<h1>`. Usa una etiqueta `<em>` alrededor del texto de búsqueda en el cuerpo para ponerlo en cursiva.
2. Añade una función de vista `search` en `views.py`. La vista debe leer una cadena de búsqueda de los parámetros de la URL (en el atributo `GET` de la solicitud). Luego, debe renderizar la plantilla que creaste en el paso anterior, pasando el valor de búsqueda que se sustituirá, utilizando el diccionario de contexto.
3. Añade un mapeo de URL a tu nueva vista en `urls.py`. La URL puede ser algo como `/book-search`.

Después de completar esta actividad, deberías poder pasar un valor de búsqueda a través de los parámetros de la URL y verlo representado en la página resultante. Debería verse así:

*Figura 1.42 – Búsqueda en Django workshop*

También deberías poder pasar caracteres especiales de HTML como `<` y `>` para ver cómo Django los escapa automáticamente en la plantilla.

*Figura 1.43 – Observa cómo se escapan los caracteres HTML para protegernos de la inyección de etiquetas*

---

### Sección: Resumen

Este capítulo fue una introducción rápida a Django. Primero te pusiste al día sobre el protocolo HTTP y la estructura de las solicitudes y respuestas HTTP. Luego vimos cómo Django utiliza el paradigma MVT y cómo analiza una URL, genera una solicitud HTTP y la envía a una vista para obtener una respuesta HTTP. Estructuramos el proyecto `bookr` y luego creamos la aplicación `reviews` para él. Luego creamos dos vistas de ejemplo para ilustrar cómo obtener datos de una solicitud y usarlos al renderizar plantillas. También experimentaste para ver cómo Django escapa la salida en HTML al renderizar una plantilla.

Hicimos todo esto con el IDE de PyCharm y aprendiste cómo configurarlo para depurar tu aplicación. El depurador te ayudará a descubrir por qué las cosas no funcionan como deberían.

En el próximo capítulo, comenzarás a aprender sobre la integración de bases de datos de Django y su sistema de modelos, para que puedas comenzar a almacenar y recuperar datos reales para tu aplicación.
