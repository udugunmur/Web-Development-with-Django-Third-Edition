# Parte 3: Características avanzadas de Django

## Capítulo 14: Pruebas de tus aplicaciones Django

En los capítulos anteriores, nos enfocamos en construir nuestra aplicación web en Django escribiendo diferentes componentes como modelos de base de datos, vistas y plantillas. Hicimos todo eso para proporcionar a nuestros usuarios una aplicación interactiva donde puedan crear un perfil y escribir reseñas para los libros que han leído.

Además de crear y ejecutar la aplicación, existe otro aspecto importante: asegurarse de que el código de la aplicación funcione de la manera en que esperamos que funcione. Esto se garantiza mediante una técnica llamada pruebas (*testing*). En las pruebas, ejecutamos las diferentes partes de la aplicación web y verificamos si la salida del componente ejecutado coincide con la salida que esperábamos. Si la salida coincide, decimos que el componente se probó con éxito, mientras que si la salida no coincide, decimos que el componente falló al funcionar según lo previsto.

En este capítulo, a medida que avanzamos por las diferentes secciones, aprenderemos por qué son importantes las pruebas, las diferentes formas de probar una aplicación web y cómo podemos construir una sólida estrategia de pruebas que nos ayude a garantizar que la aplicación web que creamos sea robusta.

En este capítulo, cubriremos los siguientes temas:
- Importancia de las pruebas
- Pruebas automatizadas
- Pruebas en Django
- Pruebas de modelos de Django
- Pruebas de vistas de Django
- RequestFactory de Django
- Clases de casos de prueba en Django

Comencemos nuestro viaje aprendiendo primero sobre la importancia de las pruebas.

---

### Sección: Importancia de las pruebas

Asegurarse de que una aplicación funcione de la manera en que fue diseñada para funcionar es un aspecto importante de los esfuerzos de desarrollo porque, de lo contrario, nuestros usuarios podrían seguir encontrando comportamientos extraños que generalmente alejarán su interés de la aplicación.

Los esfuerzos que dedicamos a las pruebas nos ayudan a garantizar que los diferentes tipos de problemas que pretendemos resolver se estén resolviendo correctamente. Imagina un caso en el que un desarrollador está creando una plataforma de programación de eventos en línea. En esta plataforma, los usuarios pueden programar eventos en sus calendarios según su zona horaria local. Ahora bien, ¿qué pasaría si, en esta plataforma, los usuarios pudieran programar eventos como se esperaba, pero debido a un error, los eventos se programaran en una zona horaria incorrecta? Son problemas como este los que tienden a ahuyentar a muchos usuarios.

Es por eso que muchas empresas gastan una gran cantidad de dinero para asegurarse de que las aplicaciones que crean se sometan a pruebas exhaustivas. De esa manera, se aseguran de no lanzar un producto con errores o un producto que esté lejos de satisfacer los requisitos del usuario.

En resumen, las pruebas nos ayudan a lograr los siguientes objetivos:
- Garantizar que los componentes de la aplicación funcionen de acuerdo con las especificaciones.
- Garantizar la interoperabilidad en diferentes plataformas de infraestructura, si una aplicación se puede implementar en un sistema operativo diferente, como Linux o Windows, y más.
- Reducir la probabilidad de introducir un error al refactorizar el código de la aplicación.

Ahora bien, una suposición común que muchas personas hacen acerca de las pruebas es que tienen que probar todos los componentes manualmente a medida que se desarrollan para asegurarse de que cada componente funcione según sus especificaciones, y repetir este ejercicio cada vez que se realiza un cambio o se agrega un nuevo componente a la aplicación. Si bien esto es cierto, no proporciona una imagen completa de las pruebas. Las pruebas como técnica se han vuelto muy poderosas con el tiempo y, como desarrollador, puedes reducir una gran cantidad de esfuerzo de prueba mediante la implementación de casos de prueba automatizados. Entonces, ¿cuáles son estos casos de prueba automatizados? O, en otras palabras, ¿qué son las pruebas automatizadas? Averigüémoslo.

---

### Sección: Pruebas automatizadas

Probar toda la aplicación repetidamente cuando se modifica un solo componente puede resultar una tarea desafiante, y más aún si esa aplicación consta de una base de código grande. El tamaño de la base de código podría deberse a la gran cantidad de funciones o a la complejidad del problema que resuelve.

A medida que desarrollamos aplicaciones, es importante asegurarse de que los cambios que se realizan en estas aplicaciones se puedan probar fácilmente para que podamos verificar si algo se está rompiendo. Ahí es donde el concepto de pruebas automatizadas (*automation testing*) resulta útil. El enfoque de las pruebas automatizadas es escribir pruebas como código para que los componentes individuales de una aplicación se puedan probar de forma aislada, así como su interacción entre sí.

Al considerar esto, se vuelve importante para nosotros definir los diferentes tipos de pruebas de automatización que se pueden realizar para las aplicaciones.

Las pruebas automatizadas se pueden categorizar ampliamente en cinco tipos diferentes:
- **Pruebas unitarias (*Unit testing*)**: En este tipo de pruebas, se prueban las unidades de código individuales aisladas. Por ejemplo, una prueba unitaria puede apuntar a un solo método o a una sola API aislada. Este tipo de prueba se realiza para asegurarse de que las unidades básicas de la aplicación funcionen de acuerdo con su especificación.
- **Pruebas de integración (*Integration testing*)**: En este tipo de prueba, las unidades de código aisladas individuales se fusionan para formar una agrupación lógica. Una vez formada esta agrupación, se realizan pruebas en este grupo lógico para asegurarse de que el grupo funcione de la manera esperada.
- **Pruebas funcionales (*Functional testing*)**: En este tipo de pruebas, se prueba la funcionalidad general de los diferentes componentes de la aplicación. Esto puede incluir diferentes APIs, interfaces de usuario y más.
- **Pruebas de humo (*Smoke testing*)**: En este tipo de prueba, se prueba la estabilidad de la aplicación desplegada para asegurarse de que la aplicación continúe funcionando a medida que los usuarios interactúan con ella, sin causar una falla o caída (*crash*).
- **Pruebas de regresión (*Regression testing*)**: Este tipo de prueba se realiza para asegurarse de que los cambios que se realizan en la aplicación no degraden la funcionalidad previamente construida de la aplicación.

Como podemos ver, las pruebas son un gran dominio que requiere tiempo para dominarse; se han escrito libros enteros sobre este tema. Para asegurarnos de destacar los aspectos importantes de las pruebas, nos centraremos en las pruebas unitarias en este capítulo.

---

### Sección: Pruebas en Django

Django es un framework repleto de funciones que tiene como objetivo hacer que el desarrollo de aplicaciones web sea rápido. Se espera que también proporcione una forma completa de probar la aplicación.

Django proporciona un módulo bien integrado que permite a los desarrolladores de aplicaciones escribir pruebas unitarias para sus aplicaciones. Este módulo se basa en la biblioteca `unittest` de Python, que viene preempaquetada con la mayoría de las distribuciones de Python.

Entonces, comencemos por comprender cómo podemos escribir casos de prueba básicos en Django y cómo aprovechar los módulos proporcionados por el framework para probar el código de nuestra aplicación.

#### Implementación de casos de prueba

Al trabajar en la implementación de mecanismos para probar tu código, lo primero que debes comprender es cómo se puede agrupar lógicamente esta implementación para que los módulos que están estrechamente relacionados entre sí se prueben en una sola unidad lógica.

Esto se puede simplificar implementando un caso de prueba (*test case*). Un caso de prueba no es más que una unidad lógica que agrupa pruebas relacionadas con unidades lógicamente similares, de modo que toda la lógica común para inicializar el entorno para los casos de prueba se pueda combinar en el mismo lugar, evitando así la duplicación de trabajo al implementar el código de prueba de la aplicación.

