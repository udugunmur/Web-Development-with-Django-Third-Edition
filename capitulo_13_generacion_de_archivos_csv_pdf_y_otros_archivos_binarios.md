# Parte 3: Características avanzadas de Django

## Capítulo 13: Generación de archivos CSV, PDF y otros archivos binarios

Hasta ahora, hemos aprendido sobre los diversos aspectos del framework Django y hemos explorado cómo podemos crear aplicaciones web utilizando Django con todas las características y personalizaciones que deseamos.

Supongamos que, al crear una aplicación web, necesitamos realizar algún análisis y preparar algunos informes. Es posible que debamos analizar los datos demográficos de los usuarios sobre cómo se utiliza la plataforma o generar datos que puedan incorporarse a los sistemas de aprendizaje automático para encontrar patrones. Queremos que nuestro sitio web muestre algunos de los resultados de nuestro análisis en un formato tabular y otros resultados como gráficos y diagramas detallados. Además, queremos permitir que nuestros usuarios exporten los informes y los examinen más a fondo en aplicaciones como Jupyter Notebook y Excel.

A medida que avanzamos en este capítulo, aprenderemos cómo hacer realidad estas ideas e implementar funciones en nuestra aplicación web que nos permitan exportar registros a formatos estructurados, como tablas, mediante el uso de archivos de valores separados por comas (CSV) o archivos de Excel. También aprenderemos a permitir que nuestros usuarios generen representaciones visuales de los datos que hemos almacenado dentro de nuestra aplicación web y los exporten en formato PDF para que puedan distribuirse fácilmente para una referencia rápida.

Comencemos nuestro viaje aprendiendo a trabajar con archivos CSV en Python. Aprender esta habilidad nos ayudará a crear una funcionalidad que permita a nuestros lectores exportar nuestros datos para un análisis posterior.

En este capítulo, cubriremos los siguientes temas:
- Uso del módulo `csv` para renderizar un archivo CSV con una vista de Django
- Uso del módulo `zipfile` para crear un archivo comprimido ZIP con una vista de Django
- Generación de archivos PDF con el paquete WeasyPrint
- Generación de archivos Excel con el paquete XlsxWriter
- Generación de gráficos con el paquete Python Image Library (Pillow)
- Generación de gráficos con Plotly
- Generación de gráficos con Pillow

---

### Sección: Requisitos técnicos

Todos los archivos de código de este capítulo se pueden encontrar en la carpeta `Chapter13` en el repositorio de GitHub de este libro. Para acceder al enlace del repositorio, sigue los pasos en la sección *Download the example code files* en el Prefacio.

---

### Sección: Trabajo con archivos CSV en Python

Existen varias razones por las que podríamos necesitar exportar los datos de nuestra aplicación. Una de las razones puede implicar el análisis de esos datos; por ejemplo, es posible que necesitemos comprender los datos demográficos de los usuarios registrados en la aplicación o extraer patrones de uso de la aplicación. También podríamos necesitar descubrir cómo funciona nuestra aplicación para los usuarios con el fin de diseñar mejoras futuras. Tales casos de uso requieren que los datos estén en un formato que se pueda consumir y analizar fácilmente. Aquí, el formato de archivo CSV viene al rescate.

CSV es un formato de archivo muy práctico que se puede utilizar para exportar rápidamente datos de una aplicación en un formato de filas y columnas. Los archivos CSV suelen tener datos separados por delimitadores simples, que se utilizan para diferenciar una columna de otra, y saltos de línea, que se utilizan para indicar el inicio de un nuevo registro (o fila) dentro de la tabla.

Python tiene un gran soporte para trabajar con archivos CSV en su biblioteca estándar, gracias al módulo `csv`. Este soporte permite leer, analizar y escribir archivos CSV. Veamos cómo podemos aprovechar el módulo `csv` proporcionado por Python para trabajar con archivos CSV y leer y escribir datos en ellos.

---

### Sección: Trabajo con el módulo csv de Python

El módulo `csv` de Python nos permite interactuar con archivos que están en formato CSV, que no es más que un formato de archivo de texto; es decir, los datos almacenados dentro de los archivos CSV son legibles por humanos.

El módulo `csv` requiere que el archivo esté abierto antes de poder aplicar los métodos suministrados por el módulo `csv`.

Primero, veamos cómo podemos usar el módulo `csv` de Python para crear nuevos archivos CSV.

#### Escritura en archivos CSV con Python

Ahora, aprendamos cómo podemos escribir datos CSV en archivos. Los siguientes pasos describen el proceso de escritura de datos en archivos CSV:

1. Abre el archivo en modo de escritura ejecutando el siguiente script:
   ```python
   csv_file = open('path to csv file', 'w')
   ```
2. Obtén un objeto escritor CSV, que puede ayudarnos a escribir datos formateados correctamente en formato CSV. Esto se puede hacer llamando al método `writer()` del módulo `csv`, que devuelve un objeto escritor que se puede usar para escribir datos compatibles con el formato CSV en un archivo CSV:
   ```python
   csv_writer = csv.writer(csv_file)
   ```
3. Una vez que el objeto escritor esté disponible, podemos comenzar a escribir los datos. Esto se facilita mediante el método `writerow()` del objeto escritor. El método `writerow()` toma una lista de valores que escribe en el archivo CSV. La lista en sí indica una sola fila y los valores dentro de la lista indican los valores de las columnas:
   ```python
   record = ['value1', 'value2', 'value3']
   csv_writer.writerow(record)
   ```
4. Si deseas escribir múltiples registros en una sola llamada, también puedes usar el método `writerows()` del escritor CSV. El método `writerows()` se comporta de manera similar al método `writerow()`, pero toma una lista de listas y puede escribir múltiples filas de una sola vez:
   ```python
   records = [['value11', 'value12', 'value13'], ['value21', 'value22', 'value23']]
   csv_writer.writerows(records)
   ```
5. Una vez escritos los registros, podemos cerrar el archivo CSV:
   ```python
   csv_file.close()
   ```

Afortunadamente, un objeto `HttpResponse` de Django es un objeto similar a un archivo (*file-like object*) y se puede utilizar para crear un escritor CSV, por lo que es muy fácil incorporar el módulo `csv` en una vista de Django para crear hojas de cálculo dinámicas a partir de nuestros modelos.

El constructor `HttpResponse` toma un argumento opcional, `content_type`, que se puede usar para especificar un tipo MIME para la respuesta. `DEFAULT_CHARSET` es `'UTF-8'`. Si los argumentos `content_type` y `DEFAULT_CHARSET` no se especifican, Django representará la línea `Content-Type` del encabezado de respuesta HTML de la siguiente manera:

```http
Content-Type: text/html; charset=UTF-8
```

El tipo MIME preferido para un archivo CSV es `text/csv`. No queremos ver los datos CSV sin procesar en el navegador web, por lo que debemos establecer la disposición del contenido (*content disposition*) en `attachment`, de modo que el CSV se descargue con un nombre de archivo específico.

```python
response['Content-Disposition'] = \
    'attachment; filename="downloadfile.csv"'
```

A veces se adapta a nuestros propósitos tener un nombre de archivo estático, pero en otras circunstancias, podríamos querer diferenciar el nombre de archivo incluyendo una clave primaria o una marca de fecha.

```python
date_stamp = datetime.datetime.now().strftime('%Y%m%d')
filename = f"download_file_{date_stamp}_{user_pk}"
response['Content-Disposition'] = f'attachment; filename="{filename}"'
```

Ahora utilicemos nuestro conocimiento del módulo `csv` para crear un informe de resumen de los datos de reseñas en formato CSV.

#### Ejercicio 13.01 – Creación de un archivo CSV de resumen de reseñas

En este ejercicio, utilizarás el módulo `csv` de Python en una vista de Django que devuelve un archivo CSV con un resumen de las reseñas de libros.

El resumen de reseñas debe contener columnas para: Título del libro (*Book title*), ISBN, Nombre de la editorial (*Publisher name*), Nombre de usuario del revisor (*Reviewer username*) y Calificación (*Rating*).

Solo queremos la información específica del libro (Título del libro, ISBN, Nombre de la editorial) en la primera línea de reseña de cada libro. Los libros sin reseñas no se agregarán al informe.

Los encabezados de columna en el archivo CSV deben aparecer como `'title'`, `'isbn'`, `'publisher'`, `'reviewer'` y `'rating'`.

La vista de Django se agregará a `reviews/views.py`:

1. Edita `reviews/views.py` y agrega una declaración de importación para el módulo `csv` y asegúrate de que se esté importando `HttpResponse`:
   `reviews/views.py`:
   ```python
   import csv
   from django.http import HttpResponse
   ```
2. El paso de recuperación de datos implicará consultar el modelo `Book` y buscar libros con al menos una reseña. Como este paso de recuperación de datos se utilizará en vistas posteriores de este capítulo, lo crearemos como una función privada, `_review_summary`, que puede ser llamada por varias funciones de vista.
   El primer paso de la vista es recuperar todos los libros del modelo `Book`:
   ```python
   def _review_summary(request):
       # Retrieve the data
       books = Book.objects.all()
   ```
3. Define el encabezado e inclúyelo como la primera fila en la estructura de datos `rows`:
   ```python
   header = ['title', 'isbn', 'publisher', 'reviewer', 'rating']
   rows = [header]
   ```
4. Itera a través de los libros y recupera las reseñas del libro actual:
   ```python
   for book in books:
       reviews = book.review_set.all()
   ```
5. Si están presentes, agrega una fila que contenga el título del libro, el ISBN, el nombre de la editorial y, de la primera reseña, el nombre de usuario del revisor y la calificación:
   ```python
   if reviews:
       rows.append([book.title, book.isbn, book.publisher.name, reviews[0].creator.username, reviews[0].rating])
   ```
6. En las reseñas posteriores, deja en blanco el título del libro, el ISBN y el nombre de la editorial, pero incluye el nombre de usuario del revisor y la calificación:
   ```python
   for review in reviews[1:]:
       rows.append(['', '', '', review.creator.username, review.rating])
   return rows
   ```
7. Ahora que hemos creado la función privada `_review_summary`, que se utiliza para recuperar los datos de las reseñas de libros, podemos codificar la vista de función `review_summary_csv`.
   Después de que la vista recupera los datos de `_review_summary`, se crea un objeto de respuesta. Necesita un `content_type` de `text/csv` y una disposición `attachment` (ya que queremos que se guarde como una descarga en lugar de aparecer en el navegador). Le daremos a la descarga el nombre de archivo `review_summary.csv`:
   ```python
   def review_summary_csv(request):
       rows = _review_summary()
       response = HttpResponse(content_type='text/csv')
       filename = "review_summary.csv"
       response['Content-Disposition'] = f'attachment; filename="{filename}"'
   ```
8. Crea un objeto `csv.writer`, usando el objeto de respuesta como un objeto similar a un archivo:
   ```python
   # Write the CSV response
   writer = csv.writer(response)
   ```
9. Escribe cada fila de `rows` en el escritor:
   ```python
   for row in rows:
       writer.writerow(row)
   ```
10. Devuelve la respuesta:
    ```python
    return response
    ```
11. El paso restante es agregar la ruta del informe de resumen a `reviews/urls.py`:
    `reviews/urls.py`:
    ```python
    urlpatterns = [
        …
        path('reviews/csv/', views.review_summary_csv, name='review_summary_csv'),
    ]
    ```
    El conjunto final de cambios debe parecerse a los de la carpeta `Chapter13` en el repositorio de GitHub de este libro.
12. Ahora, cuando visites la URL `http://localhost:8000/reviews/csv/`, se te pedirá que descargues un archivo CSV que se parece al siguiente:
    *Figura 13.01: El archivo CSV importado a Microsoft Excel*

Pronto veremos cómo podemos hacer que nuestro sitio web funcione con el formato XLSX. Pero antes de eso, echemos un vistazo rápido al uso de formatos de archivos binarios.

---

### Sección: Formatos de archivos binarios para la exportación de datos

Hasta ahora, hemos trabajado principalmente con datos textuales y cómo podemos leerlos y escribirlos desde archivos de texto. Pero a menudo, los formatos basados en texto no son suficientes. Por ejemplo, imagina que deseas exportar una imagen o un gráfico. ¿Cómo representarás una imagen o un gráfico como texto y cómo leerás y escribirás estas imágenes?

Para ayudarnos en estas situaciones, entran en juego los formatos de archivos binarios, que pueden ayudarnos a leer y escribir hacia y desde un conjunto rico y diverso de datos. Todos los sistemas operativos comerciales brindan soporte nativo para trabajar con formatos de archivos de texto y binarios, y no es ninguna sorpresa que Python proporcione una de las implementaciones más versátiles para trabajar con archivos de datos binarios. Un ejemplo simple de esto es el comando `open`, que puedes usar para indicar el formato en el que deseas abrir un archivo:

```python
file_handler = open('path to file', 'rb')
```

Aquí, `b` indica binario.

A partir de esta sección, ahora nos ocuparemos de cómo podemos trabajar con archivos binarios y utilizarlos para representar y exportar datos desde nuestra aplicación web Django. El primero de los formatos que vamos a ver es el formato de archivo XLSX popularizado por Microsoft Excel.

Por lo tanto, profundicemos en el manejo de archivos XLSX con Python.

#### Trabajo con archivos XLSX mediante el paquete XlsxWriter

En esta sección, aprenderemos más sobre el formato de archivo XLSX y comprenderemos cómo podemos trabajar con él utilizando el paquete XlsxWriter.

Si bien CSV tiene un uso muy amplio, el grado de ambigüedad al interpretarlo entre aplicaciones puede convertirlo en una opción menos deseable para intercambiar información de hojas de cálculo; lo que es más importante, muchas de las características que valoramos en las hojas de cálculo, como hojas de trabajo, formato de texto, color y cálculos basados en fórmulas.

Afortunadamente, es posible generar archivos en el formato nativo de Excel utilizando Python. Microsoft Excel es la aplicación de software líder en el campo de la contabilidad y la gestión de registros tabulares. De manera similar, el formato de archivo XLSX que fue adoptado por Excel en 2007 ha recibido un amplio soporte por parte de muchos de los principales proveedores de productos.

#### Archivos XLSX

Los archivos XLSX son archivos binarios que se utilizan para almacenar datos tabulares. Estos archivos pueden ser leídos por cualquier software que implemente soporte para este formato. El formato XLSX organiza los datos en dos particiones lógicas:
- **Libros de trabajo (*Workbooks*)**: Cada archivo XLSX se denomina libro de trabajo y se supone que contiene conjuntos de datos relacionados con un dominio en particular. En la Figura 13.2, `Examplefile.xlsx` es un libro de trabajo (1):
  *Figura 13.2: Libros de trabajo y hojas de trabajo en Excel*
- **Hojas de trabajo (*Worksheets*)**: Dentro de cada libro de trabajo, puede haber una o más hojas de trabajo, que se utilizan para almacenar datos sobre conjuntos de datos diferentes pero lógicamente relacionados en un formato tabular. En la Figura 13.2, `Sheet1` y `Sheet2` son dos hojas de trabajo (2).

Al trabajar con el formato XLSX, estas son las dos unidades en las que generalmente trabajamos. Si conoces las bases de datos relacionales, puedes pensar en los libros de trabajo como bases de datos y en las hojas de trabajo como tablas.

Ahora, intentemos comprender cómo podemos comenzar a trabajar con archivos XLSX dentro de Python.

#### El paquete de Python XlsxWriter

Python no proporciona soporte nativo para trabajar con archivos XLSX a través de su biblioteca estándar. Pero gracias a la vasta comunidad de desarrolladores dentro del ecosistema de Python, es fácil encontrar varios paquetes que pueden ayudarnos a administrar nuestra interacción con archivos XLSX. Un paquete popular en esta categoría es XlsxWriter.

XlsxWriter es un paquete mantenido activamente por la comunidad de desarrolladores que brinda soporte para interactuar con archivos XLSX. El paquete proporciona muchas funciones útiles y admite la creación y administración de libros de trabajo, así como hojas de trabajo en libros de trabajo individuales. Puedes instalarlo ejecutando el siguiente comando en la Terminal o el Símbolo del sistema:

```bash
pip install XlsxWriter
```

Una vez instalado, puedes importar el módulo `xlsxwriter` de la siguiente manera:

```python
import xlsxwriter
```

Por lo tanto, veamos cómo podemos comenzar a crear archivos XLSX con el soporte del paquete XlsxWriter.

##### Creación de un libro de trabajo
Para comenzar a trabajar en archivos XLSX, primero debemos crearlos. Un archivo XLSX también se conoce como libro de trabajo y se puede crear llamando a la clase `Workbook` desde el paquete `xlsxwriter`, de la siguiente manera:

```python
workbook = xlsxwriter.Workbook(filename)
```

