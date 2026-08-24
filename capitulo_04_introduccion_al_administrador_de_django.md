# Parte 1: Primeros pasos con Django

## Capítulo 4: Introducción al Administrador de Django

Este capítulo te introduce a la funcionalidad básica de la aplicación de administración de Django (*Django admin*). Comenzarás creando cuentas de superusuario para la aplicación Bookr, antes de pasar a ejecutar operaciones de creación, lectura, actualización y eliminación (*Create, Read, Update, Delete* o CRUD) con la aplicación de administración. Aprenderás a integrar tu aplicación Django con la aplicación de administración y también verás el comportamiento de `ForeignKey` en la aplicación de administración. Al final de este capítulo, verás cómo puedes personalizar la aplicación de administración de acuerdo con un conjunto único de preferencias mediante la creación de subclases de `AdminSite` y `ModelAdmin` para hacer que su interfaz sea más intuitiva y fácil de usar.

En este capítulo, cubriremos los siguientes temas:
- Presentación de la aplicación de administración de Django
- Creación de una cuenta de superusuario
- Operaciones CRUD mediante la aplicación de administración de Django
- Gestión de usuarios y grupos de Django
- Registro de modelos con la aplicación de administración
- Personalización de la interfaz de administración

Al final de este capítulo, podrás crear cuentas de superusuario, cuentas de usuario y grupos, y asignarles permisos. Podrás registrar tus modelos con la aplicación de administración, personalizar la funcionalidad de las páginas de lista de cambios y los formularios para administrar tus modelos, así como personalizar las propiedades globales de la aplicación de administración.

---

### Sección: Requisitos técnicos

Encuentra la solución en la carpeta `Chapter04` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos descritos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Presentación de la aplicación de administración de Django

Al desarrollar una aplicación, a menudo surge la necesidad de poblarla con datos y luego alterarlos. Ya hemos visto, en el Capítulo 2, cómo se puede hacer esto en la línea de comandos mediante `python manage.py shell`. En el Capítulo 3, aprendimos a desarrollar una interfaz de formulario web para nuestro modelo utilizando las vistas y plantillas de Django. Pero ninguno de estos enfoques es ideal para administrar los datos de las clases en `reviews/models.py`. El uso de la consola (*shell*) para administrar datos es demasiado técnico para los no programadores, y crear páginas web individuales sería un proceso laborioso, ya que nos obligaría a repetir la misma lógica de vista y funciones de plantilla muy similares para cada tabla del modelo. Afortunadamente, se ideó una solución a este problema en los primeros días de Django, cuando aún se estaba desarrollando.

El administrador de Django (*Django admin*) está escrito en realidad como una aplicación de Django. Ofrece una interfaz web intuitivamente renderizada para otorgar acceso administrativo a los datos del modelo. La interfaz de administración está diseñada para ser utilizada por los administradores del sitio web. No está destinada a ser utilizada por usuarios sin privilegios que interactúan con el sitio. En nuestro caso de un sistema de reseñas de libros, la población general de revisores de libros nunca verá la aplicación de administración. Verán las páginas de la aplicación, como las que creamos con vistas y plantillas en el Capítulo 3, y escribirán sus reseñas en dichas páginas.

Además, mientras que los desarrolladores se esfuerzan mucho en crear una interfaz web simple y atractiva para los usuarios generales, la interfaz de administración, dirigida a los usuarios administrativos, mantiene un aspecto utilitario que normalmente muestra las complejidades de los modelos que componen la aplicación y están registrados en el administrador. Es posible que no lo hayas notado, pero ya tienes una aplicación de administración en tu proyecto Bookr. Mira la lista de aplicaciones instaladas en `bookr/bookr/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    ...
]
```

Ahora, mira los patrones de URL en `bookr/bookr/urls.py`:

```python
urlpatterns = [
    path("admin/", admin.site.urls),
    ...
]
```

Si colocamos esta ruta en nuestro navegador, podemos ver que el enlace a la aplicación de administración en el servidor de desarrollo es `http://127.0.0.1:8000/admin/`. Sin embargo, antes de hacer uso de ella, debemos crear un superusuario a través de la línea de comandos.

---

### Sección: Creación de una cuenta de superusuario

Nuestra aplicación Bookr acaba de encontrar una nueva usuaria. Su nombre es Alice y quiere comenzar a agregar sus reseñas de inmediato. Bob, que ya está usando Bookr, nos acaba de informar que su perfil parece incompleto y necesita ser actualizado. Eve ya no quiere usar la aplicación y quiere que se elimine su cuenta. Por razones de seguridad, no queremos que cualquier usuario realice estas tareas por nosotros. Por eso necesitamos crear un superusuario con privilegios elevados. Comencemos haciendo precisamente eso.

En el modelo de autorización de Django, un superusuario es aquel que tiene todos los permisos. Un usuario con la marca de miembro del personal (*staff status*) establecida puede iniciar sesión en el sitio de administración, pero luego sus permisos específicos dictan lo que puede ver y hacer con los modelos registrados. Examinaremos esto más adelante en el capítulo y aprenderemos más sobre este modelo de autorización en el Capítulo 9.

Podemos crear un superusuario utilizando el script `manage.py`, que exploramos en capítulos anteriores. Nuevamente, debemos estar en el directorio del proyecto cuando ejecutamos el comando. Usaremos el subcomando `createsuperuser` ingresando el siguiente comando en la línea de comandos:

```bash
python manage.py createsuperuser
```

En este capítulo, utilizaremos direcciones de correo electrónico que pertenecen al dominio `example.com`. Esto sigue una convención establecida de utilizar este dominio reservado para pruebas y documentación. Puedes utilizar tus propias direcciones de correo electrónico si lo prefieres.

Procedamos a crear nuestro superusuario.

#### Ejercicio 4.01 – Creación de una cuenta de superusuario

En este ejercicio, crearás una cuenta de superusuario que le permitirá al usuario iniciar sesión en el sitio de administración. Esta funcionalidad también se utilizará en los próximos ejercicios para implementar cambios que solo un superusuario puede realizar. Los siguientes pasos te ayudarán a completar este ejercicio:

1. Ingresa el siguiente comando para crear un superusuario:
   ```bash
   python manage.py createsuperuser
   ```
   Al ejecutar el comando anterior, se te solicitará que crees un superusuario. Este comando te solicitará un nombre de superusuario, una dirección de correo electrónico opcional y una contraseña.

2. Agrega el nombre de usuario y el correo electrónico para el superusuario de la siguiente manera. Aquí, ingresamos `bookradmin` en el indicador y presionamos la tecla Enter. De manera similar, en el siguiente indicador, que te pide que ingreses tu dirección de correo electrónico, puedes agregar `bookradmin@example.com`. Presiona la tecla Enter para continuar:
   ```text
   Username (leave blank to use 'django'): bookradmin
   Email address: bookradmin@example.com
   Password: 
   ```
   Esto asignará el nombre `bookradmin` al superusuario. Ten en cuenta que no verás ninguna salida de inmediato mientras escribes la contraseña.

3. El siguiente indicador en la consola es para tu contraseña. Agrega una contraseña segura y presiona la tecla Enter para confirmarla una vez más:
   ```text
   Password: 
   Password (again): 
   ```
   Deberías ver el siguiente mensaje en tu pantalla:
   ```text
   Superuser created successfully.
   ```

Ten en cuenta que la contraseña se valida según los siguientes criterios:
- No puede estar entre las 20,000 contraseñas más comunes.
- Debe tener un mínimo de ocho caracteres.
- No puede contener únicamente caracteres numéricos.
- No puede derivarse del nombre de usuario, nombre, apellido o dirección de correo electrónico del usuario.

Estos criterios están determinados por `AUTH_PASSWORD_VALIDATORS` en el archivo `settings.py`. Es posible cambiar la seguridad de la contraseña requerida configurando estos validadores o escribiendo validadores personalizados.

Con esto, has creado un superusuario llamado `bookradmin` que puede iniciar sesión en la aplicación de administración. La siguiente captura de pantalla muestra cómo se ve esto en la consola:

*Figura 4.1 – Creación de un superusuario*

Visita la aplicación de administración en `http://127.0.0.1:8000/admin` e inicia sesión con la cuenta de superusuario que has creado:

*Figura 4.2 – El formulario de inicio de sesión de administración de Django*

En este ejercicio, creaste una cuenta de superusuario que utilizaremos durante el resto de este capítulo para asignar o eliminar privilegios según sea necesario.

Con una cuenta de superusuario creada, ahora podemos iniciar sesión en la aplicación de administración de Django y aprender sobre los recursos CRUD que proporciona.

---

### Sección: Operaciones CRUD mediante la aplicación de administración de Django

Volvamos a las solicitudes que recibimos de Bob, Alice y Eve. Como superusuario, tus tareas implicarán crear, actualizar, recuperar y eliminar varias cuentas de usuario, reseñas y nombres de títulos. Este conjunto de actividades se denomina colectivamente **CRUD**. Las operaciones CRUD son fundamentales para el comportamiento de la aplicación de administración. Resulta que la aplicación de administración ya reconoce los modelos de otra aplicación de Django, Autenticación y Autorización (*Authentication and Authorization*), a la que se hace referencia en `INSTALLED_APPS` como `'django.contrib.auth'`. Al iniciar sesión en `http://127.0.0.1:8000/admin/`, se nos presentan los modelos de la aplicación de autorización, como se muestra en la Figura 4.3:

