# 7. Drupal: Creación de Módulos (pages.module)

_25-10-2005_ _Juan Mellado_

Drupal no trae en su instalación por defecto ninguna opción que permita visualizar el contenido de los nodos separados por páginas. Es decir, que permita que los textos más largos se puedan mostrar en varias páginas, con algún tipo de control que permita cambiar de una página a otro. Afortunadamente esta carencia se puede solventar mediante la creación de un módulo. Este artículo explica el detalle del código de un módulo llamado ```pages.module``` que he creado para este propósito.

Para entender el contenido de este artículo quizás sea necesario leer alguno de los artículos previos de esta serie, aunque en cualquier caso siempre puede recurrirse a toda la documentación presente en la página _web_ oficial de Drupal. Allí encontré varios _posts_ en el foro relacionados con este tema, e incluso un módulo llamado ```pagination.module``` realizado por Kitt Hodsten. El código que se detalla en este artículo se basa en dicho módulo, aunque está muy simplificado, ya que he eliminado de él todas las opciones de configuración que ofrecía el módulo original.

## 7.1. pages.module

Para conseguir que el contenido de los nodos aparezca dividido en páginas se utilizará la cadena ```<!-page->``` como separador de páginas. Es decir, se deberá escribir ```<!-page->``` en los puntos en los que se quiera dividir el texto en una nueva página. Naturalmente esta cadena no se mostrará cuando se esté visualizando el nodo. De hecho, el formato de esta cadena es el de un comentario en HTML, por lo que será filtrada automáticamente por el navegador. Este sistema es el mismo que utiliza Drupal para crear los _teasers_ (resúmenes), sólo que en vez de ```<!-page->``` utiliza la cadena ```<!-break->```.

Técnicamente hablando, el principio de funcionamiento de este módulo se basa en intervenir justo en el momento en el que Drupal intente visualizar el contenido de un nodo. Tomar en ese momento el contenido, dividirlo en páginas utilizando las cadenas ```<!-page->``` como puntos de corte, y sustituir el contenido del nodo por la página actual en curso. Para saber cual página es la actual en curso se añadirá el parámetro ```?pag=N``` a la URL de los nodos, siendo ```N``` el número de la página concreta a mostrar. Y además, para permitir navegar rápidamente de una página a otra, se creará automáticamente una lista de enlaces (_links_) a cada una de las páginas.

El módulo sólo se compone de dos funciones, ```pages_help``` y ```pages_nodeapi```, cuyo código se explica de forma detallada en los siguientes dos apartados.

## 7.2. pages_help

Este _hook_ es llamado por Drupal cuando requiere mostrar por pantalla un texto de ayuda asociado al módulo. En el parámetro ```$section``` de la función se recibe el _path_ concreto para el que Drupal necesita mostrar la ayuda, y la función se limita a retornar una cadena de texto con el mensaje correspondiente.

```php
function pages_help($section) {
  switch ($section) {
    case 'admin/modules#description':
      return t('Muestra el contenido de los nodos cortados en p&aacute;ginas. El corte se realiza en las l&iacute;neas donde aparece la cadena "&lt;!--page--&gt;".');
  }
}
```

La única ayuda que muestra ```pages.module``` es la línea de descripción que aparece en el menú "_administrar->módulos_", donde puede activarse o desactivarse su uso.

## 7.3. pages_nodeapi

El _hook_ ```nodeapi``` es llamado por Drupal cada vez que realiza una operación con un nodo, independientemente de que esta operación sea un alta, modificación, borrado, o una simple visualización del contenido del nodo. Este comportamiento da a los distintos módulos instalados en el sistema la posibilidad de realizar operaciones sobre los nodos al mismo tiempo que lo hace Drupal.

```php
function pages_nodeapi(&$node, $op, $teaser = NULL, $page = NULL) {
```

La función recibe cuatro parámetros. El primero de ellos, ```$node```, es una estructura que contiene toda la información del nodo que se está procesando, como puede ser su título o su contenido, y es pasado por referencia para permitir modificar los valores que trae o añadir nuevos. El segundo parámetro, ```$op```, recibe una cadena de texto que contiene la operación concreta que se está realizando, como ```insert```, ```update```, ```delete```, o ```view```, entre otros. El tercer parámetro, ```$teaser```, contiene el _teaser_ del nodo. Y el último parámetro, ```$page```, es una variable de tipo _boolean_ que vale ```true``` si el nodo se está mostrando él sólo en una página _web_ de forma individual, y ```false``` si se está mostrando en un listado junto con otros nodos.

La primera operación que realiza la función es añadir una nueva variable al parámetro ```$node``` que indica si la página que se está visualizando es la última página del nodo:

```php
  $node->pages_lastpage = true;
```

A partir de este momento esta variable estará disponible para el resto de procesos que trabajen con la variable ```$node```.