#### Pruebas unitarias en Django

Ahora que entendemos las pruebas, veamos cómo podemos hacer pruebas unitarias dentro de Django. En el contexto de Django, una prueba unitaria consta de dos partes principales:
- Una clase `TestCase`, que envuelve los diferentes casos de prueba que se agrupan para un módulo determinado.
- Un caso de prueba real que debe ejecutarse para probar el flujo de un componente en particular.

La clase que implementa una prueba unitaria debe heredar de la clase `TestCase` proporcionada por el módulo `test` de Django. Por defecto, Django proporciona un archivo `tests.py` en cada directorio de aplicación, que se puede utilizar para almacenar los casos de prueba para el módulo de la aplicación.

Una vez que se han escrito estas pruebas unitarias, se pueden ejecutar fácilmente ejecutándolas directamente mediante el comando `test` proporcionado en `manage.py`, de la siguiente manera:

```bash
python manage.py test
```

Ahora, veamos cómo las aserciones pueden ayudarnos a construir nuestras pruebas unitarias y verificar el comportamiento esperado.

#### Uso de aserciones

Una parte importante de escribir pruebas es validar si la prueba pasó o falló. Generalmente, para implementar tales decisiones en un entorno de pruebas, utilizamos algo conocido como aserciones (*assertions*).

Las aserciones son un concepto común en las pruebas de software. Toman dos operandos y validan si el valor del operando del lado izquierdo (LHS) coincide con el valor del operando del lado derecho (RHS). Si el valor del LHS coincide con el valor del RHS, una aserción se considera exitosa, mientras que si los valores difieren, se considera que la aserción ha fallado.

Una aserción que se evalúa como `False` hace esencialmente que un caso de prueba se evalúe como un fallo, que luego se informa al usuario.

Las aserciones en Python son bastante fáciles de implementar y utilizan una palabra clave simple llamada `assert`. Por ejemplo, el siguiente fragmento de código muestra una aserción muy simple:

```python
assert 1 == 1
```

La aserción anterior toma una sola expresión, que se evalúa como `True`. Si esta aserción fuera parte de un caso de prueba, la prueba habría tenido éxito.

Ahora, veamos cómo podemos implementar casos de prueba utilizando la biblioteca `unittest` de Python. Hacerlo es bastante fácil y se puede lograr con algunos pasos sencillos de seguir:

1. Importa el módulo `unittest`, que nos permite construir los casos de prueba:
   ```python
   import unittest
   ```
2. Una vez que se ha importado el módulo, podemos crear una clase cuyo nombre comience con `Test` y que herede de la clase `TestCase` proporcionada por el módulo `unittest`:
   ```python
   class TestMyModule(unittest.TestCase):
       def test_method_a(self):
           self.assertTrue(<expression>)
   ```

Solo si la clase `TestMyModule` hereda de la clase `TestCase`, Django podrá ejecutarla automáticamente con una integración total con el framework. Una vez que se ha definido la clase, podemos implementar un nuevo método dentro de la clase llamado `test_method_a()`, que valida una aserción. Y eso es todo.

Una parte importante a tener en cuenta aquí es el esquema de nomenclatura para los casos de prueba y las funciones de prueba. Los casos de prueba que se implementen deben tener el prefijo `Test` para que los módulos de ejecución de pruebas puedan detectarlos como casos de prueba válidos y ejecutarlos. La misma regla se aplica a la denominación de los métodos de prueba (deben comenzar con `test_`).

Una vez que se ha escrito el caso de prueba, se puede ejecutar simplemente ejecutando el siguiente comando:

```bash
python manage.py test
```

Con nuestra comprensión básica de la implementación de casos de prueba clarificada, escribamos una prueba unitaria muy simple para tener una idea de cómo se comporta el framework de pruebas unitarias dentro de Django.

#### Ejercicio 14.01 – Escritura de una prueba unitaria simple

En este ejercicio, escribiremos una prueba unitaria simple para comprender cómo funciona el framework de pruebas unitarias de Django y utilizaremos este conocimiento para implementar nuestro primer caso de prueba que valida una expresión simple:

1. Para comenzar, abre el archivo `tests.py` debajo de la aplicación `reviews` del proyecto `bookr`. Por defecto, este archivo contendrá solo una línea que importa la clase `TestCase` de Django desde el módulo `test`:
   `reviews/tests.py`:
   ```python
   from django.test import TestCase
   ```
2. Agrega las siguientes líneas de código al archivo `tests.py` que acabas de abrir:
   ```python
   class TestSimpleComponent(TestCase):
       def test_basic_sum(self):
           assert 1+1 == 2
   ```
   Aquí, creamos una nueva clase llamada `TestSimpleComponent`, que hereda de la clase `TestCase` proporcionada por el módulo `test` de Django. La declaración `assert` comparará la expresión del LHS (`1+1`) con la del RHS (`2`).
3. Una vez que hayas escrito el caso de prueba, navega de regreso a la carpeta del proyecto y ejecuta el siguiente comando:
   ```bash
   python manage.py test
   ```
   Se debería generar la siguiente salida:
   ```text
   % ./manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   .
   ----------------------------------------------------------------------
   Ran 1 test in 0.001s

   OK
   Destroying test database for alias 'default'...
   ```
   La salida anterior significa que el ejecutor de pruebas de Django ejecutó 1 caso de prueba, que pasó la evaluación con éxito.
4. Con el caso de prueba confirmado como funcional y aprobado, ahora intentaremos agregar otra aserción al final del método `test_basic_sum()`, como se muestra en el siguiente fragmento de código:
   ```python
   assert 1+1 == 3
   ```
5. Con la declaración `assert` agregada en el Paso 4, ejecuta los casos de prueba ejecutando el siguiente comando desde la carpeta del proyecto:
   ```bash
   python manage.py test
   ```
   En este punto, notarás que Django informa que la ejecución de los casos de prueba ha fallado.

Con esto, comprendes cómo se pueden escribir casos de prueba en Django y cómo se pueden usar las aserciones para validar si la salida generada a partir de las llamadas a métodos bajo prueba es correcta o no.

Ahora que tenemos conocimientos básicos de aserciones, profundicemos en los diferentes tipos de aserciones presentes y su uso.

#### Tipos de aserciones

En el Ejercicio 14.01, encontramos brevemente aserciones cuando vimos la siguiente declaración de aserción:

```python
assert 1+1 == 2
```

Estas declaraciones de aserción son simples y usan la palabra clave `assert` de Python. Son posibles algunos tipos diferentes de aserciones que se pueden probar dentro de una prueba unitaria mientras se usa la biblioteca `unittest`. Veamos cuáles son:
- `assertIsNone`: Esta aserción se utiliza para comprobar si una expresión se evalúa como `None` o no. Por ejemplo, este tipo de aserción se puede utilizar en los casos en que una consulta a una base de datos devuelve `None` porque no se encontraron registros para los criterios de filtrado especificados.
- `assertIsInstance`: Esta aserción se utiliza para validar si un objeto proporcionado se evalúa como una instancia del tipo proporcionado. Por ejemplo, esta aserción se puede utilizar para validar si el valor devuelto por un método es realmente de un tipo específico, como `List`, `Dict`, `Tuple` y otros.
- `assertEqual`: Esta es una función muy básica que toma dos argumentos y verifica si los argumentos proporcionados tienen el mismo valor o no. Esto puede resultar útil cuando planeas comparar los valores de estructuras de datos que no garantizan el orden.
- `assertRaises`: Este método se utiliza para validar si el nombre del método proporcionado, cuando se llama, genera una excepción especificada o no. Esto es útil cuando escribimos casos de prueba en los que se debe probar una ruta de código que genera una excepción. Como ejemplo, este tipo de aserción puede ser útil cuando queremos asegurarnos de que un método que realiza una consulta a la base de datos genere una excepción si la conexión a la base de datos aún no está establecida.