*Figura 4.3 – La ventana de administración de Django*

Cuando se inicializa la aplicación de administración, llama a su método `autodiscover()` para detectar si alguna otra aplicación instalada contiene un módulo `admin`. Si es así, estos modelos de administración se importan. En nuestro caso, ha descubierto `'django.contrib.auth.admin'`. Ahora que los módulos están importados y nuestra cuenta de superusuario está lista, comencemos a trabajar en las solicitudes de Bob, Alice y Eve.

#### Crear (Create)

Antes de que Alice comience a escribir sus reseñas, debemos crear una cuenta para ella a través de la aplicación de administración. Una vez hecho esto, podemos ver los niveles de acceso de administración que podemos asignarle. Para crear un usuario, debemos hacer clic en la opción **+ Add** junto a **Users** (consulta la Figura 4.3) y completar el formulario, como se muestra en la Figura 4.4.

No queremos que cualquier usuario aleatorio tenga acceso a las cuentas de los usuarios de Bookr. Por lo tanto, es imperativo que elijamos contraseñas seguras y protegidas.

*Figura 4.4 – La página Add user*

Hay tres botones en la parte inferior del formulario:
- **Save and add another**: Esto crea el usuario y vuelve a mostrar la misma página *Add user*, con campos en blanco.
- **Save and continue editing**: Esto crea el usuario y carga la página *Change user*. La página *Change user* te permite agregar información adicional que no estaba presente en la página *Add user*, como *First name*, *Last name* y más (ver Figura 4.5). Ten en cuenta que *Password* no tiene un campo editable en el formulario. En su lugar, muestra información sobre la técnica de hashing con la que se almacena, además de un enlace a un formulario independiente de *Reset password*.
- **SAVE**: Esto crea el usuario y permite al usuario navegar a la página de lista *Select user to change*, como se muestra en la Figura 4.6.

*Figura 4.5 – La página Change user que se muestra después de hacer clic en Save and continue editing*

#### Actividad 4.01 – Creación de una cuenta de usuario

Ahora hemos visto cómo crear cuentas de usuario a través de la interfaz de administración de Django.
Es hora de poner en práctica tus conocimientos para crear una cuenta para un nuevo usuario, David Green, utilizando la dirección de correo electrónico `david.green@example.com`.
Como fue el caso con la cuenta de Alice White, simplemente configura esta como un usuario activo sin marcar las casillas *Staff status* o *Superuser status*.

Ahora que has creado un usuario a través de la interfaz de administración de Django, podemos examinar las otras tres operaciones CRUD: recuperar, actualizar y eliminar.

#### Recuperar (Retrieve)

Las tareas administrativas deben dividirse entre algunos usuarios y, para ello, al administrador (la persona con la cuenta de superusuario) le gustaría ver aquellos usuarios cuyas direcciones de correo electrónico terminen en `n@example.com` y asignarles las tareas a estos usuarios. Aquí es donde la funcionalidad de recuperación puede resultar útil. Después de hacer clic en el botón **SAVE** en la página *Add user* (consulta la Figura 4.4), se nos dirige a la página de lista *Select user to change* (como se muestra en la Figura 4.6), que lleva a cabo la operación de recuperación. Ten en cuenta que también se puede acceder al formulario de creación haciendo clic en el botón **ADD USER +** en la página de lista *Select user to change*. Por lo tanto, después de haber agregado algunos usuarios más, la lista de cambios se verá así:

*Figura 4.6 – La página Select user to change*

En la parte superior del formulario hay una barra de búsqueda (*Search bar*) que busca en el contenido, como el nombre de usuario, la dirección de correo electrónico y los nombres y apellidos de los usuarios. En el lado derecho hay un panel de filtro (**FILTER**) que reduce la selección según los valores de *By staff status*, *By superuser status* y *By active*. En la Figura 4.7, puedes ver qué sucede cuando buscamos la cadena `n@example.com`. Esto devolverá solo los nombres de los usuarios cuyas direcciones de correo electrónico constan de un nombre de usuario que termina con una `n` y un dominio que comienza con `example.com`. Solo podemos ver tres usuarios con direcciones de correo electrónico que coinciden con este requisito: `bookradmin@example.com`, `carol.brown@example.com` y `david.green@example.com`:

*Figura 4.7 – Búsqueda de usuarios por una parte de su dirección de correo electrónico*

La funcionalidad de recuperación, como la página de lista y la barra de búsqueda, nos permite obtener registros para operaciones de actualización y eliminación.

#### Actualizar (Update)

Recuerda que Bob quería que se actualizara su perfil. Actualicemos el perfil incompleto de Bob haciendo clic en el enlace del nombre de usuario `bob` en la lista *Select user to change*:

*Figura 4.8 – Selección de bob de la lista Select user to change*

Esto nos llevará de regreso al formulario *Change user*, donde se pueden ingresar los valores para *First name*, *Last name* y *Email address*:

*Figura 4.9 – Adición de información personal*

Como se puede ver en la Figura 4.9, estamos agregando información personal sobre Bob aquí: su nombre, apellido y dirección de correo electrónico, específicamente.

Otro tipo de operación de actualización es la eliminación temporal (*soft delete*). La propiedad booleana `Active` nos permite desactivar a un usuario en lugar de eliminar el registro completo y perder todos los datos que tienen dependencias de la cuenta. Esta práctica de usar una marca booleana para denotar un registro como inactivo o eliminado (y posteriormente filtrar estos registros marcados fuera de las consultas) se conoce como *soft delete*. De manera similar, podemos ascender al usuario a *Staff status* o *Superuser status* marcando las casillas de verificación respectivas:

*Figura 4.10 – Los permisos Active, Staff status y Superuser status*

#### Eliminar (Delete)

Eve ya no quiere usar la aplicación Bookr y ha solicitado que eliminemos su cuenta. El administrador de autenticación también se encarga de esto. Selecciona un usuario o registros de usuario en la página de lista *Select user to change* y elige la opción *Delete selected users* del menú desplegable *Action*. Luego, presiona el botón **Go** (Figura 4.11):

*Figura 4.11 – Eliminación desde la página de lista Select user to change*

Se te presentará una pantalla de confirmación y regresarás a la lista *Select user to change* una vez que hayas eliminado el objeto:

*Figura 4.12 – Confirmación de eliminación de usuario*

Verás el siguiente mensaje una vez que se elimine el usuario:

*Figura 4.13 – Notificación de eliminación de usuario*

Después de esa confirmación, verás que la cuenta de Eve ya no existe.

Hasta ahora, hemos aprendido cómo podemos agregar un nuevo usuario, obtener los detalles de otro usuario, realizar cambios en los datos de un usuario y eliminar a un usuario. Estas habilidades nos ayudaron a atender las solicitudes de Alice, Bob y Eve. A medida que crezca el número de usuarios de nuestra aplicación, gestionar las solicitudes de cientos de usuarios acabará siendo bastante difícil. Una forma de solucionar este problema sería delegar algunas de las responsabilidades administrativas a un conjunto seleccionado de usuarios. Aprenderemos cómo hacerlo en la sección siguiente.

---

### Sección: Gestión de usuarios y grupos de Django

El modelo de autenticación de Django consta de usuarios, grupos y permisos. Los usuarios pueden pertenecer a muchos grupos, que es una forma de categorizarlos. También agiliza la implementación de permisos al permitir que los permisos se asignen a colecciones de usuarios, así como a individuos.

En la sección de operaciones CRUD mediante la aplicación de administración de Django, vimos cómo una cuenta de superusuario podía atender las solicitudes de Alice, David y Bob para realizar modificaciones en sus perfiles. Fue bastante fácil de hacer y nuestra aplicación parece bien equipada para gestionar sus solicitudes.

¿Qué pasará cuando aumente el número de usuarios? ¿Podrá el usuario administrador gestionar 100 o 150 usuarios a la vez? Como puedes imaginar, esta puede ser una tarea bastante complicada. Para superar esto, podemos otorgar permisos elevados a un determinado conjunto de usuarios para que puedan ayudar a facilitar las tareas del administrador. Y ahí es donde los grupos resultan útiles. Aunque aprenderemos más sobre usuarios, grupos y permisos en el Capítulo 9, podemos comenzar a comprender los grupos y su funcionalidad creando un grupo llamado *Help Desk User* que contenga cuentas que tengan acceso a la interfaz de administración pero carezcan de funciones de alto nivel, como la capacidad de agregar, editar o eliminar grupos o de agregar o eliminar usuarios.

#### Ejercicio 4.02 – Adición y modificación de usuarios y grupos a través de la aplicación de administración

En este ejercicio, otorgaremos un cierto nivel de acceso administrativo a una de nuestras usuarias de Bookr, Carol. Primero, definiremos el nivel de acceso para un grupo y luego agregaremos a Carol al grupo. Esto le permitirá a Carol actualizar los perfiles de los usuarios y verificar los registros de actividad (*logs*). Los siguientes pasos te ayudarán a implementar este ejercicio:

1. Visita la interfaz de administración en `http://127.0.0.1:8000/admin/` e inicia sesión como `bookradmin` utilizando la cuenta configurada con el comando de superusuario.

2. En la interfaz de administración, sigue los enlaces a **Home | AUTHENTICATION AND AUTHORIZATION | Groups**:
   *Figura 4.14 – Las opciones Groups y Users en la página AUTHENTICATION AND AUTHORIZATION*

