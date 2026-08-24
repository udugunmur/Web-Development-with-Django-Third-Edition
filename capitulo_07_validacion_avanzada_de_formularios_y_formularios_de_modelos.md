# Parte 2: Creación de aplicaciones web con Django

## Capítulo 7: Validación Avanzada de Formularios y Formularios de Modelos (ModelForms)

Continuando tu viaje con la aplicación `form_project` que comenzaste en el capítulo anterior, empezarás este capítulo agregando un nuevo formulario a tu aplicación con validación personalizada de múltiples campos y limpieza de formularios. Aprenderás a establecer valores iniciales en tu formulario y a personalizar los widgets (los elementos de entrada HTML que se generan). Luego, te presentaremos la clase `ModelForm`, que permite crear automáticamente un formulario a partir de un modelo. La utilizarás en una vista para guardar automáticamente la instancia del modelo nueva o modificada.

En este capítulo, llevaremos nuestro conocimiento sobre la validación de formularios de Django más allá, introduciendo conceptos que forman los fundamentos de la mayoría de los sitios web de producción.

Por ejemplo, un determinado campo solo podría ser obligatorio si se establece otro campo. Supongamos que queremos agregar una casilla de verificación para permitir que los usuarios se suscriban a nuestro boletín mensual. Tiene un cuadro de texto debajo que les permite ingresar su dirección de correo electrónico. Con una validación básica, podemos verificar lo siguiente:
- Si el usuario ha marcado la casilla de verificación
- Si el usuario ha ingresado su dirección de correo electrónico

Cuando el usuario haga clic en el botón Enviar, podremos validar si se han completado ambos campos. Pero, ¿qué pasa si el usuario no desea suscribirse a nuestro boletín? Si hace clic en el botón Enviar, lo ideal sería que ambos campos estuvieran en blanco. Ahí es donde validar cada campo por separado podría no funcionar.

Otro ejemplo podría ser un caso en el que tenemos dos campos donde cada uno tiene un valor máximo de, digamos, 50. Pero la suma de los valores agregados a cada uno debe ser menor que 75. Comenzaremos este capítulo viendo cómo escribir reglas de validación personalizadas para resolver tales problemas.

Más adelante, a medida que avancemos en este capítulo, veremos cómo establecer valores iniciales en un formulario. Esto puede ser útil al completar automáticamente información que ya es conocida para el usuario. Por ejemplo, podemos colocar automáticamente la información de contacto de un usuario en un formulario si ese usuario ha iniciado sesión.

Terminaremos este capítulo examinando los formularios de modelos (*model forms*), que nos permitirán crear automáticamente un formulario a partir de una clase `Model` de Django. Esto reduce la cantidad de código que se debe escribir para crear una nueva instancia de `Model`.

En este capítulo, cubriremos los siguientes temas:
- Validación y limpieza personalizada de campos
- Adición de marcadores de posición (*placeholders*) y valores iniciales
- Creación o edición de modelos de Django

Al final de este capítulo, sabrás cómo agregar validación adicional de múltiples campos a los formularios de Django, cómo personalizar y configurar widgets de formulario para campos, cómo usar `ModelForms` para crear automáticamente un formulario a partir de un modelo de Django y cómo crear automáticamente instancias de `Model` a partir de `ModelForms`.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter07` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Validación y limpieza personalizada de campos

En el capítulo anterior, vimos cómo un formulario de Django convierte valores de una solicitud HTTP, que son cadenas, en objetos de Python. En un formulario de Django estándar (no personalizado), el tipo de destino depende de la clase de campo. Por ejemplo, el tipo de Python derivado de `IntegerField` es `int`, y los valores de cadena se nos entregan textualmente, tal como los ingresa el usuario. Pero también podemos implementar métodos en nuestra clase `Form` para alterar los valores de salida de nuestros campos de la manera que elijamos. Esto nos permite limpiar o filtrar los datos de entrada del usuario para que se ajusten mejor a lo que esperamos. Podríamos redondear un número entero al múltiplo de 10 más cercano para que se ajuste al tamaño de un lote al solicitar artículos específicos. O podríamos transformar una dirección de correo electrónico a minúsculas para que los datos sean consistentes para la búsqueda.

También podemos implementar algunos validadores personalizados. Veremos un par de formas diferentes de validar campos: escribiendo un validador personalizado y escribiendo un método `clean` personalizado para el campo. Cada método tiene sus pros y sus contras: un validador personalizado se puede aplicar a diferentes campos y formularios, por lo que no tienes que escribir la lógica de validación para cada campo; un método `clean` personalizado debe implementarse en cada formulario que desees limpiar, pero es más potente y permite una validación más compleja, ya que puedes usar otros campos en el formulario o cambiar el valor limpio que devuelve el campo.

#### Validadores personalizados

Un validador es simplemente una función que acepta un valor y genera `django.core.exceptions.ValidationError` si el valor no es válido; la validez está determinada por el código que escribas. El valor es un objeto de Python (es decir, `cleaned_data`, que ya se ha convertido de la cadena de solicitud POST).

He aquí un ejemplo sencillo que valida si un valor está en minúsculas:

```python
from django.core.exceptions import ValidationError

def validate_lowercase(value):
    if value.lower() != value:
        raise ValidationError(f"{value} is not lowercase.")
```

Observa que la función no devuelve nada, ni en caso de éxito ni de error. Simplemente generará `ValidationError` si el valor no es válido.

Ten en cuenta que el comportamiento y manejo de `ValidationError` son diferentes de cómo hemos visto que se comportan otras excepciones en Django. Normalmente, si generas una excepción en tu vista, terminarás con una respuesta HTTP con un código de estado 500 de Django (si no manejas la excepción en tu código).

Al generar `ValidationError` en tu código de validación/limpieza, la clase de formulario de Django capturará el error por ti; luego, el método `is_valid` del formulario devolverá `False`. No tienes que escribir controladores `try/except` alrededor del código que podría generar `ValidationError`.

El validador se puede pasar al argumento `validators` del constructor de un campo en un formulario, dentro de una lista; por ejemplo, a nuestro campo `text_input` de `ExampleForm`:

```python
class ExampleForm(forms.Form):
    text_input = forms.CharField(
        validators=[validate_lowercase])
```

Ahora, si enviamos el formulario y los campos contienen valores en mayúsculas, obtendremos un error, como se muestra en la siguiente figura:

*Figura 7.1 – Validador de texto en minúsculas en acción*

La función de validación se puede utilizar en cualquier cantidad de campos. En nuestro ejemplo, si quisiéramos que muchos campos tuvieran impuestas las minúsculas, `validate_lowercase` se podría pasar a todos ellos. Ahora, veamos cómo podríamos implementar esto de otra manera, con un método de limpieza (*clean*) personalizado.

#### Métodos de limpieza

Se crea un método de limpieza en la clase `Form` y se nombra con el formato `clean_<nombre_del_campo>`. Por ejemplo, el método de limpieza para `text_input` se llamaría `clean_text_input`, el método de limpieza para `books_you_own` se llamaría `clean_books_you_own`, y así sucesivamente.

Los métodos de limpieza no toman argumentos; en su lugar, deben usar el atributo `cleaned_data` en `self` para acceder a los datos del campo. Este diccionario contendrá los datos después de haber sido limpiados de la manera estándar de Django, como vimos en el ejemplo anterior. El método `clean` debe devolver el valor limpio, que reemplazará el valor original en el diccionario `cleaned_data`. Incluso si el método no cambia el valor, se debe devolver un valor. También puedes usar el método de limpieza para generar `ValidationError`, y el error se adjuntará al campo (lo mismo que con un validador).

Reimplementemos el validador de minúsculas como un método `clean`, de esta manera:

```python
class ExampleForm(forms.Form):
    text_input = forms.CharField()
    …
    def clean_text_input(self):
        value = self.cleaned_data["text_input"]
        if value.lower() != value:
            raise ValidationError(
                f"{value} is not lowercase.")
        return value
```

Puedes ver que la lógica es esencialmente la misma, excepto que debemos devolver el valor validado al final. Si enviamos el formulario, obtendremos el mismo resultado que la vez anterior que lo intentamos (Figura 7.1).

Veamos un ejemplo de limpieza más. En lugar de generar una excepción cuando el valor no es válido, simplemente podríamos convertir el valor a minúsculas. Implementaríamos eso con este código:

```python
class ExampleForm(forms.Form):
    text_input = forms.CharField()
    …
    def clean_text_input(self):
        value = self.cleaned_data['text_input']
        return value.lower()
