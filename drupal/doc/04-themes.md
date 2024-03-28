# 4. Drupal: Creación de Temas

_26-09-2005_ _Juan Mellado_

En este artículo explicaré como está hecho el tema (_theme_) que utiliza mi sitio _web_. Es la primera vez que hago uno, y la verdad es que me ha costado bastante. No he acabado de sentirme cómodo con CSS, muy posiblemente porque no he utilizado las herramientas adecuadas.

Como _engine theme_ he utilizado PHPTemplate, que parece ser el _engine_ que finalmente acabará adaptando Drupal como estándar después de la versión 4.6.3 en perjuicio del actual XTemplate.

Este artículo no pretende ser una guía detallada y completa del proceso de creación de temas en Drupal, sólo un relato de mi corta experiencia al respecto. Antes de proceder a la creación, modificación o instalación de un tema en Drupal se debería revisar la información que se encuentra en el sitio _web_ oficial [https://drupal.org](https://drupal.org).

## 4.1. Punto de Partida

Lo primero es pensar en el aspecto que se quiere que tenga la _web_. Yo he optado por un diseño simple, bastante clásico. Una cabecera con el nombre del sitio, una parte central dividida en dos columnas, y un pie de página. Con esto en mente ya se puede empezar a crear el tema.

Por cierto, en vez de hablar de crear temas, se debería hablar de modificar temas, ya que el proceso de creación recomendado normalmente empieza por buscar un tema ya hecho que se parezca a lo que tenemos en mente, descargarlo, copiarlo con otro nombre al directorio de temas, y empezar a modificarlo. El problema que le veo a esta forma de trabajar es que se corre el riesgo de acabar no entendiendo nada, porque los temas por lo general no están bien estructurados ni documentados. Creo que lo mejor es echar un vistazo a varios temas, entender la filosofía de funcionamiento, e ir haciendo poco a poco uno propio. Que acabará tan desorganizado e indocumentado como si lo copiáramos de uno ya hecho. Además, no creo que un tema esté nunca terminado del todo. Siempre habrá pequeñas, o grandes, modificaciones que realizar. Los temas están tan vivos como lo está el contenido de la _web_.

En la documentación que figura en la página oficial de Drupal existe información bastante detallada del proceso de creación de temas, e incluye desde el uso de ficheros ```.theme``` hasta la utilización de XTemplate y PHPTemplate.

## 4.2. PHPTemplate

PHPTemplate utiliza plantillas para definir la estructura global de las páginas y de ciertos tipos de contenido en particular.

Las plantillas son ficheros de texto con extensión ```.tpl.php``` que definen los elementos que deben aparecer en las páginas y su ubicación dentro de las mismas. Pero, y esto es importante de entender, su "ubicación" dentro de las páginas entendidas como documentos de texto, como las veríamos si las abriésemos con un editor de texto en vez de con un navegador. La ubicación "visual" de cada elemento, amén de su apariencia, se determina con ficheros ```.css```.

Los nombres de las plantillas son fijos y no pueden cambiarse. Los únicos nombres válidos son ```page.tpl.php```, ```node.tpl.php```, ```block.tpl.php```, ```comment.tpl.php```, y ```box.tpl.php```, que se utilizan para definir las estructuras de las páginas, nodos, bloques, comentarios y contenedores, respectivamente. El único fichero obligatorio es el primero, y cuando una plantilla concreta no existe, el _engine_ usa una por defecto.

Opcionalmente se puede definir la estructura de otros tipos de elementos, como las listas de _items_ por ejemplo, mediante un fichero de nombre ```template.php``` en el que se sobrescriban las funciones ```theme_``` pertinentes, como ```theme_item_list```.

El número de ficheros totales necesarios para un tema dependerá de lo que se quiera conseguir. Por ejemplo, el tema de este sitio, en estos momentos, se compone de los siguientes ficheros:

```text
page.tpl.php
node.tpl.php
comment.tpl.php
style.css
logo.png
screenshot.png
 ```

Los dos últimos no son realmente obligatorios, pero proporcionan un acabado más profesional al tema. ```logo.png``` contiene el logotipo por defecto a mostrar en el sitio, y ```screenshot.png``` una captura del aspecto de la _web_ con el tema aplicado, es una imagen de 150x90 que se muestra en el menú "_administrar->temas_".

## 4.3. page.tpl.php

La plantilla de la página, ```page.tpl.php```, fue la primera que hice. Con él empecé a entender en qué consistía realmente una plantilla y cómo interactuaba con el _theme engine_.

Las plantillas son ficheros HTML con código PHP embebido que se interpreta dentro del contexto del _engine_. Y dentro de ese contexto, el _engine_ pone a disposición de la plantilla una serie de variables con información del contenido, incluyendo idioma, usuario, fecha de publicación, y así, un largo etcétera. El número de variables disponibles, y su significado, depende enteramente del _engine_ utilizado.

El _engine_ redirige el contenido de la plantilla a la salida del servidor _web_, hacia el navegador, con el objetivo de proporcionar a éste una página HTML válida y bien formada. Y por lo tanto, lo primero que aparece en el fichero ```page.tpl.php``` es la declaración del tipo de documento:

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="<?php print $language; ?>" xml:lang="<?php print $language; ?>">
```

Como se observa, es el inicio habitual de cualquier página _web_. La única diferencia es el código PHP embebido que imprime la variable ```$language```, y que el _engine_ inicializa en función del idioma configurado en Drupal.

Siguiendo la plantilla, después de la declaración del tipo de documento, aparece la sección ```<head>``` estándar de HTML. Para definir esta sección se utiliza normalmente las variable ```$head_title```, que contiene el título de la página, la variable ```$head```, que contiene las cabeceras HTTP generadas por Drupal, y la variable ```$styles```, que contiene los "_imports_" de las hojas ```.css``` a utilizar por la página.

```php
<head>
  <title><?php print $head_title; ?></title>
  <meta http-equiv="Content-Style-Type" content="text/css" />
  <?php print $head; ?>
  <?php print $styles; ?>
</head>
```

Después de la cabecera se encuentra el inicio de la sección ```<body>```:

```php
<body <?php $onload_attributes; ?>>
```

La variable ```$onload_attributes``` debe imprimirse siempre al principio de la página para permitir la ejecución de los posibles _scripts_ que contenga la página.

A continuación aparece la sección que compondrá la cabecera de la página, con el logotipo y/o el nombre del sitio, dependiendo de cómo se haya configurado el tema.

```php
<div id="header">
  <?php if ($logo) : ?>
    <a href="/<?php print url(NULL, NULL, NULL, TRUE); ?>" title="<?php print $site; ?>"><img src="/<?php print $logo; ?>" alt="<?php print $site; ?>" /></a>
  <?php endif; ?>
  <?php if ($site_name) : ?>
    <h1 id="site-name"><a href="/<?php print url(NULL, NULL, NULL, TRUE); ?>" title="<?php print $site; ?>"><?php print $site_name; ?></a></h1>
  <?php endif;?>
</div>
```

Si se observa bien, la operativa de impresión de variables se divide en dos. Por un lado las variables que siempre tienen un valor definido, como la cabecera y estilos de la página, que se imprimen sin más. Y por otro lado las variables que no tienen porque tener siempre un valor definido, como el logotipo o el nombre del sitio, que se comprueban con una sentencia _if_ antes de mandarlas a imprimir.

Por otra parte, y esto es muy importante de entender en este momento, es que no hay porque limitarse a poner el logotipo y el nombre del sitio. Ni tan siquiera hay que hacerlo en ese orden. Es más, los temas no tienen ni tan siquiera porque tener una cabecera. Esto que estoy explicando es sólo un ejemplo, no la única forma de hacer las cosas. Al crear una plantilla podemos colocar los elementos que queramos, y en el orden que queramos. La única condición es que sea código HTML y PHP válido que genere una página bien formada.

Si se ha entendido lo anterior, el resto de código de la plantilla no tiene mayor misterio:

```php
<table id="content">
  <tr>

    <?php if (count($primary_links) || $search_box || ($sidebar_left != "")): ?>
      <td class="sidebar" id="sidebar-left">
        <?php if (count($primary_links)) : ?>
          <ul id="primary">
            <?php foreach ($primary_links as $link): ?>
              <li><?php print $link; ?></li>
            <?php endforeach; ?>
          </ul>
        <?php endif; ?>
        <?php if ($search_box): ?>
          <form action="<?php print $search_url; ?>" method="post">
            <div id="search">
              <input class="form-text" type="text" size="7" value="" name="edit[keys]" />
              <input class="form-submit" type="submit" value="Buscar" />
            </div>
          </form>
        <?php endif; ?>
        <?php print $sidebar_left; ?>
      </td>
    <?php endif; ?>

    <td class="main-content">
      <?php if ($messages != ""): ?>
        <div id="message"><?php print $messages; ?></div>
      <?php endif; ?>
      <?php if ($title != ""): ?>
        <h2 class="content-title"><?php print $title; ?></h2>
      <?php endif; ?>
      <?php if ($tabs != ""): ?>
        <?php print $tabs; ?>
      <?php endif; ?>
      <?php if ($help != ""): ?>
        <p id="help"><?php print $help; ?></p>
      <?php endif; ?>
      <?php print $content; ?>
    </td>

    <?php if ($sidebar_right != "") : ?>
      <td class="sidebar" id="sidebar-right">
        <?php print $sidebar_right; ?>
      </td>
    <?php endif; ?>

  </tr>
</table>
```

Como se observa, para definir la estructura de la página he utilizado una tabla con tres columnas, aunque hoy en día no se recomienda el uso de tablas para maquetar páginas _web_. En la columna de la izquierda se muestran los enlaces primarios (mediante un bucle), el cuadro de búsqueda, y los bloques que se hayan configurado para aparecer a la izquierda. En la columna central el contenido, incluyendo mensajes, título, pestañas, y el texto de la ayuda. Y en la columna de la derecha, los bloques que se hayan configurado para aparecer a la derecha.

He prescindido deliberadamente de algunos elementos disponibles para la página, como son la misión, el _slogan_, los enlaces secundarios, las imágenes, y el _breadcrumb_. La lista completa se encuentra en la página oficial de Drupal,aunque también resultan fáciles de encontrar dentro del propio código del _engine_.

Y por último, el pie de página y el cierre de las secciones que estaban abiertas:

```php
<div id="footer">
  <?php if ($footer_message) : ?>
    <?php print $footer_message; ?>
  <?php endif; ?>
</div>

<?php print $closure; ?>
</body>
</html>
```

La variable ```$closure``` debe imprimirse siempre al finalizar la página, por si se está utilizando alguna función en JavaScript que deba ejecutarse después de que la página se haya mostrado.

## 4.4. node.tpl.php

El fichero ```node.tpl.php``` es la plantilla de los nodos. Y a diferencia de la plantilla de la página, que se interpreta una sola vez, esta plantilla se interpreta una vez por cada nodo que se muestre en la página.

El principio de funcionamiento es el mismo. Las variables que están garantizadas que están inicializadas se imprimen sin más. Las variables que puede que no estén definidas se validan con una sentencia ```if```.

```php
<div class="node">
  <?php if ($page == 0): ?>
    <div class="node-title">
      <a href="/<?php print $node_url; ?>" title="<?php print $title; ?>"><?php print $title; ?></a>
    </div>
  <?php endif; ?>
  <div class="info">
    <?php print $name . ", " . $date; ?>
  </div>
  <div class="content">
    <?php print $content; ?>
  </div>
  <?php if ($links): ?>
    <div class="links">
      <?php print $links; ?>
    </div>
  <?php endif; ?>
</div>
```

Los nombres de las variables son lo bastante significativas como entender que información llevan. Lo único que necesite un poco más de explicación es la primera comprobación, que utiliza la variable booleana ```$page``` para verificar si el nodo se está mostrando él solo o dentro de una lista. Si se muestra solo no se imprime el título del nodo porque ya se imprimió en la plantilla de la página.

## 4.5. comment.tpl.php

La última plantilla del tema corresponde al módulo de comentarios. Es la más sencilla de todas. Es una sección con el título, autor, fecha, y el comentario en si mismo:

```php
<div class="comment">
  <div class="title">
    <?php print $title; ?>
  </div>
  <div class="info">
    <?php print $author . ", " . $date; ?>
  </div>
  <div class="content">
    <?php print $content ?>
  </div>
</div>
```

Recordar que el _engine_ pone a disposición de la plantilla más variables de las que yo estoy utilizando en este ejemplo.

## 4.6. style.css

El último fichero que compone el tema es la hoja de estilos. Es una hoja de estilos normal, sin nada especial. En ella se debe poner todas las clases que se hayan utilizado en las plantillas.

```css
#header {
  margin: 0;
  padding: 0;
  background: #3f3f7f;
  color: #ffffff;
}
#header a {
  text-decoration: none;
  color: #ffffff;
}
#header img {
  margin: 0.2em;
  padding: 0;
}

