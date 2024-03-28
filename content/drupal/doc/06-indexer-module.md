# 6. Drupal: Creación de Módulos (indexer.module)

_04-10-2005_ _Juan Mellado_

En este artículo explicaré el código de un módulo que he creado. Es un módulo que genera automáticamente páginas con listados con enlaces (_links_), ya sea de un término concreto, como "_Drupal_", o de todos los términos de un vocabulario por defecto, como "_Temas_".

El módulo lo he llamado ```indexer```, de forma que accediendo a la ruta ```indexer``` se muestra un listado de enlaces a todos los artículos de todos los términos de un vocabulario por defecto. Y accediendo a la ruta ```indexer/drupal``` se muestra un listado de enlaces a todos los artículos de la serie dedicada a "_Drupal_".

## 6.1. indexer.module

En la práctica, el módulo no es más que un fichero de nombre ```indexer.module```, copiado en el directorio ```modules```, y que contiene código PHP con una serie de _hooks_ que son llamados por Drupal.

Los _hooks_ son funciones de nombre ```hook_xxx```, como ```hook_menu```, ```hook_perm```, y ```hook_help```. Los módulos las sobreescriben con funciones equivalentes de nombre ```modulo_xxx```, como ```indexer_menu```, ```indexer_perm```, e ```indexer_help```.

En el resto de este artículo explicaré todas las funciones utilizadas en el módulo. Para instalar este módulo en cualquier servidor bastaría con crear un fichero de nombre ```indexer.module```, copiar en él todo el código listado en este artículo, y subirlo al directorio ```modules``` de la instalación de Drupal en el servidor _web_.

## 6.2. indexer_menu

El módulo ```indexer``` puede verse como si tuviera dos puntos de interacción con Drupal. El primero de ellos cuando aparece como una opción más dentro del menú principal de administración, a través de la cual se permite configurar un vocabulario a utilizar por defecto. Y el segundo punto cuando se accede a la ruta a través de la cual se listarán los enlaces.

La función ```hook_menu``` es llamada siempre por Drupal antes de crear las páginas, antes de mandarlas al navegador, y en ella puede indicarse los puntos concretos en los que el módulo interactuará con Drupal.

```php
<?php
function indexer_menu($may_cache) {
  $items = array();
 
  if ($may_cache) {
    
    $items[] = array('path' => 'admin/indexer', 'title' => t('indexer'),
      'callback' => 'indexer_admin',
      'access' => user_access('administer indexer'));

    $items[] = array('path' => 'indexer', 'title' => t('Índice'),
      'callback' => 'indexer_page',
      'access' => user_access('access content'),
      'type' => MENU_CALLBACK);
  }
 
  return $items;
}
?>
```

La función devuelve un _array_ de _ítems_. Cada _ítem_ representa un punto de interacción del módulo con Drupal.

En el caso concreto del módulo ```indexer``` se definen dos _ítems_. El primer _ítem_ representa la opción de configuración del menú de administración, y el segundo _ítem_ la página a través de la que se mostrarán los índices.

Para cada _ítem_ se definen algunas propiedades, como la ruta a través de la que se accederá, el título a mostrar en la página, la función a la que debe llamarse cuando se acceda a la ruta indicada, y los permisos que deben tener los usuarios para acceder a ella.

La propiedad ```type``` indica el tipo de interacción del módulo con la _web_. En el primer _ítem_ es el de un menú estándar por defecto, y en el segundo _ítem_ el de una función _callback_ sin menú asociado.

El parámetro ```$may_cache``` que recibe la función indica si Drupal puede guardar en _cache_ los _ítems_ devueltos. Y la función ```t```, que se aplica a todas las cadenas de texto, se encarga de gestionar la traducción de las mismas al idioma actual seleccionado.

## 6.3. indexer_pem

Los permisos que se hayan indicado en la función ```indexer_menu``` del apartado anterior deben añadirse a la lista de permisos de la _web_ a través de ```hook_perm```.

```php
<?php
function indexer_perm() {
  return array('administer indexer');
}
?>
```

Para este módulo sólo se ha creado un permiso nuevo, ```administer indexer```, pero se pueden crear tantos como se quiera. Los que ya existieran previamente, como ```access content```, no es necesario añadirlos.

El nuevo permiso aparecerá listado, como uno más, en la opción "_administrar->control de acceso_".

## 6.4. indexer_help

La función ```hook_help``` permite indicar los textos de ayuda a mostrar en los distintos puntos de interacción del módulo con la _web_.