```

Ahora, digamos que ingresamos texto en la entrada en mayúsculas:

*Figura 7.2 – Texto ingresado COMPLETAMENTE EN MAYÚSCULAS*

Si examináramos los datos limpios utilizando nuestra salida de depuración de la vista, veríamos que están en minúsculas:

*Figura 7.3 – Los datos limpios se han transformado a minúsculas*

Estos fueron solo un par de ejemplos simples de cómo validar campos usando tanto validadores como métodos de limpieza. Por supuesto, puedes hacer que cada tipo de validación sea mucho más complejo si lo deseas y transformar los datos de formas más avanzadas usando un método `clean`.

Hasta ahora, solo has aprendido métodos simples para la validación de formularios, donde has tratado cada campo de forma independiente. Un campo es válido (o no) basándose únicamente en la información que contiene y en nada más. ¿Qué sucede si la validez de un campo depende de lo que el usuario ingresó en otro campo? Un ejemplo de esto podría ser que tienes un campo de correo electrónico para recopilar la dirección de correo electrónico de alguien si desea suscribirse a una lista de correo. El campo solo es obligatorio si marca una casilla de verificación que indica que desea suscribirse. Ninguno de estos campos es obligatorio por sí solo: no queremos que sea obligatorio marcar la casilla de verificación, pero si está marcada, el campo de correo electrónico también debería ser obligatorio.

En la siguiente sección, mostraremos cómo puedes validar un formulario cuyos campos dependen unos de otros anulando el método `clean` en tu formulario.

#### Validación de múltiples campos (Multi-field validation)

Acabamos de ver los métodos `clean_<nombre_del_campo>` que se pueden agregar a un formulario de Django para limpiar un campo específico. Django también nos permite anular el método `clean`, en el cual podemos acceder a todos los datos limpios de todos los campos, y sabemos que se ha llamado a todos los métodos de campo personalizados. Esto permite validar los campos basándose en los datos de otro campo.

Haciendo referencia a nuestro ejemplo anterior con un formulario que tiene una dirección de correo electrónico que solo es obligatoria si se marca una casilla de verificación, veremos cómo podemos implementar esto usando el método `clean`.

Primero, crea una clase `Form` y agrega dos campos; hazlos opcionales con el argumento `required=False`:

```python
class NewsletterSignupForm(forms.Form):
    signup = forms.BooleanField(
        label="Sign up to newsletter?",
        required=False)
    email = forms.EmailField(
        help_text="Enter your email address to subscribe",
        required=False)
```

También hemos introducido dos nuevos argumentos que se pueden usar para cualquier campo:
- **label**: Esto te permite establecer el texto de la etiqueta para un campo. Como hemos visto, Django generará automáticamente el texto de la etiqueta a partir del nombre del campo. Si estableces el argumento `label`, puedes anular este valor predeterminado. Utiliza este argumento si deseas tener una etiqueta más descriptiva.
- **help_text**: Si necesitas que se muestre más información sobre qué entrada requiere un campo, puedes usar este argumento. De forma predeterminada, se muestra después del campo.

Cuando se renderiza, el formulario se ve así:

*Figura 7.4 – Formulario de registro por correo electrónico con etiqueta personalizada y texto de ayuda*

Si tuviéramos que enviar el formulario ahora, sin ingresar ningún dato, no pasaría nada. Ninguno de los campos es obligatorio, por lo que el formulario se valida correctamente.

Ahora, podemos agregar la validación de múltiples campos al método `clean`. Verificaremos si la casilla de verificación *Sign up to newsletter?* está marcada y luego verificaremos si el campo *Email* tiene un valor. Los métodos integrados de Django ya han validado que la dirección de correo electrónico sea válida en este punto, por lo que solo debemos verificar que exista un valor para ella. Luego usaremos el método `add_error` para establecer un error para el campo de correo electrónico. Este es un método que no has visto antes, pero es muy simple; toma dos argumentos: el nombre del campo en el que se establecerá el error y el texto del error.

Aquí está el código para el método `clean`:

```python
class NewsletterSignupForm(forms.Form):
    …
    def clean(self):
        cleaned_data = super().clean()
        if (cleaned_data["signup"] and not cleaned_data.get("email")):
            self.add_error("email", "Your email address is required "
                                    "if signing up for the newsletter.")
```

Tu método `clean` siempre debe llamar al método `super().clean()` para recuperar los datos limpios. Cuando se llama a `add_error` para agregar errores al formulario, el formulario ya no se validará (el método `is_valid` devuelve `False`).

Ahora, si enviamos el formulario sin la casilla de verificación marcada, aún no se genera ningún error, pero si marcas la casilla de verificación sin una dirección de correo electrónico, recibirás el error para el cual acabamos de escribir el código:

*Figura 7.5 – Error mostrado al intentar registrarse sin dirección de correo electrónico*

Es posible que notes que estamos recuperando el correo electrónico del diccionario `cleaned_data` mediante el método `get`. La razón para hacer esto es que si el valor del correo electrónico en el formulario no es válido, la clave `email` no existirá en el diccionario. El navegador debe evitar que el usuario envíe el formulario si se ha ingresado un correo electrónico no válido, pero un usuario podría estar usando un navegador más antiguo que no admita esta validación del lado del cliente, por lo que, por seguridad, usamos el método `get`. Dado que el campo `signup` es de tipo `BooleanField` y no es obligatorio, solo será inválido si se utiliza una función de validación personalizada. No estamos usando una aquí, por lo que es seguro acceder a su valor usando la notación de corchetes.

Hay un escenario de validación más a considerar antes de pasar a nuestro primer ejercicio, y es agregar errores que no son específicos de ningún campo. Django los llama **errores que no son de campo** (*non-field errors*). Hay muchos escenarios en los que podrías querer usarlos cuando múltiples campos dependen entre sí.

Tomemos, por ejemplo, un sitio web de compras. Tu formulario de pedido podría tener dos campos numéricos cuyos totales no pueden exceder un determinado valor. Si se excediera el total, el valor de cualquiera de los campos podría reducirse para que el total quede por debajo del valor máximo, por lo que el error no es específico de ninguno de los campos. Para agregar un error que no sea de campo, llama al método `add_error` con `None` como primer argumento.

Veamos cómo implementar esto. En este ejemplo, tendremos un formulario donde el usuario puede especificar una cierta cantidad de artículos para ordenar, para el artículo A o el artículo B. El usuario no puede ordenar más de 100 artículos en total. Los campos tendrán un valor máximo de 100 y un valor mínimo de 0, pero será necesario escribir una validación personalizada en el método `clean` para manejar la validación de la cantidad total:

```python
class OrderForm(forms.Form):
    item_a = forms.IntegerField(min_value=0, max_value=100)
    item_b = forms.IntegerField(min_value=0, max_value=100)

    def clean(self):
        cleaned_data = super().clean()
        if (cleaned_data.get("item_a", 0) + cleaned_data.get("item_b", 0) > 100):
            self.add_error(None, "The total number of items "
                                 "must be 100 or less.")