#site-name {
  margin: 0;
  padding: 0.2em;
  font-family: Helvetica, Verdana, Arial, sans-serif;
  font-size: 2.4em;
}
```

Las ventajas de usar CSS para separar el formato del aspecto son bien conocidas y no abundaré en ellas.

Crear una buena hoja de estilos es tarea complicada. Máxime, si se pretende que se visualice correctamente en varios navegadores distintos. La principal dificultad radica en el hecho de que cada navegador implementa las especificaciones HTML y CSS de forma distinta. Y la mayoría de las veces sin ajustarse al estándar.

## 4.7. Instalación

Como ya comenté en un artículo anterior, para instalar un tema en Drupal basta con crear un subdirectorio con el nombre que queramos en el directorio ```themes```, y copiar dentro los ficheros de plantillas y hojas de estilo. Para que la _web_ lo utilice hay que ir al menú "_administrar->temas_", y activarlo seleccionándolo como tema por defecto.

## 4.8. Conclusiones

Para construir un tema primero hay que crear las plantillas. Y luego las hojas CSS que definan el estilo de las clases utilizadas.

Al principio pensé que Drupal favorecía los diseños basados en columnas, ya que los bloques sólo pueden colocarse en la columna de la izquierda o en la de la derecha. Pero en realidad, una vez que se entiende la filosofía de funcionamiento, no supone una limitación. Es sólo una forma de llamar a las cosas. Izquierda y Derecha pueden transformarse en Arriba y Abajo con una hoja CSS.
