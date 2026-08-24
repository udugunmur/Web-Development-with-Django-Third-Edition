# Parte 1: Primeros pasos con Django

## Capítulo 2: Modelos y Migraciones

En este capítulo, aprenderemos sobre el mapeo objeto-relacional (*Object-Relational Mapping* u ORM) de Django, a través del cual nuestra aplicación puede interactuar y trabajar sin problemas con una base de datos relacional utilizando código Python simple, eliminando la necesidad de ejecutar consultas SQL complejas. Aprenderemos sobre modelos y migraciones, que forman parte del ORM de Django, utilizados para propagar cambios de esquema de base de datos desde la aplicación hacia la base de datos, y también para realizar operaciones de creación, lectura, actualización y eliminación (*Create, Retrieve, Update, Delete* o CRUD). Hacia el final del capítulo, estudiaremos los distintos tipos de relaciones de bases de datos y utilizaremos ese conocimiento para realizar consultas a través de registros relacionados.

Cubriremos los siguientes temas en este capítulo:
- Exploración del ORM de Django
- Creación de modelos y migraciones en Django
- Operaciones CRUD de base de datos en Django
- Operaciones de creación masiva (*bulk create*) y actualización masiva (*bulk update*)
- Realización de búsquedas complejas mediante objetos `Q`
- Actividad 2.01 – Creación de modelos para una aplicación de gestión de proyectos
- Población de la base de datos del proyecto Bookr

Al final de este capítulo, tendremos una sólida comprensión de las bases de datos y de cómo se puede utilizar Django para implementar operaciones de base de datos en una aplicación web.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter02` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos descritos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Exploración del ORM de Django

Las aplicaciones web interactúan constantemente con bases de datos, y una de las formas de hacerlo es mediante el uso de SQL. Si decides escribir una aplicación web sin un framework web como Django, y en su lugar utilizas Python solo, se podrían usar librerías de Python como `psycopg2` (para bases de datos Postgres) para interactuar directamente con las bases de datos utilizando comandos SQL. Pero al desarrollar una aplicación web con múltiples tablas y campos, los comandos SQL pueden volverse fácilmente demasiado complejos y, por lo tanto, difíciles de mantener. Por esta razón, los frameworks web populares como Django proporcionan un nivel de abstracción que nos permite trabajar fácilmente con bases de datos. La parte de Django que nos ayuda a hacer esto se llama **ORM**.

El ORM de Django convierte código Python orientado a objetos en estructuras de base de datos reales, como tablas de bases de datos con definiciones de tipos de datos, y facilita todas las operaciones de bases de datos mediante código Python simple. Gracias a esto, no tenemos que lidiar con comandos SQL al realizar operaciones de base de datos. Esto ayuda a un desarrollo más rápido de aplicaciones y facilita el mantenimiento del código fuente de la aplicación.

Django admite bases de datos relacionales como SQLite, PostgreSQL, Oracle Database y MySQL. La capa de abstracción de base de datos de Django garantiza que se pueda utilizar el mismo código fuente de Python/Django en cualquiera de las bases de datos relacionales anteriores con muy pocas modificaciones en la configuración del proyecto. Dado que SQLite forma parte de las librerías estándar de Python y Django está configurado de forma predeterminada para usar SQLite, dentro del alcance de este capítulo utilizaremos SQLite mientras aprendemos sobre los modelos y las migraciones de Django. A continuación, comprenderemos cómo se realiza la configuración de la base de datos con Django.

#### Configuración de la base de datos y creación de aplicaciones Django

Como ya hemos visto en el Capítulo 1, cuando creamos un proyecto de Django y ejecutamos el servidor de Django, la configuración de base de datos predeterminada es SQLite3. La configuración de la base de datos estará presente en el directorio del proyecto, en el archivo `settings.py`.

Asegúrate de examinar el archivo `settings.py` para la aplicación `bookr`. Revisar el archivo completo una vez te ayudará a comprender los conceptos que siguen. Puedes encontrar el archivo en la carpeta `Chapter02` en el repositorio de GitHub de este libro.

Por lo tanto, para nuestro proyecto de ejemplo, la configuración de la base de datos estará presente en la siguiente ubicación: `bookr/settings.py`. La configuración de base de datos predeterminada presente en este archivo cuando se crea un proyecto de Django es la siguiente:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

A la variable `DATABASES` se le asigna un diccionario que contiene los detalles de la base de datos para el proyecto. Dentro del diccionario, hay un diccionario anidado con una clave `default`. Esto contiene la configuración de una base de datos predeterminada para el proyecto de Django. La razón por la que tenemos un diccionario anidado con `default` como clave es que un proyecto de Django podría interactuar potencialmente con múltiples bases de datos, y la base de datos predeterminada es la que utiliza Django para todas las operaciones a menos que se especifique explícitamente. La clave `ENGINE` representa qué motor de base de datos se está utilizando; en este caso, `sqlite3`.

La clave `NAME` define el nombre de la base de datos, que puede tener cualquier valor. Pero para SQLite3, dado que la base de datos se crea como un archivo, `NAME` puede tener la ruta completa del directorio donde se debe crear el archivo. La ruta completa del archivo de base de datos se procesa uniendo (o concatenando) la ruta previamente definida en `BASE_DIR` con `db.sqlite3`. Ten en cuenta que `BASE_DIR` es el directorio del proyecto como ya se definió en el archivo `settings.py`. Debido a que `BASE_DIR` se define como `pathlib.Path`, el operador `/` en esta línea de código es el método sobrecargado `pathlib.Path.__truediv__`, en lugar de un operador numérico. Este operador se utiliza para unir dos rutas de modo que `pathlib.Path('a') / 'b'` es igual a `pathlib.Path('a/b')`.

Si utilizas otras bases de datos, como PostgreSQL, MySQL, etc., se deberán realizar cambios en la configuración de la base de datos anterior, como se muestra aquí:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'bookr',
        'USER': django,
        'PASSWORD': 0b3ef4b0d80c28db,
        'HOST': 127.0.0.1,
        'PORT': '5432',
    }
}
```

Aquí, se han realizado cambios en `ENGINE` para usar PostgreSQL. Se deben proporcionar la dirección IP del host y el número de puerto del servidor para `HOST` y `PORT`, respectivamente. Como sugieren los nombres, `USER` es el nombre de usuario de la base de datos y `PASSWORD` es la contraseña de la base de datos. Además de los cambios en la configuración, tendremos que instalar los controladores (*drivers*) o enlaces de la base de datos junto con el host y las credenciales de la base de datos. Ten en cuenta que este bloque de código es solo un ejemplo para mostrar los cambios que deberás realizar para usar una base de datos diferente, como PostgreSQL, pero como estamos usando SQLite, usaremos la configuración de base de datos que ya existe y no es necesario realizar ninguna modificación en la configuración de la base de datos. A continuación, leeremos brevemente sobre las aplicaciones de Django y algunas de las aplicaciones predeterminadas presentes en Django.

#### Migraciones de Django

Como hemos aprendido anteriormente, el ORM de Django ayuda a simplificar las operaciones de la base de datos. Una parte importante de la operación es transformar el código Python en estructuras de base de datos, como campos de bases de datos con tipos de datos y tablas definidos. En otras palabras, la transformación del código Python en estructuras de base de datos se conoce como **migración**. Por ejemplo, en lugar de crear tablas de bases de datos ejecutando comandos SQL, escribirías modelos para ellas en Python; algo que aprenderemos a hacer en la próxima sección *Creación de modelos y migraciones en Django*. Estos modelos tendrán campos que formarán los planos (*blueprints*) de las tablas de la base de datos. Los campos, a su vez, tendrán diferentes tipos de campos, lo que nos dará más información sobre el tipo de datos almacenados allí (recuerda cómo especificamos el tipo de datos de nuestro campo como `TEXT` en el paso 4 del Ejercicio 2.01).

Dado que tenemos un proyecto de Django configurado, realicemos nuestra primera migración. Aunque no hemos agregado ningún código a nuestro proyecto, podemos migrar las aplicaciones enumeradas en `INSTALLED_APPS`. Esto es necesario porque las aplicaciones instaladas de Django necesitan almacenar los datos relevantes en la base de datos para sus operaciones, y la migración creará las tablas de base de datos requeridas para almacenar los datos en la base de datos.

El siguiente es un fragmento del archivo `settings.py` del proyecto Bookr:

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

Este bloque de código contiene un conjunto de aplicaciones instaladas o predeterminadas utilizadas para el sitio de administración, autenticación, tipos de contenido, sesiones, mensajería, administración de archivos estáticos y la aplicación de reseñas que iniciamos en el Capítulo 1.

Se debe ingresar el siguiente comando en la terminal o consola para hacer esto:

```bash
python manage.py migrate
```

Al ejecutar este comando, creamos todas las estructuras de base de datos requeridas por las aplicaciones instaladas.

Para una mejor visualización de los datos en la base de datos SQLite, utilizaremos una herramienta de código abierto llamada **DB Browser for SQLite**. Esta herramienta ayuda a visualizar los datos y proporciona una consola para ejecutar comandos SQL.

