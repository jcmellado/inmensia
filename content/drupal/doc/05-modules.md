# 5. Drupal: Módulos

_30-09-2005_ _Juan Mellado_

Los módulos en Drupal permiten extender la funcionalidad de la _web_. Por ejemplo, añadiendo la posibilidad de realizar búsquedas por palabras dentro del contenido, o de agregar comentarios a los nodos.

En este artículo me limitaré a dar una pequeña introducción acerca de cómo están organizados los módulos en Drupal, y hablaré de algunos de los que he estando probado.

## 5.1. Módulos

Los módulos en Drupal son ficheros con extensión ```.module``` que contienen funciones escritas en PHP. Estas funciones actúan como _hooks_, que son llamados desde el código fuente de Drupal. Por ejemplo, cada vez que un nodo es creado, visualizado, modificado o borrado, Drupal llama a uno de estos _hooks_ pasándole el contenido del nodo. De esta forma los módulos tienen la posibilidad de modificar la información antes de que se mande definitivamente al navegador.

Los _hooks_ no sólo son llamados cuando se gestionan los nodos, también se llaman en muchas otras circunstancias. Por ejemplo, cuando se comprueba si el usuario tiene acceso a la información que está intentando consultar, cuando se empieza a crear o está a punto de servirse una página _web_, cuando se están creando o modificando comentarios, cuando se están creando los menús, y así un largo etcétera.

Todas estas llamadas que realiza Drupal a los módulos posibilita que el sistema sea altamente "maleable", logrando que se pueda modificar prácticamente el funcionamiento entero del sistema sin tener que alterar en ningún momento el núcleo (_core_) del mismo. De hecho, muchas de las características básicas de Drupal, como puede ser la posibilidad de organizar el contenido en categorías, son tratadas en la práctica como módulos.

La mayoría de los módulos en Drupal se instalan simplemente copiándolos en el directorio ```modules```. Y aunque normalmente sólo se componen de un fichero ```.module```, algunas veces pueden venir acompañados de archivos auxiliares, como hojas de estilo CSS por ejemplo.

Actualmente existe un número realmente grande de módulos disponibles. No obstante, antes de instalar módulos se deben revisar los requerimientos de los mismos. La mayoría de los módulos sólo piden que se instalen en una versión determinada de Drupal, pero otros pueden exigir la presencia de algún componente externo a Drupal, como la posibilidad de ejecutar código Perl, o quizás una librería de terceros. Incluso algunos requieren la modificación del esquema de la base de datos, mediante la adición de columnas a las tablas ya existentes, o la creación de tablas nuevas.

## 5.2. search.module

Este módulo viene con la distribución de Drupal, aunque por defecto no se encuentra activo. Sirve para _indexar_ el contenido del sitio de forma que se puedan realizar búsquedas sobre el mismo.

Después de activarlo, de la forma habitual a través del menú "_administrar->módulos_", aparecerá una nueva opción de menú "_administrar->opciones->buscar_". A través de esta opción se puede ver el nivel de indexación del contenido del sitio, el número de elementos que queremos que se indexen cada vez, y el tamaño mínimo de las palabras a indexar.

En las páginas _web_ el bloque de búsqueda aparece como un formulario que se compone de un cuadro de entrada de texto y un botón. Para que se visualice, siempre y cuando lo soporte el tema que tengamos configurado, hay que marcar la opción "_Bloque de Búsqueda_" a través del menú "_administrar->temas_". Y para que los usuarios anónimos que visitan el sitio tengan la posibilidad de realizar búsquedas, se les debe dar permiso a través de "_administrar->control de acceso_" marcando como activa la casilla del módulo "_search_".

El proceso de indexación del contenido no es automático. Se puede ejecutar manualmente a través del menú, o invocando desde el navegador al fichero ```cron.php``` que se distribuye con Drupal, y que se encuentra normalmente en el directorio raíz de la instalación. Otra posibilidad es automatizar el proceso a través del ```cron``` del servidor utilizando un ```crontab``` similar al siguiente:

```text
0 4 * * * wget -O - -q http://www.dominio.com/cron.php
```

Comentar que ```cron``` es un servicio (_demonio_) planificador de tareas que se ejecuta en el servidor. Y ```crontab``` es un comando Unix que permite ver, editar, o borrar, las lista de tareas actuales programadas. Cada tarea se programa indicando la hora en la que se quiere ejecutar, y la línea de comandos a ejecutar. Así, en el ejemplo anterior, se está indicando que se ejecute un ```wget``` sobre ```cron.php``` cada día a las cuatro de la mañana. Los asteriscos se utilizan como comodines para día, mes y año, indicando así que la tarea programada debe ejecutarse periódicamente todos los días. ```wget``` es un comando que hace peticiones HTTP, como si se hicieran desde un navegador manualmente.

Si la programación del ```cron``` resulta innacesible, o simplemente difícil de entender, siempre queda la solución de utilizar un módulo de Drupal llamado "_Poormanscron_", algo así como el ```cron``` de los pobres, que realiza todo el proceso automáticamente por nosotros de forma transparente.

## 5.3. comment.module

