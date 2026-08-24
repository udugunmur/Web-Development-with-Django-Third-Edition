# Parte 2: Creación de aplicaciones web con Django

## Capítulo 10: Administrador avanzado de Django y personalizaciones

Supongamos que queremos personalizar la página principal del sitio de administración de una gran organización. Queremos mostrar el estado de salud de los diferentes sistemas de la organización y ver cualquier alerta activa de alta prioridad. Si este fuera un sitio web interno construido sobre Django, necesitaríamos personalizarlo. Agregar este tipo de funcionalidades requerirá que los desarrolladores del equipo de TI personalicen el panel de administración predeterminado y creen su propio módulo `AdminSite` personalizado, el cual renderizará una página de índice diferente en comparación con la proporcionada por el sitio de administración predeterminado. Afortunadamente, Django facilita este tipo de personalizaciones.

En este capítulo, veremos cómo podemos aprovechar el framework de Django y su extensibilidad para personalizar la interfaz de administración predeterminada de Django. No solo aprenderemos cómo hacer que la interfaz sea más personal; también aprenderemos cómo podemos controlar los diferentes aspectos del sitio de administración para que Django cargue un sitio de administración personalizado, en lugar del que viene con el framework predeterminado. Dicha personalización puede ser útil cuando queremos introducir características en el sitio de administración que no están presentes de forma predeterminada.

Este capítulo se basa en las habilidades que practicamos en el Capítulo 4 (*Introducción al Administrador de Django*). Para recapitular, aprendimos a usar el sitio de administración de Django para tomar el control de la administración y la autorización de nuestra aplicación Bookr. También aprendimos cómo registrar modelos para leer y editar su contenido, y cómo personalizar la interfaz de administración de Django usando las propiedades de `admin.site`. Terminaremos aprendiendo cómo configurar un servidor de correo electrónico con Django y el uso del framework Django Tasks.

En este capítulo se cubren los siguientes temas:
- Personalización del modelo de usuario
- Mejoras de formularios con atributos de `ModelAdmin`
- Adición de vistas a la aplicación de administración de Django
- Configuración de un servidor de correo electrónico con Django
- Automatización de la administración con Django Tasks

Ahora, ampliemos aún más nuestro conocimiento observando cómo podemos comenzar a personalizar el sitio de administración utilizando el módulo `AdminSite` de Django para agregar nuevas y potentes funcionalidades al portal de administración de nuestra aplicación web.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter10` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Personalización del sitio de administración

Django, como framework web, ofrece muchas opciones de personalización para crear aplicaciones web. Utilizaremos esta misma libertad proporcionada por Django cuando trabajemos en la creación de la aplicación de administración para nuestro proyecto.

En el Capítulo 4 (*Introducción al Administrador de Django*), vimos cómo podemos usar las propiedades de `admin.site` para personalizar los elementos de la interfaz de administración de Django. Sin embargo, ¿qué pasa si requerimos más control sobre cómo se comporta nuestro sitio de administración? Por ejemplo, ¿qué pasaría si quisiéramos mostrar una plantilla personalizada para la página de inicio de sesión o la página de cierre de sesión cuando el usuario ingresa al panel de administración de Bookr? En este caso, las propiedades de `admin.site` proporcionadas podrían no ser suficientes y necesitaremos crear personalizaciones que puedan extender el comportamiento del sitio de administración predeterminado.

---

### Sección: Un modelo de usuario personalizado

El modelo de usuario que se proporciona con la aplicación de autenticación viene con características muy generales. A menudo ocurre que queremos retener información más específica sobre un usuario, como la dirección o el número de teléfono, o es posible que deseemos implementar un patrón de autenticación diferente al enfoque de nombre de usuario/contraseña. Estas son consideraciones de diseño importantes. Es mejor considerarlo ahora antes de acumular una gran cantidad de datos de usuario y necesitar reemplazar el modelo de usuario en una fecha posterior.

Veremos cómo personalizar el modelo de usuario ya sea extendiendo el modelo de usuario o subclasificando uno de sus modelos abstractos heredados.

#### Extender el modelo de usuario

El primer enfoque consiste en agregar un modelo para contener información complementaria sobre nuestros usuarios. Esto no implica modificaciones a la aplicación de autenticación (`auth`) ni a su modelo de usuario. En su lugar, se crea un modelo en una relación de 1 a 1 con el modelo de usuario.

Para ilustrar esto, agregaremos un modelo `ReviewerProfile` al archivo `reviews/models.py`. Este modelo contiene detalles adicionales sobre nuestro usuario, como su libro favorito y su foto de perfil. Para definir campos en modelos, necesitamos usar el atributo `settings.AUTH_USER_MODEL`.

Crearemos un modelo `ReviewerProfile`:

```python
from django.contrib.auth.models import User
…