Este fue solo un pequeño conjunto de aserciones útiles que podemos hacer en nuestros casos de prueba. El módulo `unittest`, sobre el cual se basa la biblioteca de pruebas de Django, proporciona muchas más aserciones que se pueden probar.

¿Alguna vez te has preguntado qué hacer cuando nuestras pruebas unitarias requieren que se completen previamente algunas listas o que las bases de datos se inicialicen previamente? La siguiente sección cubrirá cómo preparar esta configuración previa y cómo puede ayudarnos a evitar introducir código repetitivo que debe ejecutarse antes de cada caso de prueba.

#### Realización de configuración previa a la prueba y limpieza después de cada ejecución de caso de prueba

A veces, al escribir casos de prueba, es posible que necesitemos realizar algunas tareas repetitivas, como configurar algunas variables que serán necesarias para la prueba. Una vez finalizada la prueba, querremos limpiar todos los cambios que hayamos realizado en las variables de prueba para que cualquier prueba nueva comience con una instancia limpia.

Afortunadamente, la biblioteca `unittest` nos proporciona una forma útil de automatizar nuestros esfuerzos repetitivos de configurar el entorno antes de que se ejecute cada caso de prueba y limpiarlo después de que finalice el caso de prueba. Esto se puede lograr mediante los dos métodos que podemos implementar en `TestCase`.

El primero es el método `setUp()`. Este método se llama antes de la ejecución de cada método de prueba dentro de la clase `TestCase`. Implementa el código necesario para configurar el entorno del caso de prueba antes de que se ejecute la prueba. Este método puede ser un buen lugar para configurar cualquier instancia de base de datos local o variables de prueba que puedan ser necesarias para los casos de prueba.

El método `setUp()` solo es válido para casos de prueba escritos dentro de la clase `TestCase`.

Por ejemplo, el siguiente ejemplo ilustra una definición simple de cómo se usa el método `setUp()` dentro de una clase `TestCase`:

```python
class MyTestCase(unittest.TestCase):
    def setUp(self):
        # Do some initialization work
    def test_method_a(self):
        # code for testing method A
    def test_method_b(self):
        # code for testing method B
```

En el ejemplo anterior, cuando intentamos ejecutar los casos de prueba, el método `setUp()` que definimos aquí se llamará cada vez antes de que se ejecute un método de prueba. En otras palabras, se llamará al método `setUp()` antes de la ejecución de la llamada a `test_method_a()`, y luego se volverá a llamar antes de que se llame a la llamada a `test_method_b()`.

El segundo es el método `tearDown()`. Este método se llama una vez que la función de prueba finaliza la ejecución y limpia las variables y sus valores una vez que finaliza la ejecución del caso de prueba. Este método se ejecuta independientemente de si el caso de prueba se evalúa como `True` o `False`. Un ejemplo del uso del método `tearDown()` se muestra aquí:

```python
class MyTestCase(unittest.TestCase):
    def setUp(self):
        # Do some initialization work
    def test_method_a(self):
        # code for testing method A
    def test_method_b(self):
        # code for testing method B
    def tearDown(self):
        # perform cleanup
```

En el ejemplo anterior, el método `tearDown()` se llamará cada vez que un método de prueba finalice la ejecución; es decir, una vez que `test_method_a()` finalice la ejecución y nuevamente una vez que `test_method_b()` finalice la ejecución.

Ahora que conocemos los diferentes componentes de la escritura de casos de prueba, veamos cómo podemos probar los diferentes aspectos de una aplicación Django utilizando el framework de pruebas proporcionado.

---

### Sección: Pruebas de modelos de Django

Los modelos en Django son representaciones basadas en objetos de cómo se almacenarán los datos dentro de la base de datos de una aplicación. Proporcionan métodos que pueden ayudarnos a validar la entrada de datos proporcionada para un registro determinado, así como a realizar cualquier procesamiento en los datos antes de que se inserten en la base de datos.

Es tan fácil probar modelos en Django como crearlos. Ahora, veamos cómo se pueden probar los modelos de Django utilizando el framework Django Test.

#### Ejercicio 14.02 – Pruebas de modelos de Django

En este ejercicio, crearemos un nuevo modelo de Django y escribiremos casos de prueba para él. El caso de prueba validará si tu modelo puede insertar y recuperar correctamente los datos de la base de datos. Este tipo de casos de prueba que funcionan en modelos de bases de datos pueden resultar útiles en los casos en que un equipo de desarrolladores colabora en un proyecto grande y varios desarrolladores pueden modificar el mismo modelo de base de datos a lo largo del tiempo. La implementación de casos de prueba para los modelos de bases de datos permite a los desarrolladores identificar de forma preventiva cambios potencialmente perjudiciales que puedan introducir inadvertidamente como parte de su trabajo. Comencemos con los pasos:

1. Crea una nueva aplicación que utilizarás para los ejercicios de este capítulo. Para esto, ejecuta el siguiente comando, que configurará una nueva aplicación para tu caso de uso:
   ```bash
   python manage.py startapp bookr_test
   ```
2. Para asegurarte de que la aplicación `bookr_test` se comporte de la misma manera que cualquier otra aplicación en el proyecto Django, agrega esta aplicación a la sección `INSTALLED_APPS` del proyecto `bookr`. Para hacer esto, abre el archivo `settings.py` en tu proyecto `bookr` y agrega el siguiente código a la lista `INSTALLED_APPS`:
   `bookr/settings.py`:
   ```python
   INSTALLED_APPS = [
       ….,
       ….,
       "bookr_test",
   ]
   ```
3. Ahora, con la configuración de la aplicación completa, crea un nuevo modelo de base de datos que utilizarás para fines de prueba. Para este ejercicio, vamos a crear un nuevo modelo llamado `Publisher` que almacenará los detalles sobre la editorial del libro en nuestra base de datos. Para crear el modelo, abre el archivo `models.py` en el directorio `bookr_test` y agrégale el siguiente código:
   `bookr_test/models.py`:
   ```python
   from django.db import models

   class Publisher(models.Model):
       """A company that publishes books."""
       name = models.CharField(
           max_length=50,
           help_text="The name of the Publisher.")
       website = models.URLField(
           help_text="The Publisher's website.")
       email = models.EmailField(
           help_text="The Publisher's email address.")

       def __str__(self):
           return self.name
   ```
   En el fragmento de código anterior, creaste una nueva clase llamada `Publisher` que hereda de la clase `Model` del módulo `models` de Django, definiendo la clase como un modelo de Django que se usará para almacenar datos sobre la editorial:
   ```python
   class Publisher(models.Model)
   ```
   Dentro de este modelo, hemos agregado tres campos, que actuarán como propiedades del modelo:
   - `name`: El nombre de la editorial
   - `website`: El sitio web perteneciente a la editorial
   - `email`: La dirección de correo electrónico de la editorial
   Una vez que hicimos esto, creamos un método de clase llamado `__str__()` que define cómo se verá la representación en cadena de la clase de modelo.
