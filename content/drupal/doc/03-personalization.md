# 3. Drupal: Personalización y Temas

_22-09-2005_ _Juan Mellado_

Los temas en Drupal definen el aspecto visual de la _web_. Incluyendo, entre otras muchas cosas, los colores, los tipos de letra, y la ubicación de los distintos componentes mostrados en las páginas. En la práctica son un conjunto de ficheros que se encuentran en el directorio ```themes``` del servidor, y que incluyen plantillas con la estructura del sitio, hojas de estilo, e imágenes.

El objetivo de este artículo es detallar los cambios que he realizado para personalizar el aspecto de la _web_, a la vez que una pequeña introducción a como gestiona Drupal los temas.

Este artículo no pretende ser una guía completa y exhaustiva acerca de la elaboración, instalación, y configuración de temas en Drupal. Antes de proceder a crear, modificar, instalar o configurar algún tema debería consultarse la documentación disponible en la _web_ oficial [https://drupal.org](https://drupal.org).

## 3.1. favicon.ico

El icono de favoritos se suele mostrar junto a la URL de la página, en la barra de direcciones, y en la carpeta de favoritos del navegador.

Para cambiar el icono que viene por defecto con la instalación de Drupal basta con sobrescribir el fichero ```favicon.ico``` con otro fichero con el mismo nombre, y que contenga el icono que queramos utilizar para nuestra _web_. Este fichero normalmente se encuentra en el directorio raíz de nuestro servidor _web_, aunque se puede utilizar la etiqueta ```<link rel="shortcut icon" href="http://www.<dominio>.com/favicon.ico">``` dentro de las páginas HTML para que el navegador lo cargue de otra ubicación, e incluso con otro nombre y formato.

Para que aparezca el nuevo icono en el navegador puede ser necesario limpiar la _cache_ (historial). Y en cualquier caso, que se visualice o no el icono depende del navegador que se utilice. Algunos simplemente no soportan esta característica.

## 3.2. Configuración de temas por defecto

En el menú "_administrar->temas_" de Drupal se listan los temas instalados y se ofrecen opciones para configurarlos.

Las opciones de configuración incluyen ajustes globales que aplican a todos los temas, y ajustes particulares para cada tema en concreto. Se puede indicar que se muestre el logotipo que viene con el tema, o uno propio. Se puede configurar las URLs que deben aparecer en los enlaces primarios y secundarios. Y se puede indicar que se muestren, o no, diversos textos, como el usuario y fecha de publicación de los nodos, el nombre del sitio, el _slogan_, la misión, los enlaces primarios y secundarios, o los avatares de los usuarios.

Los enlaces primarios y secundarios se pueden utilizar como menús poniendo URLs en los mismos. Por ejemplo, para mostrar un menú con las opciones "_Blog_" y "_Artículos_" se puede utilizar el siguiente código HTML:

```html
<a href="/blog">Blog</a>
<a href="/articulos">Artículos<a>
```

## 3.3. Bloques

Los bloques son los cuadros que aparecen dentro de las columnas izquierda y derecha de la página. Los bloques los ofrecen los módulos, como el cuadro de búsqueda, pero también es posible crear bloques propios.

En el menú "_administrar->bloques->lista_" se muestran todos los bloques disponibles, y se ofrecen opciones para su configuración. Se pueden activar, dar un peso para ordenarlos, señalar en que columna deben aparecer, e indicar los tipos de nodos para los que deben mostrarse.

A través del menú "_administrar->bloques->añadir bloque_" se pueden crear bloques propios con un título, cuerpo, y descripción. En el cuerpo se puede poner una simple cadena HTML, o incluso código PHP que acceda a base de datos. El uso más común es poner enlaces a los últimos contenidos dados de alta, a los contenidos más visitados, o los últimos comentarios introducidos por los usuarios.

Un ejemplo, el siguiente bloque mostraría una lista con un enlace a los últimos nodos modificados en la base de datos:

```text
Título del bloque: Últimos temas
Descripción del bloque: Últimos temas
Formato de entrada: PHP code
Cuerpo del bloque:
```

```php
<?php
  $result = db_query_range('SELECT n.title, n.nid FROM {node} n ORDER BY n.changed DESC', 0, 5);
  while($node = db_fetch_object($result)){
    $items[] = l($node->title,'node/'.$node->nid);
  }
  print theme('item_list', $items);
?>
```

Confío en que el código sea sencillo de entender. Realiza una consulta para recuperar los nodos, les da el formato de _links_, y los manda a la página.

## 3.4. Temas

Los temas son los que definen el aspecto visual de la _web_. Reciben el contenido a mostrar en una página y le dan formato.

A fin de conseguir una mayor separación entre contenido, control, y presentación, Drupal no incorpora directamente la gestión de temas dentro del núcleo del mismo, sino que delega esta gestión en módulos externos a los que accede a través de llamadas a funciones que éstos están obligados a implementar. Algunos de estos módulos a su vez son muy genéricos, y en vez de ofrecer un único tema, lo que ofrecen es funcionalidad para trabajar con ellos con plantillas. Este tipo de módulos reciben el nombre de _engine theme_, que traducido literalmente sería algo así como "motor de temas".

En la práctica, lo que hacen estos _engines_ es extraer información de Drupal y dejarla en variables accesibles desde las plantillas. En las plantillas los valores de esas variables se transforman en código HTML.

La distribución de Drupal trae XTemplate como _engine theme_ por defecto. Sin embargo, a día de hoy, este _engine theme_ está marcado como deprecado, y es muy probable que desaparezca en futuras versiones en favor de PHPTemplate.

Cada _engine theme_ construye los temas de una forma distinta. Por ejemplo, XTemplate utiliza ficheros ```.xtmpl```, que son ficheros HTML o XHTML que definen las secciones de la página mediante etiquetas. Por su parte, PHPTemplate utiliza ficheros ```.tpl.php```, que son ficheros HTML con código PHP embebido.

En la práctica, incluso es posible construir un tema sin tener que utilizar un _engine theme_, mediante ficheros ```.theme``` que contienen funciones PHP que son llamadas por Drupal.

En lo que todos parecen estar de acuerdo es en utilizar hojas de estilos ```.css```. Incluso Drupal trata como un tema por si mismo a los ficheros de nombre ```style.css``` que se ubican dentro de un subdirectorio de un tema en particular. Esto permite hacer variaciones de dicho tema, cambiando por ejemplo su esquema de colores.

## 3.5. Instalación de Temas

Los temas pueden encontrarse y descargarse desde muchos sitios de Internet, incluida la página _web_ oficial de Drupal. No obstante, antes de instalar un tema es recomendable ver los requerimientos del mismo, para comprobar si tenemos instalado el _theme engine_ adecuado.

Ilustraré esta sección con un ejemplo, con la instalación del tema "_box_grey_" de Adrian Simmons que utiliza el _theme engine_ PHPTemplate.

La instalación de PHPTemplate requiere descargar el fichero ```phptemplate-4.6.0.tar.gz``` del sitio oficial de Drupal, y descomprimirlo en el directorio ```/themes/engine/phptemplate``` de nuestro servidor _web_.

Por su parte, la instalación del tema requiere descargar el fichero ```box_grey-4.6.0.tar.gz``` de la página oficial de Drupal, y descomprimirlo en el directorio ```/themes/box_grey```.

Una vez copiados ambos ficheros al servidor, se pueden seleccionar los nuevos temas y configurarlos como uno más en el menú "_administrar->temas_". Además, en la pestaña de configuración veremos que aparecen opciones nuevas, las propias del _theme engine_ que acabamos de instalar. Los comentarios de esta parte están en inglés, y básicamente vienen a decir que los enlaces primarios y secundarios se deben introducir a través de estas nuevas opciones, en vez de a través de las que vienen por defecto con Drupal.

## 3.6. Conclusiones

El uso de bloques y temas sirve para personalizar el aspecto de la _web_. Con los bloques indicamos el tipo de contenido a mostrar en las columnas laterales. Y con los temas, la disposición y apariencia de todos los elementos de las páginas.

Los temas constituyen todo un mundo por si mismos, y a través de la estructura modular de Drupal consiguen separar perfectamente la presentación del contenido. Instalar y configurar temas ya existentes es bastante fácil. Sin embargo, crear temas nuevos puede ser todo lo complejo que queramos.