```

Los campos (`item_a` e `item_b`) se agregan de la forma habitual, con reglas de validación estándar. Puedes ver que hemos utilizado el método `clean` de la misma manera que lo hicimos antes. Además, hemos implementado la lógica de artículos máximos dentro de este método. La siguiente línea es la que registra el error que no es de campo si se excede el número máximo de artículos:

```python
self.add_error(None, "The total number of items must be 100 or less.")
```

Una vez más, accedemos a los valores de `item_a` e `item_b` mediante el método `get`, con un valor predeterminado de 0. Esto es en caso de que el usuario tenga un navegador más antiguo (de 2011 o antes) y pueda enviar el formulario con valores no válidos.

En un navegador, la validación a nivel de campo garantiza que se hayan ingresado valores entre 0 y 100 en cada campo y evita que el formulario se envíe de otra manera:

*Figura 7.6 – El formulario no se puede enviar si un campo excede el valor máximo*

Sin embargo, si ingresamos dos valores que suman más de 100, podemos ver cómo Django muestra el error que no es de campo:

*Figura 7.7 – Error que no es de campo de Django mostrado al inicio del formulario*

Los errores que no son de campo de Django siempre se muestran al inicio de un formulario, antes de otros campos o errores.

En el siguiente ejercicio, crearemos un formulario que implementa una función de validación, un método de limpieza de campo y un método de limpieza de formulario.

#### Ejercicio 7.01 – Métodos personalizados de limpieza y validación

En este ejercicio, crearás un nuevo formulario que permite al usuario realizar un pedido de libros o revistas. Debe tener los siguientes criterios de validación:
- El usuario puede ordenar hasta 80 revistas y/o 50 libros, pero el número total de artículos no debe ser superior a 100.
- El usuario puede elegir recibir una confirmación del pedido y, si lo hace, debe ingresar una dirección de correo electrónico.
- El usuario no debe ingresar una dirección de correo electrónico si no ha optado por recibir una confirmación del pedido.
- Para asegurarse de que formen parte de nuestra empresa, la dirección de correo electrónico debe formar parte del dominio de nuestra empresa (en nuestro caso, solo utilizaremos `example.com`).
- Para mantener la coherencia con otras direcciones de correo electrónico de nuestra empresa ficticia, la dirección debe convertirse a minúsculas.

Esto parece un montón de reglas, pero con Django es simple si las abordamos una por una. Continuaremos con la aplicación `form_project` que comenzaste en el Capítulo 6 (*Formularios*). Si no has completado el Capítulo 6, puedes descargar el código de la carpeta `Chapter07` en el repositorio de GitHub de este libro.

Sigue estos pasos:

1. En PyCharm, abre el archivo `forms.py` de la aplicación `form_example`. Asegúrate de que el servidor de desarrollo de Django no se esté ejecutando; de lo contrario, podría bloquearse a medida que realizas cambios en este archivo, lo que provocaría que PyCharm salte al depurador.
2. Dado que nuestro trabajo con `ExampleForm` ha terminado, puedes eliminarlo de este archivo.
3. Crea una nueva clase llamada `OrderForm` que herede de `forms.Form`:
   ```python
   class OrderForm(forms.Form):
   ```
4. Agrega cuatro campos a la clase, de la siguiente manera:
   - `magazine_count`, de tipo `IntegerField` con `min_value` de 0 y `max_value` de 80.
   - `book_count`, de tipo `IntegerField` con `min_value` de 0 y `max_value` de 50.
   - `send_confirmation`, de tipo `BooleanField`, que no es obligatorio.
   - `email`, de tipo `EmailField`, que tampoco es obligatorio.
   La clase debería verse así:
   ```python
   class OrderForm(forms.Form):
       magazine_count = forms.IntegerField(min_value=0, max_value=80)
       book_count = forms.IntegerField(min_value=0, max_value=50)
       send_confirmation = forms.BooleanField(required=False)
       email = forms.EmailField(required=False)
   ```
5. Agrega una función de validación para verificar que la dirección de correo electrónico del usuario esté en el dominio correcto. Primero, se debe importar `ValidationError`; agrega esta línea en la parte superior del archivo:
   ```python
   from django.core.exceptions import ValidationError
   ```
6. Luego, escribe esta función después de la línea de importación (antes de la implementación de la clase `OrderForm`):
   ```python
   def validate_email_domain(value):
       if value.split("@")[-1].lower() != "example.com":
           raise ValidationError("The email address "
                                 "must be on the domain example.com.")
   ```
   La función divide la dirección de correo electrónico en el símbolo `@` y luego verifica si la parte posterior es igual a `example.com`. Esta función por sí sola validaría direcciones que no sean de correo electrónico. Por ejemplo, la cadena `not-valid@some other domain@example.com` no provocaría que se generara `ValidationError` en esta función. Esto es aceptable en nuestro caso porque, como estamos usando `EmailField`, los otros validadores de campo estándar verificarán la validez de la dirección de correo electrónico.
7. Agrega la función `validate_email_domain` como un validador al campo `email` en `OrderForm`. Actualiza la llamada al constructor de `EmailField` para agregar un argumento `validators`, pasando una lista que contenga la función de validación:
   ```python
   class OrderForm(forms.Form):
       …
       email = forms.EmailField(required=False,
                                validators=[validate_email_domain])
   ```
8. Agrega un método `clean_email` al formulario para asegurarte de que la dirección de correo electrónico esté en minúsculas:
   ```python
   class OrderForm(forms.Form):
       # truncado por brevedad
       def clean_email(self):
           return self.cleaned_data['email'].lower()
   ```
9. Ahora, agrega el método `clean` para realizar toda la validación cruzada entre campos. Primero, agregaremos la lógica para asegurarnos de que solo se ingrese una dirección de correo electrónico si se solicita una confirmación de pedido:
   ```python
   class OrderForm(forms.Form):
       # truncado por brevedad
       def clean(self):
           cleaned_data = super().clean()
           if (cleaned_data["send_confirmation"] and not cleaned_data.get("email")):
               self.add_error("email", "Please enter an email address to "
                                       "receive the confirmation message.")
           elif (cleaned_data.get("email") and not cleaned_data["send_confirmation"]):
               self.add_error("send_confirmation", "Please check this if you want to "
                                                   " receive a confirmation email.")
   ```
   Esto agregará un error al campo Email si *Send confirmation* está marcado pero no se agrega ninguna dirección de correo electrónico:
   *Figura 7.8 – Error si Send confirmation está marcado pero no se agrega una dirección de correo electrónico*
   De manera similar, se agregará un error a Email si se ingresa una dirección de correo electrónico pero no se marca *Send confirmation*:
   *Figura 7.9 – Error porque se ingresó un correo electrónico pero el usuario no ha optado por recibir confirmación*
10. Agrega la comprobación final, también dentro del método `clean`. El número total de artículos no debe ser superior a 100. Agregaremos un error que no sea de campo si la suma de `magazine_count` y `book_count` es mayor que 100:
    ```python
    class OrderForm(forms.Form):
        …
        def clean(self):
            …
            item_total = cleaned_data.get(
                "magazine_count", 0) + cleaned_data.get(
                "book_count", 0)
            if item_total > 100:
                self.add_error(None, "The total number of items must be "
                                     "100 or less.")
    ```
    Esto agregará un error que no es de campo pasando `None` como primer argumento a la llamada `add_error`.
    Consulta el archivo `forms.py` en la carpeta `Chapter07` en el repositorio de GitHub de este libro para ver el código completo.
11. Guarda `forms.py`.
12. Abre el archivo `views.py` de la aplicación `form_example`. Cambiaremos la importación del formulario para que se importe `OrderForm` en lugar de `ExampleForm`. Considera la siguiente línea de importación:
    ```python
    from .forms import ExampleForm
    ```
    Cámbiala de la siguiente manera:
    ```python
    from .forms import OrderForm
    ```
13. En la vista `form_example`, cambia las dos líneas que usan `ExampleForm` para usar `OrderForm` en su lugar. Considera la siguiente línea de código:
    ```python
    form = ExampleForm(request.POST)
    ```
    Cámbiala de la siguiente manera:
    ```python
    form = OrderForm(request.POST)
    ```
    De manera similar, considera la siguiente línea de código:
    ```python
    form = ExampleForm()
    ```
    Cámbiala de la siguiente manera:
    ```python
    form = OrderForm()
    ```
    El resto de la función puede permanecer como está.
14. No tenemos que hacer cambios en la plantilla. Inicia el servidor de desarrollo de Django y navega a `http://127.0.0.1:8000/form-example/` en tu navegador. Deberías ver el formulario renderizado de manera similar a lo que se muestra en la Figura 7.10:
    *Figura 7.10 – OrderForm en el navegador*
15. Intenta enviar el formulario con un valor de Magazine count de 80 y un valor de Book count de 50. El navegador permitirá esto, pero dado que suman más de 100, el método `clean` en el formulario generará un error que se mostrará en la página:
    *Figura 7.11 – Un error que no es de campo se muestra en el formulario cuando se excede la cantidad máxima permitida de artículos*
16. Intenta enviar el formulario con *Send confirmation* marcado pero el campo Email en blanco. Luego, completa el cuadro de texto Email, pero desmarca *Send confirmation*. Cualquiera de las dos combinaciones arrojará un error que indica que ambos deben estar presentes. El error diferirá según el campo que falte:
    *Figura 7.12 – Mensaje de error si no hay dirección de correo electrónico presente*
