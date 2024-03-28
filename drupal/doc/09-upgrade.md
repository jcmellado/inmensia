# 9. Drupal: Actualización

_20-06-2006_ _Juan Mellado_

En este artículo detallaré los pasos que he seguido para actualizar mi sitio _web_ desde la versión 4.6.6 a la 4.7.2 de Drupal, así como los cambios que tuve que realizar en el código fuente de mis módulos para adaptarlos a las modificaciones realizadas en esta nueva versión de Drupal.

## 9.1. Cambios de Versión

Para proceder a instalar una nueva versión de Drupal siempre hay que seguir el detalle de las instrucciones de actualización que vienen con el nuevo _software_. No obstante, esta información suele encontrarse en inglés y a veces no es fácil de entender exactamente lo que hay que hacer si no se tiene un mínimo de experiencia con el sistema.

Para intentar simplificar todo el asunto, se puede decir que hay dos tipos de actualizaciones: las que no implican realizar modificaciones en base de datos, y las que sí. Las primeras son generalmente más fáciles de hacer, y las segundas normalmente indican la presencia de cambios importantes en el _software_.

Un ejemplo de cambio de versión sin modificaciones de base de datos es el paso de la versión 4.6.7 a la 4.6.8. Se trataba sólo de unos parches de seguridad aplicados en el código fuente de algunos módulos de Drupal, por lo que bastaba con sobrescribir los ficheros de Drupal del servidor con los nuevos.

Por contra, un ejemplo de cambio de versión con modificaciones de base de datos es la versión 4.7.0 de Drupal. Esta versión representa un cambio muy grande con respecto a las anteriores 4.6.x, y requiere tanto la actualización de base de datos como de ficheros. Afortunadamente esta nueva versión ha mejorado notablemente el proceso de actualización y buena parte de él se realiza a través de una página _web_ a modo de asistente (_wizard_).

Como norma general, antes de cambiar a nueva versión de Drupal es conveniente revisar que módulos tenemos instalados y buscar si existen actualizaciones correspondientes a la nueva versión de Drupal que queremos instalar. Por ejemplo, si tenemos instalado el módulo ```codefilter``` en su versión 4.6.x, y queremos actualizar Drupal a la 4.7.x, entonces tendremos que comprobar que existe una versión del módulo ```codefilter``` para la versión 4.7.x. Si no existe tal versión entonces puede ser conveniente esperar a que se publique, o buscar un módulo alternativo si el cambio nos urge.

## 9.2. Actualización a la versión 4.7.2

En esta sección describiré los pasos que seguí para actualizar mi sitio _web_ desde la versión 4.6.6 de Drupal hasta la 4.7.2, última versión disponible en el momento de escribir este artículo.

Lo primero que hice fue revisar en la página oficial de Drupal los cambios realizados en las distintas versiones. Así me enteré que el desarrollo de la rama 4.6.x está actualmente en la 4.6.8, pero las modificaciones introducidas desde la 4.6.6 se limitan a algunos parches de seguridad sobre los fuentes que no implican modificaciones de base de datos, por lo que no me era realmente necesario actualizar pasando por esas versiones intermedias. Por su parte, la rama 4.7.x es una _release_ mayor con muchas modificaciones, y desde la 4.7.0 original se han publicado dos nuevas actualizaciones, pero también con pequeños parches de código por motivos de seguridad, por lo que podía instalar directamente la última versión 4.7.2 liberada desde mi 4.6.6 sin mayores problemas.

La instalación fue como sigue:

- Primero hice un _backup_ (copia de respaldo) de base de datos (con PHPMyAdmin) y de los ficheros del servidor _web_ (con FileZilla).

- A continuación entré como usuario administrador, y a través del menú de administración desactivé algunos módulos propios que sabía que iban a dejar de funcionar con la actualización, ya que no vienen con la distribución estándar de Drupal.

- Después me descargué el fichero ```drupal-4.7.2.tar.gz``` (475 Kb) con la versión 4.7.2 desde la página oficial de Drupal.

- Y copié los ficheros de la nueva versión de Drupal al servidor _web_ evitando sobrescribir mis archivos ```.htaccess``` y ```settings.php```. Esto último es importante tenerlo en cuenta, es el error más común que se comete y que deja el sitio innaccesible.

- Con el nuevo _software_ ya copiado al servidor invoqué al fichero ```update.php``` directamente desde la barra del navegador. Es decir, en el navegador hay que escribir la dirección ```http://www.<dominio>.com/update.php```

- Al ejecutar el fichero se abrió una página _web_ a modo de asistente (_wizard_) en la que prácticamente en dos pasos se actualizó la estructura de base de datos y reconfiguró el sistema por si solo. Lo mejor es dejar las opciones por defecto y limitarse a dar al botón de continuar.