class ReviewerProfile(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    location = models.CharField(max_length=100)
    favourite_book = models.ForeignKey(
        Book, null=True, on_delete=models.SET_NULL,
        help_text="Your favourite book (if it is present in Bookr)")
    profile_photo = models.ImageField(
        null=True, blank=True, upload_to="profile_photos/")
```

Ten en cuenta que hemos agregado un atributo `favourite_book` para que un revisor pueda seleccionar opcionalmente su libro favorito de la aplicación `reviews` si está presente. También hemos agregado un atributo `profile_photo` para que el usuario pueda agregar una imagen de visualización. Hemos configurado `on_delete=models.SET_NULL` porque queremos asegurarnos de que la eliminación del registro de ese libro provoque que se desactive el atributo `favourite_book` en lugar de eliminar `ReviewerProfile`.

Actualmente, estamos utilizando el modelo de usuario de `auth` con nuestra aplicación, pero es posible que deseemos preparar el código para el futuro de modo que nuestra aplicación no haga esta suposición. Esto es particularmente importante si la aplicación está destinada a ser reutilizada en otros proyectos donde no se pueden hacer suposiciones sobre el modelo de usuario.

Para hacerlo, podemos usar la función `get_user_model` de la aplicación `auth` para determinar qué modelo de usuario se está utilizando en el proyecto.

Como vimos en los capítulos anteriores, el parámetro `on_delete=models.CASCADE` garantiza que cuando se elimine el objeto `User`, `ReviewerProfile` también se elimine. De manera similar, para garantizar que `ReviewerProfile` se cree después de que se cree un objeto `User`, podemos emplear una señal de Django fácil de codificar. Esto se podría escribir en `signals.py`, que luego se puede importar en la función `ready` de la clase `AppConfig` en `apps.py`:

```python
from django.dispatch import receiver
from django.db.models.signals import post_save
…

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_revierwer_profile(sender, created, instance, **kwargs):
    if created:
        profile = ReviewerProfile(user=instance)
        profile.save()
```

Aquí, estamos usando el argumento `sender=settings.AUTH_USER_MODEL` para el decorador `receiver`. Esto se debe a que no presumimos que el modelo de usuario de `auth` se empleará en la aplicación.

Una ligera desventaja de una extensión 1 a 1 de la clase `User` es que al usar dos modelos para los datos de usuario y de perfil, ya no tenemos un único lugar en la aplicación de administración para mantener estos datos.

#### Subclasificar AbstractUser

Hay situaciones en las que es preferible reemplazar el usuario en la aplicación `auth` por uno más apropiado. Esto puede incluir el caso en el que los campos principales utilizados en la autenticación sean diferentes del nombre de usuario y la contraseña, o donde necesitemos anular el mecanismo de contraseñas o las estructuras de permisos.

Se recomienda hacer esto al principio del desarrollo si es evidente que existe tal necesidad, ya que no queremos crear escenarios de migración complicados si no es necesario.

Primero, debemos ver con más detalle cómo se implementa el modelo de usuario en la aplicación `auth`.

El modelo de usuario de la aplicación `auth` se implementa en `django/contrib/auth/models.py`. `User` es una subclase de `AbstractUser`, que a su vez subclasifica `AbstractBaseUser` (ubicado en `django/contrib/auth/base_user.py`) y `PermissionMixin`.

La Figura 10.1 describe los atributos de estas clases y de dónde provienen. Es una guía útil para establecer cómo abordar la anulación del modelo de usuario sin incurrir en ninguna complejidad de codificación innecesaria.

Antes de implementar tus propios protocolos alternativos de usuario o autenticación, vale la pena revisar qué aplicaciones existentes ya se han desarrollado para estas necesidades. Existen muchas aplicaciones para satisfacer necesidades comunes, como implementaciones de clientes OAuth2, así como aplicaciones establecidas de Django. Es crucial revisar a fondo qué opciones están disponibles antes de diseñar tu propia personalización, ya que adoptar una aplicación establecida ahorrará una gran cantidad de duplicación innecesaria de esfuerzos.

Si comienzas a personalizar la subclase `User`, serás responsable de modificarla cuando se actualice el módulo de Autenticación. También serás responsable de asegurarte de que tus personalizaciones no bloqueen las nuevas correcciones de seguridad de Django.

*Figura 10.1: La herencia del modelo de usuario*

Veamos tres escenarios diferentes para subclasificar `AbstractUser` o `AbstractBaseUser`.

##### Escenario 1: Adición de campos adicionales a la clase User
En este caso, subclasificaríamos `AbstractUser` e incluiríamos los campos adicionales.

A veces, los campos que definen al usuario son insuficientes por razones culturales o profesionales. En algunas culturas, la distinción entre nombre y apellido (*first name/last name*) no es una forma natural de ver el nombre de una persona. Por ejemplo, en muchas partes de Rusia y Europa del Este, se utiliza un segundo nombre derivado del nombre de pila de la madre o del padre, un matronímico o patronímico, respectivamente. Puede tener más sentido almacenar esto en un campo separado en el modelo de usuario.

Del mismo modo, si tu proyecto se utilizará para generar correspondencia formal, puede tener sentido almacenar los tratamientos de cortesía Sra., Srta., Sr., etc., en un campo de tratamiento separado. Los títulos profesionales como Dr., Prof. y Coronel pueden justificar su almacenamiento en un campo separado de los tratamientos de cortesía.

Si se utiliza un número de teléfono como parte esencial del proceso de registro del usuario y está vinculado de forma exclusiva a la identidad del usuario, también puede valer la pena incluirlo en esta clase. De lo contrario, considera agregar un modelo de teléfono separado que esté en una relación de varios a uno con el método de perfil de usuario.

##### Escenario 2: Reemplazo de la clave de nombre de usuario (Username)
Es un requisito frecuente implementar la autenticación de usuarios que consiste en una dirección de correo electrónico y una contraseña, y prescindir de un nombre de usuario adicional.

Esto implicaría subclasificar `AbstractUser` y reemplazar los campos de nombre de usuario y validación de nombre de usuario. Es posible que sea necesario diseñar nuevos métodos relacionados con el inicio de sesión.

##### Escenario 3: Modificación de la contraseña
Si bien podemos modificar la forma en que la contraseña se almacena en la base de datos cambiando el middleware, cambiar la definición del campo de contraseña es una propuesta completamente diferente.

Si, por alguna razón, necesitáramos reemplazar el campo de contraseña de 128 caracteres o reemplazar la autenticación por contraseña con un protocolo novedoso, entonces tendría sentido anular `AbstractBaseUser`. Este enfoque agregaría mucha complejidad y no se recomienda para principiantes.

Los escenarios presentados no son mutuamente excluyentes. Puede haber situaciones en las que el mejor curso de acción sea incluir una extensión de usuario basada en una aplicación, como el `ReviewerProfile` mostrado, así como subclasificar `AbstractUser`.

#### Pasos para reemplazar el modelo de usuario de autenticación

La siguiente lista de pasos se utiliza para crear un modelo de usuario personalizado. Implementaremos esto en el siguiente ejercicio:
1. Crear una nueva aplicación para el modelo de usuario personalizado.
2. Definir una clase de modelo de usuario personalizada, heredando de `User`, `AbstractUser` o `AbstractBaseUser` según lo amerite el caso, en función de los escenarios descritos anteriormente.
3. Incluir la aplicación en `INSTALLED_APPS`.
4. Establecer este modelo como `AUTH_USER_MODEL` en `<proyecto>/settings.py`.
5. Asegurarse de que cualquier referencia al modelo de usuario se reemplace con referencias a `settings.AUTH_USER_MODEL` o `get_user_model`.
6. Reemplazar cualquier plantilla o crear las extensiones apropiadas que queden invalidadas por los cambios.
7. Registrar el modelo con la aplicación de administración.
8. Crear y ejecutar un script de migración para estos cambios de modelo.

#### Ejercicio 10.01: Reemplazo de User con un modelo TitledUser

En este ejercicio, crearemos una clase `TitledUser` que incluye un campo de tratamiento de cortesía (*honorific*) y de título para cada usuario.

Estas opciones se definirán como `CharField` con listas de opciones fijas. El método `TitledUser.get_full_name` anulará `AbstractUser.get_full_name`. Seguirá la siguiente lógica:
- Si hay un título presente, devuelve una cadena de `"<title> <firstname> <secondname>"`.
- Si hay un tratamiento de cortesía presente, devuelve una cadena de `"<honorific> <firstname> <secondname>"`.
- De lo contrario, devuelve una cadena de `"<firstname> <secondname>"`.

1. Crea una aplicación separada llamada `titledusers`:
   ```bash
   python manage.py startapp titledusers
   ```
2. Subclasifica el modelo `User`:
   ```python
   class TitledUser(User):
   ```
3. Incluye una clase `Meta` con los atributos `verbose_name` y `verbose_name_plural` dentro de la clase `TitledUser`:
   ```python
       class Meta:
           verbose_name = "titled user"
           verbose_name_plural = "titled users"
   ```
4. Define las clases `TextChoices` para `TitleChoices` y `HonorificChoices` dentro de la clase `TitledUser`:
   ```python
       class TitleChoices(models.TextChoices):
           PROF = "PROF", "Prof"
           DR = "DR", "Dr"

       class HonorificChoices(models.TextChoices):
           MR = "MR", "Mr"
           MISS = "MISS", "Miss"
           MRS = "MRS", "Mrs"
           MS = "MS", "MS"
   ```
5. Agrega los dos campos adicionales:
   ```python
       title = models.CharField(verbose_name="Title", blank=True, choices=TitleChoices.choices)
       honorific = models.CharField(
           verbose_name="Honorific", blank=True, choices=HonorificChoices.choices)
   ```
6. Agrega el método `get_full_name`:
   ```python
       def get_full_name(self):
           if self.title:
               return f'{self.title} ' f'{self.firstname} {self.secondname}'
           elif self.honorific:
               return f'{self.honorific} ' f'{self.firstname} {self.secondname}'
           else:
               return f'{self.firstname} ' f'{self.secondname}'
   ```
7. Después de los pasos anteriores, nuestro modelo en `titledusers/models.py` se verá así:
   ```python
   from django.db import models
   from django.contrib.auth.models import User

   class TitledUser(User):
       class Meta:
           verbose_name = _('titled user')
           verbose_name_plural = _('titled users')

       class TitleChoices(models.TextChoices):
           PROF = "PROF", "Prof"
           DR = "DR", "Dr"

       class HonorificChoices(models.TextChoices):
           MR = "MR", "Mr"
           MISS = "MISS", "Miss"
           MRS = "MRS", "Mrs"
           MS = "MS", "MS"

       title = models.CharField(verbose_name="Title", blank=True, choices=TitleChoices.choices)
       honorific = models.CharField(verbose_name="Honorific", blank=True, choices=HonorificChoices.choices)

       def get_full_name(self):
           if self.title:
               return f'{self.title} ' f'{self.firstname} {self.secondname}'
           elif self.honorific:
               return f'{self.honorific} ' f'{self.firstname} {self.secondname}'
           else:
               return f'{self.firstname} ' f'{self.secondname}'
   ```
8. Incluye la nueva aplicación en la lista de aplicaciones instaladas y establece `AUTH_USER_MODEL`:
   ```python
   INSTALLED_APPS = [
       …
       'reviews',
       'titledusers',
   ]
   AUTH_USER_MODEL = 'titledusers.TitledUser'
   ```
9. Asegúrate de que cualquier referencia al modelo de usuario se reemplace con referencias a `settings.AUTH_USER_MODEL` o `get_user_model`.
   Una búsqueda en el código revela que estos solo ocurren en `reviews/management/commands/loadcsv.py` y en el código original de `reviews/models.py`.
10. Reemplaza cualquier plantilla o crea las extensiones apropiadas que queden invalidadas por los cambios. Actualmente, dado que estos cambios no afectan ningún comportamiento de nombre de usuario, correo electrónico o inicio de sesión, no necesitamos reemplazar ninguna plantilla.
11. Registra el modelo con la aplicación de administración.
    Como mínimo, el archivo `titledusers/admin.py` se verá así:
    ```python
    from django.contrib import admin

    # Register your models here.
    admin.site.register(TitledUser)
    ```
12. Crea y ejecuta un script de migración para estos cambios de modelo.

Ahora hemos creado un usuario personalizado, `TitledUser`.

#### Actividad 10.01: Implementación del modelo de perfil de revisor

El objetivo de esta actividad es seguir los pasos necesarios para implementar el modelo `ReviewerProfile` que se enumeró en la sección *Extender el modelo de usuario*.

1. Como mantenemos `auth.User` como el modelo de autenticación, no tenemos que crear un nuevo usuario. Sin embargo, debemos definir esto explícitamente en el archivo de configuración del proyecto.
2. Reemplaza cualquier plantilla o crea extensiones apropiadas que queden invalidadas por los cambios. En este punto, agregaremos plantillas de administración personalizadas para el perfil y el registro. Las plantillas existentes se modificarán para incluir el formulario `ReviewerProfile`.
3. Registra el modelo con la aplicación de administración en `admin.py` importando el modelo y luego registrándolo con `admin.site.register(ReviewerProfile)`.
4. Crea y ejecuta un script de migración para estos cambios.

El resultado de esta actividad es que hemos integrado nuestro modelo `ReviewerProfile` en el proyecto Bookr y podemos modificar los perfiles de revisor (`ReviewerProfile`) a través de la interfaz de administración.

*Figura 10.2: Edición del perfil de revisor en la aplicación de administración*

---

### Sección: Mejoras de formularios con atributos de ModelAdmin

Esta sección aborda algunos de los atributos adicionales de la clase `ModelAdmin`. Primero encontramos la clase `ModelAdmin` en el Capítulo 4 (*Introducción al Administrador de Django*) y aprendimos cómo subclasificarla para producir personalizaciones en nuestra aplicación de administración.

#### filter_horizontal / filter_vertical

Recuerda que en la página de cambio de usuario había un widget que constaba de dos listas que se utilizaban para asignar permisos al usuario. Este tipo de selección múltiple es más fácil de usar al navegar por listas largas de opciones que un menú desplegable típico. A medida que las opciones seleccionadas pasan de la lista de la izquierda a la lista de la derecha, es fácil realizar un seguimiento de qué opciones se han seleccionado.

Si queremos asignar este tipo de widget a cualquiera de nuestros campos de selección múltiple, podemos configurar el atributo `filter_horizontal` en la subclase `ModelAdmin`. El atributo `filter_horizontal` toma una tupla de cadenas como argumentos. Las cadenas son referencias a campos en el modelo o campos en claves foráneas.

También existe la opción de mostrar las dos listas una encima de la otra.

Si no queremos mostrar las dos listas una al lado de la otra, tenemos la opción de mostrarlas arriba y abajo. Este podría ser el caso si queremos que la selección se muestre en un teléfono móvil. En este caso, agregamos la cadena del campo al atributo `filter_vertical`.

#### Formularios en línea (Inline forms)

A veces también queremos representar modelos que están asociados a través de una relación `ManyToMany` o `OneToMany`.

Por ejemplo, en la página de usuario, tenemos una lista de grupos en los que se encuentra el usuario y la opción de asignarlo a grupos adicionales. Esto se logra mediante el atributo `inlines`.

Hay dos formatos de elementos de formulario en línea: `TabularInline` y `StackedInline`.

```python
class ContributorInline(admin.StackedInline):
    model = BookContributor
    extra = 0

class BookAdmin(admin.ModelAdmin):
    date_hierarchy = 'publication_date'
    list_display = ('title', 'isbn')
    list_filter = ('publisher', 'publication_date')
    search_fields = ('title', 'isbn', 'publisher__name')
    inlines = (ContributorInline,)
```

*Figura 10.3: Formularios StackedInline*

Tal vez tener esos formularios en línea adicionales en blanco parezca demasiado recargado y confuso. Se pueden eliminar utilizando el atributo `extra`:

```python
class ContributorInline(admin.StackedInline):
    model = BookContributor
    extra = 0
```

*Figura 10.4: Formularios StackedInline sin formularios en blanco*

Alternativamente, si queremos que el formulario principal imponga un número máximo de formularios en línea, podemos agregar el atributo `max_num`. Observa que ya no habrá un enlace *+ Add another Book contributor* en la parte inferior de los formularios en línea. Por supuesto, establecer un número máximo de colaboradores no tendría sentido para los libros, dado que pueden tener múltiples autores, editores y otros colaboradores, pero hay situaciones en las que esta restricción de formulario es perfectamente razonable.

```python
class ContributorInline(admin.StackedInline):
    model = BookContributor
    extra = 0
    max_num = 2
```

*Figura 10.5: Formularios StackedInline con una restricción máxima*

Este formato `StackedInline` enfatiza que los elementos en línea son elementos separados. A veces, podríamos preferir el renderizado `TabularInline`, que es más compacto y organiza los campos en columnas. Esto es particularmente útil para datos numéricos como líneas de facturas.

```python
class ContributorInline(admin.TabularInline):
    model = BookContributor
    extra = 0
```

*Figura 10.6: Formularios TabularInline*

Ten cuidado al agregar campos en línea (o campos `filter_horizontal` / `filter_vertical`) a un formulario de administración, ya que esto aumentará la complejidad de la consulta SQL subyacente utilizada para generar la página. Puede tener repercusiones en el rendimiento si tienes conjuntos de datos grandes.

#### radio_fields

De forma predeterminada, la aplicación de administración representa las opciones o las relaciones de clave foránea como menús desplegables. A veces, los botones de opción (*radio buttons*) son una forma adecuada de representar una lista corta de opciones discretas. Al usar el atributo `radio_fields`, representamos algunos campos como botones de opción.

Podríamos considerar renderizar el atributo `role` de nuestra clase `BookContributor` usando `radio_fields`. Necesitamos especificar `radio_fields` como un diccionario, especificando el nombre del campo como clave y la orientación (ya sea `admin.HORIZONTAL` o `admin.VERTICAL`) como valor:

```python
class BookContributorAdmin(admin.ModelAdmin):
    role = {'rating': admin.HORIZONTAL}

admin.site.register(BookContributor, BookContributorAdmin)
```

Esta configuración funciona bien cuando hay una pequeña lista de opciones. De lo contrario, el diseño puede parecer algo confuso o desordenado.

#### readonly_fields

A veces queremos proteger campos específicos para que no se modifiquen en las interfaces de administración. Podemos designarlos como de solo lectura mediante el atributo `readonly_fields`. Por ejemplo, es posible que desees establecer los campos que muestran la fecha de creación y la fecha de modificación de un registro como de solo lectura. La siguiente línea de código muestra cómo se puede hacer esto usando `readonly_fields`:

```python
readonly_fields = ('date_created', 'date_modified')
```

Por ejemplo, en esta situación, hemos especificado una tupla de campos de solo lectura, pero hay algunos campos que solo son de solo lectura cuando se modifica el objeto. El uso del condicional es una forma común de lograr esto, y revisaremos esta lógica en los ejercicios y actividades.

#### get_readonly_fields

A veces necesitamos asignar criterios más dinámicos para especificar cuándo un campo es de solo lectura. Por esta razón, existe un método `get_readonly_fields` que nos brinda la flexibilidad de incluir criterios como los permisos de un usuario o si el objeto se está creando o actualizando.

Supongamos que solo queríamos que los campos `creator` y `book` de la reseña fueran editables al momento de la creación, pero campos de solo lectura cuando se edita la reseña. Podríamos lograr esto especificando una condición `if obj is None`:

```python
def get_readonly_fields(self, request, obj=None):
    if obj is None or request.user.is_superuser:
        return self.readonly_fields
    return ['creator', 'book',] + self.readonly_fields
```

#### list_editable

Hay ocasiones en las que visualizas una lista de objetos y resultaría útil poder editar los campos mostrados en la lista de cambios sin tener que editar una página de cambio para cada registro individual.

Afortunadamente, el objeto `ModelAdmin` proporciona una opción para esto. Podemos asignar campos al atributo `list_editable`.

#### Ejercicio 10.02: Personalizaciones detalladas de la interfaz de administración

En este ejercicio, utilizaremos algunos de los atributos de `ModelAdmin` que hemos aprendido para llevar a cabo algunas personalizaciones detalladas de la interfaz de administración. En nuestra modificación al modelo `Review`, a las reseñas se les puede asignar una clasificación por estrellas. Nos gustaría modificar la visualización de las reseñas en su respectivo formulario de administración para que aparezcan como campos de radio verticales:

1. Incluye la calificación (`rating`) en el `list_display` de `ReviewsAdmin`. Ten en cuenta que la representación de opciones de `CharField` aparece en `list_display`.
2. Agrega un método llamado `has_profile_photo` a `ReviewerProfile`. Debe devolver una cadena vacía si no se ha subido ninguna foto de perfil o un emoji apropiado si hay una disponible. Podríamos usar un emoji de cámara, paleta de pintura o imagen enmarcada (Unicode `U+1F5BC`, HTML `🖼`).
3. Para mostrar HTML formateado en `list_display` en lugar de escaparlo, necesitamos usar la función `format_html` en `reviews/admin.py`:
   ```python
   from django.utils.html import format_html
   ```
4. Crea una clase `ReviewerProfileAdmin` en `reviews/admin.py`, luego ajusta el registro del modelo para incluir la clase de administración personalizada:
   ```python
   def has_profile_photo(self, obj):
       return obj.profile_photo and format_html(
           '<h1>🖼 </h1>') or ''
   ```
5. Las listas de cambios en los perfiles de revisor deben mostrar `user`, `location` y `has_profile_photo`.
6. Haz que el campo `location` de `ReviewerProfile` sea editable desde una página de visualización de administración.
7. El archivo `reviews/admin.py` ahora debería verse así:
   ```python
   from django.utils.html import format_html

   class ReviewerProfileAdmin(admin.ModelAdmin):
       list_display = ('user', 'location', 'has_profile_photo')
       list_editable= ('location',)

       def has_profile_photo(self, obj):
           return obj.profile_photo and format_html(
               '<h1>🖼</h1>') or ''
   …
   admin.site.register(ReviewerProfile, ReviewerProfileAdmin)
   ```
8. Ahora, una vez que hayas creado algunos perfiles de revisor, la lista de cambios se verá como la Figura 10.7:
   *Figura 10.7: La lista de cambios del perfil de revisor*

Hemos visto cómo podemos mejorar el diseño del formulario en la interfaz de administración. Ahora examinaremos las personalizaciones de paginación, búsqueda y filtros.

#### Personalizaciones de paginación

Hasta ahora, hemos estado tratando con conjuntos de datos relativamente pequeños en nuestro ejemplo, pero a medida que se acumulan nuestros nuevos datos, tenemos que renderizar más y más información en nuestras páginas. Afortunadamente, la aplicación de administración implementa opciones de paginación para que podamos devolver pequeños segmentos de datos recuperados a la vez. Esto es necesario para el rendimiento, ya que los conjuntos de consultas grandes pueden provocar problemas de rendimiento en la base de datos y ralentizarán la capacidad de respuesta del servidor web.

De forma predeterminada, los registros se paginan en conjuntos de 100. Para muchas aplicaciones, preferiríamos tener un conjunto más pequeño que no requiera desplazarse por la página para navegar.

Podemos modificar esto usando el atributo `list_per_page` de la clase `ModelAdmin`:

```python
class ReviewAdmin(admin.ModelAdmin):
    list_display = ('creator', 'book', 'rating')
    list_per_page = 10
```

Podemos ver en la Figura 10.8 que los registros ahora están paginados en 10 objetos por página (puedes probar esto con 5 elementos):

*Figura 10.8: El marco de resultados de calificaciones*

En la parte inferior del conjunto de resultados, hay un enlace *Show all* (Mostrar todo). Al hacer clic en este enlace, se devolverán todos los registros sin la paginación establecida. De forma predeterminada, el enlace *Show all* solo aparecerá si hay 200 registros o menos para devolver. Al utilizar el atributo `list_max_show_all`, podemos modificar este número.

#### Personalizaciones de búsqueda y filtrado

Revisamos la capacidad de filtrado de búsqueda de Django en el Capítulo 4 (*Introducción al Administrador de Django*). Hay opciones adicionales disponibles para personalizar el rendimiento y la apariencia del filtro.

De forma predeterminada, cuando recuperamos un conjunto de resultados de una operación de filtro, se muestra el recuento de registros del conjunto de resultados. Lo que quizás no sea evidente es que para mostrar este número, Django tiene que realizar una consulta secundaria a la base de datos después de la consulta principal que recupera el conjunto de datos.

Como la paginación es una forma eficiente de recuperar una pequeña porción de los resultados a la vez, esta segunda consulta que devuelve el recuento de registros a menudo puede ser la que crea problemas de rendimiento para el sitio, especialmente con conjuntos de registros grandes o conjuntos de datos que involucran múltiples relaciones con otros modelos. De forma predeterminada, `show_full_result_count = True`. Si encuentras que Django tarda demasiado en recuperar y renderizar tu conjunto de resultados filtrados, puede valer la pena configurar `show_full_result_count = False` y ver si esto mejora el rendimiento del filtro.

De forma predeterminada, después de editar un objeto que recuperamos de un conjunto de resultados filtrados, se nos devuelve al conjunto de filtros. Por lo general, este es el comportamiento deseado, aunque a veces es un flujo de trabajo más natural volver a la página de índice del modelo sin ninguna configuración de filtro. Si este es el caso, es apropiado especificar la opción `preserve_filters = False` en nuestra subclase `ModelAdmin`.

Estos comportamientos funcionarán igual si el conjunto de resultados se deriva de una consulta de la barra de búsqueda o de una selección de filtro:

```python
class ReviewAdmin(admin.ModelAdmin):
    …
    list_filter = ('content', 'creator__username', 'book__title')
    search_fields = ('creator__username',)
    show_full_result_count = False
    preserve_filters = False
```

#### Actividad 10.02: Consolidación de paginación, búsqueda y filtros en Reviews

Esta actividad consolidará nuestro conocimiento sobre las personalizaciones de paginación, búsqueda y filtros:

1. Estandariza la paginación en las páginas de Reviews para que todos los objetos se devuelvan con un máximo de 10 registros.
2. Elimina el enlace *Show all* para conjuntos de resultados que tengan más de 50 registros.
3. Agrega una barra de búsqueda para la página de administración de Review. Permite buscar por título de libro (`Book.title`), nombre de usuario del creador (`Creator.username`) y contenido de la reseña (`Review.content`).
4. Agrega un método a `ReviewAdmin` llamado `rating_stars`. Devolverá una cantidad de estrellas (carácter Unicode `U+2606`) según el valor de `rating`. Utiliza este método para el formato de visualización de las calificaciones de las reseñas.
5. Agrega un filtro en la página de administración de reseñas para la calificación de la reseña (`rating`).
6. Debido a que el modelo `Review` tiene relaciones de clave foránea con `Book` y `Contributor`, la estructura de la consulta de recuento de resultados bien puede causar problemas de rendimiento para conjuntos de consultas grandes. Modifica `ModelAdmin` para la clase `Review` de modo que no se muestre el recuento completo de resultados.

Al completar esta actividad, habremos completado nuestras personalizaciones de las páginas de administración de la aplicación `reviews`.

La página de lista de cambios modificada para Reviews se verá como la Figura 10.9:

*Figura 10.9: Búsqueda y filtros en la lista de cambios de reseñas*

---

### Sección: Adición de vistas al sitio de administración

Al igual que las aplicaciones generales dentro de Django, que pueden tener múltiples vistas asociadas a ellas, Django también permite a los desarrolladores agregar vistas personalizadas al sitio de administración, lo que le permite a un desarrollador aumentar el alcance de lo que puede hacer la interfaz del sitio de administración.

La capacidad de agregar tus propias vistas al sitio de administración proporciona una gran extensibilidad al panel de administración del sitio web, que se puede aprovechar para varios casos de uso adicionales. Por ejemplo, como comentamos al principio del capítulo, el equipo de TI de una gran organización puede agregar una vista personalizada al sitio de administración, que luego se puede utilizar para monitorear el estado de salud de los diferentes sistemas de TI en la organización, así como para brindar al equipo de TI la capacidad de ver rápidamente cualquier alerta urgente que deba abordarse.

Ahora, la siguiente pregunta que debemos responder es: ¿Cómo podemos agregar una vista personalizada al sitio de administración?

Resulta que agregar una nueva vista dentro de la plantilla de administración es bastante fácil y sigue el mismo enfoque que usamos al crear vistas para nuestras aplicaciones, con algunas modificaciones menores. Entonces, veamos cómo podemos agregar una nueva vista a nuestro panel de administración de Django.

#### Creación de la función de vista

El primer paso para agregar una nueva vista a la aplicación Django es crear una función de vista que implemente la lógica para manejar la vista. En los capítulos anteriores, creamos funciones de vista dentro de un archivo separado conocido como `views.py`, que se usaba para contener todas nuestras vistas basadas en métodos y basadas en clases.

En el contexto de agregar una nueva vista al panel de administración de Django, para crear una nueva vista, debemos definir una nueva función de vista dentro de nuestra clase `AdminSite` personalizada. Por ejemplo, para agregar una nueva vista que renderice una página que muestre el estado de salud de los diferentes sistemas de TI dentro de una organización, podemos crear una nueva función de vista llamada `system_health_dashboard()` dentro de nuestra implementación de clase `AdminSite` personalizada, como se muestra en el siguiente fragmento de código:

```python
class SysAdminSite(admin.AdminSite):
    def system_health_dashboard(self, request):
        # View function logic
```

Dentro de la función de vista, podemos realizar cualquier operación que deseemos para generar una vista y, finalmente, usar esa respuesta para renderizar una plantilla. Dentro de esta función de vista, hay algunas partes importantes de lógica que debemos asegurarnos de que se implementen correctamente.

La primera es establecer la propiedad `current_app` para el campo de solicitud (`request`) dentro de la función de vista. Esto es necesario para permitir que el solucionador de URLs de Django dentro de las plantillas resuelva correctamente las funciones de vista para una aplicación. Para establecer este valor dentro de la función de vista personalizada que acabamos de crear, debemos establecer la propiedad `current_app`, como se muestra en el siguiente fragmento de código:

```python
request.current_app = self.name
```

El campo `self.name` es completado automáticamente por la clase `AdminSite` de Django y no necesitamos inicializarlo explícitamente. Con esto, nuestra implementación mínima de vista personalizada aparecerá como se muestra en el siguiente fragmento de código:

```python
class SysAdminSite(admin.AdminSite):
    def system_health_dashboard(self, request):
        request.current_app = self.name
        # View function logic
```

#### Acceso a variables de plantilla comunes

Al crear una función de vista personalizada, es posible que deseemos acceder a las variables de plantilla comunes, como `site_header` y `site_title`, para representarlas correctamente en la plantilla asociada con nuestra función de vista. Resulta que esto es bastante fácil de lograr con el uso del método `each_context()` proporcionado por la clase `AdminSite`.

El método `each_context()` de la clase `AdminSite` toma un único parámetro, `request`, que es el contexto de la solicitud actual, y devuelve las variables de plantilla que se insertarán en todas las plantillas del sitio de administración.

Por ejemplo, si quisiéramos acceder a las variables de plantilla dentro de nuestra función de vista personalizada, podríamos implementar un código similar al siguiente fragmento de código:

```python
def system_health_dashboard(self, request):
    request.current_app = self.name
    context = self.each_context(request)
    # view function logic
```

El valor devuelto por el método `each_context()` es un diccionario que contiene el nombre de la variable y el valor asociado.

#### Asignación de URLs para la vista personalizada

Una vez definida la función de vista, el siguiente paso consiste en asignar esta función de vista a una URL para que un usuario pueda acceder a ella o permitir que las otras vistas se vinculen a ella. Para las vistas definidas dentro de `AdminSite`, esta asignación de URL a vistas está controlada por el método `get_urls()` implementado por la clase `AdminSite`. El método `get_urls()` devuelve la lista `urlpatterns` que se asigna a las vistas de `AdminSite`.

Si queremos agregar una asignación de URL para nuestra vista personalizada, el enfoque preferido incluye anular la implementación de `get_urls()` en nuestra clase `AdminSite` personalizada y agregar la asignación de URL allí. Este enfoque se demuestra en el siguiente fragmento de código:

```python
class SysAdminSite(admin.AdminSite):
    def get_urls(self):
        base_urls = super().get_urls() # Get the existing set of URLs
        # Define our URL patterns for custom views
        urlpatterns = [
            path("health_dashboard/", self.system_health_dashboard)
        ]
        return base_urls + urlpatterns # Return the updated mapping
```

Django generalmente llama al método `get_urls()` automáticamente y no es necesario realizar ningún procesamiento manual sobre él.

Una vez hecho esto, el último paso consiste en asegurarse de que nuestra vista de administración personalizada solo sea accesible a través del sitio de administración y que los usuarios que no sean administradores no puedan acceder a ella. Veamos cómo se puede lograr eso.

#### Restricción de vistas personalizadas al sitio de administración

Si seguiste minuciosamente todos los encabezados anteriores, ahora tendrás una vista personalizada de `AdminSite` lista para usar. Sin embargo, hay un pequeño inconveniente. Esta vista también es accesible directamente para cualquier usuario que no esté en el sitio de administración.

Para asegurarnos de que no surja tal situación, debemos restringir esta vista al sitio de administración. Esto se puede lograr de manera muy sencilla envolviendo nuestra ruta de URL dentro de la llamada `admin_view()`, como se muestra en el siguiente fragmento de código:

```python
urlpatterns = [
    self.admin_view(path("health_dashboard/", self.system_health_dashboard))
]
```

La función `admin_view` se asegura de que la ruta proporcionada esté restringida solo al panel de administración y que ningún usuario sin privilegios de administrador pueda acceder a ella.

Ahora, agreguemos una nueva vista personalizada a nuestro sitio de administración.

#### Ejercicio 10.03 – Adición de vistas personalizadas al sitio de administración

En este ejercicio, agregarás una vista personalizada al sitio de administración, la cual renderizará un perfil de usuario y le mostrará las opciones para modificar su correo electrónico y agregar una nueva foto de perfil. Para crear esta vista personalizada, sigue los pasos descritos:

1. Abre el archivo `admin.py` en el directorio `bookr_admin` y agrega las siguientes importaciones. Estas serán necesarias para crear nuestra vista personalizada dentro de la aplicación del sitio de administración:
   ```python
   from django.template.response import TemplateResponse
   from django.urls import path
   ```
2. Abre el archivo `admin.py` en el directorio `bookr_admin` y crea un nuevo método llamado `profile_view`, que toma una variable `request` como parámetro, dentro de la clase `BookrAdmin`:
   ```python
   def profile_view(self, request):
   ```
3. A continuación, dentro del método, obtén el nombre de la aplicación actual y configúralo en el contexto de la solicitud. Para esto, puedes usar la propiedad `name` de la clase, que Django completa automáticamente. Para obtener esta propiedad y configurarla en el contexto de tu solicitud, debes agregar la siguiente línea:
   ```python
   request.current_app = self.name
   ```
4. Una vez que tengas el nombre de la aplicación en el contexto de la solicitud, el siguiente paso es recuperar las variables de plantilla, que son necesarias para representar el contenido, como `site_title` y `site_header`, en las plantillas de administración. Para ello, aprovecha el método `each_context()` de la clase `AdminSite`, que proporciona el diccionario de las variables de plantilla del sitio de administración de la clase:
   ```python
   context = self.each_context(request)
   ```
5. Una vez que tengas los datos en su lugar, el último paso es devolver un objeto `TemplateResponse`, que renderizará la plantilla de perfil personalizada cuando alguien visite el punto de conexión de URL asignado a tu vista personalizada:
   ```python
   return TemplateResponse(request, "admin/admin_profile.html", context)
   ```
6. Con la función de vista ahora creada, el siguiente paso es hacer que `AdminSite` devuelva las URLs que asignan la vista a una ruta dentro de `AdminSite`. Para hacer esto, necesitas crear un nuevo método con el nombre `get_urls()`, que anula el método `AdminSite.get_urls()` y devuelve la asignación de tu nueva vista. Esto se puede hacer creando primero un nuevo método llamado `get_urls()` dentro de la clase `BookrAdmin` que creaste para tu sitio de administración personalizado:
   ```python
   def get_urls(self):
   ```
7. Dentro de este método, lo primero que debes hacer es obtener la lista de URLs que ya están asignadas al punto de conexión del administrador. Este es un paso obligatorio; de lo contrario, tu sitio de administración personalizado no podrá cargar ningún resultado asociado con las páginas de edición de modelos, la página de cierre de sesión, etc., si se pierde esta asignación. Para obtener esta asignación, llama al método `get_urls()` de la clase base de la que se deriva la clase `BookrAdmin`:
   ```python
   urls = super().get_urls()
   ```
8. Una vez capturadas las URLs de la clase base, el siguiente paso es crear una lista de URLs que asignen nuestra vista personalizada a un punto de conexión de URL en el sitio de administración. Para hacer esto, crearemos una nueva lista llamada `url_patterns` y asignaremos nuestro método `profile_view` al punto de conexión `admin_profile`. Esto se hace mediante la función de utilidad `path` de Django, que nos permite asignar la función de vista con una ruta de punto de conexión de API basada en cadenas:
   ```python
   url_patterns = [
       path("admin_profile", self.profile_view)
   ]
   return url_patterns + urls
   ```
9. Guarda el archivo `admin.py`. Debería verse como `Exercise10.03/bookr/admin.py` en la carpeta `Chapter10` en el repositorio de GitHub de este libro.
   Ten en cuenta que el valor devuelto por `get_urls()` es una lista no modificable. En el fragmento de código anterior, si intentamos cambiar el orden entre la combinación de los objetos `url_patterns` y `urls`, el método no funcionará y la URL `admin_profile` no se creará.
10. Ahora, con la clase `BookrAdmin` configurada para la nueva vista, el siguiente paso es crear tu plantilla para la página de perfil de administración. Para hacer esto, crea un nuevo archivo llamado `admin_profile.html` en el directorio `templates/admin` de la raíz de tu proyecto. Dentro de este archivo, primero, agrega una etiqueta `extends` para asegurarte de que estás extendiendo desde la plantilla de administración predeterminada:
    ```django
    {% extends "admin/index.html" %}
    ```
    Este paso garantiza que todas las hojas de estilo y el HTML de tu plantilla de administración estén disponibles para su uso dentro de tu plantilla de vista personalizada. Por ejemplo, sin tener esta etiqueta `extends`, tu vista personalizada no mostrará ningún contenido específico ya asignado al sitio de administración, como `site_header`, `site_title` o cualquier enlace para cerrar sesión o ir a otra página.
11. Una vez agregada la etiqueta `extends`, agrega una etiqueta `block` y proporciónale el valor de `content`. Esto garantiza que el código que agregues entre el par de segmentos `{% block content %}…{% endblock %}` anule cualquier valor que esté presente en la plantilla `index.html` que viene preempaquetada con el módulo de administración de Django:
    ```django
    {% block content %}
    ```
12. Dentro de la etiqueta de bloque, agrega el HTML requerido para renderizar la vista de perfil que se creó en el paso 2 de este ejercicio:
    ```html
    <p>Welcome to your profile, {{ username }}</p>
    <p>You can do the following operations</p>
    <ul>
        <li><a href="#">Change E-Mail Address</a></li>
        <li><a href="#">Add Profile Picture</a></li>
    </ul>
    {% endblock %}
    ```
13. Ahora, con los pasos anteriores completos, recarga el servidor de tu aplicación ejecutando `python manage.py runserver localhost:8000` y luego visitando `http://localhost:8000/admin/admin_profile`. Cuando se abra la página, puedes esperar ver algo como la siguiente captura de pantalla:
    *Figura 10.10: La vista de la página de perfil en el sitio de administración de Bookr*
14. La vista creada hasta ahora se renderizará bien, independientemente de si un usuario ha iniciado sesión en la aplicación de administración.
    Para asegurarte de que esta vista solo sea accesible para los administradores que hayan iniciado sesión, debes realizar una pequeña modificación dentro de tu método `get_urls()`, que definiste en el paso 2 de este ejercicio.
    Dentro del método `get_urls()`, modifica la lista `url_patterns` para que se parezca a la que se muestra aquí:
    ```python
    url_patterns = [
        path("admin_profile", self.admin_view(self.profile_view)),
    ]
    ```
    En el código anterior, envolviste tu método `profile_view` dentro del método `admin_view()`.
    El método `AdminSite.admin_view()` hace que la vista esté restringida a aquellos usuarios que hayan iniciado sesión. Si un usuario que actualmente no ha iniciado sesión en el sitio de administración intenta visitar la URL directamente, será redirigido a la página de inicio de sesión y solo en caso de un inicio de sesión exitoso se le permitirá ver el contenido de nuestra página personalizada.

Durante este ejercicio, aprovechamos nuestra comprensión existente sobre la escritura de vistas para aplicaciones de Django, fusionándola con el contexto de la clase `AdminSite` para crear una vista personalizada para nuestro panel de administración. Con este conocimiento, ahora podemos continuar y agregar funcionalidades útiles a nuestro sitio de administración de Django para potenciar su utilidad.

#### Paso de claves adicionales a plantillas mediante variables de plantilla

Dentro del sitio de administración, los valores de las variables pasadas a las plantillas se hacen mediante variables de plantilla. Estas variables de plantilla son preparadas y devueltas por el método `AdminSite.each_context()`.

Ahora, si hay un valor que deseas pasar a todas las plantillas de tu sitio de administración, puedes anular el método `AdminSite.each_context()` y agregar los campos obligatorios al contexto de la solicitud. Veamos un ejemplo para ver cómo podemos lograr este resultado.

Considera el campo `username`, que pasamos a nuestra plantilla `admin_profile` anteriormente. Si queremos pasarlo a cada plantilla dentro de nuestro sitio de administración personalizado, primero debemos anular el método `each_context()` dentro de nuestra clase `BookrAdmin`, como se muestra aquí:

```python
def each_context(self, request):
    context = super().each_context(request)
    context['username'] = request.user.username
    return context
```

El método `each_context()` toma un único argumento (no estamos considerando `self` aquí) del tipo `HTTPRequest`, que utiliza para evaluar ciertos otros valores.

Ahora, dentro de nuestro método `each_context()` anulado, primero hacemos una llamada al método `each_context()` de la clase base para recuperar el diccionario de contexto para el sitio de administración:

```python
context = super().each_context(request)
```

Una vez hecho esto, lo siguiente que debemos hacer es agregar nuestro campo `username` a `context` y establecer su valor en el valor del campo `request.user.username`:

```python
context['username'] = request.user.username
```

Una vez hecho esto, lo último que queda es devolver este contexto modificado.

Ahora, cada vez que nuestro sitio de administración personalizado renderice una plantilla, a la plantilla se le pasará esta variable adicional `username`.

---

### Sección: Configuración de un servidor de correo electrónico con Django

La aplicación Django Admin contiene características poderosas que mejoran el desarrollo rápido y robusto de aplicaciones. También proporciona un mecanismo para lidiar con contraseñas olvidadas enviando al usuario por correo electrónico un enlace seguro para restablecer la contraseña.

Para implementar esta funcionalidad de correo electrónico en Django, necesitamos configurar un servidor SMTP en el archivo `bookr/settings.py` de Django.

Se recomienda instalar el módulo `python-decouple` para que los detalles confidenciales de la cuenta del servidor de correo no se guarden en el archivo `bookr/settings.py`. Podemos instalarlo de la siguiente manera:

```bash
pip install python-decouple
```

Ahora podemos agregar las configuraciones SMTP apropiadas al archivo `<nombre_proyecto>/settings.py`. Las configuraciones que son aplicables para la configuración SMTP son `EMAIL_BACKEND`, `EMAIL_HOST`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`, `EMAIL_PORT`, `EMAIL_USE_TLS` y `EMAIL_USE_SSL`.

Las siguientes son las configuraciones actuales que funcionan con una cuenta de Gmail. Las cuentas de Google han dejado de permitir que aplicaciones de terceros acepten contraseñas de cuentas de correo electrónico. En su lugar, han introducido un sistema de generación de "contraseñas de aplicación". Las políticas de seguridad son propensas a cambios frecuentes, pero al momento de escribir este artículo, se pueden generar contraseñas de aplicaciones para cuentas con verificación en dos pasos habilitada visitando este enlace: [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).

```python
from decouple import config

EMAIL_BACKEND = \
    'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_USE_SSL = False
```

Estos cambios en el archivo `settings.py` incluyen llamadas a la función `decouple.config`. Por lo tanto, en lugar de incluir el nombre de usuario y la contraseña de la cuenta de correo electrónico en el archivo `settings.py` (que podría incluirse en la distribución del código fuente o en un repositorio público), podemos establecer estas variables de entorno en tiempo de ejecución o incluyéndolas en la carpeta del proyecto de nivel superior como un archivo `.env` o `settings.ini`. Invocarlas en tiempo de ejecución podría implicar colocarlas en un archivo `.bashrc` o `.profile`, o definirlas en la línea de comandos cuando se inicia el servidor Django:

```bash
EMAIL_HOST_USER=bookr2026 EMAIL_HOST_PASSWORD='vnpw zgkl ixpq pbcz' python manage.py runserver
```

Alternativamente, se puede agregar un archivo `.env` o `settings.ini` al directorio del proyecto. El formato `.env` se usa más comúnmente en macOS y Linux, mientras que el formato `settings.ini` se usa normalmente en entornos Windows. Decouple puede leer ambos formatos en todas las plataformas.

`EMAIL_HOST_USER` se establece en el nombre de usuario en el servidor SMTP, y `EMAIL_HOST_PASSWORD` se establece en la contraseña del correo electrónico o en la contraseña de la aplicación (en el caso de Gmail SMTP y otros proveedores de correo electrónico conscientes de la seguridad).

`.env`:
```text
EMAIL_HOST_USER=bookr2026
EMAIL_HOST_PASSWORD=vnpw zgkl ixpq pbcz
```

El archivo de configuración `settings.ini` es muy similar al formato `.env`, pero requiere una tabulación o sangría:

`settings.ini`:
```ini
[settings]
EMAIL_HOST_USER=bookr2026
EMAIL_HOST_PASSWORD=vnpw zgkl ixpq pbcz
```

Otros servidores SMTP tendrán diferentes configuraciones.

Una vez agregadas estas configuraciones, las aplicaciones de Django pueden usar el servidor de correo electrónico para enviar correos electrónicos a los usuarios.

Django proporciona funciones como `send_mail` y `send_mass_mail`, que se pueden usar para enviar correos electrónicos desde una aplicación de Django. Puedes leer más sobre esta funcionalidad en la documentación de Django en [https://docs.djangoproject.com/en/6.0/topics/email/](https://docs.djangoproject.com/en/6.0/topics/email/) y aprender practicando en el próximo Ejercicio 10.05.

---

### Sección: Automatización de la administración con Django Tasks

Muchos trabajos que necesitamos repetir en una aplicación web no se adaptan a las limitaciones del ciclo de solicitud y respuesta HTTP. No queremos limitar la capacidad de un servidor web manteniendo abiertas las conexiones HTTP mientras se producen actualizaciones prolongadas de la base de datos o lotes de actividad de red. Hay muchas tareas automatizadas involucradas en el soporte de una aplicación web que se realizan mejor sin la sobrecarga de las interacciones web.

Hasta hace poco, estos se habrían implementado típicamente como trabajos cron en Linux o Mac, o como tareas programadas en Windows, o utilizando un programador y procesador de tareas independiente como Celery.

Django 6.0 introduce el framework Django Tasks. Su funcionalidad es algo limitada en la actualidad, pero nos ofrece una forma de mantener las tareas de programación dentro de un paradigma familiar de Django.

El framework Django Tasks proporciona un mecanismo para definir unidades de trabajo y sus parámetros de programación, junto con un mecanismo para ponerlas en cola. La API define una interfaz para los backends que realmente realizan el trabajo.

Se puede definir una tarea en una aplicación Django usando el decorador `task`. Esto define las propiedades de la tarea, incluida su prioridad, la cola en la que se programará y el backend que se delegará para ejecutar la tarea.

Si tuviéramos tareas para programar que se dividieran en grupos como generación de imágenes con IA, transformaciones de bases de datos, procesamiento por lotes de correo electrónico y copia de seguridad de datos, tendría sentido configurar una cola de tareas separada para cada uno de estos grupos de tareas como medio para asignar recursos. En el ejemplo y ejercicio que siguen, solo usaremos la cola predeterminada y un backend trivial, pero configuraciones más complicadas serían apropiadas con un despliegue de producción significativo.

La aplicación Django Tasks viene con dos backends triviales: `ImmediateBackend` y `DummyBackend`. `ImmediateBackend` funciona ejecutando tareas tan pronto como se ponen en cola, mientras que `DummyBackend` devuelve un resultado de tarea cuando se pone en cola sin que se pase ningún trabajo a un trabajador (*worker*). `DummyBackend` es útil para incluir en las pruebas de una aplicación Django.

El backend se especifica en el archivo `<proyecto>/settings.py`. Una configuración simple con `ImmediateBackend` se verá así:

```python
TASKS = {
    "default": {
        "BACKEND": "django.tasks.backends.immediate."
        "ImmediateBackend"
    }
}
```

`DummyBackend` se puede configurar especificando `"django.tasks.backends.dummy.DummyBackend"`.

Las tareas se definen normalmente dentro de la carpeta de la aplicación, en un archivo llamado `tasks.py`.

Una tarea se define mediante el decorador `@task`. El decorador no requiere ningún argumento, pero tiene los siguientes argumentos opcionales:
- `priority`: Un valor entero que los backends pueden utilizar para determinar la prioridad de programación de la tarea, con un valor predeterminado de `0`.
- `queue_name`: Una cadena correspondiente a una cola definida en la especificación de Tasks. El valor predeterminado es `"default"`.
- `backend`: Una cadena correspondiente a un backend definido en la especificación de Tasks. El valor predeterminado es `"default"`.
- `takes_context`: Un valor booleano que especifica si el primer argumento de la función de tarea decorada es un objeto `TaskContext`. `TaskContext` tiene dos atributos definidos: el número de intento y `task_result`. El valor predeterminado es `False`.

He aquí un ejemplo de una tarea que se utiliza para generar una imagen y escribirla en el sistema de archivos:

`myapp/tasks.py`:
```python
from django.tasks import task
from .img_utils import _generate_cake_jpg

@task(priority=3, queue_name="img_generator")
def generate_birthday_greeting_jpg(user, background_img, favourite_cake):
    img = _generate_cake_jpg(user, background_img, favourite_cake)
    filename = f"media/{user.username}/birthday.jpg"
    img.save(filename)
```

La tarea se programará para ejecutarse llamando al método `enqueue` de la tarea. El método `enqueue` recibe los argumentos del método de tarea decorado:

```python
>>> from myapp.tasks import generate_birthday_greeting_jpg
>>> generate_birthday_greeting_jpg.enqueue(user,
...     "media/background/hundreds_and_thousands.jpg",
...     "tiramisu")
```

Una vez que se ejecute la tarea, producirá un `TaskResult` que se parecerá al que se muestra en la Figura 10.12:

*Figura 10.12: La salida de TaskResult*

Con una breve explicación del framework Django Tasks, desarrollaremos una tarea funcional en el ejercicio final de este capítulo.

#### Actividad 10.03: Creación de un panel de administración personalizado con búsqueda integrada

En esta actividad, utilizarás el conocimiento adquirido sobre los diferentes aspectos de la creación de un sitio de administración personalizado para crear un panel de administración personalizado para Bookr. Dentro de este panel, introducirás la capacidad de permitir que un usuario busque libros utilizando el nombre del libro o el nombre de la editorial del libro, y permitirás que el usuario modifique o elimine estos registros de libros.

Los siguientes pasos te ayudarán a crear un panel de administración personalizado y agregar la capacidad de buscar un registro de libro utilizando el nombre de la editorial:

1. Crea una nueva aplicación dentro del proyecto Bookr llamada `bookr_admin`, si aún no la has creado. Esto almacenará la lógica para nuestro sitio de administración personalizado.
2. Dentro del archivo `admin.py` en el directorio `bookr_admin`, crea una nueva clase, `BookrAdmin`, que herede de la clase `AdminSite` del módulo de administración de Django.
3. Dentro de la clase `BookrAdmin` recién creada del paso 2, agrega cualquier personalización para el título del sitio o cualquier otro componente de marca del panel de administración.
4. Dentro del archivo `apps.py` en el directorio `bookr_admin`, crea una nueva clase `BookrAdminConfig`, y dentro de esta nueva clase `BookrAdminConfig`, establece el atributo `default_site` en el nombre de módulo completo para nuestra clase de sitio de administración personalizada, `BookrAdmin`.
5. Dentro del archivo `settings.py` de tu proyecto Django, agrega la ruta completa de la clase `BookrAdminConfig` creada en el paso 4 como la primera aplicación instalada (`INSTALLED_APPS`).
6. Para registrar el modelo `Book` de la aplicación `reviews` dentro de Bookr, abre el archivo `admin.py` dentro del directorio `reviews` y asegúrate de que el modelo `Book` esté registrado en el sitio de administración mediante `admin.site.register(ModelClass)`.
7. Para permitir la búsqueda de un libro según el nombre de la editorial, dentro del archivo `admin.py` de la aplicación `reviews`, modifica la clase `BookAdmin` y agrégale una propiedad llamada `search_fields`, que contenga `publisher_name` como campo.
8. Para obtener el nombre de la editorial correctamente para la propiedad `search_fields`, introduce un nuevo método llamado `get_publisher` dentro de la clase `BookAdmin`, que devolverá el campo de nombre de la editorial del modelo `Book`.
9. Asegúrate de que la clase `BookAdmin` esté registrada como una clase de administración de modelos para el modelo `Book` dentro de nuestro panel de administración de Django mediante `admin.site.register(Book, BookModel)`.

Después de completar esta actividad, una vez que inicies el servidor de la aplicación y visites `http://localhost:8000/admin` y navegues al modelo Book, deberías poder buscar libros usando el nombre de la editorial y, en caso de una búsqueda exitosa, ver una página que se parece a la que se muestra en la siguiente captura de pantalla:

*Figura 10.11: La página de edición de libros dentro del panel de administración de Bookr*

La solución para esta actividad se puede encontrar en la carpeta `Chapter09` en el repositorio de GitHub de este libro.

#### Ejercicio 10.05: Uso de Django Tasks para crear una tarea periódica de envío de informes por correo

En este ejercicio, queremos utilizar el framework Django Tasks para crear una tarea periódica que envíe un correo electrónico a los suscriptores informándoles sobre las reseñas recientes en el sitio Bookr.

Escribiremos esta tarea con cierta flexibilidad para que se pueda configurar para ejecutarse diaria, semanal, mensual o anualmente:

1. Primero, agregaremos una función llamada `get_daterange_dt` a `reviews/utils.py`. Esta función tomará dos parámetros: una cadena de fecha en el formato `YYYY-mm-dd` y una cadena para describir la periodicidad: `daily`, `weekly`, `monthly` o `annual`. La función requerirá la función `monthrange` del módulo `calendar` y los tipos `datetime` y `timedelta` del módulo `datetime`:
   ```python
   from calendar import monthrange
   from datetime import datetime, timedelta
   ```
2. Luego podemos definir la función `get_daterange_dt`, que toma una cadena `start_day` y un designador de período que se limita a los valores `'daily'`, `'weekly'`, `'monthly'` y `'annual'`:
   ```python
   def get_daterange_dt(start_day, period):
       assert period in ['daily', 'weekly', 'monthly', 'annual']
       start_day_dt = datetime.strptime(start_day, '%Y-%m-%d')
   ```
3. Nuestra lógica para calcular el día final del rango de fechas diferirá según si estamos calculando un rango de un día, una semana, un mes o un año. Ten en cuenta que hay un bloque `try/except` que maneja problemas para años bisiestos:
   ```python
       if period == 'daily':
           end_day_dt = start_day_dt
       elif period == 'weekly':
           end_day_dt = start_day_dt + timedelta(days=6)
       elif period == 'monthly':
           last_day_of_month = monthrange(
               start_day_dt.year, start_day_dt.month)[1]
           end_day_dt = start_day_dt.replace(
               day=last_day_of_month)
       elif period == 'annual':
           try:
               end_day_dt = (start_day_dt.replace(
                   year=start_day_dt.year+1) - timedelta(days=1))
           except ValueError: # catch error caused by 29th
               end_day_dt = start_day_dt.replace(
                   year=start_day_dt.year+1, month=2, day=28)
   ```
4. Habiendo calculado el día final, debemos calcular el microsegundo final del mismo, ya que esto se usará en la consulta a la base de datos. Devolvemos los tipos `datetime` `start_date_dt` y `end_day_dt`.
5. Crea un archivo en la carpeta `reviews` llamado `tasks.py`. Necesitaremos importar la función `groupby` de `itertools`, la función `send_mass_mail` de Django, el decorador `task` y la función `render_to_string` de `django.template.loader`. También necesitamos importar el modelo `Review` y la función `get_daterange_dt` que acabamos de crear:
   ```python
   from itertools import groupby
   from django.core.mail import send_mass_mail
   from django.tasks import task
   from django.template.loader import render_to_string
   from .models import Review
   from .utils import get_daterange_dt
   ```
6. Ahora podemos definir una función con un decorador `task`. Estamos especificando una prioridad (pero esto solo será significativo si usamos un sistema de colas externo con tareas en competencia) mientras aceptamos la cola y el backend predeterminados. La función `review_report` toma argumentos para lo siguiente:
   - `emails`: Una lista de destinatarios de correo electrónico.
   - `from_email`: La dirección de correo electrónico remitente.
   - `period`: Valores de cadena: `'daily'`, `'weekly'`, `'monthly'`, `'annual'`.
   - `start_day`: La fecha de inicio del período de informe en el formato `'YYYY-dd-mm'`.
   - `site`: La URL del sitio web de Bookr. El valor predeterminado es `'http://127.0.0.1:8000'`.
   ```python
   @task(priority=2)
   def review_report(emails, from_email, period, start_day, site="http://127.0.0.1:8000"):
   ```
7. Llamamos a la función de utilidad `get_daterange_dt` para derivar un inicio y fin de `datetime` a partir de una fecha especificada por una cadena, como `"2026-01-01"`, y una cadena de período de tiempo. Luego creamos cadenas legibles por humanos para el día inicial y final, utilizando el especificador de tiempo específico de la configuración regional:
   ```python
       start_day_dt, end_day_dt = get_daterange_dt(
           start_day, period)
       start_day_fmt = start_day_dt.strftime('%x')
       end_day_fmt = end_day_dt.strftime('%x')
   ```
8. Utiliza el modelo de la aplicación `reviews` para construir una consulta de las reseñas que se han publicado en el período de tiempo dado y ordénalas por título de libro:
   ```python
       reviews = Review.objects.filter(
           date_created__range=[start_day_dt, end_day_dt]
       ).order_by('book__title')
   ```
9. Ahora podemos construir un contexto de datos apropiados que enviaremos a la plantilla de Django para crear el mensaje de correo electrónico. Como los datos del modelo de Django ya están ordenados por nombre de libro, podemos usar la función `itertools.groupby` para que estos datos se particionen utilizando el título como clave:
   ```python
   [('Title 1', [<Review 2>, <Review 3>]), ('Title 2', [<Review 1>]), ('Title 3', [<Review 6>, <Review 7>]) ('Title 5', [<Review 4>, <Review 5>])]
   ```
   El contexto se verá así:
   ```python
       context = dict(period=period, site=site, start_day_fmt=start_day_fmt, end_day_fmt=end_day_fmt, reviews_by_title=[(title, list(reviews)) for title, reviews in groupby(reviews, lambda x: x.book.title)])
   ```
10. A continuación, podemos generar una línea de asunto para el mensaje de correo electrónico. Variamos el mensaje dependiendo de si se relaciona con reseñas de un solo día o de un período de tiempo más largo:
    ```python
       subject = f"Your {period} Bookr reviews "
       if period=='daily':
           subject += "for {start_day_fmt}"
       else:
           subject += "from {start_day_fmt} to {end_day_fmt}"
    ```
11. Finalmente, crearemos el mensaje de correo electrónico renderizando `review_report_email.txt` en una cadena (crearemos esta plantilla en el siguiente paso). Luego creamos una lista de mensajes de correo electrónico con un mensaje para cada uno de los correos electrónicos pasados a la función. Luego usamos `send_mass_mail` de Django. Ten en cuenta que si hubiéramos implementado el mensaje utilizando la función `send_mail` y una lista de destinatarios de correo electrónico, habríamos tenido una situación en la que saldría un solo correo electrónico con múltiples destinatarios. En la mayoría de las situaciones, esto se consideraría un problema de privacidad, ya que los usuarios no habrían solicitado que sus direcciones de correo electrónico se compartieran con los demás destinatarios:
    ```python
       message = render_to_string(
           'reviews/review_report_email.txt', context)
       email_messages = [(subject, message, from_email, [email]) for email in emails]
       return send_mass_mail(
           email_messages, fail_silently=False)
    ```
12. Ahora, esta tarea requiere una plantilla de Django para representar los resultados. Seguirá esta estructura:
    ```django
    {% if reviews_by_title %} <-- header --> {% endif %}
    {% for title, reviews in reviews_by_title %} <- book title ->
        {% for review in reviews %} <- reviewer detail and link to review ->
        {% endfor %}
    {% empty %} <-- Empty report message -->
    {% endfor %}
    ```
13. Se debe agregar una plantilla para el mensaje de correo electrónico al directorio `templates` de la aplicación. Por motivos de claridad, utilizamos un mensaje de solo texto sin formato HTML.
    `reviews/templates/reviews/review_report_email.txt`:
    ```django
    {% if reviews_by_title %}Here is your {{period}} report of Bookr reviews {% if period == 'daily' %}on {{start_day_fmt}}.{% else %}from {{start_day_fmt}} to {{end_day_fmt}}.{% endif %} {% endif %}
    {% for title, reviews in reviews_by_title %}
    {{ title }}
        {% for review in reviews %}
        * {{ review.creator.username }} - {{ review.rating }}/5 - {{ site }}/reviews/{{ review.id }}
        {% endfor %}
    {% empty %}
    No reviews for {% if period == 'daily' %}{{start_day_fmt}}.{% else %} the period {{start_day_fmt}} to {{end_day_fmt}}.{% endif %}
    {% endfor %}
    ```
14. Ahora que hemos terminado de desarrollar la tarea, podemos ponerla en cola desde la línea de comandos. Este enfoque funcionará con `DummyBackend` o `ImmediateBackend`. Los parámetros `emails` y `from_email` deben ser la lista de correos electrónicos de los destinatarios y la dirección de correo electrónico configurada en el archivo `settings.py`, respectivamente. Si no obtienes ningún dato en el período de tiempo actual, asegúrate de agregar el tuyo:
    ```python
    >>> from reviews.tasks import review_report
    >>> review_report.enqueue(
    ...     ['alice@example.com', 'bob@example.com'],
    ...     'bookr2026@example.com', 'annual', '2026-01-01',
    ...     'bookr.example.com')
    ```
15. Los destinatarios de tu correo electrónico recibirán un correo electrónico similar al de la Figura 10.13:
    *Figura 10.13: El correo electrónico del informe de reseñas*

Aunque pueda parecer que hay mucho código en este ejercicio, este es un ejemplo simple de un informe de período de tiempo. Es mejor manejar esto como una tarea de Django (`Django Task`) en lugar de como una solicitud de Django (`Django request`), ya que puede llevar una cantidad significativa de tiempo generar el informe y enviar correos electrónicos a varios destinatarios, lo que podría provocar tiempos de espera del servidor web durante la ejecución.

---

### Sección: Resumen

En este capítulo, aprendimos sobre la personalización avanzada de nuestra aplicación de administración de Django. Profundizamos en las características que nos ayudan a mejorar la apariencia y el funcionamiento de nuestra aplicación de administración, haciendo que la interfaz de usuario sea más intuitiva y útil, antes de pasar a los detalles más finos de la paginación y el filtrado de resultados de búsqueda. En el próximo capítulo, veremos cómo podemos usar la creación de plantillas y las vistas basadas en clases para nuestra aplicación Django.

También aprendimos cómo conectar un proyecto de Django con un servidor de correo electrónico y cómo desarrollar servicios no HTTP con el framework Django Tasks.

A medida que avanzamos hacia el siguiente capítulo, podremos aprovechar lo que hemos aprendido hasta ahora y ampliar ese conocimiento introduciéndonos en el concepto de crear nuestras propias etiquetas y filtros personalizados para plantillas, así como la capacidad de crear nuestras vistas en un estilo orientado a objetos utilizando el concepto de vistas basadas en clases.
