# 2. Drupal: Creación de Contenido

_18-09-2005_ _Juan Mellado_

En el artículo anterior se dejó el _software_ instalado. En este artículo se mostrará cómo organizar el contenido.

Lo primero que hay que entender es que casi todo el contenido que se da de alta a través de Drupal es tratado internamente como un tipo de entidad genérica llamada "nodo". Un nodo es prácticamente cualquier texto que se introduce en la base de datos y se muestra a través de la _web_. Esto permite que a cualquier tipo de contenido se le pueda asociar de forma individual una serie de atributos comunes, independientemente que sea una noticia, un artículo, o el tema de un foro.

Los nodos se organizan en categorías. Por ejemplo, se puede indicar que un nodo es de la categoría "_Noticias_", de forma que se puedan listar fácilmente todas las noticias dadas de alta en la _web_. Pero además, el mismo nodo se podría relacionar con los términos "Deportes" y "Fútbol", para que se sepa el tipo de información que contiene.

Este artículo no pretende ser una explicación completa y exhaustiva del proceso de creación de contenido de Drupal. Antes de proceder a organizar y dar de alta contenido en la _web_ debería consultarse la ayuda que se incluye junto al _software_, así como la documentación que se encuentra disponible en la _web_ oficial [https://drupal.org](https://drupal.org).

## 2.1. Categorías

En Drupal, las categorías se utilizan para organizar el contenido. Y las categorías se implementan mediante vocabularios, términos, y las relaciones que se establecen entre ellos, que pueden ser simples o jerárquicas, incluyendo jerarquías múltiples. A este tipo de organización se le denomina "taxonomía".

Mi objetivo era organizar la _web_ en dos secciones: blog y artículos. Así que tiene creado un vocabulario llamado "_Secciones_" con dos términos: "_Blog_" y "_Artículos_".

```text
Secciones:
- Blog
- Artículos
```

Los vocabularios se dan de alta en Drupal a través del menú "_administrar->categorías->añadir vocabulario_". Y el vocabulario "_Secciones_", en particular, se dio de alta con los siguientes parámetros:

```text
Nombre del vocabulario: Secciones
Descripción: Secciones de las que se compone la web
Texto de ayuda: Sección de la web en la que aparecerá el contenido
Tipos: artículo
Términos relacionados: No
Jerarquía: Simple
Selección múltiple: No
Requerido: Sí
Peso: 0
```

Los tres primeros parámetros son auto-explicativos. El parámetro "_Tipos_" permite seleccionar la clase de nodo al que aplica el vocabulario. He marcado sólo artículos porque a priori no pienso clasificar las páginas estáticas. "_Términos relacionados_" indica si se quiere establecer relaciones entre los propios términos del vocabulario. He seleccionado "_No_" porque para este vocabulario no tiene sentido establecer este tipo de relaciones. "_Jerarquía_" establece las relaciones entre los términos. He escogido una jerarquía simple, tipo árbol. El parámetro "_Selección múltiple_" permite que cuando se dé de alta contenido se pueda elegir varios valores de este vocabulario en vez de uno sólo. He seleccionado "_No_" porque un nodo en concreto será una noticia o un artículo, no ambas cosas a la vez. Para "_Requerido_" está seleccionado "_Sí_", para forzar que se indique a qué sección tiene que ir un nodo cuando se dé de alta. Y por último, "_Peso_" está a cero porque no hay más vocabularios dados de alta todavía, y yo lo veo como un parámetro de ajuste una vez esté todo dado de alta.

Una vez creado el vocabulario se puede seleccionar en la lista de categorías y añadirle términos a través de la opción de menú "_administrar->categorías->añadir término_". El vocabulario "_Secciones_" en particular tiene dos términos dados de alta con los siguientes parámetros:

```text
Nombre del término: Blog
Descripción:
Padre: <raíz>
Sinónimos:
Peso: 0
Nombre del término: Artículos
Descripción:
Padre: <raíz>
Sinónimos:
Peso: 0
```

Los parámetros son bien sencillos y no requieren mayor explicación. Lo que es importante entender es que a partir de este momento, cuando se dé de alta contenido en la _web_, se tendrá que asociar a dicho contenido un término del vocabulario "_Secciones_".

Una nota aclaratoria. Lo que pretendo con la sección "_Blog_" es ir poniendo pequeñas notas acerca de las actualizaciones que haga en la _web_, como puede ser la publicación de un nuevo artículo. Así que otra opción, en vez de crear una categoría y asociarle contenido, podría ser utilizar uno de los módulos que trae la distribución de Drupal que permite crear _blogs_.

## 2.2. Agregar Contenido

El contenido se da de alta a través de la opción de menú "_crear contenido nuevo_". No tiene mucho misterio. Un simple formulario para la introducción del texto con unas pocas opciones acerca del autor, formato, y comentarios. Y por supuesto, el desplegable para la selección de un término del vocabulario "_Secciones_".

Una vez dado de alta el primer contenido, que en mi caso fue la noticia de inauguración de la _web_, este se mostrará automáticamente en la página principal del sitio, desapareciendo la página por defecto en la que se solicita dar de alta un primer usuario.

En el tema que viene configurado por defecto, cada nodo se muestra con título, autor y fecha. Y junto a la fecha aparece un enlace por cada término asociado al nodo. Así, junto a la noticia de la inauguración de la _web_ aparecía un enlace con el texto "_Blog_", que al pulsarlo listaba todos los nodos asociados a "_Blog_".

## 2.3. URLs amigables

Los enlaces creados por Drupal son bastantes incómodos de leer y recordar, porque añaden el parámetro ```?q=``` a todas las direcciones. Además, los vocabularios y términos son referenciados por su identificador, no por su descripción. Así, para listar todos los artículos dados de alta en la base de datos se debe acceder a través de la dirección ```http://www.<dominio>.com/?q=taxonomy/term/2```.

Para que Drupal cree URLs más amigables para los usuarios es necesario ir al menú "_administrar->opciones_" y marcar como "_Activado_" la opción URLs limpias. Con esto conseguimos que no se añada el parámetro ```?q=``` a la URLs, y que se pueda usar el enlace ```http://www.<dominio>.com/taxonomy/term/2``` para acceder al listado con todos los artículos.

No obstante, el aspecto de las URLs se puede mejorar un poco más todavía. Para ello hay que activar el módulo ```path``` a través del menú "_administrar->módulos_", lo que provocará que aparezca una nueva opción "_alias de url_" en el menú "_administrar_". A través de "_administrar->alias_" de "_url->añadir alias_" podemos dar de alta alias como el siguiente:

```text
Ruta existente en el sistema: taxonomy/term/2
Nuevo alias de ruta: articulos
```

De forma que se pueda acceder al listado de artículos a través de la dirección ```http://www.<dominio>.com/articulos```.

Una última nota, Drupal en la página principal del sitio muestra por defecto el listado de todos los nodos dados de alta en la base de datos. Podemos cambiar este comportamiento modificando el valor de la opción "_administrar->opciones->Página predefinida de inicio_". El valor por defecto es "_node_", pero puede cambiarse al alias "_articulos_", por ejemplo, para que muestre el listado de artículos.

## 2.4. Más vocabularios

Para poder clasificar los artículos que vaya escribiendo, he dado de alta otro vocabulario titulado "_Temas_":

```text
Nombre del vocabulario: Temas
Descripción: Temas tratados en la web
Texto de ayuda: Tema del que trata el contenido
Tipos: artículo
Términos relacionados: No
Jerarquía: Simple
Selección múltiple: No
Requerido: No
Peso: 0
```

Y que en este momento sólo contiene el término "_Drupal_":

```text
Nombre del término: Drupal
Descripción:
Padre: <raíz>
Sinónimos:
Peso: 0
```

## 2.5. Conclusiones

El objetivo básico de un CMS es organizar todo el contenido dado de alta. Y Drupal lo consigue a través del módulo de categorías. Mediante una interface muy simple permite dar de alta vocabularios, términos, y establecer relaciones entre ellos.

El ejemplo descrito en este artículo es muy simple y no muestra todo el potencial de Drupal. Creo que para hacerse una idea de lo que puede llegar a conseguirse hay que imaginar, por ejemplo, una gran biblioteca con miles de volúmenes. Para catalogar toda esa información haría falta varias decenas de vocabularios (autores, editoriales, temas, etc), y varios miles de términos relacionados, de forma que pudieran realizarse consultas por cualquiera de esos términos, e ir de un contenido a otro a través de términos relacionados.

De igual forma podemos imaginar un fabricante de motores con unos pocos modelos, pero cientos de piezas de todo tipo (tornillos, tuercas, arandelas, etc). De forma que las consultas sobre un motor en particular deberían poder presentarse por bloques (carburador, caja de cambios, etc), sin gran nivel de detalle, o por pequeñas piezas de un determinado tipo.

Lo importante es darse cuenta de que ambos ejemplos, el de la biblioteca y el del fabricante de motores, pueden ser gestionados por Drupal. Drupal no sirve sólo para gestionar el contenido de una _web_, sino para gestionar cualquier clase de contenido y publicarlo en _web_.