3. Usa **ADD GROUP +** en la esquina superior derecha para agregar un nuevo grupo:
   *Figura 4.15 – Adición de un nuevo grupo*

4. Nombra al grupo `Help Desk User` y asígnale los siguientes permisos, como se muestra en la Figura 4.16:
   - `Can view log entry`
   - `Can view permission`
   - `Can change user`
   - `Can view user`
   *Figura 4.16 – Selección de permisos*

5. Esto se puede hacer seleccionando los permisos de *Available permissions* y haciendo clic en la flecha hacia la derecha en el centro para que aparezcan en *Chosen permissions*. Ten en cuenta que para agregar varios permisos a la vez, puedes mantener presionada la tecla Ctrl (o Command para Macintosh) para seleccionar más de uno:
   *Figura 4.17 – Adición de permisos seleccionados a Chosen permissions*

6. Una vez que hagas clic en el botón **SAVE**, verás un mensaje de confirmación que indica `The group "Help Desk User" was added successfully`:
   *Figura 4.18 – El grupo Help Desk User se agregó exitosamente*

7. Ahora, navega a **Home | AUTHENTICATION AND AUTHORIZATION | Users** y haz clic en el enlace del usuario con el primer nombre `carol`:
   *Figura 4.19 – Clic en el nombre de usuario carol*

8. Desplázate hacia abajo hasta el conjunto de campos *Permissions* y selecciona la casilla de verificación **Staff status**. Esto es necesario para que Carol pueda iniciar sesión en la aplicación de administración:
   *Figura 4.20 – Clic en la casilla de verificación Staff status*

9. Agrega a Carol al grupo `Help Desk User` que creamos en los pasos anteriores seleccionándolo en el cuadro de selección *Available groups* (consulta la Figura 4.20) y haciendo clic en la flecha hacia la derecha para pasarlo a su lista de *Chosen groups* (como se muestra en la Figura 4.21). Ten en cuenta que a menos que hagas esto, Carol no podrá iniciar sesión en la interfaz de administración con sus credenciales:
   *Figura 4.21 – Desplazamiento del grupo Help Desk User a la lista Chosen groups para Carol*

10. Probemos si lo que hemos hecho hasta ahora ha dado el resultado correcto. Para hacer esto, cierra sesión en el sitio de administración y vuelve a iniciar sesión como Carol. Al cerrar sesión, deberías ver lo siguiente en tu pantalla:
    *Figura 4.22 – La pantalla de cierre de sesión*
    Si no recuerdas la contraseña que le diste inicialmente, puedes cambiarla en la línea de comandos escribiendo `python manage.py changepassword carol`.

    *Figura 4.23 – El panel de control de administración*

Dado que no asignamos ningún permiso de grupo, ni siquiera `auth | group | Can view group`, al grupo `Help Desk User`, cuando Carol inicia sesión, la interfaz de administración de *Groups* no está disponible para ella. De manera similar, cuando navegas a **Home | AUTHENTICATION AND AUTHORIZATION | Users** y haces clic en el enlace de un usuario, verás que no hay opciones para editar o eliminar el usuario. Esto se debe a los permisos que se otorgaron al grupo `Help Desk User`, del cual Carol es miembro. Los miembros del grupo pueden ver y editar usuarios, pero no pueden agregar ni eliminar a ningún usuario.

En este ejercicio, aprendimos cómo otorgar una cierta cantidad de privilegios administrativos a los usuarios de nuestra aplicación Django.

Ahora que comprendemos cómo funciona la aplicación de administración con los modelos de Autenticación y Autorización para usuarios y grupos, podemos centrar nuestra atención en registrar los modelos que hemos desarrollado en la aplicación `reviews`.

---

### Sección: Registro de modelos con la aplicación de administración

La aplicación de administración de Django genera interfaces CRUD funcionales para objetos de Django según las características de los modelos, con una cantidad mínima de código. En esta sección, primero veremos cómo registrar los modelos con la aplicación de administración y cómo los elementos de la interfaz, como las listas de cambios y las páginas de cambios, se derivan de las propiedades del modelo. Concluiremos con un ejercicio importante que demuestra cómo la configuración de claves foráneas en el modelo determina el comportamiento del proceso de eliminación de objetos.

#### Registro del modelo reviews

Supongamos que Carol tiene la tarea de mejorar la sección de Reseñas en Bookr; es decir, solo se deben mostrar las reseñas más relevantes y completas, y se deben eliminar las entradas duplicadas o de spam. Para esto, necesitará acceder al modelo `reviews`. Como hemos visto anteriormente con nuestra investigación de grupos y usuarios, la aplicación de administración ya contiene páginas de administración para los modelos de la aplicación de Autenticación y Autorización, pero aún no hace referencia a los modelos de nuestra aplicación Reviews. Para que la aplicación de administración reconozca los modelos, debemos registrarlos explícitamente en ella. Afortunadamente, no necesitamos modificar el código de la aplicación de administración para hacerlo, ya que podemos importar la aplicación de administración a nuestro proyecto y usar su API para registrar nuestros modelos. Esto ya se ha hecho en la aplicación de Autenticación y Autorización, así que probémoslo con nuestra aplicación Reviews. Nuestro objetivo es poder utilizar la aplicación de administración para editar los datos de nuestro modelo de reseñas.

Echa un vistazo al archivo `reviews/admin.py`. Es un archivo de marcador de posición que se generó con el subcomando `startapp` que usamos en el Capítulo 1, y actualmente contiene estas líneas:

```python
from django.contrib import admin

# Register your models here.
```

Ahora, podemos intentar ampliar esto. Para que la aplicación de administración reconozca nuestros modelos, podemos modificar el archivo `reviews/admin.py` e importar los modelos. Luego, podemos registrar los modelos con el objeto `AdminSite`, `admin.site`. El objeto `AdminSite` contiene la instancia de la aplicación de administración de Django (más adelante, aprenderemos cómo crear subclases de este objeto `AdminSite` y anular muchas de sus propiedades). Entonces, `reviews/admin.py` se verá de la siguiente manera:

```python
from django.contrib import admin
from reviews.models import (Publisher, Contributor, Book, BookContributor, Review)

admin.site.register(Publisher)
admin.site.register(Contributor)
admin.site.register(Book)
admin.site.register(BookContributor)
admin.site.register(Review)
```

El método `admin.site.register` hace que los modelos estén disponibles para la aplicación de administración agregándolos a un registro de clases contenido en `admin.site._registry`. Si optáramos por no hacer accesible un modelo a través de la interfaz de administración, simplemente no lo registraríamos. Cuando recargues `http://127.0.0.1:8000/admin/` en tu navegador, verás lo siguiente en la página de inicio de la aplicación de administración:

*Figura 4.24 – Página de inicio de la aplicación de administración*

Observa el cambio en la apariencia de la página de administración después de que se ha importado el modelo `reviews`.

Con los cinco modelos de la aplicación Reviews registrados en la aplicación de administración, podemos examinar algunas de las interfaces predeterminadas que ha creado para nosotros.

#### Listas de cambios (Change lists)

Hemos visto varios ejemplos de páginas de listas de cambios hasta ahora en nuestro recorrido por la aplicación de administración de Django.

Una lista de cambios es una página que muestra una lista de enlaces a objetos que se recuperan mediante una consulta, y también opciones para realizar acciones en varios objetos (como la eliminación), agregar nuevos, filtrar o clasificar los objetos que se muestran.

Ahora que hemos registrado nuestros modelos con la aplicación de administración, podemos echar un vistazo a las listas de cambios predeterminadas que se han creado. Si hacemos clic en el enlace **Publishers**, se nos dirigirá a `http://127.0.0.1:8000/admin/reviews/publisher/` y veremos una lista de cambios que contiene enlaces a las editoriales. Estos enlaces están designados por el campo `id` de los objetos `Publisher`. Si tu base de datos se ha poblado con el script del Capítulo 3, verás una lista con siete editoriales, como se muestra en la Figura 4.25.

Dependiendo del estado de tu base de datos y según las actividades que hayas completado, los IDs de los objetos, las URLs y los enlaces en tu sesión pueden ordenarse y numerarse de manera diferente a las siguientes capturas de pantalla.

*Figura 4.25 – La lista Select publisher to change*

Podemos seguir uno de los enlaces en la lista de cambios y examinar la página *Change publisher*.

#### La página Change publisher

La página *Change publisher* en `http://127.0.0.1:8000/admin/reviews/publisher/1/` contiene lo que podríamos esperar (ver Figura 4.26). Hay un formulario para editar los detalles de la editorial. Estos detalles se han derivado de la clase `reviews.models.Publisher`:

*Figura 4.26 – La página Change publisher*

Si hubiéramos hecho clic en el botón **ADD PUBLISHER +**, la aplicación de administración habría devuelto un formulario similar para agregar una editorial. La belleza de la aplicación de administración es que nos brinda toda esta funcionalidad CRUD con solo una línea de código (`admin.site.register(Publisher)`), utilizando la definición de los atributos de `reviews.models.Publisher` como un esquema para el contenido de la página:

```python
class Publisher(models.Model):
    """A company that publishes books."""
    name = models.CharField(
        max_length=50,
        help_text="The name of the Publisher.")
    website = models.URLField(
        help_text="The Publisher's website.")
    email = models.EmailField(
        help_text="The Publisher's email address.")
```