La llamada a la clase `Workbook` abre un archivo binario, especificado con el argumento `filename`, y devuelve una instancia del libro de trabajo que se puede usar para crear más hojas de trabajo y escribir datos.

##### Creación de una hoja de trabajo
Antes de que podamos comenzar a escribir datos en un archivo XLSX, primero debemos crear una hoja de trabajo. Esto se puede hacer fácilmente llamando al método `add_worksheet()` del objeto de libro de trabajo que obtuvimos en la sección anterior:

```python
worksheet = workbook.add_worksheet()
```

El método `add_worksheet()` crea una nueva hoja de trabajo, la agrega al libro de trabajo y devuelve un objeto que mapea la hoja de trabajo a un objeto de Python, a través del cual podemos escribir datos en la hoja de trabajo.

##### Escritura de datos en la hoja de trabajo
Una vez que se dispone de una referencia a la hoja de trabajo, podemos comenzar a escribir datos en ella llamando al método `write` del objeto de hoja de trabajo, como se muestra aquí:

```python
worksheet.write(row_num, col_num, col_value)
```

Como puedes ver, el método `write()` toma tres parámetros: un número de fila (`row_num`), un número de columna (`col_num`) y los datos que pertenecen al par (`row_num`, `col_num`), representados por `col_value`. Esta llamada se puede repetir para insertar múltiples elementos de datos en la hoja de trabajo.

##### Escritura de datos en el libro de trabajo
Una vez que se hayan escrito todos los datos, para finalizar los conjuntos de datos escritos y cerrar limpiamente el archivo XLSX, debes llamar al método `close()` en el libro de trabajo:

```python
workbook.close()
```

Este método escribe los datos que puedan estar en el búfer de archivo y finalmente cierra el libro de trabajo. Ahora, usemos este conocimiento para implementar nuestro propio código, lo que nos ayudará a escribir datos en un archivo XLSX.