17. Ahora, intenta enviar el formulario con *Send confirmation* marcado y una dirección de correo electrónico que no esté en el dominio `example.com`. Deberías recibir un mensaje que indique que tu dirección de correo electrónico debe tener el dominio `example.com`. También deberías recibir un mensaje de que se debe configurar Email; esto se debe a que el correo electrónico no termina en el diccionario `cleaned_data`. Después de todo, no es válido:
    *Figura 7.13 – El mensaje de error que se muestra cuando el dominio del correo electrónico no es example.com*
18. Finalmente, ingresa valores válidos para Magazine count y Book count (como 20 y 20). Marca *Send confirmation* e ingresa `UserName@Example.Com` como correo electrónico (asegúrate de que coincida con el uso de mayúsculas y minúsculas, incluidos los caracteres mixtos en mayúsculas y minúsculas):
    *Figura 7.14 – El formulario después de enviarse con valores válidos*
19. Cambia a PyCharm y mira en la consola de depuración. Verás que el correo electrónico se convierte a minúsculas cuando lo imprime nuestro código de depuración:
    *Figura 7.15 – Correo electrónico en minúsculas, así como otros campos*
    Este es nuestro método `clean_email` en acción: aunque ingresamos datos tanto en mayúsculas como en minúsculas, se han convertido a minúsculas en su totalidad.

En este ejercicio, creamos un nuevo `OrderForm` que implementó métodos de limpieza de formulario y de campo. Usamos un validador personalizado para asegurarnos de que el campo Email cumpliera con nuestras reglas de validación específicas: solo se permitía un dominio específico. Usamos un método de limpieza de campo personalizado (`clean_email`) para convertir la dirección de correo electrónico a minúsculas. Luego implementamos un método `clean` para validar los formularios que dependían unos de otros. En este método, agregamos errores de campo y errores que no eran de campo.

Hasta ahora, hemos aprendido sobre las validaciones de múltiples campos y cómo implementarlas para asegurarnos de que podemos validar el formulario de una sola vez en lugar de resaltar los errores uno por uno.

En la siguiente sección, cubriremos cómo agregar marcadores de posición (*placeholders*) y valores iniciales al formulario.

---

### Sección: Adición de marcadores de posición (placeholders) y valores iniciales

Hay dos cosas que nuestro primer formulario creado manualmente tenía y que nuestro formulario actual de Django todavía no tiene: marcadores de posición (*placeholders*) y valores iniciales. Agregar marcadores de posición es simple; simplemente se agregan como atributos al constructor de widget para el campo de formulario. Esto es similar a lo que ya hemos visto para configurar el tipo de `DateField` en nuestros ejemplos anteriores.

He aquí un ejemplo:

```python
class ExampleForm(forms.Form):
    text_field = forms.CharField(
        widget=forms.TextInput(
            attrs={"placeholder": "Text Placeholder"})
    )
    password_field = forms.CharField(
        widget=forms.PasswordInput(
            attrs={"placeholder": "Password Placeholder"})
    )
    email_field = forms.EmailField(
        widget=forms.EmailInput(
            attrs={"placeholder": "Email Placeholder"})
    )
    text_area = forms.CharField(
        widget=forms.Textarea(
            attrs={"placeholder": "Text Area Placeholder"})
    )
```

Así es como se ve el formulario anterior cuando se renderiza en el navegador:

*Figura 7.16 – Formulario de Django con marcadores de posición*

Por supuesto, si configuramos manualmente `Widget` para cada campo, debemos saber qué clase `Widget` usar. Los que admiten marcadores de posición son `TextInput`, `NumberInput`, `EmailInput`, `URLInput`, `PasswordInput` y `Textarea`.

Mientras examinamos la clase `Form` en sí, veremos la primera de dos formas de establecer un valor inicial para un campo. Podemos hacer esto usando el argumento `initial` en el constructor de `Field`, de esta manera:

```python
text_field = forms.CharField(initial="Initial Value", …)
```

El otro método es pasar un diccionario de datos al instanciar el formulario en nuestra vista. Las claves son los nombres de los campos. El diccionario debe tener cero o más elementos (es decir, un diccionario vacío es válido). Cualquier clave adicional se ignora. Este diccionario debe proporcionarse como el argumento `initial` en nuestra vista, de la siguiente manera:

```python
initial = {"text_field": "Text Value", "email_field": "user@example.com"}
form = ExampleForm(initial=initial)
```

O, para una solicitud POST, pasa `request.POST` como primer argumento, como de costumbre:

```python
initial = {"text_field": "Text Value", "email_field": "user@example.com"}
form = ExampleForm(request.POST, initial=initial)
```

Los valores en `request.POST` anularán los valores en `initial`. Esto significa que incluso si tenemos un valor inicial para un campo obligatorio, si se deja en blanco al enviarlo, no se validará. El campo no volverá al valor en `initial`.

Decidir si establecer valores iniciales en la clase `Form` misma o en la vista depende de ti y de tu caso de uso. Si tuvieras un formulario que se usara en múltiples vistas pero que generalmente tuviera el mismo valor, sería mejor establecer el valor inicial en el formulario. De lo contrario, puede ser más flexible usar la configuración en la vista.

En el siguiente ejercicio, agregaremos marcadores de posición y valores iniciales a la clase `OrderForm` del ejercicio anterior.

#### Ejercicio 7.02 – Marcadores de posición y valores iniciales

En este ejercicio, mejorarás la clase `OrderForm` agregando texto de marcador de posición. Simularás pasar una dirección de correo electrónico inicial al formulario. Será una dirección codificada de forma fija (*hardcoded*), pero una vez que el usuario pueda iniciar sesión, podría ser una dirección de correo electrónico asociada con su cuenta; aprenderás sobre sesiones y autenticación en el Capítulo 9 (*Sesiones y Autenticación*).

Sigue estos pasos:

1. En PyCharm, abre el archivo `forms.py` de la aplicación `form_example`. Agregarás marcadores de posición a los campos `magazine_count`, `book_count` y `email` en `OrderForm`, lo que significa que también configurarás un widget. Al campo `magazine_count`, agrega un widget `NumberInput` con `placeholder` en el diccionario `attrs`. Este valor de marcador de posición debe establecerse en `Number of Magazines`. Escribe el siguiente código:
   ```python
   magazine_count = forms.IntegerField(
       min_value=0,
       max_value=80,
       widget=forms.NumberInput(
           attrs={"placeholder": "Number of Magazines"}),
   )
   ```
2. Agrega un marcador de posición al campo `book_count` de la misma manera. El texto del marcador de posición debe ser `Number of Books`:
   ```python
   book_count = forms.IntegerField(
       min_value=0,
       max_value=50,
       widget=forms.NumberInput(
           attrs={"placeholder": "Number of Books"}),
   )
   ```
3. El cambio final en `OrderForm` es agregar un marcador de posición al campo `email`. Esta vez, el widget es `EmailInput`. El texto del marcador de posición debe ser `Your company email address`:
   ```python
   email = forms.EmailField(
       required=False,
       validators=[validate_email_domain],
       widget=forms.EmailInput(attrs={
           "placeholder": "Your company email address"}),
   )
   ```
   Ten en cuenta que los métodos `clean_email` y `clean` deben permanecer como estaban en el Ejercicio 7.01. Guarda el archivo.
4. Abre el archivo `views.py` de la aplicación `form_example`. En la función de vista `form_example`, crea una nueva variable de diccionario llamada `initial` con una clave, `email`, de esta manera:
   ```python
   initial = {"email": "user@example.com"}
   ```
5. En los dos lugares donde estás instanciando `OrderForm`, pasa también la variable `initial` mediante el argumento de palabra clave `initial`. La primera instancia es la siguiente:
   ```python
   form = OrderForm(request.POST, initial=initial)
   ```
   La segunda instancia es la siguiente:
   ```python
   form = OrderForm(initial=initial)
   ```
   El código completo para `views.py` se puede encontrar en la carpeta `Chapter07` en el repositorio de GitHub de este libro.
6. Guarda el archivo `views.py`.
7. Inicia el servidor de desarrollo de Django si aún no se está ejecutando. Navega a `http://127.0.0.1:8000/form-example/` en tu navegador. Deberías ver que tu formulario ahora tiene marcadores de posición y un valor inicial establecido:
   *Figura 7.17 – Formulario de pedido con valores iniciales y marcadores de posición*