El campo `name` de la editorial está restringido a 50 caracteres como se especifica en el modelo. El texto de ayuda que aparece en gris debajo de cada campo se deriva de los atributos `help_text` especificados en el modelo. Podemos ver que `models.CharField`, `models.URLField` y `models.EmailField` se representan en HTML como elementos de entrada de los tipos `text`, `url` y `email`, respectivamente.

Los campos del formulario vienen con validación cuando corresponde. A menos que los campos del modelo estén configurados con `blank=True` o `null=True`, el formulario arrojará un error si el campo se deja en blanco, como es el caso del campo `Publisher.name`. Del mismo modo, dado que `Publisher.website` y `Publisher.email` se definen respectivamente como instancias de `models.URLField` y `models.EmailField`, se validan en consecuencia. En la Figura 4.27, podemos ver la validación de *Name* como un campo obligatorio, *Website* como una URL y *Email* como una dirección de correo electrónico:

*Figura 4.27 – Validación de campos*

Es útil examinar cómo la aplicación de administración procesa los elementos de los modelos para comprender cómo funciona. En tu navegador, haz clic con el botón derecho en *Ver código fuente de la página* y examina el HTML que se ha generado para este formulario. Verás una pestaña del navegador que muestra un código HTML similar al de la Figura 4.28.

*Figura 4.28 – Navegador mostrando código HTML*

El formulario tiene un ID `publisher_form` y contiene un conjunto de campos con elementos HTML correspondientes a la estructura de datos del modelo `Publisher` en `reviews/models.py`, que se muestra a continuación:

```python
class Publisher(models.Model):
    """A company that publishes books."""
    name = models.CharField(
        max_length=50,
        help_text="The name of the Publisher.")
    website = models.URLField(
        help_text="The Publisher's website.")
    email = models.EmailField(
        help_text="The Publisher's email address.")
```

Ten en cuenta que para el nombre, el campo de entrada se representa así:

```html
<input type="text" name="name" value="Packt Publishing" class="vTextField" maxlength="50" required="" id="id_name">
```

Es un campo obligatorio, tiene un tipo de texto y un `maxlength` de 50, según lo definido por el parámetro `max_length` en la definición del modelo:

```python
name = models.CharField(max_length=50, help_text="The name of the Publisher.")
```

De manera similar, podemos ver el sitio web y el correo electrónico definidos en el modelo, ya que `URLField` y `EmailField` se representan en HTML como elementos de entrada de los tipos `url` y `email`, respectivamente:

```html
<input type="url" name="website" value="https://www.packtpub.com/" class="vURLField" maxlength="200" required="" id="id_website">
<input type="email" name="email" value="info@packtpub.com" class="vTextField" maxlength="254" required="" id="id_email">
```

Hemos aprendido que la aplicación de administración de Django genera representaciones HTML sensibles de los modelos de Django basadas en las definiciones de modelo que hemos proporcionado.

Ahora examinemos la página de cambio de libro.

#### La página de cambio de libro (The book change page)

De manera similar, hay una página de cambios a la que se puede acceder seleccionando **Books** en la página de administración del sitio y luego seleccionando un libro específico en la lista de cambios:

*Figura 4.29 – Selección de Books en la página de administración del sitio*

Después de hacer clic en **Books**, como se muestra en la captura de pantalla anterior, verás lo siguiente en tu pantalla:

*Figura 4.30 – La página Select book to change*

En este caso, seleccionar el libro *Architects of Intelligence* nos llevará a `http://127.0.0.1:8000/admin/reviews/book/3/change/`. En el ejemplo anterior, todos los campos del modelo se representaron como widgets de texto HTML simples. La representación de algunas otras subclases de `django.db.models.Field` utilizadas en `models.Book` es digna de un examen más detenido:

*Figura 4.31 – La página Change book*

Aquí, `publication_date` se define usando `models.DateField`. Se representa mediante un widget de selección de fecha. La representación visual de los widgets variará según los sistemas operativos y la elección del navegador:

*Figura 4.32 – Widget de selección de fecha*

Como `Publisher` se define como una relación de clave foránea, se representa como un menú desplegable con una lista de los objetos `Publisher`:

*Figura 4.33 – Menú desplegable de Publisher*

Esto nos lleva a cómo la aplicación de administración maneja la eliminación. La aplicación de administración se guía por las restricciones de clave foránea de los modelos al determinar cómo implementar la funcionalidad de eliminación. En el modelo `BookContributor`, `Contributor` se define como una clave foránea. El código en `reviews/models.py` se ve de la siguiente manera:

```python
contributor = models.ForeignKey(Contributor, on_delete=models.CASCADE)
```

Al establecer `on_delete=CASCADE` en una clave foránea, el modelo especifica el comportamiento de la base de datos requerido cuando se elimina un registro; la eliminación se propaga en cascada a otros objetos a los que hace referencia la clave foránea.

Es importante comprender el impacto de la configuración de claves foráneas en el flujo del proceso de eliminación, y el siguiente ejercicio examina esto.

#### Ejercicio 4.03 – Claves foráneas y comportamiento de eliminación en la aplicación de administración

Actualmente, todas las relaciones `ForeignKey` en los modelos de `reviews` están definidas con un comportamiento `on_delete=CASCADE`. Por ejemplo, piensa en un caso en el que un administrador elimine una de las editoriales. Esto eliminaría todos los libros asociados con la editorial. No queremos que eso suceda, y ese es precisamente el comportamiento que cambiaremos en este ejercicio:

1. Visita la lista de cambios de colaboradores en `http://127.0.0.1:8000/admin/reviews/contributor/` y selecciona un colaborador para eliminarlo. Asegúrate de que el colaborador sea el autor de un libro.

2. Haz clic en el botón **Delete**, pero no hagas clic en *Yes, I'm sure* en el cuadro de diálogo de confirmación. Verás un mensaje como el de la Figura 4.34:
   *Figura 4.34 – Cuadro de diálogo de confirmación de eliminación en cascada*
   De acuerdo con el argumento `on_delete=CASCADE` en la clave foránea, se nos advierte que eliminar este objeto `Contributor` tendrá un efecto en cascada en un objeto `BookContributor`.

3. En el archivo `reviews/models.py`, modifica el atributo `contributor` de `BookContributor` por lo siguiente y guarda el archivo:
   ```python
   contributor = models.ForeignKey(Contributor, on_delete=models.PROTECT)
   ```

4. Ahora, intenta eliminar el objeto `Contributor` nuevamente. Verás un mensaje similar al de la Figura 4.35:
   *Figura 4.35 – Error de protección de clave foránea*
   Debido a que el argumento `on_delete` es `PROTECT`, nuestro intento de eliminar el objeto con dependencias arrojará un error. Si utilizáramos este enfoque en nuestro modelo, tendríamos que eliminar los objetos en la relación `ForeignKey` antes de eliminar el objeto original. En este caso, significaría eliminar el objeto `BookContributor` antes de eliminar el objeto `Contributor`.

5. Ahora que hemos aprendido cómo la aplicación de administración maneja las relaciones `ForeignKey`, revirtamos la definición de `ForeignKey` en la clase `BookContributor` a la siguiente:
   ```python
   contributor = models.ForeignKey(Contributor, on_delete=models.CASCADE)
   ```

Hemos examinado cómo el comportamiento de la aplicación de administración se adapta a las restricciones `ForeignKey` que se expresan en las definiciones de los modelos. Si el comportamiento de `on_delete` se establece en `models.PROTECT`, la aplicación de administración devuelve un error explicando por qué un objeto protegido está bloqueando la eliminación. Esta funcionalidad puede resultar útil al crear aplicaciones del mundo real, ya que a menudo existe la posibilidad de que un error manual provoque inadvertidamente la eliminación de registros importantes. En la siguiente sección, veremos cómo podemos personalizar la interfaz de nuestra aplicación de administración para una experiencia de usuario más fluida.

---

### Sección: Personalización de la interfaz de administración

Al desarrollar una aplicación por primera vez, la conveniencia de la interfaz de administración predeterminada es excelente para crear un prototipo rápido de la aplicación. De hecho, para muchas aplicaciones o proyectos más simples que requieren un mantenimiento de datos mínimo, esta interfaz de administración predeterminada puede ser completamente adecuada. Sin embargo, a medida que la aplicación madura hasta el punto de su lanzamiento, la interfaz de administración generalmente necesitará ser personalizada para facilitar un uso más intuitivo y controlar de manera sólida los datos, sujetos a los permisos del usuario. Es posible que desees conservar ciertos aspectos de la interfaz de administración predeterminada y, al mismo tiempo, hacer algunos ajustes a ciertas funciones para adaptarlas mejor a tus propósitos. Por ejemplo, querrás que la lista de editoriales muestre los nombres completos de las editoriales en lugar de `Publisher(1)`, `Publisher(2)`, etc. Además del atractivo estético, esto facilita el uso y la navegación por la aplicación.

En esta sección, examinaremos el objeto `AdminSite` y aprenderemos sobre algunas de sus propiedades importantes que se pueden utilizar para modificar la apariencia global de la aplicación de administración. Luego, examinaremos enfoques para crear subclases de `AdminSite` dentro de nuestro proyecto para que podamos personalizarlo a partir de su apariencia predeterminada. Terminaremos probando nuestro conocimiento de estas técnicas mediante la creación de un nuevo proyecto y la personalización de su interfaz de administración.

#### Personalizaciones de Django admin en todo el sitio