```php
<?php
function indexer_help($section) {
  switch ($section) {
    case 'admin/modules#description':
      return t('Permite crear &iacute;ndices de los contenidos de un t&eacute;rmino.');
    case 'admin/indexer':
      return t('Este m&oacute;dulo permite crear &iacute;ndices de los contenidos de un
t&eacute;rmino. Con la URL <em>indexer/t&eacute;rmino</em> se obtiene un &iacute;ndice con los
contenidos del t&eacute;rmino dado, y con la URL <em>indexer</em> un &iacute;ndice de un
vocabulario por defecto. En esta p&aacute;gina puede seleccionarse el vocabulario por defecto.');
  }
}
?>
```

El parámetro ```$section``` contiene la ruta a la que se está accediendo, y lo que único que hace la función es retornar el texto con la ayuda correspondiente a la ruta dada.

## 6.5. indexer_adminis

Cuando un usuario de la _web_ accede a la opción de menú "_administrar->indexer_", Drupal llama a la función ```indexer_admin```, tal y como se le indicó que hiciera en la función ```indexer_menu```.

```php
<?php
function indexer_admin() {
  $edit = $_POST['edit'];
  if ($edit) {
    variable_set('indexer_vocabulary', $edit['indexer_vocabulary']);

    drupal_set_message(t('Se han guardado las opciones de configuraci&oacute;n.'));
  }

  print theme('page', _indexer_admin_display());
}
?>
```

Esta función puede verse como si se llamara dos veces. La primera vez cuando el usuario accede al menú, para que el módulo construya el formulario. Y la segunda vez cuando pulsa el botón de guardar, para que el módulo valide y guarde la selección del usuario. En el código de la función aparece primero la parte que se encarga de recoger la selección del usuario, y después una segunda parte con la llamada a la función que se encarga de construir el formulario. Explicaré primero esta segunda parte.

El código de construcción del formulario está en la función ```_indexer_admin_display```, que es una función privada, local al módulo, no un _hook_.

```php
<?php
function _indexer_admin_display() {
  $output  = form_select(t('Vocabulario por defecto'), 'indexer_vocabulary',
    variable_get('indexer_vocabulary', 0),
    _indexer_get_all_vocabularies(),
    t('Seleccione un vocabulario.'), 0, FALSE, TRUE);

  $output .= form_submit(t('Guardar'));
 
  return form($output);
}
?>
```

Para la construcción de formularios se utiliza un API proporcionado por Drupal que se compone de funciones de nombre ```form_xxx```, como ```form_button```, ```form_checkbox```, o ```form_textfield```. El módulo ```indexer``` en concreto, utiliza una lista desplegable, ```form_select```, donde se muestran todos los vocabularios dados de alta en la aplicación, y un botón, ```form_submit```, para guardar el vocabulario escogido. La lista de vocabularios mostrada en el desplegable se recupera de base de datos con la función ```_indexer_get_all_vocabularies``` que se verá más adelante.

A los controles del formulario se les asocian nombres, como ```indexer_vocabulary```, para que cuando se pulse el botón de guardar, Drupal recoja los valores introducidos en cada control y los guarde en un _array_ asociativo utilizando dichos nombres como clave.

Una vez visto como funciona el formulario de configuración, veamos que ocurre cuando Drupal llama a la función ```indexer_admin``` por segunda vez. En ese momento, en el parámetro ```edit``` de ```$_POST```, se encuentra el _array_ asociativo con los valores tomados de los controles del formulario, lo que da la posibilidad de almacenarlos en base de datos, para que cuando se vuelva a abrir el formulario se puedan recuperar y aparezcan así dichos valores seleccionados.

Los valores se guardan en base de datos como tuplas (clave, valor), sin necesidad de crear una nueva tabla, a través de la función ```variable_set```, y se recuperan con ```variable_get```.

## 6.6. indexer_page

La función ```indexer_page``` es la encargada de construir las páginas con los índices. Drupal la ejecuta cuando un usuario de la _web_ accede a la ruta ```indexer```, tal y como se le indicó que hiciera en la función ```indexer_menu```.

Esta función se comporta de forma distinta dependiendo de los parámetros que se indiquen en la URL. Si se indica el nombre de un término, como "_Drupal_", entonces construye un listado de enlaces (_links_) a todos los nodos del término "_Drupal_". Y si no se le indica el nombre de un término, entonces construye un listado con enlaces a todos los nodos de todos los términos del vocabulario por defecto.