En este ejercicio, agregamos marcadores de posición a los campos del formulario. Esto se hizo configurando un widget de formulario al definir el campo de formulario en la clase de formulario y estableciendo un valor de marcador de posición en el diccionario `attrs`. También establecimos un valor inicial para el formulario usando un diccionario y lo pasamos a la instancia del formulario usando el argumento de palabra clave `initial`.

En la siguiente sección, hablaremos sobre cómo trabajar con modelos de Django usando datos de formularios y cómo `ModelForm` facilita esto.

---

### Sección: Creación o edición de modelos de Django

Hemos visto cómo definir un formulario y, en el Capítulo 2 (*Modelos y Migraciones*), aprendiste a crear instancias de modelos de Django. Al usar estas cosas juntas, puedes crear una vista que muestre un formulario y también guarde una instancia de modelo en la base de datos. Esto te brinda una manera fácil de guardar datos sin tener que escribir mucho código repetitivo (*boilerplate*) ni crear formularios personalizados. En Bookr, usaremos este método para permitir que los usuarios agreguen reseñas sin necesidad de acceder al sitio de administración de Django. Sin usar `ModelForm`, puedes hacer algo como esto:
1. Puedes crear un formulario basado en un modelo existente, por ejemplo, `Publisher`. El formulario se llamaría `PublisherForm`.
2. Puedes definir manualmente los campos en `PublisherForm`, usando las mismas reglas definidas en el modelo `Publisher`, como se muestra aquí:
   ```python
   class PublisherForm(forms.Form):
       name = forms.CharField(max_length=50)
       website = forms.URLField()
       …
   ```
3. En la vista, los valores iniciales se recuperarían del modelo consultado de la base de datos y luego se pasarían al formulario mediante el argumento `initial`. Si estuvieras creando una nueva instancia, el valor inicial estaría en blanco, algo como esto:
   ```python
   if create:
       initial = {}
   else:
       publisher = Publisher.objects.get(pk=pk)
       initial = {"name": publisher.name, "website": publisher.website, …}
   form = PublisherForm(initial=initial)
   ```
4. Luego, en el flujo POST de la vista, puedes crear o actualizar el modelo basándote en `cleaned_data`:
   ```python
   form = PublisherForm(request.POST, initial=initial)
   if create:
       publisher = Publisher()
   else:
       publisher = Publisher.objects.get(pk=pk)
   publisher.name = form.cleaned_data['name']
   publisher.website = forms.cleaned_data['website']
   …
   publisher.save()
   ```

Esto requiere mucho trabajo y debemos considerar cuánta lógica duplicada tenemos. Por ejemplo, estamos definiendo la longitud del nombre en el campo de formulario `name`. Si cometiéramos un error aquí, podríamos permitir un nombre más largo en el campo de lo que permite el modelo. También debemos recordar configurar todos los campos en el diccionario `initial`, así como establecer los valores en el modelo nuevo o actualizado con `cleaned_data` del formulario. Hay muchas oportunidades para cometer errores aquí, además de tener que recordar agregar o eliminar datos de configuración de campos para cada uno de estos pasos si el modelo cambia. Todo este código tendría que duplicarse para cada modelo de Django con el que trabajes, lo que exacerba el problema de duplicación.

#### La clase ModelForm

Afortunadamente, Django proporciona un método para crear instancias de `Model` a partir de formularios de manera mucho más simple, con la clase `ModelForm`. `ModelForm` es un formulario que se crea automáticamente a partir de un modelo en particular. Heredará las reglas de validación del modelo (como si los campos son obligatorios, la longitud máxima de las instancias de `CharField`, etc.). Proporciona un argumento extra para `__init__` (llamado `instance`) para completar automáticamente los valores iniciales de un modelo existente. También agrega un método `save` para persistir automáticamente los datos del formulario en la base de datos. Todo lo que necesitas hacer para configurar `ModelForm` es especificar su modelo y qué campos deben usarse: esto se hace en el atributo `class Meta` de la clase de formulario. Veamos cómo construir un formulario a partir de `Publisher`.

Dentro del archivo que contiene el formulario (por ejemplo, el archivo `forms.py` con el que hemos estado trabajando), el único cambio es que se debe importar el modelo:

```python
from .models import Publisher
```

Luego, se puede definir la clase `Form`. La clase requiere un atributo `class Meta`, que, a su vez, debe definir un atributo `model` y los atributos `fields` o `exclude`:

```python
class PublisherForm(forms.ModelForm):
    class Meta:
        model = Publisher
        fields = ("name", "website", "email")
```

`fields` es una lista o tupla de los campos que se incluirán en el formulario. Al configurar manualmente la lista de campos, si agregas campos adicionales al modelo, también debes agregar su nombre aquí para que se muestren en el formulario.

También puedes usar el valor especial `"__all__"` en lugar de una lista o tupla para incluir automáticamente todos los campos, de esta manera:

```python
class PublisherForm(forms.ModelForm):
    class Meta:
        model = Publisher
        fields = "__all__"
```

Si el campo del modelo tiene su atributo `editable` establecido en `False`, no se incluirá automáticamente.

Por el contrario, el atributo `exclude` establece los campos que no se mostrarán en el formulario. Cualquier campo agregado al modelo se agregará automáticamente al formulario. Podríamos definir el formulario anterior usando `exclude` con una tupla vacía, ya que queremos todos los campos. El código es así:

```python
class PublisherForm(forms.ModelForm):
    class Meta:
        model = Publisher
        exclude = ()
```

Esto te ahorra algo de trabajo porque no necesitas agregar un campo tanto al modelo como a la lista de campos; sin embargo, no es tan seguro ya que podrías exponer automáticamente campos al usuario final que no deseas. Por ejemplo, si tuvieras un modelo `User` con `UserForm`, podrías agregar un campo `is_admin` al modelo `User` para otorgar privilegios adicionales a los usuarios administradores. Si este campo no estuviera en `exclude`, se mostraría al usuario. Un usuario podría convertirse a sí mismo en administrador, lo cual es algo que probablemente no querrías.

Cualquiera de estos tres enfoques para elegir los formularios que decidamos mostrar, en nuestro caso, se mostrarán igual en el navegador. Esto se debe a que estamos eligiendo mostrar todos los campos. Todos se ven así cuando se renderizan en el navegador:

*Figura 7.18 – PublisherForm*

Ten en cuenta que el `help_text` del modelo `Publisher` también se renderiza automáticamente.

El uso en una vista es similar a los otros formularios que hemos visto. Además, como se mencionó, hay un argumento adicional que se puede proporcionar, llamado `instance`. Esto se puede establecer en `None`, lo que renderizará un formulario vacío.

Suponiendo que en tu función de vista tengas algún método para determinar si estás creando o editando una instancia de modelo (hablaremos sobre cómo hacer esto más adelante), esto determinará una variable llamada `is_create` (`True` si se crea una instancia o `False` si se edita una existente). Tu función de vista para crear el formulario podría escribirse de la siguiente manera:

```python
if is_create:
    instance = None
else:
    instance = get_object_or_404(Publisher, pk=pk)

if request.method == "POST":
    form = PublisherForm(request.POST, instance=instance)
    if form.is_valid():
        # cubriremos esta rama pronto
else:
    form = PublisherForm(instance=instance)
```

Como puedes ver, en cualquiera de las ramas, la instancia se pasa al constructor de `PublisherForm`, aunque es `None` si estamos en modo de creación.

Si el formulario es válido, podemos guardar la instancia del modelo. Esto se puede hacer simplemente llamando al método `save` en el formulario. Esto creará automáticamente la instancia o simplemente guardará los cambios en la anterior:

```python
if form.is_valid():
    form.save()
    return redirect(success_url)
```

El método `save` devuelve la instancia del modelo que se guardó. Toma un argumento opcional, `commit`, que determina si los cambios deben escribirse en la base de datos. Puedes pasar `False` en su lugar, lo que te permite realizar cambios adicionales en la instancia antes de guardar manualmente los cambios. Esto sería necesario para establecer atributos que no se han incluido en el formulario. Como mencionamos, tal vez establecerías el indicador `is_admin` en `False` en una instancia de `User`:

```python
if form.is_valid():
    new_user = form.save(False)
    new_user.is_admin = False
    new_user.save()
    return redirect(success_url)
```

En la Actividad 7.02 (*Interfaz de usuario para la creación de reseñas*), al final de este capítulo, también utilizaremos esta función.