- Al acabar la actualización ya pude volver a acceder al menú de administración y comprobar que el sitio funcionaba correctamente, incluido el tema (_theme_) de la _web_.

- Lo siguiente fue modificar los parámetros de configuración que se han añadido a la nueva versión de Drupal para ponerlos a mi gusto y cargar el idioma español usando el procedimiento habitual. El fichero con la traducción al español es ```es-4.7.0.tar.gz``` (126 Kb) y también puede descargarse desde la página oficial de Drupal.

- Por último volví a activar los módulos uno a uno, descargando versiones actualizadas para los módulos ajenos, y modificando código para los módulos propios que me había hecho yo, tal y como describo en los siguientes apartados.

## 9.3. Cambios en las APIs de Drupal

En las versiones anteriores a la 4.7.0 existía una función distinta para crear cada tipo de control que podía utilizarse dentro de cada formulario. Por ejemplo, para los botones existía la función ```form_submit```, para las listas de selección ```form_select```, y así sucesivamente para cada control.

A partir de la versión 4.7.0 todas esas funciones se han eliminado del API de Drupal, ahora básicamente todo el trabajo se realiza con una única función llamada ```drupal_get_form```. Esta función admite como uno de sus argumentos un _array_ de _arrays_ que contiene todos los controles que quieren añadirse a un formulario, así como sus propiedades concretas (tipo, título, valor por defecto, etc).

Veamos esto con un ejemplo de un formulario de uno de mis módulos. El código para la versión 4.6.x era de la siguiente forma:

```php
<?php
  $output = form_select(t('Vocabulario por defecto'),
    'tagger_vocabulary',
    variable_get('tagger_vocabulary', 0),
    _tagger_get_all_vocabularies(),
    t('Seleccione un vocabulario.'),
    0, FALSE, TRUE);

  $output .= form_submit(t('Guardar'));
 
  return form($output);
?>
```

Y para hacerlo funcionar con la versión 4.7.x tuve que cambiarlo por el siguiente:

```php
<?php
  $form['tagger_vocabulary'] = array(
      '#type' => 'select',
      '#title' => t('Vocabulario por defecto'),
      '#options' => _tagger_get_all_vocabularies(),
      '#default_value' => variable_get('tagger_vocabulary', 0)
    );

  $form['submit'] = array('#type' => 'submit', '#value' => t('Guardar'));

  return(drupal_get_form('tagger_admin_display', $form));
?>
```

Como se observa, lo que se hace en el nuevo código es sustituir las llamadas a la funciones de Drupal por la creación de un _array_ de elementos con los controles que se quieren incluir en el formulario. De esta forma se ha reducido el número de funciones y se ha facilitado que en un futuro se puedan incluir nuevos tipos de controles y propiedadse de una forma más sencilla y uniforme.

## 9.4. Cambios en la Base de Datos de Drupal

En uno de mis módulos tenía una consulta (_query_) que accedía a base de datos para recuperar el resumen (_teaser_) de los nodos para mostrarlos en un listado. A partir de la versión 4.7.0 esta columna ha cambiado su ubicación, pasando de estar en la tabla ```node``` a encontrarse en la tabla ```node_revisions```. Por lo que tuve que cambiar esta consulta:

```php
<?php
      $nodes = db_query('SELECT n.nid, n.title, n.teaser FROM {node} n
        INNER JOIN {term_node} t ON n.nid = t.nid
        WHERE t.tid = %d AND n.status = 1 ORDER BY n.created', $term->tid);
?>
```

Por esta otra:

```php
<?php
      $nodes = db_query('SELECT n.nid, n.title, r.teaser FROM {node} n
        INNER JOIN {node_revisions} r ON n.nid = r.nid AND n.vid = r.vid
        INNER JOIN {term_node} t ON n.nid = t.nid
        WHERE t.tid = %d AND n.status = 1 ORDER BY n.created', $term->tid);
?>
```

## 9.5. Conclusiones

Actualizar una versión de Drupal por una más moderna es una tarea que se debe acometer antes o después. Lo mejor es tomárselo con calma, sacar siempre lo primero un _backup_ de base de datos y ficheros, y planificar el cambio, tomando nota de los módulos que tenemos instalados y comprobando si existen versiones actualizadas.

Los cambios de versión más normales sólo implicarán actualizar uno o dos ficheros de módulos con parches por temas de seguridad o errores críticos. En esos casos incluso no hará falta sobrescribir todos los ficheros, sólo ver cuales han cambiado de fecha con respecto a la versión anterior y copiar solo esos al servidor.