```php
<?php
function indexer_page() {
  $output = '';

  $tname = array_pop(explode('/', $_GET['q']));
  if ($tname != 'indexer') {
    $terms = db_query('SELECT t.tid FROM {term_data} t
      WHERE t.name = \'%s\'', $tname);
    if ($term = db_fetch_object($terms)) {
      $tid = $term->tid;
    }
  }

  if ($tid) {
    $terms = db_query('SELECT t.tid, t.name FROM {term_data} t
      WHERE t.tid = %d', $tid);
  }
  else {
    $vid = variable_get('indexer_vocabulary', 0);
    if ($vid) {
      $terms = db_query('SELECT t.tid, t.name FROM {term_data} t
        WHERE t.vid = %d ORDER BY t.name', $vid);
    }
  }

  if ($terms) {
    while($term = db_fetch_object($terms)) {
      $output .= '<p><h3>' . $term->name . '</h3></p>';

      $output .= '<ul>';
      $nodes = db_query('SELECT n.nid, n.title FROM {node} n
        INNER JOIN {term_node} t ON n.nid = t.nid
        WHERE t.tid = %d AND n.status = 1 ORDER BY n.created', $term->tid);
      while($anode = db_fetch_object($nodes)) {
        $output .= '<li>' . l($anode->title, 'node/' . $anode->nid) . '</li>';
      }
      $output .= '</ul>';
    }
  }

  print theme('page', $output);
}
?>
```

La función se compone de tres partes. La primera parte en la que se extrae el nombre del término de la URL y se comprueba que sea un término válido existente en la tabla ```term_data```. La segunda parte en la que se recupera el detalle del término indicado, o de todos los términos del vocabulario por defecto si no se especificó un término válido. Y una tercera parte en la que se accede a la tabla ```node```, a través de ```term_node```, para recuperar el título de los nodos con los que se construye el índice con la lista de enlaces.

El modelo de datos es sencillo y no debería suponer ningún problema interpretar las consultas. Como mucho comentar que en la última consulta se utiliza la condición ```status = 1``` para recuperar sólo los nodos que hayan sido publicados.

Todos los accesos a base de datos que realiza el módulo utilizan el API de Drupal. Dicho API ofrece funciones del tipo ```db_xxx```, como ```db_query```, o ```db_fetch_object```, que permiten independizar el _software_ de una base de datos concreta.

Los nombres de las tablas aparecen entre llaves "```{}```" para permitir que las funciones del API las detecten y modifiquen su nombre cuando se estén utilizando prefijos para los nombres de las tablas. Es decir, cuando hay varias versiones de Drupal instaladas en una misma base de datos, y para distinguirlas se ha añadido un prefijo a cada una de las tablas, como ```v463_xxx``` por ejemplo.

La función ```l``` se utiliza en Drupal para construir todos los enlaces (_links_) de una misma forma, muy importante cuando se tienen habilitadas las opciones de alias de URLs.

## 6.7. Miscelánea

Las funciones privadas auxiliares, siguiendo la guía de estilo propuesto por los desarrolladores de Drupal, empiezan con un guión bajo.

La función ```_indexer_get_all_vocabularies``` se utiliza para obtener la lista de vocabularios dados de alta en base de datos.

```php
<?php
function _indexer_get_all_vocabularies() {
  $vocabularies = array();

  $result = db_query('SELECT v.vid, v.name FROM {vocabulary} v ORDER BY v.name');
  while($vocabulary = db_fetch_object($result)) {
     $vocabularies[$vocabulary->vid] = $vocabulary->name;
    }

  return $vocabularies;
}
?>
```

## 6.8. Conclusiones

Uno de los puntos fuertes de Drupal es la facilidad con la que puede extenderse o modificarse el funcionamiento del mismo a través de módulos. No es frecuente encontrar _software_ que lo ponga tan fácil.

En este artículo se ha expuesto el ejemplo de un módulo muy sencillo creado para la versión 4.6.3 de Drupal. Aunque ni se han implementado todos los _hooks_ existentes, ni la forma en que se ha hecho es la única forma de construir un módulo.

A modo de resumen, y como aspecto positivo, señalar que hacer un módulo sencillo es relativamente fácil si se tiene cierta experiencia como programador y se sabe leer el modelo de datos. Drupal da bastantes facilidades a la hora de construir formularios, acceder a base de datos, e integrar el funcionamiento de nuestro módulo dentro del funcionamiento normal de Drupal. Aunque apuntar también que las APIs de Drupal parecen cambiar con cada nueva versión, y eso podría implicar tener que modificar el código de los módulos para que funcionen con cada nueva versión.