4. Ahora, con el modelo creado, debes migrar este modelo antes de poder ejecutar una prueba en él. Para hacer esto, ejecuta los siguientes comandos:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```
5. Con el modelo ahora configurado, escribe el caso de prueba con el que vas a probar el modelo que creaste en el Paso 3. Para esto, abre el archivo `tests.py` debajo del directorio `bookr_test` y agrégale el siguiente código:
   `bookr_test/tests.py`:
   ```python
   from django.test import TestCase
   from .models import Publisher

   class TestPublisherModel(TestCase):
       """Test the publisher model."""

       def setUp(self):
           self.p = Publisher(
               name='Packt',
               website='www.packt.com',
               email='contact@packt.com'
           )

       def test_create_publisher(self):
           self.assertIsInstance(self.p, Publisher)

       def test_str_representation(self):
           self.assertEqual(str(self.p), "Packt")
   ```
   En el fragmento de código anterior, hay un par de cosas que vale la pena explorar.
   Al principio, después de importar la clase `TestCase` del módulo `test` de Django, importaste el modelo `Publisher` del directorio `bookr_test`, que se utilizará para las pruebas.
   Una vez que se hayan importado las bibliotecas requeridas, debemos crear una nueva clase llamada `TestPublisherModel` que herede de la clase `TestCase` y se use para agrupar las pruebas unitarias relacionadas con el modelo `Publisher`:
   ```python
   class TestPublisherModel(TestCase):
   ```
   Dentro de esta clase, definimos un par de métodos. Primero, definimos un nuevo método llamado `setUp()` y le agregamos el código de creación del objeto de modelo para que el objeto de modelo se cree cada vez que se ejecuta un nuevo método de prueba dentro de este caso de prueba. Este objeto de modelo se almacena como un miembro de la clase para que se pueda acceder a él dentro de otros métodos sin problemas:
   ```python
   def setUp(self):
       self.p = Publisher(
           name='Packt',
           website='www.packt.com',
           email='contact@packt.com'
       )
   ```
   El primer caso de prueba valida si el objeto de modelo para el modelo `Publisher` se creó correctamente o no. Para hacer esto, creamos un nuevo método llamado `test_create_publisher()`, dentro del cual verificamos si el objeto de modelo creado apunta a un objeto del tipo `Publisher`. Si este objeto de modelo no se creó correctamente, tu prueba fallará:
   ```python
   def test_create_publisher(self):
       self.assertIsInstance(self.p, Publisher)
   ```
   Si revisas detenidamente, verás que estamos usando el método `assertIsInstance()` de la biblioteca `unittest` aquí para asegurar si el objeto de modelo pertenece al tipo `Publisher` o no.
   La siguiente prueba valida si la representación en cadena del modelo es la misma que esperábamos que fuera. A partir de la definición del código, la representación en cadena del modelo `Publisher` debería generar el nombre de la editorial. Para probar esto, creamos un nuevo método llamado `test_str_representation()` y verificamos si la representación en cadena generada del modelo coincide con la que estamos esperando:
   ```python
   def test_str_representation(self):
       self.assertEqual(str(self.p), "Packt")
   ```
   Para realizar esta validación, utilizamos el método `assertEqual` de la biblioteca `unittest`, que valida si los dos valores proporcionados son iguales o no.
6. Con los casos de prueba ahora en su lugar, puedes ejecutarlos para verificar qué sucede. Para ejecutar estos casos de prueba, ejecuta el siguiente comando:
   ```bash
   python manage.py test
   ```
   Una vez que el comando finalice la ejecución, verás una salida que se asemeja a la que se muestra aquí:
   ```text
   % python manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   ..
   ----------------------------------------------------------------------
   Ran 2 tests in 0.002s

   OK
   Destroying test database for alias 'default'...
   ```
   Como puedes ver en la salida anterior, los casos de prueba se ejecutan con éxito, validando así que las operaciones, como la creación de un nuevo objeto `Publisher` y su representación en cadena cuando se recupera, se están realizando correctamente.

Con este ejercicio, vimos cómo podemos escribir casos de prueba para nuestros modelos de Django fácilmente y validar su funcionalidad creando objetos, recuperándolos y representándolos.

Además, observa la siguiente línea importante en la salida del Ejercicio 14.02:

```text
"Destroying test database for alias 'default'..."
```

Esto sucede porque cuando hay casos de prueba que requieren que los datos se conserven dentro de una base de datos, en lugar de usar la base de datos de producción, Django crea una nueva base de datos vacía para los casos de prueba, que utiliza para persistir el valor para el caso de prueba.

Ahora que sabemos cómo probar modelos en Django, en la siguiente sección aprenderemos cómo probar vistas en Django.

---

### Sección: Pruebas de vistas de Django

Las vistas en Django controlan la representación de la respuesta HTTP para los usuarios en función de la URL que visitan en una aplicación web. En esta sección, aprenderemos a probar vistas dentro de Django. Imagina que estás trabajando en un sitio web donde se requieren muchos puntos de conexión de interfaz de programación de aplicaciones (API). Una pregunta interesante para hacer es: ¿cómo podrás validar cada nuevo punto de conexión? Si se hace manualmente, primero tendrás que implementar la aplicación cada vez que se agregue un nuevo punto de conexión y luego visitar manualmente el punto de conexión en el navegador para validar si funciona bien o no. Tal enfoque puede funcionar cuando el número de puntos de conexión es pequeño, pero puede volverse extremadamente engorroso si hay cientos de puntos de conexión.

Django proporciona una forma muy completa de probar las vistas de las aplicaciones. Esto sucede mediante el uso de una clase de cliente de prueba proporcionada por el módulo `test` de Django. Esta clase se puede utilizar para visitar URLs asignadas a las vistas y capturar la salida generada al visitar el punto de conexión de la URL. Luego, podemos usar la salida capturada para probar si las URLs están generando una respuesta correcta o no. Este cliente se puede utilizar importando la clase `Client` del módulo `test` de Django y luego inicializándola, como se muestra en el siguiente fragmento:

```python
from django.test import Client
c = Client()
```

El objeto `Client` admite varios métodos que se pueden utilizar para simular las diferentes llamadas HTTP que un usuario puede realizar, a saber, GET, POST, PUT, DELETE y otras. Un ejemplo de cómo hacer tal solicitud se ve así:

```python
response = c.get('/welcome')
```

La respuesta generada por la vista es luego capturada por el cliente y expuesta como un objeto de respuesta, que luego se puede consultar para validar la salida de la vista.

Con este conocimiento, veamos cómo podemos escribir casos de prueba para nuestras vistas de Django.

#### Ejercicio 14.03 – Escritura de pruebas unitarias para vistas de Django

En este ejercicio, utilizaremos el cliente de prueba de Django para escribir un caso de prueba para nuestra vista de Django, que se asignará a una URL específica. Estos casos de prueba te ayudarán a validar si tu función de vista genera la respuesta correcta cuando se visita mediante su URL asignada. Comencemos con los pasos:

1. Para este ejercicio, vas a utilizar la aplicación `bookr_test` que se creó en el Paso 1 del Ejercicio 14.02. Para comenzar, abre el archivo `views.py` en el directorio `bookr_test` y agrégale el siguiente código:
   `bookr_test/views.py`:
   ```python
   from django.http import HttpResponse

   def greeting_view(request):
       """Greet the user."""
       return HttpResponse("Hey there, welcome to Bookr!"
                           "Your one stop place to review books.")
   ```
   Aquí, has creado una vista simple de Django que se usará para saludar al usuario con un mensaje de bienvenida cada vez que visite un punto de conexión asignado a la vista provista.
2. Una vez que se ha creado la vista, debes asignar esta vista a un punto de conexión de URL que luego se pueda visitar en un navegador o en un cliente de prueba. Para hacer esto, agrega un archivo `urls.py` en el directorio `bookr_test` e incluye el siguiente código con la asignación en la lista `urlpatterns`:
   `bookr_test/urls.py`:
   ```python
   from django.urls import path
   from . import views

   urlpatterns = [
       path('test/greeting', views.greeting_view, name='greeting_view'),
   ]
   ```
   En el fragmento de código anterior, asignamos `greeting_view` al punto de conexión `'test/greeting'` para la aplicación configurando la ruta en la lista `urlpatterns`.
3. Una vez configurada esta ruta, debemos asegurarnos de que nuestro proyecto también la identifique. Para esto, debemos agregar esta entrada a la asignación de URL del proyecto `bookr`. Para lograrlo, abre el archivo `urls.py` en el directorio `bookr` y agrega la siguiente línea al final de la lista `urlpatterns`:
   `bookr/urls.py`:
   ```python
   urlpatterns = [
       ….,
       ….,
       path('', include('bookr_test.urls')),
   ]
   ```
4. Una vez configurada la vista, valida que funcione correctamente. Hazlo ejecutando el siguiente comando:
   ```bash
   python manage.py runserver localhost:8080
   ```
   Luego, visita `http://localhost:8080/test/greeting` en tu navegador web. Una vez que se abra la página, deberías ver el siguiente texto que agregaste en el Paso 1 a la vista de saludo que se muestra en el navegador:
   ```text
   Hey there, welcome to Bookr! Your one stop place to review books.
   ```
