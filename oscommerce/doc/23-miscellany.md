# 23. osCommerce: Miscelánea

_15-06-2007_ _Juan Mellado_

## 23.1. Instalar osCommerce en un directorio distinto de "catalog"

Es posible instalar osCommerce en otro directorio distinto a ```catalog```. No obstante, el _breadcrumb_ (secuencia de enlaces de navegación por los que se va pasando) puede no mostrar la ruta correcta. Puede aparecer un enlace a _catalog_ después de _inicio_, aunque en realidad ese directorio no exista.

Buscando por los fuentes de osCommerce se puede ver que la filosofía de funcionamiento del _breadcrumb_ es muy sencilla. Básicamente consiste en que cada página manda su propia ubicación a una clase donde se compone la secuencia final con todos los enlaces. De forma que para eliminar uno de los enlaces del principio hay que buscar en los primeros ficheros que se invocan a la hora de componer una página _web_. En particular ```includes/application_top.php```:

```php
<?php
// include the breadcrumb class and start the breadcrumb trail
  require(DIR_WS_CLASSES . 'breadcrumb.php');
  $breadcrumb = new breadcrumb;
  $breadcrumb->add(HEADER_TITLE_TOP, HTTP_SERVER);
  $breadcrumb->add(HEADER_TITLE_CATALOG, tep_href_link(FILENAME_DEFAULT));
?>
```

Y lo único que hay que hacer es eliminar, o comentar, la línea con el enlace no deseado:

```php
<?php
//  $breadcrumb->add(HEADER_TITLE_CATALOG, tep_href_link(FILENAME_DEFAULT));
?>
```

## 23.2. Eliminar las opciones de notificación

Los clientes de una tienda desarrollada con osCommerce tienen la posibilidad de activar opciones de notificación sobre determinados productos de su interés, e incluso sobre absolutamente todos los productos disponibles. Lo quiere decir que cuando se produzca alguna variación sobre los productos seleccionados, o sobre cualquiera de entre todos los disponibles, se le mandará una notificación.

Si se quieren eliminar dichas opciones hay que localizar los enlaces a la opciones de notificación dentro de los ficheros PHP y quitarlos. Por ejemplo, dentro de la cuenta del usuario existe un enlace que lleva a la página de notificaciones:

```php
<tr>
<td class="main"><?php echo tep_image(DIR_WS_IMAGES . 'arrow_green.gif')
 . ' <a href="' . tep_href_link(FILENAME_ACCOUNT_NOTIFICATIONS, '', 'SSL') . '">'
 . EMAIL_NOTIFICATIONS_PRODUCTS . '</a>'; ?></td>
</tr>
```

Para evitar que salga el enlace lo único que hay que hacer es quitar esas líneas. Pero por supuesto, quitar el enlace no sirve para mucho si se deja el fichero destino en el servidor, ya que un usuario avanzado siempre podría invocarlo poniendo la URL adecuada en su navegador: ```www.___.com/account_notifications.php```. Es decir, esos ficheros que implementan opciones que no queremos utilizar también hay que borrarlo del servidor, no basta con que no haya ningún enlace a ellos.

## 23.3. Obtener el nombre de la categoría actual

Se puede utilizar el siguiente código para obtener el nombre de la categoría actual seleccionada, con el objeto de poder mostrar dicho nombre dentro de una página, en vez de sólo en el bloque de selección de categorías:

```php
<?php
$cnq = tep_db_query("select categories_name from " . TABLE_CATEGORIES_DESCRIPTION
. " where categories_id = '" . (int)$current_category_id . "'");
$cna = tep_db_fetch_array($cnq);
$cn = $cna["categories_name"];
?>
```

La variable ```$cnq``` es una consulta SQL para obtener el nombre de la categoría actual, que se encuentra siempre disponible en la variable ```$current_category_id```. La variable ```$cna``` almacena el resultado de la consulta. Y la variable ```$cn``` es la que contendrá finalmente el nombre de la categoría actual seleccionado.

Para hilar más fino, y considerar que la tienda puede tener configurados varios idiomas, habría que considerar en la consulta SQL el idioma actual seleccionado por el usuario, que se encuentra siempre disponible en la variable ```$languages_id```:

```php
<?php
$cnq = tep_db_query("select categories_name from " . TABLE_CATEGORIES_DESCRIPTION
. " where categories_id = '" . (int)$current_category_id
. "' and language_id = '" . (int)$language_id . "'");
?>
```