La función decide a continuación si tiene que realizar algún proceso con el nodo dado. Para ello comprueba los valores de los parámetros ```$op``` y ```$page```, y sólo hace algo en el caso de que estos sean igual a ```view``` y ```true``` respectivamente, es decir, sólo cuando Drupal esté tratando de visualizar un nodo de forma individual:

```php
  switch ($op) {
    case 'view':

      if ($page) {
```

A continuación se divide físicamente el texto del nodo en páginas mediante la función ```explode``` de PHP, que retorna el _array_ resultante de cortar en trozos una cadena de texto (```$node->body```) en función de un determinado patrón de corte (```<!-page->```). Una vez cortado el texto en páginas se cuenta el número de ellas y se guarda en la variable ```$last```:

```php
        $pages = explode('<!--page-->', $node->body);
   
        $last = count($pages);

        if ($last > 1) {
```

Lo que sigue después es la extracción del parámetro ```pag``` de la URL del nodo que se está visualizando. Si no se encuentra dicho parámetro se sobreentiende que la página actual se corresponde con la primera página, y si existe el parámetro se valida para ver que sea un valor numérico mayor o igual que ```1```, y menor o igual que el número total de páginas:

```php
          $pag = $_GET['pag'];

          if ( !isset($pag) || empty($pag) ) {
            $pag = 1;
          }

          if ($pag < 1) {
            $pag = 1;
          }
          if ($pag > $last) {
            $pag = $last;
          }
```

A continuación se crea una lista de enlaces (_links_) a cada una de las páginas mediante un bucle. Cada uno de los enlaces se forma añadiendo a la URL actual el parámetro ```pag=N```, siendo ```N``` cada uno de los números de páginas en los que se ha dividido el texto. La página actual en curso se concatena como un simple texto, sin más, no como un enlace, para distinguirla del resto:

```php
          for($i = 1; $i <= $last; $i ++) {
            if ($pag == $i) {
              $links .= $i;
            } else {
              $links .= l($i, $_GET['q'], array(), 'pag=' . $i);
            }
            $links .= ' ';
          }
```

El último paso que realiza la función consiste en sustituir el contenido del nodo por el de la página actual en curso sobreescribiendo el valor de ```$node->body_```, además de añadirle la lista de enlaces y actualizar el indicador de última página:

```php
          $node->body = $pages[$pag - 1];
         
          $node->pages_links = t('P&aacute;gina: ') . $links;
         
          $node->pages_lastpage = ($pag == $last);
        }
      }
      break;
  }
}
```

## 7.4. node.tpl.php

El código del módulo visto hasta ahora realiza todo el trabajo de cortar el contenido en páginas y permitir navegar entre las distintas páginas añadiendo el parámetro ```?pag=N``` adecuado a la URL. Sin embargo, la lista de enlaces a las páginas, aunque la construye, no la visualiza, se limita a almacenarla en una variable del nodo. La tarea de visualizar la lista de enlaces se deja para el tema (_theme_), que decide la ubicación y aspecto con el que debe mostrarse.

La forma en la que puede accederse a los nuevos valores añadidos a la variable ```$node``` depende enteramente del _engine theme_ que se esté utilizando. Para este ejemplo supondré que se está utilizando PHPTemplate, de forma que sería en el fichero ```node.tpl.php``` en donde se podría añadir un nuevo bloque de código que compruebe si la lista de enlaces está definida para mostrarla en la página _web_:

```php
  <?php if ($node->pages_links): ?>
    <div class="pages-links">
      <?php print $node->pages_links; ?>
    </div>
  <?php endif; ?>
```

Opcionalmente se puede utilizar la otra variable añadida, ```pages_lastpage```, para realizar alguna acción en función de si está en la última página o no:

```php
  <?php if ($node->pages_lastpage): ?>
//...
  <?php endif; ?>
```

## 7.5. style.css

El último paso de todo el proceso consiste en modificar la hoja de estilos del tema (_theme_) que se está utilizando en la _web_ para darle la apariencia visual deseada a la lista de enlaces.

```css
.pages-links {
  font-weight: bold;
  text-align: right;
}
```

## 7.6. Conclusiones

Este sencillo módulo demuestra una vez más la facilidad con la que Drupal permite personalizar los sitios _web_, añadiéndoles características no presentes en la distribución por defecto, o mejorando las existentes, gracias principalmente a su diseño basado en _hooks_.

El _hook_ ```nodeapi``` tiene muchas más posibilidades que las vistas en este módulo. No sólo se limita a permitir modificar el contenido de los nodos a la hora de visualizarlos. Por ejemplo, algunos módulos crean tablas propias para contener información relacionada con los nodos, como es el caso del módulo de comentarios, y utilizan el _hook_ ```nodeapi``` para mantener la coherencia de base de datos, de forma que al borrar un nodo borran también los comentarios asociados al mismo. Otros módulos utilizan este _hook_ para modificar la ventana a través de la que se dan de alta los nodos y añaden sus propias opciones además de las que trae por defecto Drupal.