5. Ahora estamos listos para escribir los casos de prueba para `greeting_view`. Para este ejercicio, vamos a escribir un caso de prueba que verifica si, al visitar el punto de conexión `/test/greeting`, obtienes un resultado exitoso o no. Para implementar este caso de prueba, abre el archivo `tests.py` en el directorio `bookr_test` y agrega el siguiente código al final del archivo:
   `bookr_test/tests.py`:
   ```python
   from django.test import TestCase

   class TestGreetingView(TestCase):
       """Test the greeting view."""

       def test_greeting_view(self):
           response = self.client.get('/test/greeting')
           self.assertEqual(response.status_code, 200)
   ```
   En el fragmento de código anterior, definimos un caso de prueba que ayuda a validar si la vista de saludo funciona bien o no.
   Esto se hace importando primero el cliente de prueba de Django, que te permite probar las vistas asignadas a las URLs haciendo llamadas a ellas y analizando la respuesta generada:
   ```python
   from django.test import TestCase, Client
   ```
   Una vez realizada la importación, debemos crear una nueva clase llamada `TestGreetingView` que agrupará los casos de prueba relacionados con la vista de saludo que creamos en el Paso 2:
   ```python
   class TestGreetingView(TestCase):
   ```
   Dentro de este caso de prueba, definimos el método `test_greeting_view()`. El método `test_greeting_view()` implementa tu caso de prueba. Dentro de este, primero hacemos una llamada HTTP GET a la URL que está asignada a la vista de saludo y almacenamos la respuesta generada por la vista dentro del objeto de respuesta creado:
   ```python
   response = self.client.get('/test/greeting')
   ```
   Una vez que finaliza esta llamada, tendrás su código de respuesta HTTP, contenido y encabezados disponibles dentro de la variable `response`. A continuación, con el siguiente código, hacemos una aserción, validando si el código de estado generado por la llamada coincide con el código de estado para llamadas HTTP exitosas (HTTP 200):
   ```python
   self.assertEqual(response.status_code, 200)
   ```
   Con esto, ya estamos listos para ejecutar las pruebas.
6. Con el caso de prueba escrito, mira qué sucede cuando ejecutas el caso de prueba. Esto se puede hacer ejecutando el siguiente comando:
   ```bash
   python manage.py test
   ```
   Una vez que se ejecuta el comando, puedes esperar ver una salida como la que se muestra en el siguiente fragmento:
   ```text
   % python manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   ...
   ----------------------------------------------------------------------
   Ran 3 tests in 0.006s

   OK
   Destroying test database for alias 'default'...
   ```
   Como puedes ver en la salida, tus casos de prueba se ejecutaron con éxito, validando así que la respuesta generada por el método `greeting_view()` se ajusta a tus expectativas.

En este ejercicio, aprendimos cómo podemos implementar un caso de prueba para una función de vista de Django y utilizar el `TestClient` de Django para afirmar que la salida generada por la función de vista coincide con la que el desarrollador debería ver.

En la siguiente sección, aprenderemos a probar vistas que tienen habilitada la autenticación.

#### Pruebas de vistas con autenticación

En el ejemplo anterior, vimos cómo podemos probar vistas dentro de Django. Una parte importante que debe destacarse sobre esta vista es que cualquier persona podía acceder a la vista que creamos y no está protegida por ninguna autenticación ni verificación de inicio de sesión. Ahora, imagina un caso en el que solo se debe poder acceder a una vista si el usuario ha iniciado sesión. Por ejemplo, considera implementar una función de vista que represente una página de perfil de un usuario registrado de nuestra aplicación web. Para asegurarte de que solo los usuarios que hayan iniciado sesión puedan ver la página de perfil de su cuenta, es posible que desees restringir la vista solo a los usuarios que hayan iniciado sesión.

Con esto, ahora tenemos una pregunta importante: ¿Cómo podemos probar vistas que requieren autenticación?

Afortunadamente, el cliente de prueba de Django proporciona esta funcionalidad, a través de la cual podemos iniciar sesión en nuestras vistas y luego ejecutar pruebas sobre ellas. Este resultado se puede lograr mediante el método `login()` del cliente de prueba de Django. Cuando se llama a este método, el cliente de prueba de Django realiza una operación de autenticación contra el servicio. Si la autenticación tiene éxito, almacena la cookie de inicio de sesión internamente, que luego puede usar para ejecuciones de prueba posteriores. El siguiente fragmento de código muestra cómo puedes configurar el cliente de prueba de Django para simular un usuario que ha iniciado sesión:

```python
login = self.client.login(username='testuser', password='testpassword')
```

El método `login` requiere un nombre de usuario y una contraseña para el usuario de prueba con el que vamos a realizar la prueba, como se mostrará en el siguiente ejercicio. Entonces, echemos un vistazo a cómo podemos probar un flujo que requiere autenticación de usuario.

#### Ejercicio 14.04 – Escritura de casos de prueba para validar usuarios autenticados

En este ejercicio, escribiremos casos de prueba para vistas que requieren que el usuario esté autenticado. Como parte de esto, validaremos la salida generada por el método de vista cuando un usuario que no ha iniciado sesión intenta visitar la página y cuando un usuario que ha iniciado sesión intenta visitar la página asignada a la función de vista. Comencemos con los pasos:

1. Para este ejercicio, vas a utilizar la aplicación `bookr_test` que creaste en el Paso 1 del Ejercicio 14.02. Para comenzar, abre el archivo `views.py` bajo la aplicación `bookr_test` y agrégale el siguiente código:
   `bookr_test/views.py`:
   ```python
   from django.http import HttpResponse
   from django.contrib.auth.decorators import \
       login_required
   ```
2. Una vez agregado el fragmento de código anterior, crea una nueva función llamada `greeting_view_user()` al final del archivo, como se muestra en el siguiente fragmento de código:
   ```python
   @login_required
   def greeting_view_user(request):
       """Greeting view for the user."""
       user = request.user
       return HttpResponse(f"Welcome to Bookr! {user}")
   ```
   Aquí, hemos creado una vista simple de Django que se usará para saludar al usuario que ha iniciado sesión con un mensaje de bienvenida cada vez que visite un punto de conexión asignado a la vista provista.
3. Una vez que se ha creado esta vista, debemos asignar esta vista a un punto de conexión de URL que luego se pueda visitar en un navegador o en un cliente de prueba. Para hacer esto, abre el archivo `urls.py` en el directorio `bookr_test` y agrégale el siguiente código:
   `bookr_test/urls.py`:
   ```python
   from django.urls import path
   from . import views

   urlpatterns = [
       …
       path('test/greet_user', views.greeting_view_user, name='greeting_view_user'),
   ]
   ```
   En el fragmento de código anterior, asignamos `greeting_view_user` al punto de conexión `'test/greet_user'` para la aplicación configurando la ruta en la lista `urlpatterns`:
   ```python
   path('test/greet_user', views.greeting_view_user, name='greeting_view_user')
   ```
   Si seguiste los ejercicios anteriores, esta URL ya debería estar configurada para su detección en el proyecto y no se requieren pasos adicionales para configurar la asignación de URL.