Demasiadas variables globales en osCommerce y PHP en general, ¿no?

## 23.4. Mini-carro de la compra

La imagen del mini-carro de la compra, que puede verse permanentemente en la esquina superior derecha de la _web_, con el número de artículos que contiene en todo momento, es muy sencilla de obtener. Para hacer que aparezca hay modificar el fichero ```catalog/includes/header.php```, sustituyendo la columna de la tabla de la cabecera que contiene las imágenes por defecto:

```php
<td align="right" valign="bottom">
<?php echo '<a href="' . tep_href_link(FILENAME_ACCOUNT, '', 'SSL') . '">'
. tep_image(DIR_WS_IMAGES . 'header_account.gif', HEADER_TITLE_MY_ACCOUNT)
. '</a>&nbsp;&nbsp;<a href="' . tep_href_link(FILENAME_SHOPPING_CART) . '">'
. tep_image(DIR_WS_IMAGES . 'header_cart.gif', HEADER_TITLE_CART_CONTENTS)
. '</a>&nbsp;&nbsp;<a href="' . tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL') . '">'
. tep_image(DIR_WS_IMAGES . 'header_checkout.gif', HEADER_TITLE_CHECKOUT) . '</a>'; ?>
&nbsp;&nbsp;</td>
```

Por otras dos columnas que contengan la imagen del carro en miniatura y el número de artículos:

```php
<td align="right" valign="middle">
<?php echo '<a href="' . tep_href_link(FILENAME_SHOPPING_CART, '', 'SSL') . '">'
. tep_image(DIR_WS_IMAGES . 'mini-cart.png', HEADER_TITLE_CART_CONTENTS) . '</a>'; ?></td>
<td align="right" valign="middle" class="header" width="50px">
<?php echo '<a href="' . tep_href_link(FILENAME_SHOPPING_CART, '', 'SSL')
. '" class="header" title="' . HEADER_TITLE_CART_CONTENTS . '">'
. $cart->count_contents() . '&nbsp;art&iacute;culo' . ($cart->count_contents()!=1?'s':'')
. '</a>'; ?></td>
```

La imagen ```mini-cart.png``` debe estar ubicada en el directorio común donde se encuentran todas las imágenes, para que el _path_ retornado por la función ```tep_image``` sea correcto.

El número de artículos actuales en la cesta está siempre disponible a través del método ```$cart->count_contents()```, por lo que lo único que hay que hacer es imprimirlo. Para que quede un poco mejor, he añadido la palabra ```artículo``` o ```artículos``` (plural) en función del número de productos en la cesta. Lógicamente para una tienda en la que se prevea el uso de varios idiomas habría que poner estos textos en los ficheros correspondientes dentro de ```catalog/includes/languages```.

## 23.5. Correo con aviso legal

Añadir un aviso legal a todos los correos se hace normalmente configurando el servidor de correos para que añada ese texto a todos los correos salientes, pero también se puede hacer de forma más sencilla tocando directamente la función de osCommerce que se encarga de enviar correos. Más concretamente, la función ```tep_mail```, que se encuentra en el fichero ```includes\functions\general.php```:

```php
<?php
function tep_mail($to_name, $to_email_address, $email_subject, $email_text, $from_email_name, $from_email_address) {
    if (SEND_EMAILS != 'true') return false;

    $email_text .= "\n\n";
    $email_text .= "Cuide el medio ambiente, no imprima este correo si no es necesario.\n";
    $email_text .= "\n";
    $email_text .= "************************* AVISO LEGAL *************************\n";
    $email_text .= "Este mensaje y sus documentos anexos son privados y confidenciales y estan\n";
    $email_text .= "dirigidos exclusivamente a sus destinatarios. Si por error, ha recibido este mensaje\n";
    $email_text .= "y no se encuentra entre los destinatarios, por favor, no use, informe, distribuya,\n";
    $email_text .= "imprima o copie su contenido por ningun medio. Le rogamos lo comunique al remitente\n";
    $email_text .= "y borre todo su contenido.\n";
    $email_text .= "La empresa no asume ningun tipo de responsabilidad legal por el contenido de este\n";
    $email_text .= "mensaje. Cualquier opinion manifestada en el pertenece solo al autor y no representa\n";
    $email_text .= "necesariamente la opinion de la compañía salvo que expresamente se especifique lo contrario.";

    // Instantiate a new mail object
    $message = new email(array('X-Mailer: osCommerce Mailer'));
...
?>
```