Si tu modelo utiliza campos `ManyToMany` y llamas a `form.save(False)`, también debes llamar a `form.save_m2m()` para guardar cualquier relación de muchos a muchos que se haya establecido. No es necesario llamar a este método si llamas al método `form.save` con `commit` establecido en `True` (es decir, el valor predeterminado).

Los formularios de modelos se pueden personalizar realizando cambios en sus atributos `Meta`. Se puede configurar el atributo `widgets`. Puede contener un diccionario codificado en los nombres de campo, con clases o instancias de widgets como valores. Por ejemplo, así es como se configura `PublisherForm` para que tenga marcadores de posición:

```python
class PublisherForm(forms.ModelForm):
    class Meta:
        model = Publisher
        fields = "__all__"
        widgets = {
            "name": forms.TextInput(attrs={
                "placeholder": "The publisher's name."
            })
        }
```

Los valores se comportan igual que al configurar el argumento de palabra clave `widget` en la definición del campo; pueden ser una clase o una instancia. Por ejemplo, para mostrar `CharField` como una entrada de contraseña, se puede utilizar la clase `PasswordInput`; no es necesario crear una instancia:

```python
widgets = {"password": forms.PasswordInput}
```

Los formularios de modelos también se pueden enriquecer con campos adicionales agregados de la misma manera que se agregan a un formulario normal. Por ejemplo, supongamos que quisiéramos ofrecer la opción de enviar un correo electrónico de notificación después de guardar un `Publisher`. Podemos agregar un campo `email_on_save` a `PublisherForm` de esta manera:

```python
class PublisherForm(forms.ModelForm):
    email_on_save = forms.BooleanField(required=False,
                                       help_text="Send notification email on save")
    class Meta:
        model = Publisher
        fields = "__all__"
```

Cuando se renderiza, el formulario se ve así:

*Figura 7.19 – PublisherForm con un campo adicional*

Los campos adicionales se colocan después de los campos de `Model`. Los campos adicionales no se manejan automáticamente: no existen en el modelo, por lo que Django no intentará guardarlos en la instancia del modelo. En su lugar, debes manejar cómo se guardan sus valores examinando los valores de `cleaned_data` del formulario, como lo harías con un formulario estándar, por ejemplo (dentro de tu función de vista):

```python
if form.is_valid():
    if form.cleaned_data.get("email_on_save"):
        send_email()  # asumimos que esta función está definida en otra parte
    form.save()  # guarda la instancia independientemente de si
                 # se envió el correo electrónico o no
    return redirect(success_url)
```

En el siguiente ejercicio, escribirás una nueva función de vista en Bookr que te permitirá crear y editar un `Publisher`.

#### Ejercicio 7.03 – Creación y edición de una editorial

En este ejercicio, volveremos a Bookr. Queremos agregar la capacidad de crear y editar un `Publisher` sin usar el administrador de Django. Para hacer esto, agregaremos un `ModelForm` para el modelo `Publisher`. Se utilizará en una nueva función de vista. Esta función de vista tomará un argumento opcional, `pk`, que será el ID del objeto `Publisher` que se está editando o `None` para crear un nuevo `Publisher`. Agregaremos dos nuevos mapas de URL para facilitar esto. Cuando esto esté completo, podremos ver y actualizar cualquier editorial usando su ID. Por ejemplo, la información para el Editor 1 se podrá ver/editar en la ruta de URL `/publishers/1/`:

1. En PyCharm, abre el archivo `forms.py` de la aplicación `reviews`. Después de la importación de `forms`, importa el modelo `Publisher`:
   ```python
   from .models import Publisher
   ```
2. Crea una clase `PublisherForm`, heredando de `forms.ModelForm`:
   ```python
   class PublisherForm(forms.ModelForm):
   ```
3. Define el atributo `class Meta` en `PublisherForm`. Los atributos que requiere `Meta` son el modelo (`Publisher`) y los campos (`"__all__"`):
   ```python
   class PublisherForm(forms.ModelForm):
       class Meta:
           model = Publisher
           fields = "__all__"
   ```
4. Guarda `forms.py`. El archivo completo se puede encontrar en la carpeta `Chapter07` en el repositorio de GitHub de este libro.
5. Abre el archivo `views.py` de la aplicación `reviews`. En la parte superior del archivo, importa `PublisherForm`:
   ```python
   from .forms import PublisherForm, SearchForm
   ```
6. Asegúrate de importar las funciones `get_object_or_404` y `redirect` desde `django.shortcuts`, si aún no lo estás haciendo:
   ```python
   from django.shortcuts import (
       render, get_object_or_404, redirect)
   ```
7. Además, asegúrate de importar el modelo `Publisher` si aún no lo estás haciendo. Es posible que ya estés importando este y otros modelos:
   ```python
   from .models import Book, Contributor, Publisher
   ```
8. La importación final que necesitarás es el módulo `messages`. Esto nos permitirá registrar un mensaje para informar al usuario que se editó o creó un objeto `Publisher`:
   ```python
   from django.contrib import messages
   ```
   Una vez más, agrega esta importación si aún no la tienes.
9. Crea una nueva función de vista llamada `publisher_edit`. Toma dos argumentos: `request` y `pk` (el ID del objeto `Publisher` a editar). Esto es opcional, y si es `None`, se creará un objeto `Publisher` en su lugar:
   ```python
   def publisher_edit(request, pk=None):
   ```
10. Dentro de la función de vista, debemos intentar cargar la instancia de `Publisher` existente si `pk` no es `None`. De lo contrario, `publisher` debería ser `None`:
    ```python
    def publisher_edit(request, pk=None):
        if pk is not None:
            publisher = get_object_or_404(Publisher, pk=pk)
        else:
            publisher = None
    ```
11. Después de obtener una instancia de `Publisher` o `None`, completa la rama para una solicitud POST. Crea una instancia del formulario de la misma manera que viste anteriormente en este capítulo, pero ahora, asegúrate de que tome `instance` como argumento de palabra clave. Luego, si el formulario es válido, guárdalo usando el método `form.save()`. Este método devolverá la instancia de `Publisher` actualizada, que se almacena en la variable `updated_publisher`. Luego, registra un mensaje de éxito diferente según si el `Publisher` se creó o se actualizó. Finalmente, redirige de nuevo a esta vista `publisher_edit`, ya que `updated_publisher` siempre tendrá un ID en este punto:
    ```python
    def publisher_edit(request, pk=None):
        …
        if request.method == "POST":
            form = PublisherForm(request.POST, instance=publisher)
            if form.is_valid():
                updated_publisher = form.save()
                if publisher is None:
                    messages.success(request, f'Publisher "{updated_publisher}" '
                                              f'was created.')
                else:
                    messages.success(request, f'Publisher "{updated_publisher}" '
                                              f'was updated.')
                return redirect("publisher_edit", updated_publisher.pk)
    ```
    Si el formulario no es válido, la ejecución pasará de largo y simplemente devolverá la llamada a la función `render` con el formulario no válido (esto se implementará en el Paso 13). La redirección utiliza un mapa de URL con nombre, que se agregará más adelante en este ejercicio.
12. A continuación, completa la rama del código que no es POST. En este caso, simplemente crea una instancia del formulario con tu instancia:
    ```python
    def publisher_edit(request, pk=None):
        …
        if request.method == "POST":
            …
        else:
            form = PublisherForm(instance=publisher)
    ```
13. En el Paso 15, crearás un archivo `form-example.html` para renderizar, pero podemos agregar la llamada a la función `render` ahora. Renderízalo en la vista con la función `render` pasando el método HTTP y el formulario como contexto:
    ```python
    def publisher_edit(request, pk=None):
        …
        return render(request, "form-example.html", {"method": request.method, "form": form})
    ```
    Guarda este archivo. Puedes consultar esto en la carpeta `Chapter07` en el repositorio de GitHub de este libro.