Vimos en la Figura 4.2 una página titulada *Log in | Django site admin* que contenía un formulario de administración de Django. Sin embargo, un usuario administrativo de la aplicación Bookr puede sentirse algo perplejo por toda esta jerga de Django, y sería muy confuso y una receta para el error si tuviera que lidiar con múltiples aplicaciones de Django que tuvieran aplicaciones de administración idénticas. Como desarrollador de una aplicación intuitiva y fácil de usar, debes personalizar esto. Propiedades globales como estas se especifican como atributos del objeto `AdminSite`. La siguiente tabla detalla algunas de las personalizaciones más simples para mejorar la usabilidad de la interfaz de administración de tu aplicación:

| Atributo de AdminSite | Valor base | Descripción |
| :--- | :--- | :--- |
| `site_title` | `"Django site admin"` | Rellena la etiqueta `<title>` en cada página de la interfaz de administración. |
| `site_header` | `"Django administration"` | Establece el encabezado en el formulario de inicio de sesión. |
| `index_title` | `"Site administration"` | Establece el encabezado en la página de índice de administración (donde se enumeran los modelos). |
| `index_template` | `None` | Proporciona la ruta para encontrar la plantilla de índice de administración. Si no se establece, se utiliza la plantilla `admin/index.html`. |
| `app_index_template` | `None` | Proporciona la ruta para encontrar la plantilla de índice de administración de la aplicación. Si no se establece, se utiliza la plantilla `admin/app_index.html`. |
| `login_template` | `None` | Proporciona la ruta para encontrar la plantilla de inicio de sesión. Si no se establece, se utiliza la plantilla `admin/login.html`. |
| `logout_template` | `None` | Proporciona la ruta para encontrar la plantilla de cierre de sesión. Si no se establece, se utiliza la plantilla `registration/logged_out.html`. |
| `password_change_template` | `None` | Proporciona la ruta para encontrar la plantilla de cambio de contraseña. Si no se establece, se utiliza la plantilla `registration/password_change_form.html`. |
| `password_change_done_template` | `None` | Proporciona la ruta para encontrar la plantilla de confirmación de cambio de contraseña. Si no se establece, se utiliza la plantilla `registration/password_change_done.html`. |

*Figura 4.36 – Atributos importantes de AdminSite*

Si esto parece un poco abstracto, podría ser útil examinar las propiedades de esta tabla mediante la consola de Django.

#### Examen del objeto AdminSite desde la consola de Python

Echemos un vistazo más profundo a la clase `AdminSite`. Ya nos hemos encontrado con un objeto de la clase `AdminSite`. Es el objeto `admin.site` que usamos en la sección de registro del modelo de reseñas. Si el servidor de desarrollo no se está ejecutando, inícialo ahora con el subcomando `runserver` de la siguiente manera:

```bash
python manage.py runserver
```

Podemos examinar el objeto `admin.site` importando la aplicación de administración en la consola de Django, utilizando el script `manage.py` nuevamente:

```python
python manage.py shell
>>> from django.contrib import admin
```

Podemos examinar interactivamente los valores predeterminados de `site_title`, `site_header` e `index_title` y ver que coinciden con los valores esperados de `'Django site admin'`, `'Django administration'` y `'Site administration'` que ya hemos observado en las páginas web renderizadas de la aplicación de administración de Django:

```python
>>> admin.site.site_title
'Django site admin'
>>> admin.site.site_header
'Django administration'
>>> admin.site.index_title
'Site administration'
```

La clase `AdminSite` también especifica qué formularios y vistas se utilizan para representar la interfaz de administración y determinar su comportamiento global.

Ahora que hemos examinado las propiedades del objeto, intentaremos crear una subclase para poder personalizarlas en nuestro proyecto de Django.

#### Subclase de AdminSite: un primer enfoque

Podemos hacer algunas modificaciones en el archivo `reviews/admin.py`. En lugar de importar el módulo `django.contrib.admin` y usar su objeto `site`, importaremos `AdminSite`, crearemos una subclase y crearemos una instancia de nuestro objeto `admin_site` personalizado. Considera el siguiente fragmento de código. Aquí, `BookrAdminSite` es una subclase de `AdminSite` que contiene valores personalizados para `site_title`, `site_header` e `index_title`; `admin_site` es una instancia de `BookrAdminSite`, y podemos usar esto en lugar del objeto predeterminado `admin.site` para registrar nuestros modelos. El archivo `reviews/admin.py` se verá de la siguiente manera:

```python
from django.contrib.admin import AdminSite
from reviews.models import (Publisher, Contributor, Book, BookContributor, Review)

class BookrAdminSite(AdminSite):
    title_header = 'Bookr Admin'
    site_header = 'Bookr administration'
    index_title = 'Bookr site admin'

admin_site = BookrAdminSite(name='bookr')

admin_site.register(Publisher)
admin_site.register(Contributor)
admin_site.register(Book)
admin_site.register(BookContributor)
admin_site.register(Review)
```

Ahora que hemos creado nuestro propio objeto `admin_site` que anula el comportamiento del objeto `admin.site`, debemos eliminar las referencias existentes en nuestro código al objeto `admin.site`. En `bookr/bookr/urls.py`, debemos apuntar `admin` al nuevo objeto `admin_site` y actualizar nuestros patrones de URL. De lo contrario, seguiríamos usando el sitio de administración predeterminado y nuestras personalizaciones se ignorarían. El cambio se verá de la siguiente manera:

```python
from reviews.admin import admin_site
import reviews.views
from django.urls import include, path

urlpatterns = [
    path("admin/", admin_site.urls),
    path("book-search", reviews.views.book_search),
    path("", include("reviews.urls")),
]
```

Esto produce los resultados esperados en la pantalla de inicio de sesión:

*Figura 4.37 – Personalización de la pantalla de inicio de sesión*

Sin embargo, ahora hay un problema: hemos perdido la interfaz para los objetos de autenticación (`auth`). Anteriormente, la aplicación de administración descubría los modelos registrados en `reviews/admin.py` y `django.contrib.auth.admin` a través del proceso de descubrimiento automático (*autodiscovery*), pero ahora hemos anulado este comportamiento al crear un nuevo `AdminSite`.

*Figura 4.38 – El AdminSite personalizado carece de autenticación y autorización*

Podríamos tomar el camino de hacer referencia a ambos objetos `AdminSite` en los patrones de URL en `bookr/bookr/urls.py`, pero este enfoque significaría que terminaríamos con dos aplicaciones de administración separadas para autenticación y reseñas. Por lo tanto, la URL `http://127.0.0.1:8000/admin` te llevará a la aplicación de administración original derivada del objeto `admin.site`, mientras que `http://127.0.0.1:8000/bookradmin` te llevará a nuestro `BookrAdminSite`, `admin_site`. Esto no es lo que queremos hacer, ya que todavía nos queda la aplicación de administración sin las personalizaciones que agregamos cuando creamos la subclase `BookrAdminSite`:

```python
from django.contrib import admin
from reviews.admin import admin_site
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('bookradmin/', admin_site.urls),
]
```

Para evitar este problema, veamos qué podemos hacer en la siguiente subsección.

#### Subclase de AdminSite: una mejor manera

Este ha sido un problema complejo con la interfaz de administración de Django que ha llevado a muchas soluciones *ad hoc* en versiones anteriores. Sin embargo, desde que se lanzó Django 2.1, ha habido una forma sencilla de integrar una interfaz personalizada para la aplicación de administración sin interrumpir el descubrimiento automático ni ninguna de sus otras funciones predeterminadas. Veamos cómo hacer esto con los siguientes pasos:

1. Como `BookrAdminSite` es específico del proyecto, el código realmente no pertenece a nuestra carpeta `reviews`. Por lo tanto, mueve `BookrAdminSite` a un nuevo archivo, `bookr/bookr/admin.py`, en el nivel superior del directorio del proyecto Bookr:
   ```python
   from django.contrib import admin

   class BookrAdminSite(admin.AdminSite):
       title_header = 'Bookr Admin'
       site_header = 'Bookr administration'
       index_title = 'Bookr site admin'
   ```
   La ruta de configuración de URLs en `bookr/bookr/urls.py` cambia a `path('admin/', admin.site.urls)`, y definimos `ReviewsAdminConfig`.

2. Agrega un archivo llamado `reviews/adminconfig.py` que contenga estas líneas:
   ```python
   from django.contrib.admin.apps import AdminConfig

   class ReviewsAdminConfig(AdminConfig):
       default_site = "admin.BookrAdminSite"
   ```

3. Reemplaza `django.contrib.admin` con `reviews.adminconfig.ReviewsAdminConfig`, de modo que `INSTALLED_APPS` en el archivo `bookr/bookr/settings.py` se verá de la siguiente manera:
   ```python
   INSTALLED_APPS = [
       'reviews.adminconfig.ReviewsAdminConfig',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'reviews'
   ]
   ```

Con la especificación `ReviewsAdminConfig` de `default_site`, ya no necesitamos reemplazar las referencias a `admin.site` con un objeto `AdminSite` personalizado, `admin_site`. Podemos reemplazar esas llamadas a `admin_site` con las llamadas a `admin.site` que teníamos originalmente. Ahora, `reviews/admin.py` vuelve a lo siguiente:

```python
from django.contrib import admin
from reviews.models import (Publisher, Contributor, Book, BookContributor, Review)

admin.site.register(Publisher)
admin.site.register(Contributor)
admin.site.register(Book, BookAdmin)
admin.site.register(BookContributor)
admin.site.register(Review)
```