4. Una vez configurada la vista, lo siguiente que debemos hacer es validar si funciona correctamente. Para hacer esto, ejecuta el siguiente comando:
   ```bash
   python manage.py runserver localhost:8080
   ```
   Luego, visita `http://localhost:8080/test/greet_user` en tu navegador web.
   Si aún no has iniciado sesión, al visitar la URL anterior, serás redirigido a la página de inicio de sesión del proyecto.
5. Ahora, escribe los casos de prueba para `greeting_view_user`, que verifican si, al visitar el punto de conexión `/test/greet_user`, obtienes un resultado exitoso o no. Para implementar este caso de prueba, abre el archivo `tests.py` en el directorio `bookr_test` y agrégale el siguiente código:
   `bookr_test/tests.py`:
   ```python
   from django.contrib.auth.models import User

   class TestLoggedInGreetingView(TestCase):
       """Test the greeting view for the authenticated users."""

       def setUp(self):
           test_user = User.objects.create_user(
               username='testuser',
               password='test@#628password')
           test_user.save()
           self.client = Client()

       def test_user_greeting_not_authenticated(self):
           response = self.client.get('/test/greet_user/')
           self.assertEqual(response.status_code, 302)

       def test_user_authenticated(self):
           login = self.client.login(username='testuser', password='test@#628password')
           response = self.client.get('/test/greet_user/')
           self.assertEqual(response.status_code, 200)
   ```
   En el fragmento de código anterior, implementamos un caso de prueba que verifica las vistas que tienen habilitada la autenticación antes de que se pueda ver su contenido.
   Aquí, primero, importamos las clases y métodos requeridos que se utilizarán para definir el caso de prueba e inicializar un cliente de prueba:
   ```python
   from django.test import TestCase, Client
   ```
   Lo siguiente que necesitas es el modelo `User` del módulo `auth` de Django:
   ```python
   from django.contrib.auth.models import User
   ```
   Este modelo es necesario porque, para los casos de prueba que requieren autenticación, necesitaremos inicializar un nuevo usuario de prueba. A continuación, creamos una nueva clase llamada `TestLoggedInGreetingView`, que envuelve tus pruebas relacionadas con la vista `greeting_user` (que requiere autenticación). Dentro de esta clase, definiste tres métodos llamados `setUp()`, `test_user_greeting_not_authenticated()` y `test_user_authenticated()`. El método `setUp()` se utiliza para inicializar un usuario de prueba que utilizarás para la autenticación. Este es un paso obligatorio porque un entorno de prueba dentro de Django es un entorno completamente aislado que no utiliza datos de tu aplicación de producción, por lo que todos los modelos y objetos requeridos deben instanciarse por separado dentro del entorno de prueba.
   Luego, creamos el usuario de prueba e iniciamos el cliente de prueba usando el siguiente código:
   ```python
   test_user = User.objects.create_user(
       username='testuser',
       password='test@#628password')
   test_user.save()
   self.client = Client()
   ```
   A continuación, escribimos el caso de prueba para el punto de conexión `greet_user` cuando el usuario no está autenticado. Aquí, deberías esperar que Django redirija al usuario al punto de conexión de inicio de sesión. Esta redirección se puede detectar comprobando el código de estado HTTP de la respuesta, que debe establecerse en HTTP 302, lo que indica una operación de redirección:
   ```python
   def test_user_greeting_not_authenticated(self):
       response = self.client.get('/test/greet_user/')
       self.assertEqual(response.status_code, 302)
   ```
   A continuación, escribimos otro caso de prueba para verificar si el punto de conexión `greet_user` se representa correctamente cuando el usuario está autenticado. Para autenticar al usuario, primero debes llamar al método `login()` del cliente de prueba y realizar la autenticación proporcionando el nombre de usuario y la contraseña del usuario de prueba que creaste en el método `setUp()`, de la siguiente manera:
   ```python
   login = self.client.login(username='testuser', password='test@#628password')
   ```
   Una vez que se haya completado el inicio de sesión, debemos realizar una solicitud HTTP GET al punto de conexión `greet_user` y validar si el punto de conexión genera un resultado correcto o no verificando el código de estado HTTP de la respuesta devuelta:
   ```python
   response = self.client.get('/test/greet_user/')
   self.assertEqual(response.status_code, 200)
   ```
6. Con los casos de prueba escritos, es hora de comprobar cómo se ejecutan. Para esto, ejecuta el siguiente comando:
   ```bash
   python manage.py test
   ```
   Una vez finalizada la ejecución, podemos esperar ver una respuesta que se asemeja a la siguiente:
   ```text
   % python manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   .....
   ----------------------------------------------------------------------
   Ran 5 tests in 0.366s

   OK
   Destroying test database for alias 'default'...
   ```
   Como podemos ver en la salida anterior, nuestros casos de prueba han pasado con éxito, validando que la vista que creamos genera la respuesta deseada de redirigir al usuario si el usuario no está autenticado en el sitio web y permitir que el usuario vea la página si el usuario está autenticado.

En este ejercicio, implementamos un caso de prueba en el que podemos probar qué salida genera una función de vista en función de si el usuario está autenticado o no.

En la siguiente sección, exploraremos cómo crear solicitudes en pruebas unitarias de Django sin el uso de un cliente de prueba proporcionado por Django, sino proporcionando datos de solicitud sin procesar.

---

### Sección: RequestFactory de Django

Hasta ahora, hemos estado usando el cliente de prueba de Django para probar las vistas que hemos creado para nuestra aplicación. La clase de cliente de prueba simula un navegador y utiliza esta simulación para realizar llamadas a las APIs requeridas. Pero, ¿qué pasa si no quisiéramos usar el cliente de prueba y su simulación asociada de ser un navegador, sino que quisiéramos probar las funciones de vista pasando directamente el parámetro de solicitud? ¿Cómo podemos hacer eso?

Para ayudarnos en tales casos, podemos aprovechar la clase `RequestFactory` proporcionada por Django. La clase `RequestFactory` nos ayuda a proporcionar el objeto de solicitud que podemos pasar a nuestras funciones de vista para evaluar si están funcionando. El siguiente objeto para la clase `RequestFactory` se puede crear instanciando la clase de la siguiente manera:

```python
factory = RequestFactory()
```

El objeto `factory` que hemos creado solo admite métodos HTTP como `get()`, `post()`, `put()` y otros para simular una llamada a cualquier punto de conexión de URL. Ahora, aprendamos cómo modificar el caso de prueba que escribimos en el Ejercicio 14.04 para que use `RequestFactory`.

#### Ejercicio 14.05 – Uso de RequestFactory para probar vistas

En este ejercicio, utilizaremos `RequestFactory` para probar funciones de vista en Django:

1. Para este ejercicio, vamos a utilizar la función de vista `greeting_view_user` existente que creamos anteriormente en el Paso 1 del Ejercicio 14.04, que se ve así:
   ```python
   @login_required
   def greeting_view_user(request):
       """Greeting view for the user."""
       user = request.user
       return HttpResponse(f"Welcome to Bookr! {user}")
   ```
2. A continuación, modifica el caso de prueba existente, `TestLoggedInGreetingView`, definido dentro del archivo `tests.py` en el directorio `bookr_test`. Abre el archivo `tests.py` y realiza los siguientes cambios:
   Primero, debemos agregar la siguiente importación para usar la clase `RequestFactory` dentro de los casos de prueba:
   ```python
   from django.test import (TestCase, Client, RequestFactory)
   ```
   Lo siguiente que necesitamos es una importación para la clase `AnonymousUser` del módulo `auth` de Django y el método de vista `greeting_view_user` del módulo `views`. Esto es necesario para probar las funciones de vista con un usuario simulado que no ha iniciado sesión. Esto se puede hacer agregando el siguiente código:
   ```python
   from django.contrib.auth.models import (AnonymousUser, User)
   from .views import greeting_view_user
   ```