14. Abre `urls.py` en el directorio `reviews`. Agrega dos nuevos mapas de URL; ambos irán a la vista `publisher_edit`. Uno capturará el ID de `Publisher`, que querremos editar y pasar a la vista como el argumento `pk`. El otro usará `new` en su lugar y no pasará el argumento `pk`, lo que indicará que queremos crear un nuevo `Publisher`. A tu variable `urlpatterns`, agrega el mapeo de ruta `"publishers/<int:pk>/"` a la vista `reviews.views.publisher_edit` con el nombre `"publisher_edit"`. Además, agrega el mapeo de ruta `"publishers/new/"` a la vista `reviews.views.publisher_edit` con el nombre `"publisher_create"`:
    ```python
    urlpatterns = [
        …
        path("publishers/<int:pk>/", views.publisher_edit, name="publisher_edit"),
        path("publishers/new/", views.publisher_edit, name="publisher_create")
    ]
    ```
    Dado que el segundo mapeo no captura nada, el valor `pk` que se pasa a la función de vista `publisher_edit` es `None`.
    Guarda el archivo `urls.py`. La versión completa de referencia se encuentra en la carpeta `Chapter07` en el repositorio de GitHub de este libro.
15. Crea un archivo `form-example.html` dentro del directorio `templates` de la aplicación `reviews` utilizando la función New HTML File en PyCharm. Dado que esta es una plantilla independiente (no extiende ninguna otra plantilla), debemos renderizar los mensajes dentro de ella. Agrega este código justo después de la etiqueta de apertura `<body>` para iterar a través de todos los mensajes y mostrarlos:
    ```django
    {% for message in messages %}
        <p><em>{{ message.level_tag|title }}:</em> {{ message }}</p>
    {% endfor %}
    ```
    Esto recorrerá los mensajes que hemos agregado y mostrará la etiqueta (en nuestro caso, Success) y luego el mensaje.
16. Luego, agrega el código normal de renderizado y envío del formulario:
    ```django
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <p>
            <input type="submit" value="Submit">
        </p>
    </form>
    ```
    Guarda y cierra este archivo. Puedes consultar la versión completa de este archivo en la carpeta `Chapter07` en el repositorio de GitHub de este libro.
17. Inicia el servidor de desarrollo de Django, luego navega a `http://127.0.0.1:8000/publishers/new/`. Deberías ver un `PublisherForm` en blanco:
    *Figura 7.20 – Formulario de editorial en blanco*
18. El formulario ha heredado las reglas de validación del modelo, por lo que no puedes enviarlo con demasiados caracteres para Name o con un valor de Website o Email no válido. Ingresa información válida y luego envía el formulario. Después del envío, deberías ver el mensaje de Success; el formulario se completará con la información que se guardó en la base de datos:
    *Figura 7.21 – Formulario después del envío*
    Observa que la URL también se ha actualizado y ahora incluye el ID de la editorial que se creó. En este caso, es `http://127.0.0.1:8000/publishers/10/`; el ID en tu configuración dependerá de cuántas instancias de `Publisher` ya estaban en tu base de datos.
    Observa que si actualizas la página, no recibirás un mensaje que confirme si deseas reenviar los datos del formulario. Esto se debe a que redirigimos después de guardar, por lo que es seguro actualizar esta página tantas veces como desees y no se crearán nuevas instancias de `Publisher`. Si no lo hubieras redirigido, cada vez que se actualizara la página, se crearía una nueva instancia de `Publisher`.
19. Si tienes otras instancias de `Publisher` en tu base de datos, puedes cambiar el ID en la URL para editar otras. Dado que el ID en esta instancia es 3, podemos asumir que el Publisher 1 y el Publisher 2 ya existen y podemos sustituir sus ID para ver los datos existentes. Aquí está la vista de la información existente del Publisher 1 (en `http://127.0.0.1:8000/publishers/1/`): tu información puede ser diferente:
    *Figura 7.22 – Información existente de Publisher 1*
20. Intenta realizar cambios en la instancia de `Publisher` existente. Observa que después de guardar, el mensaje es diferente: le dice al usuario que la instancia de `Publisher` se actualizó en lugar de crearse:
    *Figura 7.23 – Editorial después de actualizar en lugar de crear*

En este ejercicio, implementamos un `ModelForm` a partir de un modelo (`PublisherForm` se creó a partir de `Publisher`) y vimos cómo Django generó automáticamente los campos del formulario con las reglas de validación correctas. Luego usamos el método integrado `save` del formulario para guardar los cambios en la instancia de `Publisher` (o crearla automáticamente) dentro de la vista `publisher_edit`. Asignamos dos URLs a la vista. La primera fue para editar un `Publisher` existente y pasar un `pk` a la vista. La otra no pasó este `pk` a la vista, lo que indica que se debe crear la instancia de `Publisher`. Finalmente, usamos el navegador para experimentar con la creación de una nueva instancia de `Publisher` y luego editar una existente.

---

### Sección: Actividad 7.01 – Estilo e integración del formulario de editorial

En el Ejercicio 7.03 (*Creación y edición de una editorial*), agregaste `PublisherForm` para crear y editar instancias de `Publisher`. Construiste esto con una plantilla independiente que no extendía ninguna otra plantilla, por lo que carecía de los estilos globales. En esta actividad, crearás una página de detalle de formulario genérica que mostrará un formulario de Django, similar a `form-example.html`, pero extendiéndose desde una plantilla base. La plantilla aceptará una variable para mostrar el tipo de modelo que se está editando. También actualizarás la plantilla principal `base.html` para renderizar los mensajes de Django, utilizando el estilo de Bootstrap.

Estos pasos te ayudarán a completar esta actividad:

1. Comienza editando el archivo `base.html` del proyecto. Envuelve el bloque de contenido en un contenedor `div` para un diseño más agradable con algo de espaciado. Rodea el bloque `content` existente con un elemento `<div>` con `class="container-fluid"`.
2. Renderiza cada mensaje en `messages` (similar al Paso 15 del Ejercicio 7.03). Agrega el bloque `{% for %}` después del contenedor `<div>` que acabas de crear, pero antes del bloque de contenido. Debes usar las clases del framework Bootstrap; este fragmento te ayudará:
   ```django
   <div role="alert" class="alert alert-{{ {'error': 'danger'}.get(level_tag, level_tag)}}" >
       {{ message }}
   </div>
   ```
   La clase Bootstrap y las etiquetas de mensajes de Django tienen nombres correspondientes en su mayor parte (por ejemplo, `success` y `alert-success`). La excepción es la etiqueta `error` de Django. La clase Bootstrap correspondiente es `alert-danger`. Consulta más información sobre las alertas de Bootstrap en [https://getbootstrap.com/docs/5.3/components/alerts/](https://getbootstrap.com/docs/5.3/components/alerts/). Esta es la razón por la que necesitas usar la etiqueta de plantilla `if` en este fragmento.
3. Crea una nueva plantilla llamada `instance-form.html`, dentro del directorio de plantillas con espacio de nombres de la aplicación `reviews`.
4. Ahora, `instance-form.html` debe extenderse del archivo `base.html` de la aplicación `reviews`.
5. El contexto que se pasa a esta plantilla contendrá una variable llamada `instance`. Esta será la instancia de `Publisher` que se está editando, o `None` si estamos creando una nueva instancia de `Publisher`. El contexto también contendrá una variable `model_type`, que es una cadena que indica el tipo de modelo (en este caso, `Publisher`). Utiliza estas dos variables para completar la etiqueta de plantilla del bloque `title`:
   - Si la `instance` es `None`, el título debe ser `New Publisher`.
   - De lo contrario, el título debe ser `Editing Publisher <Publisher Name>`.
6. `instance-form.html` debe contener una etiqueta de plantilla de bloque `content` para anular el bloque `content` de `base.html`.
7. Agrega un elemento `<h2>` dentro del bloque `content` y complétalo usando la misma lógica que el título. Para un mejor estilo, envuelve el nombre de la editorial en un elemento `<em>`.
8. Agrega un elemento `<form>` a la plantilla con un valor `method` de `post`. Como estamos publicando en la misma URL, no es necesario especificar un valor de `action`.
9. Incluye la etiqueta de plantilla del token CSRF en el cuerpo de `<form>`.
10. Renderiza el formulario de Django (su variable de contexto será `form`) dentro de `<form>`, usando el método `as_p`.
11. Agrega un elemento `<button>` de envío al formulario. Su texto debe depender de si estás editando o creando. Usa el texto `Save` para editar o `Create` para crear. Puedes usar las clases de Bootstrap para el estilo del botón aquí. Debe tener el atributo `class="btn btn-primary"`.
12. En `reviews/views.py`, la vista `publisher_edit` no necesita muchos cambios. Actualiza la llamada a `render` para renderizar `instance-form.html` en lugar de `form-example.html`.
13. Actualiza el diccionario de contexto que se pasa a la llamada a `render`. Debe incluir la instancia de `Publisher` (la variable `publisher` que ya estaba definida) y la cadena `model_type`. El diccionario de contexto ya incluye `form` (una instancia de `PublisherForm`). Puedes eliminar la clave `method`.
14. Como ya terminamos con la plantilla `form-example.html`, se puede eliminar.

Cuando hayas terminado, la página de creación de editorial (en `http://127.0.0.1:8000/publishers/new/`) debería verse como se muestra en la Figura 7.24:

*Figura 7.24 – La página de creación de Publisher*

Al editar un `Publisher` (por ejemplo, en `http://127.0.0.1:8000/publishers/1/`), tu página debería verse como se muestra en la Figura 7.25:

*Figura 7.25 – La página Editing Publisher*

Después de guardar una instancia de `Publisher`, ya sea creando o editando, deberías ver un mensaje de éxito en la parte superior de la página (Figura 7.26):

*Figura 7.26 – Mensaje de éxito renderizado como una alerta de Bootstrap*

---

### Sección: Actividad 7.02 – Interfaz de usuario para la creación de reseñas

La Actividad 7.01 (*Estilo e integración del formulario de editorial*) fue bastante extensa; sin embargo, al completarla, has creado una base que facilita agregar otras vistas de edición y creación. Experimentarás esto de primera mano en esta actividad cuando crees formularios para crear y editar reseñas. Debido a que la plantilla `instance-form.html` se creó de forma genérica, puedes reutilizarla en otras vistas.

En esta actividad, crearás un `ModelForm` de reseña y luego agregarás una vista `review_edit` para crear o editar una instancia de `Review`. Puedes reutilizar `instance-form.html` de la Actividad 7.01 y pasar diferentes variables de contexto para que funcione con el modelo `Review`. Al trabajar con reseñas, operarás dentro del contexto de un libro, es decir, la vista `review_edit` debe aceptar una `pk` de `Book` como argumento. Obtendrás este `Book` por separado y lo asignarás a la instancia de `Review` que crees.

Estos pasos te ayudarán a completar esta actividad:

1. En `forms.py`, agrega una subclase `ReviewForm` de `ModelForm`; su modelo debe ser `Review` (asegúrate de importar el modelo `Review`). `ReviewForm` debe excluir los campos `date_edited` y `book` ya que el usuario no debería configurarlos en el formulario. La base de datos permite cualquier calificación, pero podemos anular el campo `rating` con un `IntegerField` que requiera un valor mínimo de 0 y un valor máximo de 5.
2. Crea una nueva vista llamada `review_edit`. Debe aceptar dos argumentos después de `request`: `book_pk`, que es obligatorio, y `review_pk`, que es opcional (el valor predeterminado es `None`). Obtén la instancia de `Book` y la instancia de `Review` mediante el acceso directo `get_object_or_404` (llámalo una vez para cada tipo). Al obtener la reseña, asegúrate de que la reseña pertenezca al libro. Si `review_pk` es `None`, entonces la instancia de `Review` también debería ser `None`.
3. Si el método de solicitud es POST, crea una instancia de `ReviewForm` usando `request.POST` y la instancia de `Review`. Asegúrate de importar este `ReviewForm`. Si el formulario es válido, guarda el formulario pero cambia el argumento `commit` de `save` a `False`. Luego, establece el atributo `book` en la instancia de `Review` devuelta en el libro que se obtuvo en el Paso 2.
4. Si la instancia de `Review` se está actualizando en lugar de crearse, también debes establecer el atributo `date_edited` en la fecha y hora actuales. Usa la función `from django.utils import timezone` y llama a `timezone.now()`. Luego, guarda la instancia de `Review`.
5. Finaliza la rama del formulario válido registrando un mensaje de éxito y redirigiendo de vuelta a la vista `book_detail`. Dado que el modelo `Review` no contiene una descripción de texto significativa, usa el título del libro en el mensaje; por ejemplo, `Review for "<book title>" created.`.
6. Si el método de solicitud no es POST, crea una instancia de `ReviewForm` y simplemente pasa la instancia de `Review`.
7. Renderiza la plantilla `instance-form.html`. En el diccionario de contexto, incluye los mismos elementos que se usaron en `publisher_view`: `form`, `instance` y `model_type` (`Review`). Incluye dos elementos adicionales: `related_model_type`, que debe ser `Book`, y `related_instance`, que será la instancia de `Book`.
8. Edita `instance-form.html` para agregar un lugar donde mostrar la información de la instancia relacionada agregada en el Paso 7. Debajo del elemento `<h2>`, agrega un elemento `<p>` que solo se muestre si tanto `related_model_type` como `related_instance` están configurados. Debe mostrar el texto `For <related_model_type> <related_instance>` (por ejemplo, `For Book Advanced Deep Learning with Keras`). Coloca la salida de `related_instance` en un elemento `<em>` para una mejor legibilidad.
9. En el archivo `urls.py` de la aplicación `reviews`, agrega mapas de URL a la vista `review_edit`. Las URLs `/books/` y `/books/<pk>/` ya se han configurado. Agrega `/books/<book_pk>/reviews/new/` para crear una reseña y `/books/<book_pk>/reviews/<review_pk>/` para editar una reseña. Asegúrate de darles nombres, como `review_create` y `review_edit`.
10. Dentro de la plantilla `book_detail.html`, agrega enlaces en los que un usuario pueda hacer clic para crear o editar una reseña. Agrega un enlace dentro del bloque `content`, justo antes de la etiqueta de plantilla de cierre `endblock`. Debe usar la etiqueta de plantilla `url` para vincular a la vista `review_edit` cuando esté en modo de creación. Además, usa el atributo `class="btn btn-primary"` para hacer que el enlace se muestre como un botón de Bootstrap. El texto del enlace debe ser `Add Review`.
11. Finalmente, agrega un enlace para editar una reseña, dentro del bucle `for` que itera sobre las reseñas del libro. Después de todas las instancias de `<span>` con clase `text-info`, agrega un enlace a la vista `review_edit` usando la etiqueta de plantilla `url`. Deberás proporcionar `book.pk` y `review.pk` como argumentos. El texto del enlace debe ser `Edit Review`.

Cuando hayas terminado, la página de comentarios de reseñas debería verse como se muestra en la Figura 7.27:

*Figura 7.27 – La página Book Details con el botón Add Review agregado*

Verás el botón **Add Review**. Al hacer clic en él, accederás a la página **New Review**, que debería verse como se muestra en la Figura 7.28:

*Figura 7.28 – La página New Review*

Ingresa algunos detalles en el formulario y haz clic en **Create**. Serás redirigido a la página de detalles del libro. Deberías ver el mensaje de éxito y tu reseña, como se muestra en la Figura 7.29:

*Figura 7.29 – La página Book Details con una reseña agregada*

También verás el enlace **Edit Review**. Si haces clic en él, accederás a un formulario que se ha completado previamente con los datos de tu reseña (consulta la Figura 7.30):

*Figura 7.30 – Formulario de reseña al editar una reseña*

Después de guardar una reseña existente, deberías ver que la fecha *Modified on* se ha actualizado en la página de detalles del libro (Figura 7.31):

*Figura 7.31 – El campo de fecha Modified on ahora ha sido completado*

---

### Sección: Resumen

Este capítulo proporcionó una inmersión profunda en los formularios. Vimos cómo mejorar los formularios de Django con validación personalizada, reglas avanzadas para limpiar datos y validar campos. También vimos cómo los métodos de limpieza personalizados pueden transformar los datos que obtenemos de los formularios. Una característica agradable que vimos que se puede agregar a los formularios es la capacidad de establecer valores iniciales y marcadores de posición en los campos para que el usuario no tenga que completarlos.

Luego vimos cómo usar la clase `ModelForm` para crear automáticamente un formulario a partir de un modelo de Django. Vimos cómo mostrar solo algunos campos al usuario y cómo aplicar reglas de validación de formularios personalizadas a `ModelForm`. También vimos cómo Django puede guardar automáticamente la instancia de modelo nueva o actualizada en la base de datos dentro de la vista. En las actividades de este capítulo, mejoramos Bookr aún más al agregar formularios para crear y editar editoriales y enviar reseñas.

En el próximo capítulo, continuaremos con el tema del envío de entradas del usuario y analizaremos cómo maneja Django la carga y descarga de archivos.