Si aún no lo has hecho, visita la URL [https://sqlitebrowser.org/](https://sqlitebrowser.org/) y, desde la sección de descargas, instala la aplicación según tu sistema operativo e iníciala.

Echemos un vistazo a la base de datos en la que se han realizado cambios después de ejecutar el comando `migrate` mediante DB Browser for SQLite.

El archivo de base de datos se creará en el directorio del proyecto bajo el nombre `db.sqlite3`.

Las operaciones de base de datos también se pueden realizar mediante una consola de línea de comandos utilizando el comando `sqlite3 db.sqlite3`.

Sigue estos pasos para verificar que la migración se haya realizado correctamente y que se hayan creado las tablas de la base de datos:

1. Abre DB Browser, haz clic en **Open Database**, navega hasta encontrar el archivo `db.sqlite3` y ábrelo. Deberías ver un conjunto de tablas recién creadas por la migración de Django. Se verá de la siguiente manera en DB Browser:
   *Figura 2.1: Contenido del archivo db.sqlite3*

2. Ahora, explora la estructura de la base de datos recién creada haciendo clic en las tablas de la base de datos, y deberíamos ver lo siguiente:
   *Figura 2.2: Exploración a través de la estructura de base de datos recién creada*
   Observa que las tablas de la base de datos creadas tienen diferentes campos, cada uno con sus respectivos tipos de datos.

3. Haz clic en la pestaña **Browse Data** en DB Browser y selecciona una tabla del menú desplegable. Por ejemplo, después de hacer clic en la tabla `auth_group_permissions`, deberías ver algo como esto:
   *Figura 2.3: Visualización de la tabla auth_group_permissions*

Verás que todavía no hay datos disponibles para estas tablas porque la migración de Django solo crea la estructura de la base de datos o el plano (*blueprint*), y los datos reales en la base de datos se almacenan durante la operación de la aplicación. Ahora que hemos migrado las aplicaciones integradas o predeterminadas de Django, intentemos crear una aplicación y realizar una migración de Django.

---

### Sección: Creación de modelos y migraciones en Django

Un modelo de Django es esencialmente una clase de Python que contiene el plano para crear una tabla en una base de datos. El archivo `models.py` puede tener muchos de estos modelos, y cada modelo se transforma en una tabla de base de datos. Los atributos de la clase forman los campos y las relaciones de la tabla de base de datos según las definiciones del modelo.

Para nuestra aplicación `reviews`, necesitamos crear los siguientes modelos y sus tablas de base de datos:
- **Book**: Debe almacenar información sobre libros.
- **Contributor**: Debe almacenar información sobre la(s) persona(s) que contribuyeron a escribir el libro, como el autor, coautor o editor.
- **Publisher**: Como su nombre lo indica, se refiere a la editorial del libro.
- **Review**: Debe almacenar todas las reseñas de libros escritas por los usuarios de la aplicación.

Cada libro en nuestra aplicación necesitará tener una editorial, así que creemos `Publisher` como nuestro primer modelo. Ingresa el siguiente código en `reviews/models.py`:

```python
from django.db import models

class Publisher(models.Model):
    """A company that publishes books."""
    name = models.CharField(
        max_length=50,
        help_text="The name of the Publisher.")
    website = models.URLField(
        blank=True,
        help_text="The Publisher's website.")
    email = models.EmailField(
        blank=True,
        help_text="The Publisher's email address.")
```

Puedes ver el archivo `models.py` completo para la aplicación `bookr` en la carpeta `Chapter02` en el repositorio de GitHub de este libro.

La primera línea de código importa el módulo `models` de Django. Si bien esta línea se generará automáticamente en el momento de la creación de la aplicación Django, asegúrate de agregarla si no está presente. Después de la importación, el resto del código define una clase llamada `Publisher`, una subclase de `models.Model` de Django. Además, esta clase tendrá atributos o campos como `name`, `website` y `email`. Los siguientes son los tipos de campos utilizados al crear este modelo:
- **CharField**: Este tipo de campo almacena campos de cadena más cortos, como Packt Publishing. Para cadenas muy grandes, usamos `TextField`.
- **EmailField**: Es similar a `CharField` pero valida si la cadena representa una dirección de correo electrónico válida; por ejemplo, `customersupport@packtpub.com`.
- **URLField**: Nuevamente, es similar a `CharField`, pero valida si la cadena representa una URL válida; por ejemplo, `https://www.packtpub.com/`.

A continuación, veremos las opciones de campo utilizadas al crear cada uno de estos campos.

#### Opciones de campo (Field options)

Django proporciona una forma de definir opciones de campo para el campo de un modelo. Estas opciones de campo se utilizan para establecer un valor o una restricción, etc. Por ejemplo, podemos establecer un valor predeterminado para un campo usando `default=<value>` para asegurarnos de que cada vez que se cree un registro en la base de datos para el campo, se establezca en un valor predeterminado especificado por nosotros. Las siguientes son las tres opciones de campo que hemos utilizado al definir el modelo `Publisher`:
- **blank**: El valor predeterminado es `False`. Si esta opción se establece en `True`, se permitirán valores en blanco para este campo durante la validación del formulario de Django.
- **help_text**: Esta opción de campo nos ayuda a agregar texto descriptivo para un campo que se incluye automáticamente en los formularios de Django.
- **max_length**: Esta opción se proporciona a `CharField`, donde define la longitud máxima del campo en términos de número de caracteres.

Otras opciones de campo que se utilizan con frecuencia incluyen las siguientes:
- **null**: El valor predeterminado es `False`. Si esta opción se establece en `True`, se permite un valor `NULL` para el campo a nivel de base de datos.
- **unique**: El valor predeterminado es `False`. Si esta opción se establece en `True`, la columna de la base de datos correspondiente al campo se definirá como única. Los valores duplicados para este campo generarán un error de validación.

Django tiene muchos más tipos y opciones de campo que se pueden explorar en la extensa documentación oficial de Django. A medida que desarrollemos nuestra aplicación de muestra de reseñas de libros, aprenderemos sobre los tipos y campos utilizados para el proyecto. Ahora, migremos los modelos de Django a la base de datos siguiendo estos pasos:

1. Ejecuta el siguiente comando en la consola o terminal para hacer eso (ejecútalo desde la carpeta donde está almacenado tu archivo `manage.py`):
   ```bash
   python manage.py makemigrations reviews
   ```
   La salida del comando se ve así:
   ```text
   Migrations for 'reviews':
     reviews/migrations/0001_initial.py
       - Create model Publisher
   ```
   El comando `makemigrations <appname>` crea los scripts de migración para la aplicación dada, en este caso, para la aplicación `reviews`. Observa que después de ejecutar `makemigrations`, hay un nuevo archivo creado dentro de la carpeta `migrations`:
   *Figura 2.4: Nuevo archivo bajo la carpeta migrations*
   Este es el script de migración creado por Django. Cuando ejecutamos `makemigrations` sin el nombre de la aplicación, los scripts de migración se crearán para todas las aplicaciones del proyecto.

2. A continuación, enumeremos el estado de migración del proyecto. Recuerda que anteriormente aplicamos migraciones a las aplicaciones instaladas de Django, y ahora hemos creado una nueva aplicación, `reviews`. Ejecuta el siguiente comando en la consola o terminal, y mostrará el estado de las migraciones de modelos en todo el proyecto (ejecútalo desde la carpeta donde está almacenado tu archivo `manage.py`):
   ```bash
   python manage.py showmigrations
   ```
   La salida para el comando anterior es la siguiente:
   ```text
   admin
    [X] 0001_initial
    [X] 0002_logentry_remove_auto_add
    [X] 0003_logentry_add_action_flag_choices
   auth
    [X] 0001_initial
    [X] 0002_alter_permission_name_max_length
    [X] 0003_alter_user_email_max_length
    [X] 0004_alter_user_username_opts
    [X] 0005_alter_user_last_login_null
    [X] 0006_require_contenttypes_0002
    [X] 0007_alter_validators_add_error_messages
    [X] 0008_alter_user_username_max_length
    [X] 0009_alter_user_last_name_max_length
    [X] 0010_alter_group_name_max_length
    [X] 0011_update_proxy_permissions
   contenttypes
    [X] 0001_initial
    [X] 0002_remove_content_type_name
   reviews
    [ ] 0001_initial
   sessions
    [X] 0001_initial
   ```
   Aquí, la marca `[X]` indica que se han aplicado las migraciones. Observa la diferencia de que se han aplicado todas las migraciones de las otras aplicaciones, excepto la de `reviews`. El comando `showmigrations` se puede ejecutar para comprender el estado de la migración, pero este no es un paso obligatorio al realizar migraciones de modelos.

3. A continuación, comprendamos cómo transforma Django un modelo en una tabla de base de datos real. Django proporciona el comando `sqlmigrate` para mostrar el SQL real generado por una migración. Este no es un paso obligatorio en las migraciones de modelos, pero puede ser útil para una mayor comprensión. Deberíamos ver la siguiente salida:
   ```sql
   BEGIN;
   --
   -- Create model Publisher
   --
   CREATE TABLE "reviews_publisher" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "name" varchar(50) NOT NULL, "website" varchar(200) NULL, "email" varchar(254) NULL);
   COMMIT;
   ```
   El fragmento anterior muestra el comando SQL equivalente utilizado cuando Django migra la base de datos. En este caso, estamos creando la tabla `reviews_publisher` con los campos `name`, `website` y `email` con los tipos de campo definidos. Además, el campo `name` se define como `NOT NULL`, lo que implica que las entradas para este campo no pueden ser nulas y deben tener un valor. Las opciones de campo `null=True` o `unique=True` darán lugar a una definición SQL con `NULL` o `UNIQUE`, respectivamente.

En la siguiente sección, aprenderemos sobre las claves primarias y su importancia al almacenar datos en una base de datos.

#### Claves primarias (Primary keys)

Supongamos que una tabla de base de datos llamada `users`, como sugiere su nombre, almacena información sobre los usuarios. Digamos que tiene más de 1,000 registros y hay al menos 3 usuarios con el mismo nombre, Joe Burns. ¿Cómo identificamos unívocamente a estos usuarios desde la aplicación? La solución es tener una forma de identificar de forma única cada registro en la base de datos. Esto se hace mediante **claves primarias** (*primary keys*). Una clave primaria es única para una tabla de base de datos y, como regla, una tabla no puede tener dos filas con la misma clave primaria. En Django, cuando la clave primaria no se menciona explícitamente en los modelos de base de datos, Django crea automáticamente `id` como clave primaria (de tipo entero), que se incrementa automáticamente a medida que se crean nuevos registros.

En la sección anterior, observa la salida del comando `python manage.py sqlmigrate`. Al crear la tabla `Publisher`, el comando SQL `CREATE TABLE` agregó un campo más llamado `id` a la tabla. Un `id` se define como `PRIMARY KEY AUTOINCREMENT`. En las bases de datos relacionales, se utiliza una clave primaria para identificar de forma única una entrada en la base de datos. Por ejemplo, la tabla `book` tiene `id` como clave primaria, que tiene números que comienzan desde 1. Este valor se incrementa en uno a medida que se crean nuevos registros. El valor entero de `id` siempre es único en toda la tabla `book`. Dado que el script de migración ya se ha creado ejecutando `makemigrations`, ahora migremos el modelo recién creado en la aplicación `reviews` ejecutando el siguiente comando:

```bash
python manage.py migrate reviews
```

Deberíamos obtener la siguiente salida:

```text
Operations to perform:
  Apply all migrations: reviews
Running migrations:
  Applying reviews.0001_initial... OK
```

Esta operación crea la tabla de base de datos para la aplicación `reviews`. El siguiente es un fragmento de DB Browser que indica que se ha creado la nueva tabla `reviews_publisher` en la base de datos:

*Figura 2.5: La tabla reviews_publisher creada después de ejecutar el comando de migración*

Hasta ahora, hemos explorado cómo crear un modelo y migrarlo a la base de datos. Ahora trabajemos en la creación del resto de los modelos para nuestra aplicación de reseñas de libros. Como ya hemos visto, la aplicación tendrá tablas de base de datos para `Book`, `Publisher`, `Contributor` y `Review`.

Agreguemos los modelos `Book` y `Contributor`, como se muestra en el siguiente fragmento de código, en `reviews/models.py`:

```python
class Book(models.Model):
    """A published book."""
    title = models.CharField(
        max_length=70,
        help_text="The title of the book.")
    publication_date = models.DateField(
        verbose_name="Date the book was published.")
    isbn = models.CharField(
        blank=True,
        max_length=20,
        verbose_name="ISBN number of the book.")


class Contributor(models.Model):
    """A contributor to a Book, e.g. author, editor, co-author."""
    first_names = models.CharField(
        max_length=50,
        help_text="The contributor's first name or names.")
    last_names = models.CharField(
        blank=True,
        max_length=50,
        help_text="The contributor's last name or names.")
    email = models.EmailField(
        blank=True,
        help_text="The contact email for the contributor.")
```

El código se explica por sí mismo. El modelo `Book` tiene los campos `title`, `publication_date` e `isbn`. El modelo `Contributor` tiene los campos `first_names` y `last_names` y el correo electrónico del colaborador. También hay algunos modelos recién añadidos, además de los que hemos visto en el modelo `Publisher`. Tienen `DateField` como un nuevo tipo de campo, que, como su nombre indica, se utiliza para almacenar una fecha. También se utiliza una nueva opción de campo llamada `verbose_name`. Proporciona un nombre descriptivo para el campo. A continuación, veremos cómo funcionan las relaciones en una base de datos relacional.

---

### Sección: Relaciones en bases de datos

Uno de los puntos fuertes de las bases de datos relacionales es la capacidad de establecer relaciones entre los datos almacenados en las tablas de la base de datos. Las relaciones ayudan a mantener la integridad de los datos al establecer las referencias correctas entre las tablas, lo que a su vez ayuda a mantener la base de datos. Las reglas de relación, por otro lado, garantizan la coherencia de los datos y evitan duplicados.

En una base de datos relacional, pueden existir los siguientes tipos de relaciones:
- Muchos a uno (*Many to one*)
- Muchos a muchos (*Many to many*)
- Uno a uno (*One to one*)

Exploremos cada relación en detalle en las siguientes secciones.

#### Muchos a uno (Many to one)

En esta relación, muchos registros (filas/entradas) de una tabla pueden hacer referencia a un registro (fila/entrada) en otra tabla. Por ejemplo, puede haber muchos libros producidos por una editorial. Este es un ejemplo de una relación de muchos a uno. Para establecer esta relación, necesitamos usar las **claves foráneas** (*foreign keys*) de la base de datos. Una clave foránea en una base de datos relacional establece la relación entre un campo de una tabla y una clave primaria de una tabla diferente.

Por ejemplo, supongamos que tenemos datos sobre empleados que pertenecen a diferentes departamentos almacenados en una tabla llamada `employee_info` con su ID de empleado como clave primaria junto con una columna que almacena el nombre de su departamento; esta tabla también contiene una columna que almacena el ID del departamento. Ahora, hay otra tabla llamada `departments_info`, que tiene el ID del departamento como clave primaria. En este caso, entonces, el ID del departamento es una clave foránea en la tabla `employee_info`.

En nuestra aplicación `bookr`, el modelo `Book` puede tener una clave foránea que hace referencia a la clave primaria de la tabla `Publisher`. Dado que ya hemos creado los modelos para `Book`, `Contributor` y `Publisher`, ahora establezcamos una relación de muchos a uno entre los modelos `Book` y `Publisher`. Para el modelo `Book`, agrega la última línea:

```python
class Book(models.Model):
    """A published book."""
    title = models.CharField(
        max_length=70,
        help_text="The title of the book.")
    publication_date = models.DateField(
        verbose_name="Date the book was published.")
    isbn = models.CharField(
        blank=True,
        max_length=20,
        verbose_name="ISBN number of the book.")
    publisher = models.ForeignKey(
        Publisher,
        on_delete=models.CASCADE)
```

Ahora el campo `publisher` recién agregado establece una relación de muchos a uno entre `Book` y `Publisher` usando una clave foránea. Esta relación asegura la naturaleza de una relación de muchos a uno, que es que muchos libros pueden tener una editorial:
- `models.ForeignKey`: Esta es la opción de campo para establecer una relación de muchos a uno.
- `Publisher`: Cuando establecemos relaciones con diferentes tablas en Django, nos referimos al modelo que crea la tabla; en este caso, la tabla `Publisher` es creada por el modelo `Publisher` (o la clase de Python `Publisher`).
- `on_delete`: Esta es una opción de campo que determina la acción a tomar tras la eliminación del objeto referenciado. En este caso, la opción `on_delete` se establece en `CASCADE` (`models.CASCADE`), lo que elimina los objetos referenciados.

Por ejemplo, supongamos que una editorial ha publicado un conjunto de libros. Si por alguna razón la editorial tiene que ser eliminada de la aplicación, la siguiente acción es `CASCADE`, lo que significa eliminar todos los libros referenciados de la aplicación. Hay muchas más acciones `on_delete`, tales como las siguientes:
- **PROTECT**: Impide la eliminación del registro a menos que se eliminen todos los objetos referenciados.
- **SET_NULL**: Establece un valor nulo si el campo de la base de datos se ha configurado previamente para almacenar valores nulos.
- **SET_DEFAULT**: Se establece en un valor predeterminado al eliminar el objeto referenciado.

Solo usaremos la opción `CASCADE` para nuestra aplicación de reseñas de libros.

#### Muchos a muchos (Many to many)

En esta relación, múltiples registros en una tabla pueden tener una relación con múltiples registros en una tabla diferente. Por ejemplo, un libro puede tener múltiples coautores, y cada autor (colaborador) podría haber escrito múltiples libros. Entonces, esto forma una relación de muchos a muchos entre las tablas `Book` y `Contributor`:

*Figura 2.6: Relación de muchos a muchos entre libros y coautores*

En `models.py`, para el modelo `Book`, añade la última línea como se muestra aquí:

```python
class Book(models.Model):
    """A published book."""
    title = models.CharField(
        max_length=70,
        help_text="The title of the book.")
    publication_date = models.DateField(
        verbose_name="Date the book was published.")
    isbn = models.CharField(
        blank=True,
        max_length=20,
        verbose_name="ISBN number of the book.")
    publisher = models.ForeignKey(
        Publisher,
        on_delete=models.CASCADE)
    contributors = models.ManyToManyField(
        'Contributor',
        through="BookContributor")
```

El campo `contributors` recién agregado establece una relación de muchos a muchos con `Book` y `Contributor` utilizando el tipo de campo `ManyToManyField`:
- `models.ManyToManyField`: Este es el tipo de campo para establecer una relación de muchos a muchos.
- `through`: Esta es una opción de campo especial para relaciones de muchos a muchos. Cuando tenemos una relación de muchos a muchos entre dos tablas, si queremos almacenar información adicional sobre la relación, podemos usar esto para establecer la relación a través de una tabla intermediaria.

Por ejemplo, tenemos dos tablas, `Book` y `Contributor`, donde necesitamos almacenar la información sobre el tipo de colaborador del libro, como Autor, Coautor o Editor. Luego, el tipo de colaborador se almacena en una tabla intermediaria llamada `BookContributor`. Así es como se ve la tabla/modelo `BookContributor`. Asegúrate de incluir este modelo en `reviews/models.py`:

```python
class BookContributor(models.Model):
    class ContributionRole(models.TextChoices):
        AUTHOR = "AUTHOR", "Author"
        CO_AUTHOR = "CO_AUTHOR", "Co-Author"
        EDITOR = "EDITOR", "Editor"

    book = models.ForeignKey(
        Book,
        on_delete=models.CASCADE)
    contributor = models.ForeignKey(
        Contributor,
        on_delete=models.CASCADE)
    role = models.CharField(
        verbose_name="The role this contributor had in"
        " the book.",
        choices=ContributionRole.choices,
        max_length=20)
```

Una tabla intermediaria, como `BookContributor`, establece relaciones mediante el uso de claves foráneas tanto para la tabla `Book` como para la tabla `Contributor`. También puede tener campos adicionales que pueden almacenar información sobre la relación que tiene el modelo `BookContributor` con los siguientes campos:
- `book`: Esta es una clave foránea para el modelo `Book`. Como vimos anteriormente, `on_delete=models.CASCADE` eliminará una entrada de la tabla de relaciones cuando el libro relevante se elimine de la aplicación.
- `contributor`: Nuevamente, esta es una clave foránea para el modelo/tabla `Contributor`. Esto también se define como `CASCADE` tras la eliminación.
- `role`: Este es el campo del modelo intermediario, que almacena la información adicional sobre la relación entre `Book` y `Contributor`.
- `class ContributionRole(models.TextChoices)`: Esto se puede usar para definir un conjunto de opciones creando una subclase de `models.TextChoices`. Por ejemplo, `ContributionRole` es una subclase creada a partir de `TextChoices`, que es utilizada por el campo de roles para definir `Author`, `Co-Author` y `Editor` como un conjunto de opciones.
- `choices`: Esto se refiere a un conjunto de opciones definidas en los modelos, y son útiles al crear formularios de Django usando los modelos.

Cuando no se proporciona la opción de campo `through` al establecer una relación de muchos a muchos, Django crea automáticamente una tabla intermediaria para gestionar la relación.

#### Relaciones de uno a uno (One-to-one)

En esta relación, un registro en una tabla tendrá una referencia a solo un registro en una tabla diferente. Por ejemplo, una persona solo puede tener una licencia de conducir, por lo que una persona con una licencia de conducir podría formar una relación de uno a uno:

*Figura 2.7: Ejemplo de una relación uno a uno*

`OneToOneField` se puede utilizar para establecer una relación uno a uno, como se muestra aquí:

```python
class DriverLicence(models.Model):
    person = models.OneToOneField(
        Person,
        on_delete=models.CASCADE)
    licence_number = models.CharField(max_length=50)
```

Ahora que hemos explorado las relaciones de bases de datos, volvamos a nuestra aplicación `bookr` y agreguemos un modelo más allí.

#### Adición del modelo Review

Ya agregamos los modelos `Book` y `Publisher` al archivo `reviews/models.py`. El último modelo que vamos a agregar es el modelo `Review`. El siguiente fragmento de código debería ayudarnos a hacer esto:

```python
from django.contrib import auth

class Review(models.Model):
    content = models.TextField(
        help_text="The Review text.")
    rating = models.IntegerField(
        help_text="The rating the reviewer has given.")
    date_created = models.DateTimeField(
        auto_now_add=True,
        help_text="The date and time the"
        " review was created.")
    date_edited = models.DateTimeField(
        null=True,
        help_text="The date and time the review"
        " was last edited.")
    creator = models.ForeignKey(
        auth.get_user_model(),
        on_delete=models.CASCADE)
    book = models.ForeignKey(
        Book,
        on_delete=models.CASCADE,
        help_text="The Book that this review is for.")
```

El modelo/tabla `Review` se utilizará para almacenar comentarios de reseñas y calificaciones proporcionadas por los usuarios para los libros. Tiene los siguientes campos:
- `content`: Este campo almacena el texto de una reseña de libro; por lo tanto, el tipo de campo utilizado es `TextField`, ya que puede almacenar una gran cantidad de texto.
- `rating`: Este campo almacena la calificación de la reseña de un libro. Como la calificación será un número entero, el tipo de campo utilizado es `IntegerField`.
- `date_created`: Este campo almacena la hora y la fecha en que se escribió la reseña; por lo tanto, el tipo de campo es `DateTimeField`.
- `date_edited`: Este campo almacena la fecha y la hora cada vez que se edita una reseña. Nuevamente, el tipo de campo es `DateTimeField`.
- `creator`: Este campo especifica el creador de la reseña o la persona que escribe la reseña del libro. Observa que esta es una clave foránea para `auth.get_user_model()`, que hace referencia al modelo `User` del módulo de autenticación integrado de Django. Tiene una opción de campo `on_delete=models.CASCADE`. Esto explica que cuando un usuario se elimina de la base de datos, se eliminarán todas las reseñas escritas por ese usuario.
- `book`: Las reseñas tienen un campo llamado `book`, una clave foránea para el modelo `Book`. Esto se debe a que las reseñas deben escribirse para una aplicación de reseñas de libros, y un libro puede tener muchas reseñas, por lo que esta es una relación de muchos a uno. Esto también se define con una opción de campo `on_delete=models.CASCADE` porque una vez que se elimina el libro, no tiene sentido conservar las reseñas en la aplicación. Por lo tanto, cuando se elimina un libro, también se eliminarán todas las reseñas que hagan referencia al libro.

En la siguiente sección, aprenderemos sobre los métodos de modelo y los implementaremos.

#### Métodos de modelo (Model methods)

En Django, podemos escribir métodos dentro de una clase de modelo. Se denominan **métodos de modelo** y pueden ser métodos personalizados o especiales que anulan los métodos predeterminados de los modelos de Django. Uno de estos métodos es `__str__()`. Este método devuelve la representación en cadena de las instancias del modelo y puede resultar especialmente útil al utilizar la consola interactiva (*shell*) de Django. En el siguiente ejemplo, donde se añade el método `__str__()` al modelo `Publisher`, la representación en cadena del objeto `Publisher` será el nombre de la editorial; por lo tanto, se devuelve `self.name`, con `self` refiriéndose al objeto `Publisher`:

```python
class Publisher(models.Model):
    """A company that publishes books."""
    name = models.CharField(
        max_length=50,
        help_text="The name of the Publisher.")
    website = models.URLField(
        help_text="The Publisher's website.")
    email = models.EmailField(
        blank=True,
        help_text="The Publisher's email address.")

    def __str__(self):
        return self.name
```

Añade los métodos `__str__()` a `Contributor` y `Book` también, de la siguiente manera:

```python
class Book(models.Model):
    """A published book."""
    title = models.CharField(
        max_length=70,
        help_text="The title of the book.")
    publication_date = models.DateField(
        verbose_name="Date the book was published.")
    isbn = models.CharField(
        blank=True,
        max_length=20,
        verbose_name="ISBN number of the book.")
    publisher = models.ForeignKey(
        Publisher,
        on_delete=models.CASCADE)
    contributors = models.ManyToManyField(
        'Contributor',
        through="BookContributor")

    def __str__(self):
        return self.title


class Contributor(models.Model):
    """A contributor to a Book, e.g. author, editor, co-author."""
    first_names = models.CharField(
        max_length=50,
        help_text="The contributor's first name or names.")
    last_names = models.CharField(
        blank=True,
        max_length=50,
        help_text="The contributor's last name or names.")
    email = models.EmailField(
        blank=True,
        help_text="The contact email for the contributor.")

    def __str__(self):
        return self.first_names
```

De manera similar, la representación en cadena de `Book` es `title`, por lo que el valor devuelto es `self.title`, donde `self` se refiere al objeto `Book`. La representación en cadena de `Contributor` es el primer nombre del colaborador; por lo tanto, se devuelve `self.first_names`. Aquí, `self` se refiere al objeto `Contributor`. A continuación, veremos la migración de la aplicación `reviews`.

#### Migración de la aplicación reviews

Dado que tenemos todo el archivo de modelo listo, ahora migremos los modelos a la base de datos, similar a lo que hicimos antes con las aplicaciones instaladas. Como la aplicación `reviews` tiene un conjunto de modelos creados por nosotros, es importante crear los scripts de migración antes de ejecutar la migración. Los scripts de migración ayudan a identificar cualquier cambio en los modelos y propagarán estos cambios a la base de datos mientras se ejecuta la migración. Sigue estos pasos para crear scripts de migración y luego migrar modelos a la base de datos:

1. Ejecuta el siguiente comando para crear los scripts de migración:
   ```bash
   python manage.py makemigrations reviews
   ```
   Deberíamos obtener una salida similar a esta:
   ```text
   reviews/migrations/0002_auto_20191007_0112.py
     - Create model Book
     - Create model Contributor
     - Create model Review
     - Create model BookContributor
     - Add field contributors to book
     - Add field publisher to book
   ```
   Los scripts de migración se crearán en una carpeta llamada `migrations` en la carpeta de la aplicación.

2. A continuación, migra todos los modelos a la base de datos utilizando el comando `migrate`:
   ```bash
   python manage.py migrate reviews
   ```
   Deberíamos ver la siguiente salida:
   ```text
   Operations to perform:
     Apply all migrations: reviews
   Running migrations:
     Applying reviews.0002_auto_20191007_0112... OK
   ```

Después de ejecutar este comando, creamos con éxito las tablas de base de datos definidas en la aplicación `reviews`. Puedes utilizar DB Browser for SQLite para explorar las tablas que acabas de crear después de la migración.

Para hacerlo, abre DB Browser for SQLite, haz clic en el botón **Open Database** y navega hasta el directorio de tu proyecto. Selecciona el archivo de base de datos `db.sqlite3` para abrirlo. Ahora deberíamos poder explorar los nuevos conjuntos de tablas creados. La siguiente captura de pantalla muestra las tablas de la base de datos definidas en la aplicación `reviews`:

*Figura 2.8: Tablas de base de datos según lo definido en la aplicación reviews*

En esta sección, aprendimos más sobre los modelos y las migraciones de Django y cómo las clases simples de Python pueden transformarse en tablas de bases de datos. También aprendimos cómo varios atributos de clase se traducen en columnas de base de datos apropiadas siguiendo los tipos de campo definidos. Más adelante, aprendimos sobre claves primarias y diferentes tipos de relaciones que pueden existir en una base de datos. También creamos modelos para la aplicación de reseñas de libros y migramos esos modelos, traduciéndolos en tablas de base de datos. En la siguiente sección, aprenderemos cómo realizar operaciones CRUD de base de datos utilizando el ORM de Django.

---

### Sección: Operaciones CRUD de base de datos en Django

Como hemos creado las tablas de base de datos necesarias para la aplicación de reseñas de libros, trabajemos en la comprensión de las operaciones básicas de base de datos con Django.

**CRUD** es un acrónimo de Crear, Leer, Actualizar y Eliminar (*Create, Read, Update, Delete*), las cuatro operaciones principales de un sistema de base de datos. En SQL, estas operaciones se implementan con las declaraciones `INSERT`, `SELECT`, `UPDATE` y `DELETE`.

El ORM de Django proporciona la misma funcionalidad sin tener que lidiar con declaraciones SQL. Las operaciones de base de datos de Django son código Python simple, por lo que superamos la molestia de mantener declaraciones SQL dentro del código Python. Echemos un vistazo a cómo se realizan.

Para ejecutar las operaciones CRUD, entraremos a la consola de comandos de Django ejecutando el siguiente comando:

```bash
python manage.py shell
```

Cuando se inicia la consola interactiva, se ve de la siguiente manera:

```text
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```

En el siguiente ejercicio, realizaremos una operación de creación (*create*).

#### Ejercicio 2.01 – Creación de una entrada en la base de datos de bookr

En este ejercicio, crearemos una nueva entrada en la base de datos guardando una instancia del modelo. En otras palabras, crearemos una entrada en una tabla de base de datos sin ejecutar explícitamente una consulta SQL. Sigue estos pasos para hacer esto:

1. Primero, importa la clase/modelo `Publisher` desde `reviews.models`:
   ```python
   >>> from reviews.models import Publisher
   ```

2. Crea un objeto o una instancia de la clase `Publisher` pasando todos los valores de campo (`name`, `website` y `email`) requeridos por el modelo `Publisher`:
   ```python
   >>> publisher = Publisher(name='Packt Publishing', website='https://www.packtpub.com', email='info@packtpub.com')
   ```

3. A continuación, para escribir el objeto en la base de datos, es importante llamar al método `save()`, porque hasta que se llame a este, no se creará una entrada en la base de datos:
   ```python
   >>> publisher.save()
   ```

4. Ahora podemos ver una nueva entrada creada en la base de datos usando DB Browser:
   *Figura 2.9: Una entrada creada en la base de datos*

5. Usa los atributos del objeto para realizar cambios adicionales en el objeto y guarda los cambios en la base de datos:
   ```python
   >>> publisher.email = 'info@packtpub.com'
   >>> publisher.email = 'customersupport@packtpub.com'
   >>> publisher.save()
   ```

6. Podemos ver los cambios usando DB Browser de la siguiente manera:
   *Figura 2.10: Una entrada con el campo email actualizado*

En este ejercicio, creamos una entrada en la base de datos creando una instancia del objeto de modelo y usamos el método `save()` para escribir el objeto de modelo en la base de datos.

Ten en cuenta que siguiendo el método anterior, los cambios en la instancia de la clase no se guardan hasta que se llama al método `save()`. Sin embargo, si usamos el método `create()`, Django guarda los cambios en la base de datos en un solo paso. Usaremos este método en el ejercicio que sigue.

#### Ejercicio 2.02 – Uso del método create() para crear una entrada

A diferencia del ejercicio anterior, donde ejecutamos dos pasos separados creando primero un objeto y luego usando el método `save()` para crear una entrada en la base de datos, en este ejercicio crearemos un registro en la tabla de colaboradores usando el método `create()` en un solo paso:

1. Primero, importa la clase `Contributor` como antes:
   ```python
   >>> from reviews.models import Contributor
   ```

2. Invoca el método `create()` para crear un objeto en la base de datos en un solo paso. Asegúrate de pasar todos los parámetros requeridos (`first_names`, `last_names` y `email`):
   ```python
   >>> contributor = Contributor.objects.create(
   ...     first_names="Rowel",
   ...     last_names="Atienza",
   ...     email="RowelAtienza@example.com")
   ```

3. Usa DB Browser para verificar que el registro del colaborador se haya creado en la base de datos. Si tu DB Browser aún no está abierto, abre el archivo de base de datos `db.sqlite3` como acabamos de hacer en la sección anterior. Haz clic en **Browse Data** y selecciona la tabla deseada (en este caso, la tabla `reviews_contributor` del menú desplegable *Table*, como se muestra en la captura de pantalla) y verifica el registro de base de datos recién creado:
   *Figura 2.11: Verificación de la creación del registro en DB Browser*

En este ejercicio, aprendimos que usando el método `create()`, podemos crear un registro para un modelo en una base de datos en un solo paso.

#### Creación de un objeto con una clave foránea

Similar a cómo creamos un registro en las tablas `Publisher` y `Contributor`, ahora creemos uno para la tabla `Book`. Si recuerdas, el modelo `Book` tiene una clave foránea para `Publisher` que no puede tener un valor nulo. Por lo tanto, una forma de completar la clave foránea de la editorial es proporcionando el objeto de editorial creado en el campo `publisher` del libro, como se muestra en el siguiente ejercicio.

#### Ejercicio 2.03 – Creación de registros para una relación de muchos a uno

En este ejercicio, crearemos un registro en la tabla `Book`, incluida una clave foránea para el modelo `Publisher`. Como ya sabes, la relación entre `Book` y `Publisher` es una relación de muchos a uno, por lo que primero debes obtener el objeto `Publisher` y luego usarlo al crear el registro del libro. Para hacer eso, sigue estos pasos:

1. Primero, importa las clases `Book` y `Publisher`:
   ```python
   >>> from reviews.models import Book, Publisher
   ```

2. Recupera el objeto de la editorial de la base de datos mediante el siguiente comando. El método `get()` se utiliza para recuperar un objeto de la base de datos. Todavía no hemos explorado las operaciones de lectura de bases de datos. Por ahora, usa el siguiente comando; profundizaremos en la lectura/recuperación de bases de datos en la siguiente sección:
   ```python
   >>> publisher = Publisher.objects.get(
   ...     name='Packt Publishing')
   ```

3. Al crear un libro, necesitamos suministrar un objeto de fecha, ya que `publication_date` es un campo de fecha en el modelo `Book`. Por lo tanto, importa `date` desde `datetime` para que se pueda suministrar un objeto de fecha al crear el objeto de libro, como se muestra en el siguiente código:
   ```python
   >>> from datetime import date
   ```

4. Usa el método `create()` para crear un registro del libro en la base de datos. Asegúrate de pasar todos los campos, a saber, `title`, `publication_date`, `isbn` y el objeto `publisher`:
   ```python
   >>> book = Book.objects.create(
   ...     title="Advanced Deep Learning with Keras",
   ...     publication_date=date(2018, 10, 31),
   ...     isbn="9781788629416",
   ...     publisher=publisher)
   ```

Ten en cuenta que dado que `publisher` es una clave foránea y no admite valores nulos (no puede contener un valor nulo), es obligatorio pasar un objeto `publisher`. Cuando no se proporciona el objeto de clave foránea obligatorio `publisher`, la base de datos generará un error de integridad (*integrity error*).

La Figura 2.12 muestra la tabla `Book`, donde se crea la primera entrada. Observa que el campo de clave foránea (`publisher_id`) apunta al `id` (clave primaria) de la tabla `Publisher`. La entrada `publisher_id` en el registro del libro apunta a un registro de `Publisher` que tiene el `id` (clave primaria) 1, como se muestra en las siguientes dos capturas de pantalla:

*Figura 2.12: La clave foránea publisher_id de reviews_book apunta a la clave primaria de reviews_publisher*

En este ejercicio, aprendimos que al crear un registro de base de datos, se puede asignar un objeto a un campo si es una clave foránea. Sabemos que el modelo `Book` también tiene una relación de muchos a muchos con el modelo `Contributor`. Ahora exploremos las formas de establecer relaciones de muchos a muchos a medida que creamos registros en la base de datos.

#### Ejercicio 2.04 – Creación de registros con relaciones de muchos a muchos

En este ejercicio, crearemos una relación de muchos a muchos entre `Book` y `Contributor` utilizando el modelo de relación `BookContributor`:

1. En caso de que hayas reiniciado la consola y hayas perdido los objetos de editorial y libro, recupéralos de la base de datos mediante el siguiente conjunto de instrucciones de Python:
   ```python
   >>> from reviews.models import Book
   >>> from reviews.models import Contributor
   >>> contributor = Contributor.objects.get(
   ...     first_names='Rowel')
   >>> book = Book.objects.get(
   ...     title="Advanced Deep Learning with Keras")
   ```

2. La forma de establecer una relación de muchos a muchos es almacenando la información sobre la relación en el modelo intermediario o el modelo de relación; en este caso, `BookContributor`. Como ya hemos obtenido los registros del libro y del colaborador de la base de datos, usemos estos objetos al crear un registro para el modelo de relación `BookContributor`. Para hacerlo, primero crea una instancia de la clase de relación `BookContributor` y luego guarda el objeto en la base de datos. Al hacerlo, asegúrate de pasar los campos requeridos, a saber, el objeto `book`, el objeto `contributor` y `role`:
   ```python
   >>> from reviews.models import BookContributor
   >>> book_contributor = BookContributor(book=book, contributor=contributor, role='AUTHOR')
   >>> book_contributor.save()
   ```

Observa que especificamos el rol como `AUTHOR` al crear el objeto `book_contributor`. Este es un ejemplo clásico de almacenamiento de datos de relación al establecer una relación de muchos a muchos. El rol puede ser `AUTHOR`, `CO_AUTHOR` o `EDITOR`.

Esto estableció la relación entre el libro *Advanced Deep Learning with Keras* y el colaborador Rowel (siendo Rowel el autor del libro).

En este ejercicio, establecimos una relación de muchos a muchos entre `Book` y `Contributor` utilizando el modelo de relación `BookContributor`. Con respecto a la verificación de la relación de muchos a muchos que acabamos de crear, veremos esto en detalle en algunos ejercicios más adelante en este capítulo.

#### Ejercicio 2.05 – Una relación de muchos a muchos usando el método add()

En este ejercicio, estableceremos una relación de muchos a muchos utilizando el método `add()`. Cuando no usamos la relación para crear objetos, podemos usar `through_defaults` para pasar un diccionario con los parámetros que definen los campos requeridos. Continuando con el ejercicio anterior, agreguemos un colaborador más al libro titulado *Advanced Deep Learning with Keras*. Esta vez, el colaborador es un editor del libro:

1. Si has reiniciado la consola, ejecuta los siguientes dos comandos para importar y obtener la instancia de libro deseada:
   ```python
   >>> from reviews.models import Book, Contributor
   >>> book = Book.objects.get(
   ...     title="Advanced Deep Learning with Keras")
   ```

2. Usa el método `create()` para crear un colaborador, como se muestra aquí:
   ```python
   >>> contributor = Contributor.objects.create(
   ...     first_names='Packt',
   ...     last_names='Example Editor',
   ...     email='PacktEditor@example.com')
   ```

3. Agrega el colaborador recién creado al libro mediante el método `add()`. Asegúrate de proporcionar el parámetro de relación de rol como un diccionario (`dict`). Ingresa el siguiente código:
   ```python
   >>> book.contributors.add(contributor, through_defaults={'role': 'EDITOR'})
   ```

De este modo, utilizamos el método `add()` para establecer una relación de muchos a muchos entre el libro y el colaborador mientras almacenamos el rol de datos de relación como Editor. Ahora echemos un vistazo a otras formas de hacer esto.

#### Uso de los métodos create() y set() para relaciones de muchos a muchos

Supongamos que el libro *Advanced Deep Learning with Keras* tiene un total de dos editores. Usemos el siguiente método para agregar otro editor al libro. Si el colaborador aún no está presente en la base de datos, podemos usar el método `create()` para crear simultáneamente una entrada, así como para establecer la relación con el libro:

```python
>>> book.contributors.create(first_names='Packtp', last_names='Editor Example', email='PacktEditor2@example.com', through_defaults={'role': 'EDITOR'})
```

Del mismo modo, también podemos usar el método `set()` para agregar una lista de colaboradores para un libro. Creemos una editorial, un conjunto de dos colaboradores que son los coautores y un objeto de libro. Primero, importa el modelo `Publisher`, si aún no está importado, usando el siguiente código:

```python
>>> from reviews.models import Publisher
```

El siguiente código nos ayudará a hacerlo:

```python
>>> publisher = Publisher.objects.create(
...     name='Pocket Books',
...     website='https://pocketbookssampleurl.com',
...     email='pocketbook@example.com')
>>> contributor1 = Contributor.objects.create(
...     first_names='Stephen',
...     last_names='Stephen',
...     email='StephenKing@example.com')
>>> contributor2 = Contributor.objects.create(
...     first_names='Peter',
...     last_names='Straub',
...     email='PeterStraub@example.com')
>>> book = Book.objects.create(title='The Talisman', publication_date=date(2012, 9, 25), isbn='9781451697216', publisher=publisher)
```

Dado que se trata de una relación de muchos a muchos, podemos agregar una lista de objetos de una sola vez usando el método `set()`. Podemos usar `through_defaults` para especificar el rol de los colaboradores; en este caso, son coautores:

```python
>>> book.contributors.set([contributor1, contributor2], through_defaults={'role': 'CO_AUTHOR'})
```

---

### Sección: Operaciones de lectura (Read operations)

Django nos proporciona métodos que nos permiten leer/recuperar de la base de datos. Podemos recuperar un solo objeto de la base de datos mediante el método `get()`. Ya hemos creado algunos registros en las secciones anteriores, así que usemos el método `get()` para recuperar un objeto.

#### Ejercicio 2.06 – Uso del método get() para recuperar un objeto

En este ejercicio, recuperaremos un objeto de la base de datos mediante el método `get()`. Sigue estos pasos:

1. Obtén un objeto `Publisher` que tenga un campo `name` con el valor `Pocket Books`:
   ```python
   >>> from reviews.models import Publisher
   >>> publisher = Publisher.objects.get(
   ...     name='Pocket Books')
   ```

2. Vuelve a ingresar el objeto de editorial recuperado y presiona Enter:
   ```python
   >>> publisher
   <Publisher: Pocket Books>
   ```
   Observa que la salida se muestra en la consola. Esto se llama **representación en cadena de un objeto**. Es el resultado de agregar el método de modelo `__str__()` como lo hicimos en la sección *Métodos de modelo* para la clase `Publisher`.

3. Al recuperar el objeto, tenemos acceso a todos los atributos del objeto. Como se trata de un objeto Python, se puede acceder a los atributos del objeto mediante la notación de punto (`.` seguido del nombre del atributo). Por lo tanto, podemos recuperar el nombre de la editorial con el siguiente comando:
   ```python
   >>> publisher.name
   'Pocket Books'
   ```

4. De manera similar, para recuperar el sitio web de la editorial, usa lo siguiente:
   ```python
   >>> publisher.website
   'https://pocketbookssampleurl.com'
   ```

5. La dirección de correo electrónico de la editorial se puede recuperar de la siguiente manera:
   ```python
   >>> publisher.email
   'pocketbook@example.com'
   ```

En este ejercicio, aprendimos cómo obtener un solo objeto mediante el método `get()`. Sin embargo, existen varias desventajas al usar este método. Averigüemos cuáles son a continuación.

#### Devolución de un objeto mediante el método get()

Es importante tener en cuenta que el método `get()` solo puede obtener un objeto. Si hay otro objeto que tiene el mismo valor que el campo mencionado, podemos esperar un mensaje de error que indique que se devolvió más de uno (*MultipleObjectsReturned*). Por ejemplo, si hay dos entradas en la tabla `Publisher` con el mismo valor para el campo `name`, podemos esperar un error. En tales casos, existen formas alternativas de recuperar esos objetos, que exploraremos en las secciones posteriores.

También podemos recibir un mensaje de error que indique que la consulta coincidente no existe (*DoesNotExist*) cuando la consulta `get()` no devuelve ningún objeto. El método `get()` se puede utilizar con cualquiera de los campos del objeto para recuperar un registro. En el siguiente caso, estamos utilizando el campo `website`:

```python
>>> publisher = Publisher.objects.get(
...     website='https://pocketbookssampleurl.com')
```

Después de recuperar el objeto, aún podemos obtener el nombre de la editorial, como se muestra aquí:

```python
>>> publisher.name
'Pocket Books'
```

Otra forma de recuperar un objeto es mediante el uso de su clave primaria, `pk`, como se puede ver aquí:

```python
>>> Publisher.objects.get(pk=2)
<Publisher: Pocket Books>
```

El uso de `pk` para la clave primaria es una forma más genérica de utilizar el campo de clave primaria. Pero para la tabla `Publisher`, dado que sabemos que `id` es la clave primaria, simplemente podemos usar el nombre del campo `id` para crear nuestra consulta `get()`:

```python
>>> Publisher.objects.get(id=2)
<Publisher: Pocket Books>
```

Para `Publisher` y todas las demás tablas, la clave primaria es `id`, que Django creó automáticamente. Esto sucede cuando no se menciona un campo de clave primaria en el momento de la creación de la tabla. Pero puede haber casos en los que un campo se pueda declarar explícitamente como una clave primaria.

En el siguiente ejercicio, veremos cómo consultar y recuperar todos los objetos para un modelo determinado.

#### Ejercicio 2.07 – Uso del método all() para recuperar un conjunto de objetos

Podemos usar el método `all()` para recuperar un conjunto de todos los objetos. En este ejercicio, usaremos este método para recuperar los nombres de todos los colaboradores. Para hacer eso, sigue estos pasos:

1. Agrega el siguiente código para recuperar todos los objetos de la tabla `Contributor`:
   ```python
   >>> from reviews.models import Contributor
   >>> Contributor.objects.all()
   <QuerySet [<Contributor: Rowel>, <Contributor: Packt>, <Contributor: Packtp>, <Contributor: Stephen>, <Contributor: Peter>]>
   ```
   Tras la ejecución, obtendremos un `QuerySet` de todos los objetos.

2. Podemos usar la indexación de listas para buscar un objeto específico o para iterar sobre la lista usando un bucle para realizar cualquier otra operación:
   ```python
   >>> contributors = Contributor.objects.all()
   ```

3. Dado que `Contributor` es una lista de objetos, podemos usar la indexación para acceder a cualquier elemento de la lista, como se muestra en el siguiente comando:
   ```python
   >>> contributors[0]
   <Contributor: Rowel>
   ```
   En este caso, el primer elemento de la lista es un colaborador con un valor `first_names` de `'Rowel'` y un valor `last_names` de `'Atienza'`, como podemos ver en el siguiente código:
   ```python
   >>> contributors[0].first_names
   'Rowel'
   >>> contributors[0].last_names
   'Atienza'
   ```

En este ejercicio, aprendimos cómo recuperar todos los objetos utilizando el método `all()` y cómo utilizar el conjunto de objetos recuperado como una lista.

#### Recuperación de objetos mediante filtrado

Si tenemos más de un objeto para un valor de campo, no podemos usar el método `get()` ya que el método `get()` solo puede devolver un objeto. Para tales casos, tenemos el método `filter()`, que puede recuperar todos los objetos que coincidan con una condición específica.

#### Ejercicio 2.08 – Uso del método filter() para recuperar objetos

En este ejercicio, usaremos el método `filter()` para obtener un conjunto específico de objetos para una determinada condición. Específicamente, recuperaremos los nombres de todos los colaboradores cuyo primer nombre sea Peter. Para hacer eso, sigue estos pasos:

1. Primero, crea dos colaboradores más:
   ```python
   >>> from reviews.models import Contributor
   >>> Contributor.objects.create(first_names='Peter', last_names='Wharton', email='PeterWharton@example.com')
   >>> Contributor.objects.create(first_names='Peter', last_names='Tyrrell', email='PeterTyrrell@example.com')
   ```

2. Para recuperar aquellos colaboradores que tienen un valor `first_names` de Peter, agrega el siguiente código:
   ```python
   >>> Contributor.objects.filter(first_names='Peter')
   <QuerySet [<Contributor: Peter>, <Contributor: Peter>, <Contributor: Peter>]>
   ```

3. El método `filter()` devuelve el objeto incluso si solo hay uno. Podemos ver esto aquí:
   ```python
   >>> Contributor.objects.filter(first_names='Rowel')
   <QuerySet [<Contributor: Rowel>]>
   ```

4. Además, el método `filter()` devuelve un `QuerySet` vacío si ninguno coincide con la consulta. Esto se puede ver aquí:
   ```python
   >>> Contributor.objects.filter(first_names='Nobody')
   <QuerySet []>
   ```

En este ejercicio, vimos el uso de filtros para recuperar un conjunto de algunos objetos filtrados por una determinada condición. En la siguiente sección, aprenderemos sobre el filtrado mediante el uso de búsquedas de campo (*field lookups*).

#### Filtrado por búsquedas de campo (Field lookups)

Ahora, supongamos que queremos filtrar y consultar un conjunto de objetos utilizando los campos de los objetos proporcionando ciertas condiciones. En este caso, podemos usar una búsqueda de doble guion bajo (`__`). Por ejemplo, el objeto `Book` tiene un campo `publication_date`; digamos que queremos filtrar y obtener todos los libros que se publicaron después del 01-01-2014. Podemos buscarlos fácilmente utilizando el método de doble guion bajo. Para hacer esto, primero importaremos el modelo `Book`:

```python
>>> from reviews.models import Book
>>> book = Book.objects.filter(
...     publication_date__gt=date(2014, 1, 1))
```

Aquí, `publication_date__gt` indica la fecha de publicación, que es mayor que (*greater than* o `gt`) una determinada fecha especificada; en este caso, 01-01-2014. Similar a esto, tenemos las siguientes abreviaturas:
- **lt**: Menor que (*less than*)
- **lte**: Menor o igual que (*less than or equal to*)
- **gte**: Mayor o igual que (*greater than or equal to*)

El resultado después del filtrado se puede ver aquí:

```python
>>> book
<QuerySet [<Book: Advanced Deep Learning with Keras>]>
```

Aquí está la fecha de publicación del libro que forma parte del conjunto de consultas, lo que confirma que la fecha de publicación fue posterior al 01-01-2014:

```python
>>> book[0].publication_date
datetime.date(2018, 10, 31)
```

#### Uso de coincidencia de patrones para operaciones de filtrado

Para los resultados filtrados, también podemos buscar si el parámetro contiene una parte de la cadena que estamos buscando:

```python
>>> book = Book.objects.filter(
...     title__contains='Deep learning')
```

Aquí, `title__contains` busca todos aquellos objetos con títulos que contienen `'Deep learning'` como parte de la cadena:

```python
>>> book
<QuerySet [<Book: Advanced Deep Learning with Keras>]>
>>> book[0].title
'Advanced Deep Learning with Keras'
```

Del mismo modo, podemos usar `icontains` si la coincidencia de cadenas no debe distinguir entre mayúsculas y minúsculas (*case-insensitive*). El uso de `startswith` coincide con cualquier cadena que comience con la cadena especificada.

#### Recuperación de objetos mediante el método exclude()

En la sección anterior, aprendimos cómo obtener un conjunto de objetos que coincidan con una determinada condición. Ahora, supongamos que queremos hacer lo contrario; es decir, queremos obtener todos aquellos objetos que no coincidan con una determinada condición. En tales casos, podemos usar el método `exclude()` para excluir una determinada condición y obtener todos los objetos requeridos. Esto quedará más claro con un ejemplo. La siguiente es una lista de todos los colaboradores:

```python
>>> Contributor.objects.all()
<QuerySet [<Contributor: Rowel>, <Contributor: Packt>, <Contributor: Packtp>, <Contributor: Stephen>, <Contributor: Peter>, <Contributor: Peter>, <Contributor: Peter>]>
```

Ahora, de esta lista, excluiremos a todos aquellos colaboradores que tengan el valor `first_names` de Peter:

```python
>>> Contributor.objects.exclude(first_names='Peter')
<QuerySet [<Contributor: Rowel>, <Contributor: Packt>, <Contributor: Packtp>, <Contributor: Stephen>]>
```

Vemos aquí que la consulta devolvió a todos aquellos colaboradores cuyo primer nombre no es Peter.

#### Recuperación de objetos mediante el método order_by()

Podemos recuperar una lista de objetos mientras ordenamos por un campo especificado mediante el método `order_by()`. Por ejemplo, en el siguiente fragmento de código, ordenamos los libros por su fecha de publicación:

```python
>>> books = Book.objects.order_by("publication_date")
>>> books
<QuerySet [<Book: The Talisman>, <Book: Advanced Deep Learning with Keras>]>
```

Examinemos el orden de la consulta. Como el conjunto de consultas es una lista, podemos usar la indexación para verificar la fecha de publicación de cada libro:

```python
>>> books[0].publication_date
datetime.date(2012, 9, 25)
>>> books[1].publication_date
datetime.date(2018, 10, 31)
```

Observa que la fecha de publicación del primer libro con un índice 0 es más antigua que la fecha de publicación del segundo libro con un índice 1. Por lo tanto, esto confirma que la lista de libros consultada se ha ordenado correctamente según sus fechas de publicación. También podemos usar un prefijo con el signo negativo para el parámetro de campo para ordenar los resultados en orden descendente. Esto se puede ver en el siguiente fragmento de código:

```python
>>> books = Book.objects.order_by("-publication_date")
>>> books
<QuerySet [<Book: Advanced Deep Learning with Keras>, <Book: The Talisman>]>
```

Dado que hemos puesto como prefijo un signo negativo a la fecha de publicación, observa que el conjunto de libros consultado ahora se ha devuelto en el orden opuesto, donde el primer objeto de libro con un índice 0 tiene una fecha más reciente que el segundo libro:

```python
>>> books[0].publication_date
datetime.date(2018, 10, 31)
>>> books[1].publication_date
datetime.date(2012, 9, 25)
```

También podemos ordenar utilizando un campo de cadena o uno numérico. Por ejemplo, el siguiente código se puede utilizar para ordenar libros por su clave primaria o `id`:

```python
>>> books = Book.objects.order_by('id')
>>> books
<QuerySet [<Book: Advanced Deep Learning with Keras>, <Book: The Talisman>]>
```

El conjunto de libros consultado se ha ordenado según el `id` del libro en orden ascendente:

```python
>>> books[0].id
1
>>> books[1].id
2
```

Nuevamente, para ordenar en orden descendente, se puede usar el signo negativo como prefijo, de la siguiente manera:

```python
>>> books = Book.objects.order_by('-id')
>>> books
<QuerySet [<Book: The Talisman>, <Book: Advanced Deep Learning with Keras>]>
```

Ahora, el conjunto de libros consultado se ha ordenado según el `id` del libro en orden descendente:

```python
>>> books[0].id
2
>>> books[1].id
1
```

Para ordenar por un campo de cadena en orden alfabético, podemos hacer algo como esto:

```python
>>> books = Book.objects.order_by('title')
>>> books
<QuerySet [<Book: Advanced Deep Learning with Keras>, <Book: The Talisman>]>
```

Dado que hemos utilizado el título del libro para ordenar, el conjunto de consultas se ha ordenado en orden alfabético. Podemos ver esto de la siguiente manera:

```python
>>> books[0]
<Book: Advanced Deep Learning with Keras>
>>> books[1]
<Book: The Talisman>
```

Similar a lo que hemos visto para los tipos de ordenamiento anteriores, el prefijo del signo negativo puede ayudarnos a ordenar en orden alfabético inverso, como podemos ver aquí:

```python
>>> books = Book.objects.order_by('-title')
>>> books
<QuerySet [<Book: The Talisman>, <Book: Advanced Deep Learning with Keras>]>
```

Esto conducirá a la siguiente salida:

```python
>>> books[0]
<Book: The Talisman>
>>> books[1]
<Book: Advanced Deep Learning with Keras>
```

Otro método útil que ofrece Django es `values()`. Nos ayuda a obtener un conjunto de consultas de diccionarios en lugar de objetos. En el siguiente fragmento de código, estamos usando esto para un objeto `Publisher`:

```python
>>> publishers = Publisher.objects.all().values()
>>> publishers
<QuerySet [{'id': 1, 'name': 'Packt Publishing', 'website': 'https://www.packtpub.com', 'email': 'customersupport@packtpub.com'}, {'id': 2, 'name': 'Pocket Books', 'website': 'https://pocketbookssampleurl.com', 'email': 'pocketbook@example.com'}]>
>>> publishers[0]
{'id': 1, 'name': 'Packt Publishing', 'website': 'https://www.packtpub.com', 'email': 'customersupport@packtpub.com'}
>>> publishers[0]
{'id': 1, 'name': 'Packt Publishing', 'website': 'https://www.packtpub.com', 'email': 'customersupport@packtpub.com'}
```

En las siguientes secciones, exploraremos cómo realizar consultas a través de relaciones.

---

### Sección: Consultas a través de relaciones (Querying across relationships)

Como hemos estudiado en este capítulo, la aplicación `reviews` tiene dos tipos de relaciones: de muchos a uno y de muchos a muchos. Hasta ahora, hemos aprendido varias formas de realizar consultas utilizando `get()`, filtros, búsquedas de campos, etc. Ahora estudiemos cómo realizar consultas a través de relaciones. Hay varias formas de abordar esto: podríamos usar claves foráneas, instancias de objetos y más. Exploremos esto con la ayuda de algunos ejemplos.

#### Consultas utilizando claves foráneas

Cuando tenemos relaciones entre dos modelos/tablas, Django proporciona una forma de realizar una consulta utilizando las relaciones. El comando que se muestra en esta sección recuperará todos los libros publicados por Packt Publishing realizando una consulta utilizando relaciones de modelos. De manera similar a lo que hemos visto anteriormente, esto se hace mediante la búsqueda de doble guion bajo. Por ejemplo, el modelo `Book` tiene una clave foránea de `publisher` que apunta al modelo `Publisher`. Utilizando esta clave foránea, podemos realizar una consulta utilizando guiones bajos dobles y el campo `name` en el modelo `Publisher`. Esto se puede ver en el siguiente código:

```python
>>> Book.objects.filter(publisher__name='Packt Publishing')
<QuerySet [<Book: Advanced Deep Learning with Keras>]>
```

#### Consultas utilizando el nombre del modelo

Otra forma de consultar es utilizar una relación para realizar la consulta en sentido inverso (*reverse lookup*), utilizando el nombre del modelo en minúsculas. Por ejemplo, digamos que queremos consultar la editorial que publicó el libro *Advanced Deep Learning with Keras* utilizando relaciones de modelos en la consulta. Para esto, podemos ejecutar la siguiente declaración para recuperar el objeto de información de `Publisher`:

```python
>>> Publisher.objects.get(
...     book__title='Advanced Deep Learning with Keras')
<Publisher: Packt Publishing>
```

Aquí, `book` es el nombre del modelo en minúsculas. Como ya sabemos, el modelo `Book` tiene una clave foránea `publisher` con un valor de nombre `Packt Publishing`.

#### Consultas a través de relaciones de clave foránea utilizando la instancia del objeto

También podemos recuperar la información utilizando la clave foránea del objeto. Supongamos que queremos consultar el nombre de la editorial para el título *The Talisman*:

```python
>>> book = Book.objects.get(title='The Talisman')
>>> book.publisher
<Publisher: Pocket Books>
```

El uso del objeto aquí es un ejemplo en el que usamos la dirección inversa para obtener todos los libros publicados por una editorial mediante el método `set.all()`:

```python
>>> publisher = Publisher.objects.get(name='Pocket Books')
>>> publisher.book_set.all()
<QuerySet [<Book: The Talisman>]>
```

También podemos crear consultas utilizando cadenas de consultas:

```python
>>> Book.objects.filter(
...     publisher__name='Pocket Books').filter(
...     title='The Talisman')
<QuerySet [<Book: The Talisman>]>
```

Realicemos algunos ejercicios más para afianzar nuestro conocimiento de los distintos tipos de consultas que hemos aprendido hasta ahora.

#### Ejercicio 2.09 – Consultas a través de una relación de muchos a muchos mediante búsqueda de campos

Sabemos que `Book` y `Contributor` tienen una relación de muchos a muchos. En este ejercicio, sin crear un objeto, realizaremos una consulta para recuperar todos los colaboradores que contribuyeron a escribir el libro titulado *The Talisman*:

1. Primero, importa la clase `Contributor`:
   ```python
   >>> from reviews.models import Contributor
   ```

2. Ahora, agrega el siguiente código para consultar el conjunto de colaboradores en *The Talisman*:
   ```python
   >>> Contributor.objects.filter(
   ...     book__title='The Talisman')
   ```
   Deberíamos ver lo siguiente:
   ```text
   <QuerySet [<Contributor: Stephen>, <Contributor: Peter>]>
   ```

A partir del resultado anterior, podemos ver que Stephen y Peter son los colaboradores que contribuyeron a escribir el libro *The Talisman*. La consulta utiliza el modelo `book` (escrito en minúsculas) y realiza una búsqueda de campo para el campo `title` utilizando el doble guion bajo, como se muestra en el comando.

En este ejercicio, aprendimos cómo realizar consultas a través de relaciones de muchos a muchos mediante una búsqueda de campos. Ahora veamos cómo usar otro método para llevar a cabo la misma tarea.

#### Ejercicio 2.10 – Una consulta de muchos a muchos utilizando objetos

En este ejercicio, utilizando un objeto `Book`, busca todos los colaboradores que contribuyeron a escribir el libro con el título *The Talisman*. Los siguientes pasos nos ayudarán a hacerlo:

1. Importa el modelo `Book`:
   ```python
   >>> from reviews.models import Book
   ```

2. Recupera un objeto de libro con el título *The Talisman*, agregando la siguiente línea de código:
   ```python
   >>> book = Book.objects.get(title='The Talisman')
   ```

3. Luego recupera todos los colaboradores que trabajaron en el libro *The Talisman* utilizando el objeto de libro. Agrega el siguiente código para hacerlo:
   ```python
   >>> book.contributors.all()
   <QuerySet [<Contributor: Stephen>, <Contributor: Peter>]>
   ```

Nuevamente, podemos ver que Stephen y Peter son los colaboradores que trabajaron en el libro *The Talisman*. Dado que el libro tiene una relación de muchos a muchos con los colaboradores, hemos utilizado el método `contributors.all()` para obtener un conjunto de consultas de todos los colaboradores que trabajaron en el libro. Ahora, intentemos usar el método `set` para realizar una tarea similar.

#### Ejercicio 2.11 – Una consulta de muchos a muchos mediante el método set()

En este ejercicio, utilizaremos un objeto colaborador para obtener todos los libros escritos por el colaborador llamado Rowel:

1. Importa el modelo `Contributor`:
   ```python
   >>> from reviews.models import Contributor
   ```

2. Obtén un objeto colaborador cuyo `first_names` sea `'Rowel'` mediante el método `get()`:
   ```python
   >>> contributor = Contributor.objects.get(
   ...     first_names='Rowel')
   ```

3. Utilizando el objeto colaborador y el método `book_set()`, obtén todos los libros escritos por el colaborador:
   ```python
   >>> contributor.book_set.all()
   <QuerySet [<Book: Advanced Deep Learning with Keras>]>
   ```

Dado que `Book` y `Contributor` tienen una relación de muchos a muchos, podemos usar el método `set()` para consultar un conjunto de objetos asociados con el modelo. En este caso, `contributor.book_set.all()` devolvió todos los libros escritos por el colaborador.

#### Ejercicio 2.12 – Uso del método update()

En este ejercicio, usaremos el método `update()` para actualizar un registro existente:

1. Cambia `first_names` para un colaborador que tenga el apellido Tyrrell:
   ```python
   >>> from reviews.models import Contributor
   >>> Contributor.objects.filter(
   ...     last_names='Tyrrell').update(
   ...     first_names='Mike')
   1
   ```
   El valor de retorno muestra la cantidad de registros que se han actualizado. En este caso, se ha actualizado un registro.

2. Obtén el colaborador que se acaba de modificar mediante el método `get()` y verifica que el primer nombre se haya cambiado a Mike:
   ```python
   >>> Contributor.objects.get(
   ...     last_names='Tyrrell').first_names
   'Mike'
   ```

Si la operación de filtrado tiene más de un registro, el método `update()` actualizará el campo especificado en todos los registros devueltos por el filtro.

En este ejercicio, aprendimos cómo usar el método `update()` para actualizar un registro en la base de datos. Ahora, finalmente, intentemos eliminar un registro de la base de datos mediante el método `delete()`.

#### Ejercicio 2.13 – Uso del método delete()

Un registro existente en la base de datos se puede eliminar mediante el método `delete()`. En este ejercicio, eliminaremos un registro de la tabla de colaboradores que tiene un valor `last_name` de Wharton:

1. Obtén el objeto mediante el método `get` y usa el método `delete`, como se muestra aquí:
   ```python
   >>> from reviews.models import Contributor
   >>> Contributor.objects.get(
   ...     last_names='Wharton').delete()
   (1, {'reviews.Contributor': 1})
   ```
   Observa que llamamos al método `delete()` sin asignar el objeto colaborador a una variable. Dado que el método `get()` devuelve un solo objeto, podemos acceder al método del objeto sin crear realmente una variable para él.

2. Verifica que el objeto colaborador con `last_name` como `'Wharton'` haya sido eliminado:
   ```python
   >>> Contributor.objects.get(last_names='Wharton')
   Traceback (most recent call last):
     File "<console>", line 1, in <module>
     File "/../site-packages/django/db/models/manager.py", line 82, in manager_method
       return getattr(self.get_queryset(), name)(*args, **kwargs)
     File "/../site-packages/django/db/models/query.py", line 417, in get
       self.model._meta.object_name
   reviews.models.Contributor.DoesNotExist: Contributor matching query does not exist.
   ```

Como puedes ver, al ejecutar la consulta, obtuvimos un error de objeto no existe (*DoesNotExist*). Esto es lo esperado ya que el registro ha sido eliminado. En este ejercicio, aprendimos cómo usar el método `delete` para eliminar un registro de la base de datos.

---

### Sección: Operaciones de creación y actualización masivas (Bulk create and bulk update)

Cuando tenemos un conjunto grande de registros que deben crearse o actualizarse, podemos realizar operaciones de creación y actualización masivas utilizando los métodos `bulk_create()` y `bulk_update()`, respectivamente. Cuando tenemos una gran cantidad de registros para crear o actualizar, el uso de métodos como estos puede ser eficiente al realizar operaciones de base de datos.

Por lo general, así es como se llama a un método `bulk_create` proporcionando una lista de objetos:

```python
Person.objects.bulk_create([
    Person(name='Robert', address='5 Falcon Street, Byron Bay, NSW 2481'),
    Person(name='Mark', address="31 West Street, Orange, NSW 2800"),
])
```

Del mismo modo, la actualización masiva también se realiza en una lista de objetos similares. Los atributos del objeto se pueden cambiar como se muestra en el siguiente ejemplo y actualizarse en la base de datos en un solo comando mediante el método `bulk_update()`, como se muestra en el siguiente fragmento:

```python
persons[0].address = "2 Church Street, Byron Bay, NSW 2481"
persons[1].address = "35 Steel Street, Orange, NSW 2800"
Person.objects.bulk_update(persons, ["address"])
```

Veremos más ejemplos de ambos en los siguientes ejercicios.

#### Ejercicio 2.14 – Creación de múltiples registros mediante bulk_create

En este ejercicio, utilizaremos el método `bulk_create()` para crear múltiples entradas en la tabla `Publisher` de una sola vez. Para hacer eso, sigue estos pasos:

1. Importa el modelo `Publisher` si aún no lo has importado en tu consola de Django:
   ```python
   >>> from reviews.models import Publisher
   ```

2. Crea múltiples registros en la tabla `Publisher` pasando una lista de los objetos `Publisher` al método `bulk_create()`:
   ```python
   >>> Publisher.objects.bulk_create([
   ...     Publisher(name="New Town Publisher", website="www.newtown.example.com", email='newtown@example.com'),
   ...     Publisher(name="Byron Bay Press", website="www.byronbay.example.org", email='byronbay@example.org'),
   ...     Publisher(name="Katoomba Publisher", website="www.katoomba.example.net", email='katoomba@example.net)])
   [<Publisher: New Town Publisher>, <Publisher: Byron Bay Press>, <Publisher: Katoomba Publisher>]
   ```

El método devuelve una lista de objetos creados como entradas en la base de datos. Si es necesario crear una gran cantidad de entradas, esta podría ser una de las formas más eficientes de llevar a cabo la tarea.

A continuación, intentaremos actualizar registros masivamente de manera eficiente utilizando el método `bulk_update`.

#### Ejercicio 2.15 – Actualización de múltiples registros mediante bulk_update

Similar al ejercicio anterior, podemos actualizar múltiples registros de una sola vez mediante el método `bulk_update`:

1. Primero, importa el modelo `Publisher` si aún no lo has importado:
   ```python
   >>> from reviews.models import Publisher
   ```

2. En el siguiente paso, seleccionaremos un par de objetos `Publisher` como una lista. Suponiendo que ambas editoriales se fusionaron en una sola empresa y necesitan compartir el mismo sitio web; ahora actualicemos ambos objetos en un conjunto de consultas:
   ```python
   >>> publishers = [
   ...     Publisher.objects.get(name='New Town Publisher'),
   ...     Publisher.objects.get(name='Byron Bay Press')]
   >>> publishers[0].website = "nswpublisher.example.com"
   >>> publishers[1].website = "nswpublisher.example.com"
   >>> Publisher.objects.bulk_update(publishers, ["website"])
   2
   ```

Al ejecutar el método `bulk_update`, devuelve la cantidad de objetos actualizados; en este caso, son dos objetos. Nuevamente, esta es una forma eficiente de actualizar más de un objeto de una sola vez.

---

### Sección: Realización de búsquedas complejas mediante objetos Q

Los **objetos Q** se utilizan para realizar consultas complejas, especialmente cuando una consulta involucra operaciones `AND` u `OR` en una cláusula `WHERE`. Por ejemplo, si necesitamos hacer una consulta similar a la consulta SQL que se muestra aquí:

```sql
SELECT * FROM Person WHERE name LIKE 'Rob%' OR name LIKE 'Bob%';
```

La declaración SQL anterior busca cualquier persona cuyo nombre comience con Rob o Bob.
`LIKE 'Rob%'`: aquí, la palabra clave `LIKE` busca coincidencias de patrones en una cadena para comprobar si la cadena comienza con el valor especificado Rob.

Los objetos Q utilizan los operadores `&` y `|` para las operaciones `AND` y `OR` al combinar las cláusulas `WHERE`. La consulta anterior se puede escribir de la siguiente manera utilizando objetos Q:

```python
Person.objects.get(Q(name__startswith='Rob') | Q(name__startswith='Bob'))
```

#### Ejercicio 2.16 – Realización de una consulta compleja mediante un objeto Q

A continuación, utilizando el concepto aprendido anteriormente, los objetos Q nos permiten realizar una consulta para dos de las editoriales:

1. Importa `Publisher` si aún no lo has hecho antes, y también importa `Q` desde `django.db.models`, como se muestra en el siguiente comando:
   ```python
   >>> from django.db.models import Q
   >>> from reviews.models import Publisher
   ```

2. Ejecuta la consulta del objeto Q como se muestra a continuación. Aquí, verificamos si alguno de los objetos `Publisher` tiene un `name` que comience con New o Idea:
   ```python
   >>> Publisher.objects.filter(Q(name__startswith="New") | Q(name__startswith="Idea"))
   <QuerySet [<Publisher: New Town Publisher>]>
   ```
   Como no hay ninguna editorial cuyo nombre comience con Idea y había un objeto cuyo nombre comenzaba con New, se devolvió un objeto.

3. Este es un ejemplo de consulta de objeto Q con el operador `AND`. En esta consulta, estamos consultando un objeto `Publisher` cuyo nombre comienza con New y también cuyo nombre termina con Publisher:
   ```python
   >>> Publisher.objects.filter(Q(name__startswith="New") & Q(name__endswith="Publisher"))
   <QuerySet [<Publisher: New Town Publisher>]>
   ```
   En este caso, solo se devolvió un objeto, que es New Town Publisher, que cumple las condiciones establecidas en la consulta. De esta manera, se pueden combinar múltiples cláusulas `where` mediante los operadores `AND` u `OR` para obtener los resultados deseados.

#### Ejercicio 2.17 – Verificación de si un QuerySet contiene un objeto dado

En este ejercicio, utilizaremos un método `contains` para verificar si la consulta contiene un objeto especificado:

1. Importa el modelo `Publisher` si aún no lo has importado:
   ```python
   >>> from reviews.models import Publisher
   ```

2. Ejecuta una consulta, como se muestra a continuación. En este ejemplo, estamos ejecutando una consulta de objeto Q y el resultado tiene dos editoriales:
   ```python
   >>> publishers = Publisher.objects.filter(
   ...     Q(name__startswith="New") | Q(name__endswith="Publisher"))
   >>> publishers
   <QuerySet [<Publisher: New Town Publisher>, <Publisher: Katoomba Publisher>]>
   ```

3. Obtén un nuevo objeto de editorial y verifica si es parte del conjunto de consultas anterior:
   ```python
   >>> new_town_publisher = Publisher.objects.get(
   ...     name='New Town Publisher')
   >>> new_town_publisher
   <Publisher: New Town Publisher>
   >>> publishers.contains(new_town_publisher)
   True
   ```
   Si el objeto está presente, devuelve un valor booleano como `True` o `False`.

4. He aquí un ejemplo de cuando un determinado conjunto de consultas no tiene el objeto especificado, en cuyo caso el resultado devuelto es `False`:
   ```python
   >>> publishers.contains(Publisher.objects.get(
   ...     name='Byron Bay Press'))
   False
   ```

En este ejercicio, aprendimos cómo se puede utilizar un método simple, `contains()`, para verificar si un objeto es parte de un conjunto de consultas (*queryset*).

En general, en toda esta sección, aprendimos y usamos la consola interactiva de línea de comandos de Django. Utilizando los modelos creados en la sección anterior para la aplicación de reseñas de libros, utilizamos la consola interactiva de Django para realizar operaciones CRUD. También exploramos varias formas de filtrar y consultar (Leer) desde la base de datos. Todas estas operaciones de base de datos resultarán útiles al desarrollar cualquier aplicación de Django. Utilizando los conceptos aprendidos hasta ahora en este capítulo, en la siguiente sección, crearemos modelos para una aplicación de muestra y realizaremos algunas de las operaciones CRUD.

---

### Sección: Actividad 2.01 – Creación de modelos para una aplicación de gestión de proyectos

Imagina que estamos desarrollando una aplicación de gestión de proyectos llamada **Juggler**. Juggler es una aplicación que puede rastrear múltiples proyectos, y cada proyecto puede tener múltiples tareas asociadas. Los siguientes pasos nos ayudarán a completar esta actividad:

1. Utilizando las técnicas que hemos aprendido hasta ahora, crea un proyecto de Django llamado `juggler`.
2. Crea una aplicación de Django llamada `projectp`.
3. Agrega la aplicación `projectp` en el archivo `juggler/settings.py`.
4. Crea dos clases de modelos relacionadas llamadas `Project` y `Task` en `projectp/models.py`.
5. Crea scripts de migración y migra las definiciones de los modelos a la base de datos.
6. Abre la consola de Django e importa los modelos.
7. Llena la base de datos con un ejemplo y escribe una consulta que muestre la lista de tareas asociadas con un proyecto determinado.

La solución completa para esta actividad se puede encontrar en la carpeta `Chapter02` en el repositorio de GitHub de este libro.

---

### Sección: Población de la base de datos del proyecto Bookr

Aunque sabemos cómo crear registros de bases de datos individuales, será bueno poblar la base de datos de Bookr con un conjunto de datos que ejemplifique publicaciones del mundo real. Hemos incluido un archivo JSON que se puede utilizar para cargar datos de publicaciones y reseñas en la base de datos mediante el subcomando `loaddata` del script `manage.py`.

Encontrarás el archivo `reviews.json` en la carpeta `Chapter02` en el repositorio de GitHub de este libro.

Sigue los siguientes pasos para poblar la base de datos del proyecto:

1. Copia el archivo `reviews.json` en la carpeta de la aplicación `reviews`. Este archivo se puede encontrar en el repositorio de GitHub de este libro.

2. Ahora, recreemos una base de datos nueva. Elimina tu archivo de base de datos SQL presente en la carpeta del proyecto:
   ```bash
   rm db.sqlite3
   ```

3. Para crear una base de datos nueva con el esquema correcto, ejecuta el comando de migración de Django:
   ```bash
   python manage.py migrate
   ```
   Ahora podemos ver el archivo `db.sqlite3` recién creado bajo la carpeta `bookr`.

4. Ejecuta el comando de gestión `loaddata` para poblar la base de datos:
   ```bash
   python manage.py loaddata reviews/reviews.json
   ```

5. Mediante DB Browser for SQLite, verifica que todas las tablas creadas por el proyecto `bookr` estén pobladas.

Con esto, hemos poblado con éxito la base de datos de nuestra aplicación, lo que resultará muy útil para trabajar en los próximos capítulos de este libro.

---

### Sección: Resumen

En este capítulo, aprendimos cómo Django proporciona una valiosa capa de abstracción llamada ORM que nos ayuda a interactuar sin problemas con bases de datos relacionales utilizando código Python simple sin tener que componer comandos SQL. Como parte del ORM, aprendimos sobre los modelos de Django, las migraciones y cómo ayudan a propagar los cambios en los modelos de Django a una base de datos.

Afianzamos nuestro conocimiento de las bases de datos al aprender sobre las relaciones de bases de datos y sus tipos clave en bases de datos relacionales. También trabajamos con la consola de Django, utilizando código Python para realizar las mismas consultas CRUD que realizamos anteriormente utilizando SQL. Más adelante, aprendimos cómo recuperar nuestros datos de una manera más refinada utilizando coincidencia de patrones y búsquedas de campo. A medida que aprendimos sobre estos conceptos, también hicimos un progreso considerable en nuestra aplicación Bookr. Creamos modelos para nuestra aplicación `reviews` y adquirimos todas las habilidades necesarias para interactuar con los datos almacenados dentro de la base de datos de la aplicación. En el próximo capítulo, aprenderemos cómo crear vistas de Django, enrutamiento de URLs y plantillas.