No es posible cubrir todos los métodos y características que ofrece el paquete XlsxWriter en este capítulo. Para obtener más información, puedes leer la documentación oficial: [https://xlsxwriter.readthedocs.io/contents.html](https://xlsxwriter.readthedocs.io/contents.html).

#### Formateo de una hoja de cálculo

Revisemos la hoja de cálculo del ejemplo CSV y veamos cómo se podría implementar con formato XLSX adicional mediante `xlsxwriter`.

Primero, necesitamos instalar el módulo `xlsxwriter`. Está disponible en PyPI y se puede instalar rápidamente usando `pip`, o se puede encontrar en GitHub en [https://github.com/jmcnamara/XlsxWriter](https://github.com/jmcnamara/XlsxWriter):

```bash
pip install xlsxwriter
```

Esta sección no tiene como objetivo proporcionar una documentación completa de este módulo, pero ofrece un esquema simple de las características con las que es probable que nos encontremos.

Comenzamos importando el módulo `xlsxwriter` y luego creando un libro de trabajo pasando un nombre de archivo o un objeto similar a un archivo al constructor:

```python
import xlsxwriter
workbook = xlsxwriter.Workbook('output.xlsx')
```

Luego podemos crear hojas de trabajo específicas en el libro de trabajo con el método `add_worksheet`:

```python
worksheet = workbook.add_worksheet('Review Summary')
```

La clase `worksheet` contiene muchos métodos para trabajar con hojas de trabajo. Si tu objetivo es imprimir tu hoja de trabajo, o al menos guardarla en PDF, los métodos `set_landscape` y `set_portrait` resultan útiles.

También podemos establecer anchos de columna específicos usando el método `set_column`. El ancho predeterminado es de aproximadamente 8.43 caracteres en la fuente predeterminada Calibri 11. Podemos especificar un ancho basado en caracteres para un rango de columnas en la hoja de cálculo.

Por ejemplo, si queremos que la primera columna tenga el ancho predeterminado, pero que de la segunda a la cuarta columna tengan un ancho de 20 caracteres, seguido de un ancho de 50 caracteres para la octava columna, podemos usar el método `worksheet.set_column` para lograr esto:

```python
worksheet.set_column(1, 3, 20)  # Columns B:D width 20
worksheet.set_column(7, 7, 50)  # Column H width 50
```

Además de numerar las columnas en la hoja de cálculo desde cero y especificar dos valores enteros para el rango de columnas, también es posible especificar una columna mediante la notación A1 de Excel, por lo que esto se podría escribir de manera equivalente de la siguiente manera:

```python
worksheet.set_column('B:D', 20)  # Columns B:D width 20
worksheet.set_column('H:H', 50)  # Column H width 50
```

Por regla general, es más fácil usar la notación A1 si estamos codificando detalles específicos de columnas de forma fija en nuestro código, pero si estamos escribiendo hojas de cálculo donde formateamos celdas dinámicamente, el enfoque de coordenadas numéricas es más simple.

Afortunadamente, `xlsxwriter` proporciona funciones de utilidad que traducen entre los sistemas de coordenadas numéricas y de cuadrícula A1:

```python
>>> xlsxwriter.utility.xl_col_to_name(4)
'E'
>>> xlsxwriter.utility.xl_rowcol_to_cell(4,6)
'G5'
>>> xlsxwriter.utility.xl_rowcol_to_cell(26,3)
'D27'
>>> xlsxwriter.utility.xl_rowcol_to_cell(3,26)
'AA4'
>>> xlsxwriter.utility.xl_cell_to_rowcol('A1')
(0, 0)
>>> xlsxwriter.utility.xl_cell_to_rowcol('AA10')
(9, 26)
>>> xlsxwriter.utility.xl_range(0, 4, 5, 4)
'E1:E6'
```

Estas utilidades resultan muy útiles cuando intentamos especificar fórmulas de Excel que requieren la notación A1.

Utiliza siempre las funciones de `xlsxwriter.utility` para la conversión de fila-columna a A1. Es posible que tengas la tentación de escribir tu propio código simple para calcular el nombre de la celda, pero dicho código es propenso a errores, especialmente cuando lo utilizas para valores de rango alto que no has probado adecuadamente.

Una vez que tenemos una hoja de trabajo, podemos asignar valores específicos a celdas, filas y columnas en la hoja de trabajo usando los métodos `worksheet.write`, `worksheet.write_row` y `worksheet.write_column`. Todos estos métodos utilizan la notación A1 o la notación de coordenadas para escribir celdas, como podemos ver en el siguiente ejemplo, donde es más fácil usar la notación A1 para especificar filas o columnas explícitamente, pero si necesitamos determinar dinámicamente la ubicación de la celda, entonces es más fácil especificarla mediante coordenadas (fila, columna).

El código de Python utiliza estos métodos de escritura para producir la hoja de cálculo de Excel:

```python
import xlsxwriter

workbook = xlsxwriter.Workbook('WriteRowsAndColumns.xlsx')
worksheet = workbook.add_worksheet('Title')
worksheet.write_column('A1', list('WEB'))
worksheet.write_row('B4', list('DEVELOPMENT'))
worksheet.write_column('A5', list('WITH'))
worksheet.write_row('B9', list('DJANGO'))
for i, letter in enumerate('PACKT'):
    worksheet.write(12, i*2, letter)
workbook.close()
```

Esto nos lleva al formateo de celdas. `xlsxwriter` incluye una clase `Format` que implementa todo el formato que encontramos en las hojas de cálculo de Excel: fuentes, color, tamaño de fuente, negrita, cursiva, texto tachado, representación de fechas, formato numérico, etc. Podemos crear un objeto `Format` utilizando el método `add_format` en la clase de libro de trabajo. Podemos preespecificar el formato llamando a `add_format` con un argumento de diccionario que contenga los parámetros de formato, o podemos crear un formato vacío y luego anular los valores predeterminados con métodos específicos en el objeto de formato. Los dos fragmentos de código siguientes muestran formas equivalentes de crear el mismo formato:

```python
format_dict = {
    'bold': True,      # set bold
    'font_size': 12,   # specify font size
    'bg_color': 'gray' # set background color
}
header_format = workbook.add_format(format_dict)
header_format = workbook.add_format()
header_format.set_bold()
header_format.set_font_size(12)
header_format.set_bg_color('gray')
```

Una vez que tenemos un objeto de formato, podemos especificarlo como el argumento final de uno de los métodos de escritura:

```python
worksheet.writerow('A1', header, header_format)
```

Para comprender mejor la clase `Format`, puedes leer la documentación completa en [https://xlsxwriter.readthedocs.io/format.html](https://xlsxwriter.readthedocs.io/format.html). Los ejemplos y ejercicios restantes de esta sección solo utilizarán el diccionario de propiedades para especificar el formato.

No estamos limitados a llenar hojas de cálculo con contenido estático. `xlsxwriter` incluye la capacidad de especificar fórmulas de Excel en celdas de hojas de trabajo específicas:

```python
worksheet.write_formula('F9', '=RAND()')
```

El material de referencia de XlsxWriter en [https://xlsxwriter.readthedocs.io](https://xlsxwriter.readthedocs.io/) puede ayudarte a desbloquear el poder del formato de archivo XLSX y crear hojas de cálculo increíbles. Apenas hemos arañado la superficie de `xlsxwriter`, pero hemos aprendido lo suficiente para recrear el archivo CSV de reseñas como una hoja de cálculo de Excel, aumentándola con formato y cálculos dinámicos.

Ahora que hemos examinado la funcionalidad básica del paquete XlsxWriter, podemos revisar el informe de resumen de reseñas del Ejercicio 13.01 e implementarlo como una hoja de cálculo formateada.

#### Ejercicio 13.02 – Desarrollo de una hoja de cálculo formateada

En el Ejercicio 13.01, creamos la mecánica de una vista de Django que construye una hoja de cálculo CSV a partir de una consulta de un modelo de Django. En este ejercicio, utilizaremos el paquete XlsxWriter para desarrollar una hoja de cálculo formateada.

Nuestro enfoque será desarrollar un escritor de informes XLSX genérico que tome solicitudes de recuperación de datos y parámetros de formato para escribir solicitudes específicas.

Reutilizaremos la función privada `_review_summary` que desarrollamos en el Ejercicio 13.01 y mejoraremos la salida con formato para que se produzca la siguiente hoja de cálculo XLSX.

1. La vista de Django `review_summary_xlsx` se agregará a `reviews/views.py`. Toma un objeto de solicitud como argumento. Inicialmente, recupera datos mediante la función privada `_review_summary`. Como el informe contiene una variedad de tipos de formato de celda, desarrollaremos parámetros para pasar al método `add_format` de la hoja de trabajo:
   `reviews/views.py`:
   ```python
   def review_summary_xlsx(request):
       rows = _review_summary()
       format_settings = dict(
           header=dict(bold=True, font_size=12, bg_color='gray'),
           short_text=dict(align='left'),
           isbn=dict(num_format='0', align='left'),
           content=dict(align='left', valign='top', text_wrap=True),
           rating=dict(num_format='#', align='right'),
           footer=dict(bold=True, font_size=12),
           average=dict(num_format='#.###', align='right', bold=True, font_size=12),
       )
   ```
2. Como se pueden definir anchos de columna en una hoja de cálculo de Excel, definiremos una serie de anchos de columna:
   ```python
   # Column A 40, B 15, C-D 30, E 50, F 5
   column_widths = [('A:A', 40), ('B:B', 14), ('C:D', 30), ('E:E', 50), ('F:F', 5)]
   ```
3. Las configuraciones de formato deben mapearse a columnas individuales en el informe. Por lo tanto, `title`, `reviewer` y `content` utilizarán el formato `short_text`, mientras que `isbn` y `rating` tendrán sus propios formatos numéricos específicos:
   ```python
   # title, isbn, publisher, reviewer, content, rating
   col_formats = ('short_text', 'isbn', 'short_text', 'short_text', 'content', 'rating')
   ```
4. Creamos un búfer en memoria `BytesIO` para almacenar la hoja de cálculo. El búfer será escrito por `xlsxwriter.Workbook`:
   ```python
   output = BytesIO()
   workbook = xlsxwriter.Workbook(output)
   ```
5. Llamaremos a una función de utilidad `format_workbook` que toma los objetos del libro de trabajo, las filas de datos y nuestra información de formato como parámetros:
   ```python
   workbook = format_workbook(
       workbook, rows, format_settings, reader.fieldnames, col_formats, column_widths, footer_avg=dict(label='Average Rating', id='rating')
   )
   ```
6. Al final del proceso, cerramos el libro de trabajo, rebobinamos el búfer y adjuntamos el búfer con la disposición y el tipo de contenido adecuados a la respuesta HTTP:
   ```python
   workbook.close()
   # Rewind the buffer.
   output.seek(0)
   # Create the HTTP response.
   filename = 'review_details.xlsx'
   ctype = (
       'application/vnd.openxmlformats-'
       'officedocument.spreadsheetml.sheet'
   )
   response = HttpResponse(output, content_type=ctype)
   response['Content-Disposition'] = \
       f'attachment; filename={filename}'
   return response
   ```
7. Ahora podemos definir el método `format_workbook` en el archivo `reviews/utils.py`. Comenzamos importando las funciones auxiliares necesarias de `xlsxwriter.utility`, que se utilizarán para convertir coordenadas numéricas a la notación de celda A1:
   `reviews/utils.py`:
   ```python
   from xlsxwriter.utility import xl_rowcol_to_cell, xl_range
   ```
8. El método `format_workbook` toma argumentos para objetos de libro de trabajo, los datos de filas, `format_settings`, `col_formats` y `column_widths` que creamos en la vista de función `review_summary_xlsx`. El argumento opcional `footer_avg` se utiliza para especificar si habrá un cálculo de promedio en el pie de página:
   ```python
   def format_workbook(workbook, rows, format_settings, header, col_formats, column_widths, footer_avg=None):
   ```
9. Se agrega una hoja de trabajo llamada `'Review Summary'` al libro de trabajo:
   ```python
   worksheet = workbook.add_worksheet('Review Summary')
   ```
10. Los parámetros de formatos suministrados por `format_settings` se utilizan para definir un diccionario de formatos:
    ```python
    formats = {format_id: workbook.add_format(format_set) for format_id, format_set in format_settings.items()}
    ```
11. Los anchos de columna especificados en `column_widths` se aplican a la hoja de trabajo:
    ```python
    for col_range, width in column_widths:
        worksheet.set_column(col_range, width)
    ```
12. El encabezado se escribe en el archivo y se formatea:
    ```python
    worksheet.write_row('A1', header, formats['header'])
    for col_num, cell_data in enumerate(header):
        worksheet.write(0, col_num, cell_data, formats['header'])
    ```
13. Cada fila recuperada de `_review_summary` se agrega a la hoja de trabajo, y los formatos se aplican en función de su especificación en `col_formats`:
    ```python
    for row_num, row in enumerate(rows):
        if not row:
            continue
        for col_num, col in enumerate(list(row.values())):
            worksheet.write(row_num+1, col_num, col, formats[col_formats[col_num]])
    ```
14. Podemos incluir el cálculo del promedio en la hoja de trabajo antes de devolver el libro de trabajo formateado:
    ```python
    # Add footer with average rating
    if footer_avg:
        row_id = len(rows) + 1
        col_id = header.index(footer_avg['id'])
        col_label_id = (col_id - 1 if col_id > 0 else len(col_formats))
        worksheet.write(
            row_id, col_label_id, f"{footer_avg['label']}:", formats['footer'])
        avg_cell = xl_rowcol_to_cell(len(rows)+1, col_id)
        avg_range = xl_range(1, col_id, row_id-1, col_id)
        worksheet.write_formula(avg_cell, f"=AVERAGE({avg_range})")
    return workbook
    ```
15. Ahora que se ha creado la vista, debemos incluir la ruta de URL adecuada en `reviews/urls.py`:
    `reviews/urls.py`:
    ```python
    path('reviews/xlsx/', views.review_summary_xlsx, name='review_summary_xlsx'),
    ```
    El conjunto final de cambios debe parecerse a los de la carpeta `Chapter13` en el repositorio de GitHub de este libro.
16. Ahora, cuando visites la URL `http://localhost:8000/reviews/xlsx/`, se te pedirá que descargues un archivo que se parece al siguiente:
    *Figura 13.3: La hoja de cálculo XLSX*

Hemos visto en este ejemplo que la arquitectura MVT de Django es lo suficientemente flexible como para permitir la creación de una respuesta binaria. A continuación, aplicaremos este enfoque para crear archivos comprimidos ZIP desde Django.

---

### Sección: Archivos ZIP

El formato de archivo comprimido ZIP ha tenido un uso popular desde el lanzamiento de PKZIP en 1989. Los archivos ZIP contienen archivos individuales comprimidos o colecciones y preservan la estructura de directorios.

El módulo `zipfile` es parte de la biblioteca estándar de Python y contiene los recursos para generar archivos ZIP mediante programación. El módulo contiene una clase `ZipFile` que se puede utilizar para leer, escribir y anexar a archivos ZIP.

El constructor `ZipFile` toma un argumento obligatorio `file` y tres argumentos opcionales:
- `file`: Puede ser la ruta del archivo ZIP o un objeto similar a un archivo.
- `mode`: Puede ser `'r'`, `'w'`, `'a'` o `'x'`, para lectura, escritura, anexado o exclusivo (el valor predeterminado es `'r'`).
- `compression`: El módulo `zipfile` define cinco constantes que representan el tipo de compresión que se aplica al archivo comprimido ZIP:
  - `zipfile.ZIP_STORED` (sin compresión)
  - `zipfile.ZIP_DEFLATED` (zlib)
  - `zipfile.ZIP_BZIP2` (bzip2)
  - `zipfile.ZIP_LZMA` (LZMA)
  - `zipfile.ZIP_ZSTANDARD` (compression.zstd)
  El valor predeterminado es sin compresión. Como no todos los clientes de software ZIP utilizan las bibliotecas bzip2, LZMA y compression.zstd, `zipfile.ZIP_DEFLATED` es la opción más segura si pretendes comprimir tu archivo ZIP.
- `allowZipFile64`: Este es un booleano para permitir archivos con extensiones ZIP64. Por defecto, es `True`.

Considera una situación donde un directorio contiene los siguientes archivos:

```text
csv/book_list.csv
csv/reviews_2010.csv
csv/reviews_2011.csv
csv/reviews_2012.csv
csv/reviews_2013.csv
…
csv/reviews_2025.csv
```

Queremos crear mediante programación un archivo comprimido ZIP que solo almacene los archivos que coincidan con el comodín `csv/reviews_20??.csv`. Afortunadamente, Python tiene un módulo para este tipo de coincidencia de comodines llamado `glob`.

Este sencillo programa resuelve nuestro problema:

```python
from zipfile import ZipFile, ZIP_DEFLATED
import glob

file_paths = glob.glob('csv/reviews_20??.csv')
with ZipFile('all_reviews.zip', 'w', ZIP_DEFLATED) as z:
    for file_path in file_paths:
        z.write(file_path)
```

Al igual que en el caso de los datos CSV, podemos crear fácilmente una vista de Django que tome una solicitud y escriba un archivo ZIP en una respuesta:

```python
import datetime
import glob
from io import BytesIO
from zipfile import ZipFile
from django.http import HttpResponse

def review_zip(request):
    output = BytesIO()
    file_paths = glob.glob('csv/reviews_20??.csv')
    with ZipFile(output, 'w') as z:
        for file_path in file_paths:
            z.write(file_path)
    # Create the HTTP response.
    date_stamp = datetime.datetime.now().strftime('%Y%m%d')
    filename = f"all_reviews_{date_stamp}.zip"
    ctype = 'application/zip'
    cdisposition = f"attachment; filename={filename}"
    response = HttpResponse(output, content_type=ctype)
    response['Content-Disposition'] = cdisposition
    return response
```

---

### Sección: Trabajo con archivos PDF en Python

El formato de documento portátil (*Portable Document Format*, PDF) fue desarrollado a finales de la década de 1980 por Adobe Inc. con el fin de crear un formato uniforme de descripción de páginas para la impresión que funcionara en una variedad de sistemas operativos, firmware e impresoras.

Existen numerosos módulos disponibles para generar contenido PDF dinámico. Muchos de estos implican el aprendizaje de APIs complejas para construir un objeto PDF mediante programación. Como estamos en el proceso de aprender sobre Django y hemos profundizado en sus mecanismos de plantillas, sería fantástico si pudiéramos aprovechar este conocimiento para generar archivos PDF.

Afortunadamente, existen varios paquetes que generarán archivos PDF basados en contenido de texto como HTML o lenguaje de marcado. Un paquete de Python que se está desarrollando activamente al momento de escribir este artículo es WeasyPrint.

Nuevamente, WeasyPrint está ampliamente documentado y los desarrolladores interesados pueden visitar [https://weasyprint.readthedocs.io/](https://weasyprint.readthedocs.io/) para obtener más información. Los ejemplos aquí son solo una introducción breve y sencilla al poder de WeasyPrint.

WeasyPrint se puede instalar usando `pip` (o se puede obtener en GitHub en [https://github.com/Kozea/WeasyPrint](https://github.com/Kozea/WeasyPrint)):

```bash
pip install weasyprint
```

WeasyPrint incluso se puede usar en la línea de comandos para convertir una página HTML en un documento PDF sin necesidad de codificar nada. Todo lo que se requiere es especificar `weasyprint <url> <pdffile>`. Los resultados pueden variar ya que WeasyPrint no implementa todo el formato HTML exactamente como se vería en un navegador y desconoce JavaScript u otro contenido dinámico dentro de un sitio web.

```bash
weasyprint https://python.org python_org.pdf
```

Si quisiéramos lograr lo que hicimos en la línea de comandos con un script de Python, podríamos usar el siguiente fragmento de código:

```python
from weasyprint import HTML, CSS

html = HTML('https://python.org')
html.write_pdf('python_org.pdf')
```

La clase `HTML` en `weasyprint` también es capaz de tomar cualquier archivo local, así como contenido de cadena HTML sin procesar, y usar esos archivos para generar archivos PDF. Usaremos esta capacidad en el próximo ejercicio para producir un PDF de detalles de libros que se genera desde dentro de la aplicación Bookr.

Esta es una introducción muy breve al uso de WeasyPrint y cubre poco de su amplia funcionalidad. Para obtener más información, visita la documentación de WeasyPrint en [https://weasyprint.readthedocs.io](https://weasyprint.readthedocs.io/).

#### Ejercicio 13.03 – Generación de una versión en PDF de una página web en Python

En el siguiente ejemplo, utilizaremos esta técnica y generaremos un archivo PDF que contiene información sobre un libro específico y sus reseñas en el sistema Bookr. Estamos familiarizados con este código, ya que simplemente sigue el patrón MVT de Django, pero esta vez la vista utiliza WeasyPrint para representar la respuesta como un PDF en lugar de HTML:

1. Agrega un método `rating_to_stars` a `Review` en `models.py`:
   `models.py`:
   ```python
   def rating_to_stars(self):
       if not self.rating:
           return ''
       elif isinstance(self.rating, int):
           return '*'*self.rating
       return self.rating
   ```
2. Crea una nueva plantilla de Django en `reviews/templates/reviews/book_detail_pdf.html`:
   `reviews/templates/reviews/book_detail_pdf.html`:
   ```html
   <html>
   <head>
       <title>Bookr - {{ book.title }} </title>
   </head>
   <body>
       <img src="/static/reviews/logo.png">
       <div>
           <h1>{{ book.title }}</h1>
           <p>Published: {{ book.publication_date }}</p>
           <p>ISBN: {{ book.isbn }}</p>
           <p>Publisher: {{ book.publisher.name}} </p>
           <p>{% if book.contributors.all %}By {% endif %}
           {% for contributor in book.contributors.all %}
               {{ contributor.first_names }} {{ contributor.last_names }}
           {% endfor %}</p>
       </div>
       <div><img src="/media/{{ book.cover }}" /></div>
       <div>
           {% if reviews %}
               <h2>Reviews:</h2>
           {% endif %}
           {% for review in reviews %}
               <p class="review-content">{{ review.content }}</p>
               <p class="review-creator">{{ review.creator.username }} {{ review.rating_to_stars }}</p>
               <p class="review-date">{{ review.date_edited }}</p>
           {% empty %}
               <h2> No reviews </h2>
           {% endfor %}
       </div>
   </body>
   </html>
   ```
3. Necesitamos agregar una función de vista, `book_detail_pdf`, a `reviews/views.py`.
   Primero, necesitamos importar las clases `HTML` y `CSS` de `weasyprint`:
   ```python
   from weasyprint import HTML, CSS
   …
   ```
4. Tenemos una función de vista existente llamada `book_detail`. Todo el código en esta función es aplicable a nuestra función actual, excepto la línea final que renderiza la vista como HTML:
   ```python
   return render(request, "reviews/book_detail.html", context)
   ```
5. Por lo tanto, refactorizaremos este código existente para que la función `book_detail` llame a una función privada, `_book_detail(request, pk)`, que devuelva el objeto de solicitud y el diccionario de contexto. Reutilizaremos `_book_detail` cuando escribamos la función `book_detail_pdf`:
   ```python
   from django.template.loader import get_template
   from weasyprint import HTML, CSS
   …

   def _book_detail(request, pk):
       book = get_object_or_404(Book, pk=pk)
       …
       if request.user.is_authenticated:
           …
           request.session['viewed_books'] = viewed_books
       return request, context

   def book_detail(request, pk):
       request, context = _book_detail(request, pk)
       return render(request, "reviews/book_detail.html", context)
   ```
6. Con el paquete ahora instalado, crea un nuevo archivo llamado `book_detail_pdf` que contendrá la lógica de generación de PDF. Dentro de este archivo, escribe el siguiente código:
   ```python
   def book_detail_pdf(request, pk):
       request, context = _book_detail(request, pk)
       # Render the HTML
       template = get_template('reviews/book_detail_pdf.html')
       html = template.render(context=context, request=request)
       css = CSS(string='')
       # Create a response
       fname = f"book_detail_{book.isbn or book.id}.pdf"
       cdisposition = f"inline; filename={fname}"
       response = HttpResponse(content_type='application/pdf')
       response['Content-Disposition'] = cdisposition
       response['Content-Transfer-Encoding'] = 'binary'
       # Write the PDF binary to the response
       return HTML(
           string=html,
           base_url=request.build_absolute_uri()
       ).write_pdf(response, stylesheets=[css])
   ```
   Ahora, intentemos entender qué hace este código. En la primera línea, importamos la clase `HTML` del paquete `weasyprint`, que instalaste en el Paso 1:
   ```python
   from weasyprint import HTML
   ```
   Esta clase `HTML` nos proporciona un mecanismo a través del cual podemos leer el contenido HTML de un sitio web si tenemos su URL.
   En el siguiente paso, creamos un nuevo método llamado `book_detail_pdf()` que toma dos parámetros: el objeto de solicitud y la clave primaria del libro para el que se generará el PDF:
   ```python
   def book_detail_pdf(request, pk):
   ```
   A continuación, pasamos el HTML renderizado al objeto de clase `HTML` que importamos anteriormente. Esto hizo que la biblioteca `weasyprint` analizara la cadena HTML y leyera su contenido HTML. Una vez hecho esto, llamamos al método `write_pdf()` del objeto de clase `HTML` y proporcionamos el nombre del archivo en el que se debe escribir el contenido:
   ```python
   pdf_stream = HTML(string=html, base_url=request.build_absolute_uri()).write_pdf(stylesheets=[css])
   ```
   *Figura 13.4: El PDF de detalles del libro generado con WeasyPrint*

#### Actividad 13.01 – Adición de funcionalidad adicional a una vista

La inclusión de información resumida en un informe hace que la información sea más fácil de comprender para el usuario y de comparar con otros informes.

Esta actividad se centrará en agregar funcionalidad adicional a una vista para incluir informes de datos agregados en el PDF. En la parte superior de cada sección de Reseña en el PDF de detalles del libro, agrega un gráfico simple que muestre cuántas estrellas recibió cada película. Este gráfico se implementará como una tabla HTML.

El módulo `collections` de la biblioteca estándar de Python proporciona una clase `Counter`. Puedes usar esta clase para crear una función en `reviews/utils.py` llamada `ratings_to_histogram`. A `ratings_to_histogram` se le pasa una lista de calificaciones de reseñas y produce una lista de valores que se utilizarán en la plantilla para producir un gráfico que muestra la distribución de calificaciones para el libro:

```python
>>> ratings_to_histogram([4, 5, 3, 4])
[('*****', 1), ('****', 2), ('***', 1), ('**', 0), ('*', 0)]
```

Incluye `book_rating_histogram` como una clave en el contexto de la vista `book_detail_pdf` en `reviews/views.py`. Tendrá un valor de la siguiente manera:

```python
ratings = [review.rating for review in reviews]
book_rating_histogram = ratings_to_histogram(rating)
```

En `reviews/templates/reviews/book_detail_pdf.html`, realiza los siguientes cambios para incorporar el gráfico:
1. Asegúrate de que el gráfico solo aparezca en libros que hayan recibido al menos una reseña.
2. Los datos del histograma se pueden mostrar en una tabla con dos columnas dentro de una etiqueta `div` con un ID de HTML de `histogram`. La columna izquierda contendrá las estrellas y la columna derecha contendrá su conteo. Esto se puede lograr iterando mediante un bucle `for` como este:
   ```django
   {% for stars, count in book_rating_histogram %}
   ```
3. Asigna a los bloques de datos `div` existentes los IDs de HTML de `logoheader`, `bookinfo`, `bookcover` y `reviews`. Coloca el título en un `div` propio llamado `title`. Agrega un color de fondo al `div` `logoheader` para que las partes blancas del logotipo sean visibles.
4. Incluye algo de CSS para que el `div` `histogram` flote a la derecha de `bookinfo`. Para contrastar con el resto del informe, utiliza texto blanco sobre un fondo gris oscuro.

Cuando hayas terminado esta actividad, deberías ver un gráfico simple como este insertado en el PDF:

*Figura 13.5: Gráfico de calificaciones*

El diseño general ahora se verá como el PDF en la Figura 13.6:

*Figura 13.6: Gráfico de calificaciones*

Hasta ahora, hemos aprendido cómo podemos generar diferentes tipos de archivos binarios con Python, lo que puede ayudarnos a exportar nuestros datos de manera estructurada o ayudarnos a imprimir versiones en PDF de nuestras páginas. A continuación, aprenderemos a generar representaciones gráficas de nuestros datos utilizando Python.

La solución para esta actividad se puede encontrar en la carpeta `Chapter13` en el repositorio de GitHub de este libro.

---

### Sección: Generación de gráficos con Plotly

Los gráficos son una excelente manera de representar visualmente datos que cambian dentro de una dimensión específica. Nos encontramos con gráficos con bastante frecuencia en nuestra vida cotidiana, ya sean gráficos meteorológicos para una semana, movimientos del mercado de valores o boletas de calificaciones de desempeño de los estudiantes.

Plotly es una biblioteca popular para producir visualizaciones enriquecidas con JavaScript. Está implementada tanto en Python como en el lenguaje de programación R, muy popular entre estadísticos y científicos. Permite la visualización rápida de datos con docenas de gráficos configurables, incluidos histogramas, gráficos circulares, gráficos de dispersión, diagramas de Venn, mapas de calor y muchos gráficos en 3D. Uno de sus beneficios es que Plotly es particularmente interesante para nosotros debido a su facilidad de integración con Django.

Comenzaremos con un ejemplo simple de gráfico circular.

Supongamos que tenemos un conjunto de datos con las calificaciones que los usuarios han asignado a las reseñas en Bookr, según la Figura 13.7:

| Calificación | Conteo |
| :--- | :--- |
| ***** | 510 |
| **** | 735 |
| *** | 475 |
| ** | 386 |
| * | 235 |

*Figura 13.7: El conjunto de datos de clasificación por estrellas*

#### Instalación

Para instalarlo en tu sistema, puedes ejecutar el siguiente comando:

```bash
pip install plotly
```

Ahora que has hecho eso, veamos cómo podemos generar una visualización gráfica utilizando Plotly.

#### Configuración de una figura

Antes de que podamos comenzar a generar un gráfico, debemos inicializar un objeto `Figure` de Plotly, que esencialmente actúa como un contenedor para nuestro gráfico. Un objeto `Figure` de Plotly es bastante fácil de inicializar; se puede hacer utilizando el siguiente fragmento de código:

```python
from plotly.graph_objs import graphs

figure = graphs.Figure()
```

El constructor `Figure()` del módulo `graph_objects` de la biblioteca `plotly` devuelve una instancia del contenedor de gráficos `Figure`, dentro del cual se puede generar un gráfico. Una vez que el objeto `Figure` está en su lugar, debemos generar un trazado.

#### Generación de un gráfico

Un trazado (*plot*) es una representación visual de un conjunto de datos. Este trazado podría ser un gráfico circular, un gráfico de dispersión, un gráfico de líneas, etc. Para implementar un gráfico circular de los datos en la Figura 13.7, puedes utilizar el siguiente fragmento de código:

```python
labels = ['*****', '****', '***', '**', '*']
values = [510, 735, 475, 386, 235]
pie_plot = go.Pie(labels=labels, values=values, sort=False)
figure.add_trace(scatter_plot)
figure.show()
```

El método `add_trace()` es responsable de agregar un objeto de trazado a la figura y generar su visualización dentro de la figura. De forma predeterminada, cuando se llama al método `Figure.show()`, la figura se renderizará en un navegador web.

*Figura 13.8: Un gráfico circular de Plotly*

#### Renderizado de un gráfico en una página web

Una vez que el gráfico se ha agregado a la figura, se puede representar en una página web llamando al método `plot()` del módulo de trazado sin conexión de la biblioteca `plotly`. Esto se muestra en el siguiente fragmento de código:

```python
from plotly.offline import plot

visualization_html = plot(figure, output_type='div')
```

El método `plot()` toma dos parámetros principales: el primero es la figura que se debe representar y el segundo es la etiqueta HTML del contenedor dentro del cual se generará el HTML de la figura. El método `plot` devuelve HTML completamente integrado que se puede incrustar en cualquier página web o formar parte de la plantilla para representar un gráfico.

Ahora, con esta comprensión de cómo funciona el trazado de gráficos, probemos un ejercicio práctico en el que generaremos un gráfico para nuestro conjunto de datos de muestra.

#### Gráficos avanzados – Gráficos de burbujas

Para nuestro siguiente ejemplo, desarrollaremos un gráfico de burbujas. Este es un gráfico de dispersión con marcadores ponderados. En el conjunto de datos con el que estamos tratando, trazaremos las reseñas promedio de una variedad de libros y estableceremos el tamaño de los marcadores.

Para nuestro conjunto de datos, usaremos este archivo CSV, que contiene información sobre libros y sus calificaciones:

```csv
publication_year,average_rating,title,num_rating
2014,2.4285714285714284,'Book A',7
2016,2.25,'Book B',4
2018,4.0,'Book C',5
2016,2.3333333333333335,'Book D',3
2011,4.0,'Book E',1
2015,2.875,'Book F',16
2019,2.6315789473684212,'Book G',19
2020,3.375,'Book H',16
2018,5.0,'Book I',1
2011,2.6666666666666665,'Book J',9
2013,5.0,'Book K',1
2013,3.5384615384615383,'Book L',13
2016,2.5789473684210527,'Book M',19
2010,2.2941176470588234,'Book N',17
2014,3.3333333333333335,'Book O',12
2015,3.0526315789473686,'Book P',19
2020,2.857142857142857,'Book Q',7
2018,3.0,'Book R',16
2015,2.5,'Book S',18
2020,3.230769230769231,'Book T',13
```

Necesitamos importar `plotly` y el módulo `graph_objects` y luego leer el archivo CSV:

```python
import csv
import plotly as py
import plotly.graph_objects as go

with open('data.csv') as f:
    reader = csv.DictReader(f)
    reviews = list(reader)
```

El constructor `Scatter` toma los valores para el eje X y el eje Y y devuelve un objeto que se puede usar para construir un gráfico de dispersión. En nuestro ejemplo, estamos trazando el año de publicación en el eje X y la calificación promedio en el eje Y. Estamos definiendo el marcador utilizando la cantidad de calificaciones presentes en el conjunto de datos. De esta manera, los libros con más calificaciones se verán más prominentes que aquellos con menos calificaciones:

```python
scatter_0 = go.Scatter(
    x=[review['publication_year'] for review in reviews],
    y=[review['average_rating'] for review in reviews],
    text=[review['title'] for review in reviews],
    mode='markers+text',
    textposition='middle center',
    marker=dict(
        size=[int(review['num_rating']) for review in reviews],
        sizemode='area',
        sizeref=0.001,
    )
)
```

Una vez que se ha generado el objeto de gráfico de dispersión, el siguiente paso es agregar este gráfico a `Figure`. Esto se puede hacer de la siguiente manera:

```python
figure = go.Figure()
figure.add_trace(scatter_0)
figure.update_xaxes(range=[2009, 2021])
figure.update_yaxes(range=[-0.2, 5.2])
figure.show()
```

El gráfico de burbujas resultante aparecerá como en la Figura 13.9.

En el próximo ejercicio, incorporaremos este tipo de objeto Plotly en una vista de Django que toma sus datos de una consulta dinámica de la base de datos.

*Figura 13.9: El gráfico de burbujas en Plotly*

#### Ejercicio 13.04 – Generación de gráficos en Python

En este ejercicio, agregaremos una vista de Django que crea un gráfico de burbujas con los datos de reseñas de la aplicación Bookr:

1. Edita `reviews/views.py` e incluye las declaraciones de importación requeridas:
   `reviews/views.py`:
   ```python
   import datetime
   import plotly.graph_objects as go
   from django.db.models import Avg
   from plotly.offline import plot
   ```
2. Define la vista `review_statistics` con los argumentos para `min_publication_year` y `max_publication_year`. Calcula el inicio del primer año y el final del segundo año para que podamos realizar búsquedas inclusivas en este rango de fechas. (Por ejemplo, si `max_publication_year` es 2025, queremos que nuestra búsqueda incluya publicaciones hasta el último segundo del 31 de diciembre de 2025):
   ```python
   def review_statistics(request, min_publication_year, max_publication_year):
       min_year = datetime.datetime(
           min_publication_year, 1, 1)
       max_year = datetime.datetime(
           max_publication_year, 12, 31, 23, 59, 59)
   ```
3. Ahora podemos recuperar las reseñas filtradas en nuestro rango de fechas e incluir agregados de la calificación promedio y el conteo de calificaciones:
   ```python
   reviews = Review.objects.filter(
       book__publication_date__gte=min_year,
       book__publication_date__lte=max_year).values(
       'book__id', 'book__title', 'book__publication_date'
   ).annotate(average_rating=Avg('rating')
   ).annotate(num_rating=Count('rating'))
   ```
4. Podemos crear el gráfico de dispersión utilizando la declaración de clase que discutimos en la sección anterior:
   ```python
   scatter_0 = go.Scatter(
       x=[review['book__publication_date'].year for review in reviews],
       y=[review['average_rating'] for review in reviews],
       text=[review['book__title'] for review in reviews],
       mode='markers+text',
       textposition='middle center',
       marker=dict(
           size=[review['num_rating'] for review in reviews],
           sizemode='area',
           sizeref=0.001,
       )
   )
   ```
5. Ahora podemos crear el diseño y la figura y anotar los ejes con las escalas de fecha de publicación y calificación:
   ```python
   layout = go.Layout(autosize=True, height=900)
   fig = go.Figure(data=[scatter_0], layout=layout)
   fig.update_xaxes(range=[min_publication_year, max_publication_year])
   fig.update_yaxes(range=[-0.2, 5.2])
   ```
6. Crearemos un elemento `div` y lo representaremos mediante una plantilla de Django:
   ```python
   plotly_div = plot(fig, output_type='div')
   return render(request, "reviews/statistics.html", context={'plotly_div': plotly_div})
   ```
7. La plantilla de Django se puede definir de la siguiente manera:
   `reviews/templates/reviews/statistics.html`:
   ```html
   <!DOCTYPE HTML>
   <html>
   <head>
       <title>Review Bubbleplot</title>
   </head>
   <body>
       {% autoescape off %}
           {{ plotly_div }}
       {% endautoescape %}
   </body>
   </html>
   ```
8. Ahora necesitaremos agregar la siguiente ruta a `reviews/urls.py`:
   `reviews/urls.py`:
   ```python
   path('reviews/statistics/'
        '<int:min_publication_year>/'
        '<int:max_publication_year>',
        views.review_statistics,
        name='review_statistics'),
   ```

Al usar Plotly en una vista de Django, hemos abierto nuestras aplicaciones de Django a las sofisticadas visualizaciones de datos que ofrece esta biblioteca. Ahora aprenderemos sobre la generación dinámica de imágenes utilizando la biblioteca Pillow.

---

### Sección: Generación de imágenes en Python con Pillow

Nos encontramos con Pillow en el Capítulo 8 (*Carga de archivos con formularios de Django*), donde la usamos para cambiar el tamaño de las imágenes cargadas.

Si bien Pillow contiene una amplia gama de funciones para la creación y manipulación de imágenes, no investigaremos estos aspectos en su totalidad. Los ejemplos de esta sección analizan transformaciones de imágenes simples como la rotación y el cambio de tamaño.

1. Primero, debemos importar la clase `Image` del módulo `PIL`:
   ```python
   from PIL import Image
   ```
2. Ahora, definiremos una función privada que toma dos rutas de imagen como argumentos:
   ```python
   def _logo_transormations(logo, background):
   ```
3. Cargaremos estas imágenes utilizando el método `Image.open` y las convertiremos a un modo de color común:
   ```python
   bg = Image.open(background)
   bg_img = bg.convert('RGBA')
   logo_img = Image.open(logo).convert('RGBA')
   ```
4. Ahora pegaremos el logotipo en el fondo en la posición (100, 50):
   ```python
   bg_img.paste(logo_img, box=(100, 50), mask=logo_img)
   ```
5. Luego rotaremos otra imagen del logotipo 135 grados en sentido antihorario y la pegaremos más a la derecha del primer logotipo:
   ```python
   rot = logo_img.rotate(135, Image.NEAREST, expand=1)
   bg_img.paste(rot, box=(650, 50), mask=rot)
   ```
6. Luego cambiaremos el tamaño del logotipo para que tenga una relación de aspecto de 1:1, lo pegaremos en la parte inferior del fondo y devolveremos la imagen completa:
   ```python
   resize = logo_img.resize((216, 216), Image.NEAREST)
   bg_img.paste(resize, box=(100, 400), mask=resize)
   return bg_img
   ```
7. Finalmente, tenemos el código para llamar al método privado:
   ```python
   # bookr logo: 432 x 216 pixels
   logo = 'reviews/static/reviews/logo.png'
   # wooden background: 1280 x 720 pixels
   background = 'media/backgrounds/wood_background.jpg'
   transformed_img = _logo_transormations(logo, background)
   transformed_img.save('logo_transforms.png', quality=95)
   ```

La imagen de fondo de madera está disponible en la carpeta `Chapter13` en el repositorio de GitHub de este libro.

La ejecución de este código creará la siguiente imagen:

*Figura 13.10: logo_transformation.png*

Te darás cuenta muy rápidamente de que esta no es una forma muy intuitiva de manipular gráficos, y lleva mucho tiempo lograr incluso un efecto básico al usar esta biblioteca en comparación con el tiempo empleado con una aplicación de imagen estándar como Microsoft Paint o Adobe Photoshop. El beneficio de este enfoque es que es muy fácil automatizar este código y aplicarlo a la creación de imágenes mediante scripts o al renderizado dinámico de una imagen en un servidor web.

Como vimos en los ejemplos anteriores, es muy fácil envolver este tipo de datos binarios en una `HttpResponse` de Django. Aquí hay una vista que podemos agregar a `reviews/views.py` que usa la función privada `_logo_transformations` y devuelve una imagen PNG:

```python
from PIL import Image
…

def logo_transormations(request):
    logo = 'reviews/static/reviews/logo.png'
    background = 'media/backgrounds/wood_background.jpg'
    transformed_img = _logo_transormations(
        logo, background)
    response = HttpResponse(content_type="image/png;")
    transformed_img.save(response, 'PNG', quality=95)
    return response
```

#### Actividad 13.02 – Creación de gráficos dinámicos en Django con Pillow

La idea de este ejemplo es generar una imagen de fondo para el perfil de un revisor que contenga una pila de libros que ha revisado. El código es una versión dinámica de nuestro ejemplo anterior.

Las portadas de los libros que el usuario ha revisado se guardan en una lista llamada `covers`. Luego, cuando iteramos a través de las portadas, cada portada se pega en el fondo en un ángulo y posición diferentes. Esto se asemeja a una pila desordenada de libros que el usuario ha revisado.

Esta actividad te ayudará a consolidar tu conocimiento de Pillow y su uso dentro de Django. Aprenderás que Pillow se puede utilizar para generar imágenes JPEG:

1. Necesitamos incluir un archivo `wood_background.jpg` en `media/backgrounds`. Puedes copiar el de esta actividad desde la carpeta `Chapter13` en el repositorio de GitHub de este libro o proporcionar el tuyo propio.
2. Crea una función en `reviews/views.py` llamada `get_review_profile_img`. Tomará `image_type`, `user_id`, `background_path` como argumentos y devolverá una imagen.
3. En la función, crea una lista llamada `covers` que conste de las rutas de las imágenes de portada relativas a `MEDIA_ROOT`. Obtén esta lista filtrando objetos `Review` para que `creator_id` coincida con el parámetro `user_id`.
4. Combina la ruta `MEDIA_ROOT` con `background_path`, abre esta imagen y conviértela a `'RGBA'`. Necesitamos usar RGBA en lugar de RGB porque estamos haciendo uso del canal alfa de las imágenes para solucionar problemas de transparencia.
5. Crearemos una lista de orientaciones y alturas de los libros. La primera se utilizará para especificar las rotaciones de grados y la segunda se utilizará para especificar el posicionamiento en el eje Y de cada libro. Estos valores dan resultados satisfactorios, pero considera probar los tuyos propios:
   ```python
   orientations = [30, -15, 15, 0, -30, 45, -45]
   heights = [125, 150, 175, 150, 125, 100, 150]
   ```
6. Definiremos un valor de desplazamiento izquierdo y derecho para el posicionamiento de la imagen. Estos valores tienen en cuenta el tamaño de la imagen de portada al calcular cada posición en el eje X de la imagen de fondo:
   ```python
   left_displacement = -10
   right_displacement = 280
   ```
7. Ahora enumera a través de la lista `covers` y asigna una orientación y altura a la portada. El bucle `for` comenzará así:
   ```python
   for i, cover in enumerate(covers):
   ```
   La posición del libro en el eje X se determina proporcionalmente a la longitud de la imagen. Podemos asignar un valor como este:
   ```python
   x_pos = int(left_displacement + i*(bg_img - right_displacement)/len(covers))
   ```
8. Abre la imagen de portada y conviértela a RGBA.
9. Rota la imagen según la orientación específica y pégala en las posiciones X e Y derivadas anteriormente sobre la imagen de fondo.
10. Si el tipo de imagen es `'jpg'`, debemos convertirla de nuevo a RGB. La clase `Image` de Pillow puede guardar archivos JPEG, pero los archivos RGBA primero deben convertirse a RGB.
11. Devuelve la imagen. Ahora que has creado una función que produce la imagen del perfil de revisión, debe envolverse en una vista que tome un argumento de solicitud y devuelva una respuesta.
12. Crea una vista en `reviews/views.py` que tome los argumentos `request` y `user_id` llamada `reviewer_profile_jpg`. Llamará a `get_review_profile_img`, especificando `'jpg'` para `image_type`, `user_id` y `'backgrounds/wood_background.jpg'` como `background_path`, devolviendo una imagen.
13. Crea un objeto `BytesIO` y guarda la imagen en él.
14. Devuelve una `HttpResponse` con el valor del objeto `BytesIO` y `content_type` establecido en `'image/jpeg'`.
15. Agrega un patrón de URL de `'reviews/profile/<int:user_id>/background_jpg/'` para `reviewer_profile_jpg` a `urlpatterns` en `reviews/urls.py`. Al final de esta actividad, podrás ver una imagen como la siguiente cuando visites una imagen de fondo de perfil, como `http://localhost:8000/reviews/profile/8/background_jpg`.

Ahora has adquirido cierta comprensión de cómo se puede utilizar Pillow en Django para crear imágenes dinámicas.

*Figura 13.11: La imagen de fondo del perfil*

La solución para esta actividad se puede encontrar en la carpeta `Chapter13` en el repositorio de GitHub de este libro.

---

### Sección: Resumen

En este capítulo, analizamos cómo podemos manejar archivos binarios y cómo la biblioteca estándar de Python, que viene precargada con las herramientas necesarias, nos permite manejar formatos de archivo de uso común como CSV. Luego pasamos a aprender cómo leer y escribir archivos CSV en Python utilizando el módulo `csv` de Python. Más adelante, trabajamos con el paquete XlsxWriter, que nos permite generar archivos compatibles con Microsoft Excel directamente desde nuestro entorno de Python sin que tengamos que preocuparnos por el formato interno del archivo.

A continuación, investigamos el uso de `zipfile` para generar archivos ZIP y utilizamos la biblioteca WeasyPrint para generar versiones en PDF de páginas HTML. Esta habilidad puede resultar útil cuando queremos brindar a nuestros usuarios una opción fácil para imprimir la versión HTML de nuestra página con cualquier estilo CSS agregado de nuestra elección. Luego discutimos cómo podemos generar gráficos interactivos en Python y representarlos como páginas HTML que se pueden ver dentro del navegador utilizando la biblioteca Plotly. En la última sección de este capítulo, profundizamos en Pillow y aprendimos cómo se puede utilizar para crear una imagen dinámica a partir de una solicitud de Django.

En el próximo capítulo, veremos cómo podemos probar los diferentes componentes que hemos estado implementando en los capítulos anteriores para asegurarnos de que los cambios de código no rompan la funcionalidad de nuestro sitio web.