Hay otros aspectos de `AdminSite` que podemos personalizar, pero los veremos en el Capítulo 9, una vez que tengamos una comprensión más completa de las plantillas y los formularios de Django.

La siguiente actividad tiene como objetivo evaluar las habilidades que has aprendido para configurar y personalizar la aplicación de administración de Django para un nuevo proyecto de Django.

#### Actividad 4.02 – Personalización del objeto AdminSite

Has aprendido a modificar los atributos del objeto `AdminSite` en un proyecto de Django. Esta actividad te desafiará a utilizar estas habilidades para personalizar un nuevo proyecto y anular su título de sitio, encabezado de sitio y encabezado de índice. Además, reemplazarás el mensaje de cierre de sesión creando una plantilla específica del proyecto y configurándola en nuestro objeto `AdminSite` personalizado. Estás desarrollando un proyecto de Django que implementa un tablero de mensajes llamado **Comment8or**. Comment8or está orientado a un grupo demográfico técnico, por lo que debes hacer que la fraseología sea sucinta y abreviada:
- El sitio de administración de Comment8or se denominará `c8admin`. Esto aparecerá en el encabezado del sitio y en el título del índice.
- Para el encabezado del título, dirá `c8 site admin`.
- El mensaje de cierre de sesión predeterminado de administración de Django es `Thanks for spending some quality time with the Web site today.`. En Comment8or, dirá `Bye from c8admin`.

Estos son los pasos que debes seguir para completar esta actividad:
1. Siguiendo el proceso que aprendiste en el Capítulo 1, crea un nuevo proyecto de Django llamado `comment8or`, una aplicación llamada `messageboard` y ejecuta las migraciones. Crea un superusuario llamado `c8admin`.
2. En el código fuente de Django, hay una plantilla para la página de cierre de sesión ubicada en `django/contrib/admin/templates/registration/logged_out.html`. Haz una copia de ella en el directorio de tu proyecto en `bookr/comment8or/templates/comment8or` y modifica el mensaje en la plantilla siguiendo los requisitos.
3. Dentro del proyecto, crea un archivo `admin.py` que implemente un objeto `AdminSite` personalizado. Establece los valores apropiados para los atributos `index_title`, `title_header`, `site_header` y `logout_template`, según los requisitos.
4. Agrega una subclase personalizada de `AdminConfig` a `bookr/messageboard/adminconfig.py`.
5. Reemplaza la aplicación de administración con la subclase personalizada `AdminConfig` en `bookr/comment8or/settings.py`.
6. Configura el ajuste `TEMPLATES` para que la plantilla del proyecto sea reconocible.

Cuando se creó el proyecto por primera vez, la página de inicio de sesión se veía así:
*Figura 4.39 – Página de inicio de sesión para el proyecto*

La página de índice de la aplicación se veía así:
*Figura 4.40 – Página de índice de la aplicación para el proyecto*

Finalmente, la página de cierre de sesión se veía así:
*Figura 4.41 – Página de cierre de sesión para el proyecto*

Después de haber completado esta actividad con todas las personalizaciones, la página de inicio de sesión se verá así:
*Figura 4.42 – Página de inicio de sesión después de la personalización*

La página de índice de la aplicación se verá así:
*Figura 4.43 – Página de índice de la aplicación después de la personalización*

Finalmente, la página de cierre de sesión se verá así:
*Figura 4.44 – Página de cierre de sesión después de la personalización*

Has personalizado exitosamente la aplicación de administración mediante la creación de subclases de `AdminSite`.

En la siguiente sección, examinaremos las clases `ModelAdmin` y aprenderemos cómo se pueden utilizar para personalizar la interfaz de administración para modelos individuales.

---

### Sección: Personalización de las clases ModelAdmin

Ahora que hemos aprendido cómo se puede usar una subclase de `AdminSite` para personalizar la apariencia global de la aplicación de administración, veremos cómo personalizar la interfaz de la aplicación de administración para modelos individuales. Debido a que la interfaz de administración se genera automáticamente a partir de la estructura de los modelos, tiene una apariencia demasiado genérica y debe personalizarse por cuestiones de estética y facilidad de uso. Haz clic en uno de los enlaces de **Books** en la aplicación de administración y compáralo con el enlace de **Users**. Ambos enlaces te llevan a las páginas de listas de cambios. Estas son las páginas que visita un administrador de Bookr cuando quiere agregar libros nuevos o agregar o alterar los privilegios de un usuario. Como se explicó anteriormente en la sección *Listas de cambios*, una página de lista de cambios presenta una lista de objetos de modelo con la opción de seleccionar un grupo de ellos para su eliminación masiva (u otra actividad masiva), examinar un objeto individual para editarlo o agregar un nuevo objeto. Observa la diferencia entre las dos páginas de listas de cambios con el fin de hacer que nuestra página básica de Books sea tan completa como la página de Users. La siguiente captura de pantalla de la aplicación de Autenticación y Autorización contiene funciones útiles como una barra de búsqueda, encabezados de columna ordenables para campos de usuario importantes y un filtro de resultados:

*Figura 4.45 – La lista de cambios de Users contiene las funciones personalizadas de ModelAdmin*

Al crear una subclase de `ModelAdmin`, podemos crear una clase que establezca la funcionalidad para las páginas de administración de un modelo específico.

En esta sección, aprenderemos cómo se pueden desarrollar las clases `ModelAdmin` para personalizar los campos en la página de lista de cambios e implementar filtros y barras de búsqueda. Veremos cómo se pueden agrupar los campos del modelo en formularios CRUD de administración o excluirse del formulario si los campos deben conservar un valor predeterminado o generado por el sistema.

#### Los campos de visualización de lista (The list display fields)

En la página de lista de cambios de Users, verás lo siguiente:
- Se presenta una lista de objetos de usuario, resumidos por sus atributos `USERNAME`, `EMAIL ADDRESS`, `FIRST NAME`, `LAST NAME` y `STAFF STATUS`.
- Estos atributos individuales son ordenables. El orden de clasificación se puede cambiar haciendo clic en los encabezados.
- Hay una barra de búsqueda en la parte superior de la página.
- En la columna de la derecha, hay un filtro de selección que permite la selección de varios campos de usuario, incluidos algunos que no aparecen en la visualización de la lista.

Sin embargo, el comportamiento de la página de lista de cambios de Books es mucho menos útil. Los libros se enumeran por sus títulos, pero no en orden alfabético. La columna de título no se puede ordenar y no hay opciones de filtro o búsqueda presentes:

*Figura 4.46 – La lista de cambios de Books*

Recuerda del Capítulo 2 que definimos los métodos `__str__` en las clases `Publisher`, `Book` y `Contributor`. En el caso de la clase `Book`, tenía una representación `__str__()` que devuelve el título del objeto del libro:

```python
class Book(models.Model):
    ...
    def __str__(self):
        return self.title
```

Si no hubiéramos definido el método `__str__()` en la clase `Book`, lo habría heredado de la clase base `Model`, `django.db.models.Model`.
Esta clase base proporciona una forma abstracta de dar una representación de cadena de un objeto. Por ejemplo, cuando tenemos `Book` con una clave primaria, en este caso, el campo `id` con un valor de 17, terminaremos con una representación de cadena de `Book object (17)`:

*Figura 4.47 – La lista de cambios de Books utilizando la representación __str__ del modelo*

Podría ser útil en nuestra aplicación representar un objeto `Book` como un compuesto de varios campos. Por ejemplo, si quisiéramos que los libros se representaran como `Title (ISBN)`, el siguiente fragmento de código produciría los resultados deseados:

```python
class Book(models.Model):
    ...
    def __str__(self):
        return f"{self.title} ({self.isbn})"
```

Este es un cambio útil en sí mismo, ya que hace que la representación del objeto sea más intuitiva en la aplicación:

*Figura 4.48 – Una porción de la lista de cambios de Books con la representación de cadena personalizada*

No estamos limitados a usar la representación `__str__` del objeto en el campo `list_display`. Las columnas que aparecen en la visualización de lista están determinadas por la clase `ModelAdmin` de la aplicación de administración de Django. En la consola de Django, podemos importar la clase `ModelAdmin` y examinar su atributo `list_display`:

```python
python manage.py shell
>>> from django.contrib.admin import ModelAdmin
>>> ModelAdmin.list_display
('__str__',)
```

Esto explica por qué el comportamiento predeterminado de `list_display` es mostrar una tabla de una sola columna de las representaciones `__str__` de los objetos, de modo que podamos personalizar la visualización de la lista anulando este valor. La mejor práctica es crear una subclase de `ModelAdmin` para cada objeto. Si quisiéramos que la visualización de la lista de libros contuviera dos columnas separadas para *Title* e *ISBN*, en lugar de tener una sola columna que contenga ambos valores, como en la Figura 4.48, crearíamos una subclase de `ModelAdmin` llamada `BookAdmin` y especificaríamos el `list_display` personalizado. El beneficio de hacer esto es que ahora podemos ordenar los libros por título y por ISBN. Podemos agregar esta clase a `reviews/admin.py`:

```python
class BookAdmin(admin.ModelAdmin):
    list_display = ("title", "isbn")
```

Ahora que hemos creado una clase `BookAdmin`, debemos hacer referencia a ella cuando registremos nuestra clase `reviews.models.Book` en el sitio de administración. En el mismo archivo, también debemos modificar el registro del modelo para usar `BookAdmin` en lugar del valor predeterminado de `admin.ModelAdmin`, por lo que la llamada `admin.site.register` ahora se convierte en la siguiente:

```python
admin.site.register(Book, BookAdmin)
```

Una vez que se hayan realizado estos dos cambios en el archivo `reviews/admin.py`, obtendremos una página de lista de cambios de Books que se verá así:

*Figura 4.49 – Una porción de la lista de cambios de Books con una visualización de lista de dos columnas*

Esto nos da una idea de cuán flexible es `list_display`. Puede tomar cuatro tipos de valores:
1. Toma nombres de campo del modelo, como `title` o `isbn`.
2. Toma una función que toma la instancia del modelo como argumento, como esta función que proporciona una versión con iniciales del nombre de una persona:
   ```python
   def initialled_name(obj):
       """
       obj.first_names='Jerome David', obj.last_names='Salinger' => 'Salinger, JD'
       obj.first_names='Plato', obj.last_names='' => 'Plato'
       """
       initials = "".join([name[0] for name in obj.first_names.split(" ")])
       if obj.last_names:
           return f"{obj.last_names}, {initials}"
       return obj.first_names

   class ContributorAdmin(admin.ModelAdmin):
       list_display = (initialled_name,)
   ```
3. Toma un método de la subclase `ModelAdmin` que toma el objeto del modelo como un único argumento. Ten en cuenta que esto debe especificarse como un argumento de cadena, ya que estaría fuera de alcance y no definido dentro de la clase:
   ```python
   class BookAdmin(admin.ModelAdmin):
       list_display = ("title", "isbn13")

       def isbn13(self, obj):
           """
           '9780316769174' => '978-0-31-676917-4'
           '0316769174' => '0316769174'
           None => ''
           """
           if obj.isbn:
               if len(obj.isbn) == 13:
                   return "-".join([obj.isbn[0:3], obj.isbn[3:4], obj.isbn[4:6], obj.isbn[6:12], obj.isbn[12:13]])
               return obj.isbn
           return ""
   ```
4. Toma un método (o un atributo que no sea un campo) de la clase del modelo, como `__str__`, siempre que acepte el objeto del modelo como argumento. Por ejemplo, podríamos convertir `isbn13` en un método en la clase de modelo `Book`:
   ```python
   class Book(models.Model):
       def isbn13(self):
           """
           '9780316769174' => '978-0-31-676917-4'
           '0316769174' => '0316769174'
           None => ''
           """
           if self.isbn:
               if len(self.isbn) == 13:
                   return "-".join([self.isbn[0:3], self.isbn[3:4], self.isbn[4:6], self.isbn[6:12], self.isbn[12:13]])
               return self.isbn
           return ""
   ```

Ahora, al ver la lista de cambios de Books en `http://127.0.0.1:8000/admin/reviews/book/`, podemos ver el campo `ISBN13` con guiones:

*Figura 4.50 – Una porción de la lista de cambios de Books con el ISBN13 con guiones*

Vale la pena señalar que los campos calculados como `__str__` o nuestros métodos `isbn13` no generan campos ordenables en la página de resumen a menos que se especifique. Además, no podemos incluir campos del tipo `ManyToManyField` en `list_display`.

Si bien las columnas que se derivan de los atributos del modelo obtienen sus encabezados y propiedades de columna de los atributos del campo, podemos especificar propiedades de las columnas de visualización que son campos calculados mediante el decorador `display`.

#### El decorador de visualización (The display decorator)

Cuando se utiliza un elemento invocable (*callable*) en `list_display`, como en los casos de `initialled_name` e `isbn13`, podemos usar el decorador `admin.display` para especificar el nombre de la columna que aparecerá en el encabezado de la lista de cambios mediante el argumento `description`. También podemos usarlo para evitar la limitación de que los campos calculados no se puedan ordenar especificando `ordering` en el invocable. El argumento `empty_value` se puede utilizar para especificar cómo se muestra un valor `None` o una cadena vacía. La visualización de `empty_value` predeterminada es un solo carácter de guion:

```python
@admin.display(
    ordering="isbn",
    description="ISBN-13",
    empty_value="-"
)
def isbn13(self, obj):
    """
    '9780316769174' => '978-0-31-676917-4'
    '0316769174' => '0316769174'
    None => ''
    """
    ...
```

El argumento `boolean` de `admin.display` se puede utilizar para marcar un valor que se representará en forma booleana:

```python
@admin.display(
    boolean=True,
    description="Has ISBN",
)
def has_isbn(self, obj):
    """
    '9780316769174' => True
    """
    return bool(obj.isbn)
```

En conjunto, estas configuraciones del decorador de visualización nos darán columnas de visualización que se verán así:

*Figura 4.51 – La lista de cambios de Books con la configuración de admin display*

#### El filtro (The filter)

Una vez que la interfaz de administración necesita lidiar con una cantidad significativa de registros, es conveniente reducir los resultados que aparecen en las páginas de listas de cambios. Los filtros más simples seleccionan valores individuales. Por ejemplo, el filtro de usuario representado en la Figura 4.6 permite la selección de usuarios eligiendo *By staff status*, *By superuser status* y *By active*. Hemos visto en el filtro de usuario que `BooleanField` se puede utilizar como filtro. También podemos implementar filtros en `CharField`, `DateField`, `DateTimeField`, `IntegerField`, `ForeignKey` y `ManyToManyField`. En este caso, al agregar `publisher` como una `ForeignKey` de `Book`, se define en la clase `Book` de la siguiente manera:

```python
publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
```

Los filtros se implementan mediante el atributo `list_filter` de una subclase de `ModelAdmin`. En nuestra aplicación Bookr, filtrar por título de libro o ISBN sería poco práctico, ya que produciría una gran lista de opciones de filtro que devolverían solo un registro. El filtro que ocuparía el lado derecho de la página ocuparía más espacio que la lista de cambios real. Una opción práctica sería filtrar los libros por editorial. Definimos un método `__str__` personalizado para el modelo `Publisher` que devuelve el atributo de nombre de la editorial, por lo que nuestras opciones de filtro se mostrarán como nombres de editoriales.

Podemos especificar nuestro filtro de lista de cambios en `reviews/admin.py` en la clase `BookAdmin` de esta manera:

```python
list_filter = ("publisher",)
```

Así es como debería verse ahora la página de lista de cambios de Books:

*Figura 4.52 – La página de lista de cambios de Books con el filtro de editorial*

Con esa línea de código, hemos implementado un útil filtro de editoriales en la página de lista de cambios de Books.
Para consolidar nuestro conocimiento sobre filtros, agregaremos más filtros a la página de lista de cambios de Books.

#### Ejercicio 4.04 – Adición de los filtros date list_filter y date_hierarchy

Hemos visto que la clase `admin.ModelAdmin` proporciona atributos útiles para personalizar filtros en las páginas de listas de cambios. Por ejemplo, filtrar por fecha es una funcionalidad crucial para muchas aplicaciones y también puede ayudarnos a hacer que nuestra aplicación sea más fácil de usar. En este ejercicio, examinaremos cómo se puede implementar el filtrado por fecha incluyendo un campo de fecha en el filtro y veremos el filtro `date_hierarchy`:

1. Edita el archivo `reviews/admin.py` y modifica el atributo `list_filter` en la clase `BookAdmin` para incluir `'publication_date'`:
   ```python
   class BookAdmin(admin.ModelAdmin):
       list_display = ("title", "isbn13")
       list_filter = ("publisher", "publication_date")
   ```

2. Recarga la página de lista de cambios de Books y confirma que el filtro ahora incluye configuraciones de fecha:
   *Figura 4.53 – Confirmación de que la página de lista de cambios de Books incluye configuraciones de fecha*
   Este filtro de fecha de publicación sería conveniente si el proyecto Bookr estuviera recibiendo muchos lanzamientos nuevos y quisiéramos filtrar los libros por lo que se publicó en los últimos siete días o un mes. A veces, sin embargo, nos gustaría filtrar por un año específico o un mes específico en un año específico. Afortunadamente, la clase `admin.ModelAdmin` viene con un atributo de filtro personalizado orientado a navegar por jerarquías de información temporal. Se llama `date_hierarchy`.

3. Agrega un atributo `date_hierarchy` a `BookAdmin` y establece su valor en `publication_date`:
   ```python
   class BookAdmin(admin.ModelAdmin):
       date_hierarchy = "publication_date"
       list_display = ("title", "isbn13")
       list_filter = ("publisher", "publication_date")
   ```

4. Recarga la página de lista de cambios de Books y confirma que la jerarquía de fechas aparece arriba del menú desplegable *Action*:
   *Figura 4.54 – Confirmación de que la jerarquía de fechas aparece arriba del menú desplegable Action*

5. Selecciona un año de la jerarquía de fechas y confirma que contiene una lista de meses en ese año que contienen títulos de libros y una lista total de libros:
   *Figura 4.55 – Confirmación de que la selección de un año de la jerarquía de fechas muestra los libros publicados ese año*

6. Confirma que al seleccionar uno de estos meses se filtra aún más por días del mes:
   *Figura 4.56 – Filtrado de meses hasta días del mes*

El filtro `date_hierarchy` es una forma conveniente de personalizar una lista de cambios que contiene un gran conjunto de datos ordenables por tiempo para facilitar una selección de registros más rápida, como vimos en este ejercicio.

Veamos ahora la implementación de una barra de búsqueda en nuestra aplicación.

#### La barra de búsqueda (The search bar)