Este módulo permite que se puedan añadir comentarios a los nodos.

El módulo viene con la distribución de Drupal, pero al igual que el de búsqueda del apartado anterior, se encuentra desactivado por defecto. Al activarlo, a través de "_administrar->módulos_", aparecerá una nueva opción de menú "_administrar->comentarios_". En esta opción puede verse la lista de los últimos comentarios enviados y los que están pendientes de aprobación. Además, permite configurar la forma en la que deben visualizarse los comentarios en las páginas, tanto la lista de comentarios en sí misma, como el formulario para su envío. Opcionalmente, y si se configura la característica de moderación de comentarios, pueden crearse roles y umbrales de moderación, y consultarse los votos efectuados en cada comentario.

Para los que usuarios anónimos puedan ver y enviar comentarios, se les debe dar los permisos oportunos a través del menú "_administrar->control de acceso_", además de a través de las opciones propias del módulo de comentarios. Como mínimo deberían activarse los permisos "_acceder a comentarios_", "_enviar comentarios_", y "_enviar comentarios sin aprobación_", aunque este último no es realmente obligatorio, dependerá de la política de cada sitio en concreto.

A partir de la activación de este módulo, a continuación de todos y cada uno de los contenidos mostrados en el sitio, aparecerá un enlace con el texto "_añadir nuevo comentario_", si no se ha enviado ningún comentario todavía, o "_(n) comentarios_", si se ha enviado alguno, siendo "_n_" el número de comentarios enviados.

## 5.4. upload.module

Este módulo permite que se puedan adjuntar ficheros a los nodos. Es un módulo que viene con la distribución de Drupal, y tras activarlo aparece una nueva opción de menú en "_administrar->opciones->cargas_". En esta opción se puede configurar el tamaño máximo permitido para los ficheros adjuntos y la resolución máxima (_Ancho x Alto_) en caso de que el fichero se trate de una imagen.

A partir de la activación de este módulo, y dentro del formulario de creación de contenido nuevo, aparecerá un cuadro de "_Adjuntos_" a través del que se podrán seleccionar los ficheros a adjuntar al nodo. Los ficheros se seleccionan escribiendo la ruta completa en la que se encuentran ubicados, o a través del botón "_Examinar..._" que abre la típica ventana de selección de archivos sobre el contenido de nuestro disco duro local. Los ficheros adjuntados a un nodo se listan al final del mismo en forma de enlaces (_links_) para que puedan ser vistos o descargados por los visitantes de la _web_.

A través de la opción _"administrar->control de acceso_" se pueden configurar los permisos "_cargar ficheros_" y "_ver ficheros cargados_" del módulo.

## 5.5. codefilter.module

Este módulo no viene con la distribución de Drupal, pero se puede descargar desde la _web_ oficial de Drupal. Es obra de Steven Wittens, que a su vez se inspiró en el módulo "_project_" de Kjartan Mannes. Su objetivo es el de mostrar en colores la sintaxis del código PHP embebido dentro del texto de los nodos.

Es un módulo de tipo filtro, es decir, filtra el contenido antes de su presentación. Detecta las ocurrencias tanto de ```<?php``` y ```?>```, como de ```<code>``` y ```</code>```, marcando todo el bloque con la etiqueta (_tag_) ```<div class=”codeblock”>```. Evita la necesidad de sustituir manualmente los ```<``` por ```&<```, los ```>``` por ```&>```, y extrae los elementos que componen el código PHP, como los nombres de las funciones por ejemplo, y les añade etiquetas de estilo con el color correspondiente. En realidad, todo el trabajo lo hace la función estándar ```highlight_string``` que viene con PHP, lo que hace este módulo es poner dicha función a disposición de los usuarios de Drupal.

El proceso de instalación es sencillo, basta con crear un subdirectorio para el módulo en el directorio ```modules```. Una vez activado, de la forma habitual a través de "_administrar->módulos_", se puede configurar mediante "_administrar->formatos de entrada_" indicando los formatos para los cuales se debe aplicar el filtro de código, así como qué filtro tiene prioridad en caso de existir más de uno.

## 5.6. Conclusiones

El sistema de módulos en Drupal es realmente fantástico. Los _hooks_ permiten modificar el comportamiento del sistema de una manera muy sencilla. Es como una cadena de montaje en la que se va pasando individualmente por cada uno de los módulos instalados hasta obtener el producto final resultante de la interacción de todos ellos en conjunto.

La instalación de un nuevo módulo resulta por lo general muy sencilla. Aunque esto depende de los requerimientos de cada módulo en particular. Como mínimo se debe tener los conocimientos necesarios para acceder al servidor y copiar los ficheros del módulo en el directorio ```modules```. Tener una opción de instalación directamente a través del menú de administración podría ser de ayuda para los usuarios sin estos conocimientos.

La idea de programar un módulo resulta bastante tentadora, sobre todo porque el API es claro e intuitivo, como lo demuestra el gran número y la diversidad de los módulos existentes en la actualidad. Trataré de escribir un artículo en un futuro acerca de la programación de un módulo propio a través de un ejemplo.