3. Una vez agregadas las declaraciones de importación, modifica el método `setUp()` de la clase `TestLoggedInGreetingView` y cambia su contenido para que se asemeje a lo siguiente:
   ```python
   def setUp(self):
       self.test_user = User.objects.create_user(
           username='testuser',
           password='test@#628password')
       self.test_user.save()
       self.factory = RequestFactory()
   ```
   En este método, primero creamos un objeto de usuario y lo almacenamos como un miembro de la clase para que podamos usarlo más adelante en las pruebas. Una vez que se creó el objeto de usuario, instanciamos una nueva instancia de la clase `RequestFactory` para probar nuestra función de vista.
4. Con el método `setUp()` ahora definido, modifica las pruebas existentes para que utilicen la instancia de `RequestFactory`. Para la prueba de una llamada no autenticada a la función de vista, modifica el método `test_user_greeting_not_authenticated` para que tenga los siguientes contenidos:
   ```python
   def test_user_greeting_not_authenticated(self):
       request = self.factory.get('/test/greet_user')
       request.user = AnonymousUser()
       response = greeting_view_user(request)
       self.assertEqual(response.status_code, 302)
   ```
   En este método, primero creamos un objeto de solicitud usando la instancia de `RequestFactory` que definimos en el método `setUp()`. Una vez que hicimos eso, asignamos una instancia de `AnonymousUser()` a la propiedad `request.user`. La asignación de la instancia `AnonymousUser()` a la propiedad hace que la función de vista piense que el usuario que realiza la solicitud no ha iniciado sesión:
   ```python
   request.user = AnonymousUser()
   ```
   Una vez que hicimos esto, hicimos una llamada al método de vista `greeting_view_user()` y le pasamos el objeto de solicitud que creaste. Una vez que la llamada sea exitosa, debemos capturar la salida del método en la variable de respuesta utilizando el siguiente código:
   ```python
   response = greeting_view_user(request)
   ```
   Para el usuario no autenticado, esperamos obtener una respuesta de redirección que se puede probar comprobando el código de estado HTTP de la respuesta, de la siguiente manera:
   ```python
   self.assertEqual(response.status_code, 302)
   ```
5. Ahora, continúa y modifica el otro método, `test_user_authenticated()`, de manera similar usando la instancia de `RequestFactory`, de la siguiente manera:
   ```python
   def test_user_authenticated(self):
       request = self.factory.get('/test/greet_user')
       request.user = self.test_user
       response = greeting_view_user(request)
       self.assertEqual(response.status_code, 200)
   ```
   Como puedes ver, la mayor parte del código se parece al código que escribiste en el método `test_user_greeting_not_authenticated`, con un pequeño cambio: en este método, en lugar de usar `AnonymousUser` para la propiedad `request.user`, estás usando el `test_user` que creaste en el método `setUp()`:
   ```python
   request.user = self.test_user
   ```
6. Con los cambios realizados, es hora de ejecutar las pruebas.
   Para ejecutar las pruebas y validar si `RequestFactory` funciona como se esperaba o no, ejecuta el siguiente comando:
   ```bash
   python manage.py test
   ```
   Una vez que se ejecuta el comando, puedes esperar ver una salida que se asemeja a la que se muestra aquí:
   ```text
   % python manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   ......
   ----------------------------------------------------------------------
   Ran 6 tests in 0.248s

   OK
   Destroying test database for alias 'default'...
   ```
   Como podemos ver en la salida, los casos de prueba escritos por nosotros han pasado con éxito, validando así el comportamiento de la clase `RequestFactory`.

Con este ejercicio, aprendimos cómo podemos escribir casos de prueba para funciones de vista aprovechando `RequestFactory` y pasando el objeto de solicitud directamente a la función de vista, en lugar de simular una visita a la URL mediante el enfoque del cliente de prueba, lo que permite pruebas más directas.

En la siguiente sección, probaremos vistas basadas en clases.

#### Pruebas de vistas basadas en clases

En el ejercicio anterior, vimos cómo podemos probar vistas definidas como métodos. Pero ¿qué pasa con las vistas basadas en clases? ¿Cómo podemos probarlas?

Resulta que es bastante fácil probar vistas basadas en clases. Por ejemplo, si tenemos una vista basada en clases definida con el nombre `ExampleClassView(View)`, para probar la vista, todo lo que necesitamos hacer es usar la siguiente sintaxis:

```python
response = ExampleClassView.as_view()(request)
```

Es tan simple como eso.

Una aplicación Django generalmente consta de varios componentes diferentes que pueden funcionar de forma aislada, como los modelos, y algunos otros componentes que necesitan interactuar con la asignación de URL y otras partes del framework para funcionar. Probar estos diferentes componentes puede requerir algunos pasos que son comunes solo para esos componentes. Por ejemplo, al probar un modelo, primero es posible que queramos crear ciertos objetos de la clase `Model` antes de comenzar a probar, o para las vistas, primero es posible que queramos inicializar un cliente de prueba con las credenciales del usuario.

Resulta que Django también proporciona algunas otras clases basadas en la clase `TestCase` que se pueden usar para escribir casos de prueba de tipos específicos sobre el tipo de componente que se utiliza. Veamos las diferentes clases proporcionadas por Django.

---

### Sección: Clases de casos de prueba en Django

Más allá de la clase base `TestCase` proporcionada por Django, que se puede usar para definir una multitud de casos de prueba para diferentes componentes, Django también proporciona algunas clases especializadas derivadas de la clase `TestCase`.

Estas clases se utilizan para tipos específicos de casos de prueba según las capacidades que proporcionan al desarrollador. Veamos las diferentes clases proporcionadas por Django.

#### La clase SimpleTestCase

Esta clase se deriva de la clase `TestCase` proporcionada por el módulo `test` de Django y debe usarse al escribir casos de prueba simples que prueben las funciones de vista. Por lo general, esta clase no se prefiere cuando tu caso de prueba implica realizar consultas a la base de datos. La clase también proporciona muchas funciones útiles, como las siguientes:
- Capacidad para comprobar las excepciones generadas por una función de vista.
- Capacidad para probar campos de formulario.
- Un cliente de prueba incorporado.
- Capacidad para verificar una redirección mediante una función de vista.
- Puede coincidir con la igualdad de dos salidas HTML, JSON o XML generadas por las funciones de vista.

Ahora que tenemos una idea básica de lo que es una clase `SimpleTestCase`, sigamos adelante e intentemos comprender otro tipo de clase de caso de prueba que ayuda a escribir casos de prueba sobre la interacción con bases de datos.

#### La clase TransactionTestCase

Esta clase se deriva de la clase `SimpleTestCase` y debe usarse al escribir casos de prueba que involucren interacción con la base de datos, como consultas a la base de datos, creaciones de objetos de modelo y más.

La clase proporciona las siguientes características adicionales:
- Capacidad para restablecer la base de datos a un estado predeterminado antes de que se ejecute un caso de prueba.
- Puede omitir pruebas basadas en las características de la base de datos, lo que puede resultar muy útil si la base de datos que se utiliza para las pruebas no admite todas las funciones de la base de datos de producción.

#### La clase LiveServerTestCase

Esta clase es como la clase `TransactionTestCase`, pero con una pequeña diferencia: los casos de prueba escritos en la clase, en lugar de usar el cliente de prueba predeterminado, usan un servidor en vivo creado por Django.