Esto nos lleva a la parte restante de la funcionalidad que queríamos implementar: la barra de búsqueda. Al igual que los filtros, una barra de búsqueda básica es bastante sencilla de implementar. Solo necesitamos agregar el atributo `search_fields` a la clase `ModelAdmin`. Los campos de caracteres obvios en nuestra clase `Book` para buscar son `title` e `isbn`. Actualmente, la lista de cambios de Books aparece con una jerarquía de fechas en la parte superior de la lista de cambios. La barra de búsqueda aparecerá encima de esto:

*Figura 4.57 – La lista de cambios de Books antes de agregar la barra de búsqueda*

Podemos comenzar agregando este atributo a `BookAdmin` en `reviews/admin.py` y examinando el resultado:

```python
search_fields = ("title", "isbn")
```

El resultado se vería así:

*Figura 4.58 – La lista de cambios de Books con la barra de búsqueda*

Ahora, podemos realizar una búsqueda de texto simple en campos que coincidan con el campo de título o el ISBN. Esta búsqueda requiere coincidencias exactas de cadenas, por lo que "color" no coincidirá con "colour". También carece del procesamiento semántico profundo que esperamos de herramientas de búsqueda más sofisticadas como Elasticsearch. La búsqueda de ISBN es una función muy útil si tienes un escáner de código de barras. Limitar nuestra búsqueda a campos en el modelo Books es bastante restrictivo. Es posible que también queramos buscar por nombre de editorial. Afortunadamente, `search_fields` es lo suficientemente flexible como para lograr esto. Para buscar en un `ForeignKeyField` o `ManyToManyField`, solo necesitamos especificar el nombre del campo en el modelo actual y el campo en el modelo relacionado, separados por dos guiones bajos. En este caso, `Book` tiene una clave foránea, `publisher`, y queremos buscar en el campo `Publisher.name`, por lo que se puede especificar como `'publisher__name'` en `BookAdmin.search_fields`:

```python
search_fields = ("title", "isbn", "publisher__name")
```

Si quisiéramos restringir un campo de búsqueda a una coincidencia exacta en lugar de devolver resultados que contengan la cadena de búsqueda, el campo puede tener como sufijo `'__exact'`. Por lo tanto, reemplazar `'isbn'` con `'isbn__exact'` requerirá que coincida el ISBN completo y no podremos obtener una coincidencia utilizando una parte del ISBN.

De manera similar, restringimos el campo de búsqueda para que solo devuelva resultados que comiencen con la cadena de búsqueda mediante el sufijo `'__startswith'`. Calificar el campo de búsqueda del nombre de la editorial como `'publisher__name__startswith'` significa que obtendremos resultados buscando "pack" pero no "ackt".

Esto concluye nuestro examen de las personalizaciones comunes de las páginas de listas de cambios. También es posible que queramos personalizar el comportamiento de los formularios de creación y actualización en nuestra aplicación.

#### Exclusión y agrupación de campos (Excluding and grouping fields)

Hay ocasiones en las que es apropiado restringir la visibilidad de algunos de los campos del modelo en la interfaz de administración. Esto se puede lograr con el atributo `exclude`.

Esta es la pantalla del formulario de reseña con el campo *Date edited* visible. Ten en cuenta que el campo *Date created* no aparece; ya es una vista oculta porque `date_created` está definido en el modelo con el parámetro `auto_now_add`:

*Figura 4.59 – El formulario de reseña*

Si quisiéramos excluir el campo *Date edited* del formulario de reseña, lo haríamos en la clase `ReviewAdmin`:

```python
exclude = ("date_edited",)
```

Entonces, el formulario de reseña aparecería sin *Date edited*:

*Figura 4.60 – El formulario de reseña con el campo Date edited excluido*

Por el contrario, podría ser más prudente restringir los campos de administración a aquellos que hayan sido permitidos explícitamente. Esto se logra con el atributo `fields`. La ventaja de este enfoque es que si se agregan nuevos campos en el modelo, no estarán disponibles en el formulario de administración a menos que se hayan agregado a la tupla `fields` en la subclase `ModelAdmin`:

```python
fields = ("content", "rating", "creator", "book")
```

Esto nos dará el mismo resultado que vimos anteriormente.

Otra opción es usar el atributo `fieldsets` de la subclase `ModelAdmin` para especificar el diseño del formulario como una serie de campos agrupados. Cada agrupación en `fieldsets` consiste en un título seguido de un diccionario que contiene una clave `"fields"` que apunta a una lista de cadenas de nombres de campo:

```python
fieldsets = (
    ("Linkage", {"fields": ("creator", "book")}),
    ("Review content", {"fields": ("content", "rating")}),
)
```

El formulario de reseña debería verse de la siguiente manera:

*Figura 4.61 – El formulario de reseña con fieldsets*

Si queremos omitir el título en un conjunto de campos (*fieldset*), podemos hacerlo asignándole el valor `None`:

```python
fieldsets = (
    (None, {"fields": ("creator", "book")}),
    ("Review content", {"fields": ("content", "rating")}),
)
```

Ahora, el formulario de reseña debería aparecer como se muestra en la siguiente captura de pantalla:

*Figura 4.62 – El formulario de reseña con el primer fieldset sin título*

Ahora que hemos aprendido sobre las listas de cambios y las personalizaciones de formularios, podemos poner en práctica nuestros conocimientos con una actividad integral.

#### Actividad 4.03 – Personalización de los administradores de modelos

En nuestro modelo de datos, la clase `Contributor` se utiliza para almacenar datos de los colaboradores de libros; pueden ser autores, colaboradores o editores. Esta actividad se centra en modificar la clase `Contributor` y agregar una clase `ContributorAdmin` para mejorar la facilidad de uso de la aplicación de administración. Actualmente, la lista de cambios de `Contributor` se establece de forma predeterminada en una sola columna, *FirstNames*, basada en el método `__str__` creado en el Capítulo 2. Investigaremos algunas formas alternativas de representar esto. Estos pasos te ayudarán a completar la actividad:

1. Edita `reviews/models.py` para agregar funcionalidad adicional al modelo `Contributor`.
2. Agrega un método `initialled_name` a `Contributor` que no tome argumentos (como el método `Book.isbn13`).
3. El método `initialled_name` devolverá una cadena que contiene `Contributor.last_names` seguido de una coma y las iniciales de los nombres dados. Por ejemplo, para un objeto `Contributor` con `first_names` de `Jerome David` y `last_names` de `Salinger`, `initialled_name` devolverá `Salinger, JD`. Si `Contributor.last_names` está en blanco, se devolverá `Contributor.first_names`. He aquí un ejemplo:
   ```python
   >>> Contributor(first_names="Plato", last_names="").initialled_name()
   'Plato'
   ```
4. Reemplaza el método `__str__` para `Contributor` con uno que llame a `initialled_name()`. En este punto, la lista de visualización de Contribuidores se verá así:
   *Figura 4.63 – La lista de visualización de Contribuidores*
5. Edita el archivo `reviews/admin.py`. Modificaremos `ContributorAdmin`, que fue subclasificado de `admin.ModelAdmin` en la sección *Los campos de visualización de lista*.
6. Modifícalo para que en la lista de cambios de Contributors, los registros se muestren con dos columnas ordenables: *Last Names* y *First Names*. Podemos eliminar el atributo `list_display` existente de `ContributorAdmin` junto con la función `initialled_name`.
7. Agrega una barra de búsqueda que busque en *Last Names* y *First Names*. Modifícala para que la búsqueda de *Last Names* coincida con el inicio de *Last Names*.
8. Agrega un filtro en *Last Names*.

Al completar la actividad, deberías ver algo como esto:

*Figura 4.64 – Salida esperada*

Se pueden realizar cambios como estos para mejorar la funcionalidad de la interfaz de usuario de administración. Al implementar las columnas *First Names* y *Last Names* como columnas separadas en la lista de cambios de Contributors, le estamos dando al usuario la opción de ordenar por cualquiera de los campos. Al considerar qué columnas son más útiles en la recuperación de búsquedas y selecciones de filtros, podemos mejorar la recuperación eficiente de registros.

---

### Sección: Resumen

En este capítulo, vimos cómo crear superusuarios a través de la línea de comandos de Django y cómo usarlos para acceder a la aplicación de administración. Luego, después de un breve recorrido por la funcionalidad básica de la aplicación de administración, examinamos cómo registrar nuestros modelos con ella para producir una interfaz CRUD para nuestros datos.

Posteriormente, aprendimos cómo refinar esta interfaz modificando las funciones de todo el sitio. Modificamos la forma en que la aplicación de administración presenta los datos del modelo al usuario registrando clases de administración de modelos personalizadas con el sitio de administración. Esto nos permitió realizar cambios detallados en la representación de las interfaces de nuestros modelos. Estas modificaciones incluyeron la personalización de las páginas de listas de cambios agregando columnas adicionales, filtros, jerarquías de fechas y barras de búsqueda. También modificamos el diseño de las páginas de administración de modelos agrupando y excluyendo campos.

Esta fue solo una inmersión preliminar en la funcionalidad de la aplicación de administración. Volveremos a examinar la rica funcionalidad de `AdminSite` y `ModelAdmin` en el Capítulo 10. Pero primero, debemos aprender algunas funciones más intermedias de Django. En el próximo capítulo, aprenderemos a organizar y servir contenido estático, como CSS, JavaScript e imágenes, desde una aplicación Django.