Esta capacidad de ejecutar el servidor en vivo para las pruebas resulta útil cuando se escriben casos de prueba que prueban las páginas web renderizadas y cualquier interacción con ellas, lo cual no es posible mientras se usa el cliente de prueba predeterminado.

Dichos casos de prueba pueden aprovechar herramientas como Selenium, que se pueden utilizar para crear casos de prueba interactivos que modifican el estado de la página renderizada interactuando con ella.

#### Modularización del código de prueba

En los ejercicios anteriores, vimos cómo podemos escribir casos de prueba para diferentes componentes de nuestro proyecto. Pero un aspecto importante a tener en cuenta es que, hasta ahora, hemos escrito los casos de prueba para todos los componentes en un solo archivo.

Este enfoque está bien cuando la aplicación no tiene muchas vistas y modelos, pero esto puede volverse problemático a medida que nuestra aplicación crece porque nuestro único archivo `tests.py` será difícil de mantener.

Para evitar encontrarnos con tales escenarios, debemos intentar modularizar nuestros casos de prueba para que los casos de prueba de los modelos se mantengan separados de los casos de prueba relacionados con las vistas y otras propiedades. Para lograr esta modularización, todo lo que tenemos que hacer es seguir dos sencillos pasos:

1. Crea un nuevo directorio llamado `tests` dentro de nuestro directorio de aplicación ejecutando el siguiente comando:
   ```bash
   mkdir tests
   ```
2. Crea un nuevo archivo vacío llamado `__init__.py` dentro del directorio `tests` ejecutando el siguiente comando:
   ```bash
   touch __init__.py
   ```

Django requiere este archivo `__init__.py` para detectar correctamente el directorio `tests` que creamos como un módulo y no como un directorio normal.

Una vez que hayas completado los pasos anteriores, puedes continuar y crear nuevos archivos de prueba para los diferentes componentes de tu aplicación. Por ejemplo, para escribir casos de prueba para tus modelos, puedes crear un nuevo archivo llamado `test_models.py` dentro del directorio `tests` y agregar cualquier código asociado para las pruebas de tus modelos dentro de este archivo.

Además, no necesitas tomar ningún otro paso adicional para ejecutar tus pruebas. El mismo comando funcionará perfectamente bien para tu base de código de prueba modularizada:

```bash
python manage.py test
```

Con esto, sabemos cómo podemos escribir casos de prueba para nuestros proyectos. Entonces, ¿qué tal si probamos este conocimiento y escribimos casos de prueba para el proyecto Bookr en el que estamos trabajando?

#### Ejecución de pruebas más rápida mediante paralelización

Una aplicación bien construida tiene varios casos de prueba que proporcionan una alta cobertura de código y prueban todas las diferentes rutas de código en la aplicación. Esto también significa que cada vez que se compila o prueba la aplicación, se deben ejecutar todas estas pruebas. Esto ralentiza el proceso de compilación y, a menudo, se convierte en un motivo de frustración para los desarrolladores.

De forma predeterminada, cuando ejecutamos pruebas con Django, ejecuta una prueba a la vez. Sin embargo, esto se puede hacer más rápido. Una forma muy sencilla en que Django nos permite reducir el tiempo de compilación/ejecución es permitiendo a los desarrolladores ejecutar las pruebas en paralelo.

El siguiente comando muestra cómo ejecutar pruebas en Django de forma paralela:

```bash
python manage.py test --parallel
```

Cuando se ejecuta el comando anterior, Django ejecuta todas las pruebas en la aplicación web Django. El parámetro `--parallel` hace que Django ejecute un número N de pruebas en paralelo en lugar de hacerlo de manera serial. La cantidad de pruebas que se ejecutarán en paralelo estará determinada por la cantidad de núcleos disponibles en tu máquina de desarrollo.

También puedes decirle explícitamente a Django cuántas pruebas puede ejecutar en paralelo mencionando el recuento justo después del parámetro `--parallel`, como se muestra aquí:

```bash
python manage.py test --parallel 4
```

Esto le indicará a Django que ejecute solo 4 casos de prueba en paralelo.

Cuando las pruebas se ejecutan en paralelo, cada caso de prueba obtiene su propia base de datos y otras variables de entorno para evitar problemas que surgen debido a la ejecución de código en un entorno concurrente.

Ahora que entendemos cómo realizar pruebas unitarias en nuestras aplicaciones en Django, sigamos adelante e implementemos nuestro conocimiento implementando casos de prueba de Bookr.

#### Actividad 14.01 – Pruebas de modelos y vistas en Bookr

En esta actividad, implementarás casos de prueba para el proyecto Bookr. Implementarás casos de prueba para validar el funcionamiento de los modelos creados dentro de la aplicación `reviews` del proyecto `bookr`, y luego implementarás un caso de prueba simple para validar la vista `index` dentro de la aplicación `reviews`.

Los siguientes pasos te ayudarán a completar esta actividad:

1. Crea un directorio llamado `tests` dentro del directorio de la aplicación `reviews` para que todos nuestros casos de prueba para la aplicación `reviews` puedan modularizarse.
2. Crea un archivo `__init__.py` vacío para que el directorio no se considere un directorio general, sino un directorio de módulo de Python.
3. Crea un nuevo archivo llamado `test_models.py` para implementar el código de prueba de los modelos. Dentro de este archivo, importa los modelos que deseas probar.
4. Dentro del archivo `test_models.py`, crea una nueva clase que herede de la clase `TestCase` del módulo `django.test` e implemente métodos para validar el proceso de creación y lectura de los objetos de modelo.
5. Para probar la función de vista, crea un nuevo archivo llamado `test_views.py` dentro del directorio `tests` que se creó en el Paso 1.
6. Dentro del archivo `test_views.py`, importa la clase `Client` de prueba del módulo `django.test` y la función de vista `index` del archivo `views.py` de la aplicación `reviews`.
7. Dentro del archivo `test_views.py` que creaste en el Paso 5, crea una nueva clase `TestCase` e implementa métodos para validar la vista `index`.
8. Dentro de la clase `TestCase` que creaste en el Paso 7, crea una nueva función llamada `setUp()`, dentro de la cual debes inicializar una instancia de `RequestFactory`, que se usará para crear un objeto de solicitud que se pueda pasar directamente a la función de vista para probarla.
9. Una vez que hayas completado los pasos anteriores, escribe los casos de prueba y ejecútalos ejecutando `python manage.py test` para validar que los casos de prueba pasen.

Al completar esta actividad, todos los casos de prueba deberían pasar con éxito.

---

### Sección: Resumen

En este capítulo, aprendimos a escribir casos de prueba para diferentes componentes de nuestro proyecto de aplicación web con Django. Aprendimos por qué las pruebas juegan un papel crucial en el desarrollo de cualquier aplicación web y los diferentes tipos de técnicas de prueba que se emplean en la industria para garantizar que el código de la aplicación que entregan sea estable y esté libre de errores.

Luego, vimos cómo podemos usar la clase `TestCase` proporcionada por el módulo `test` de Django para implementar pruebas unitarias, que se pueden usar para probar tanto modelos como vistas. También vimos cómo podemos usar el cliente de prueba de Django para probar funciones de vista, que requieren o no que el usuario esté autenticado. También echamos un vistazo a otro enfoque de usar `RequestFactory` para probar vistas basadas en funciones y vistas basadas en clases.

Concluimos este capítulo comprendiendo las clases predefinidas proporcionadas por Django y dónde deben usarse, al tiempo que analizamos cómo podemos modularizar nuestra base de código de prueba para que parezca limpia.

A medida que avanzamos hacia el próximo capítulo, aprenderemos cómo implementar nuestra aplicación Django en un servidor. Esto cubrirá la arquitectura del servidor web, la verificación de que un proyecto esté listo y las opciones disponibles para la implementación.
