# 13. osCommerce: Configuración - Download

_24-03-2007_ _Juan Mellado_

Este conjunto de parámetros controlan las opciones de descarga de productos desde la _web_. Resultan de utilidad cuando los productos vendidos en la tienda se encuentran en algún tipo de soporte informático y pueden descargarse directamente desde la _web_ una vez hecho el pedido.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Enable download (false)

Este parámetro establece de forma global si la descarga de productos está activa o no. El valor por defecto ```false``` hace que esté desactivada y que el resto de parámetros de este apartado no tengan ningún efecto. Para permitir las descargas se debe cambiar a ```true```.

Al activar las descargas se mostrarán tres nuevos campos en el formulario de atributos de productos. En estos nuevos campos se podrá especificar el nombre del fichero a descargar (que deberá estar ubicado en el directorio ```catalog/download```), el número máximo de días durante los cuales el cliente podrá realizar la descarga, y el número máximo de intentos de descarga que podrá realizar.

## Download by redirect (false)

Este parámetro indica si osCommerce debe utilizar el método de redirección para realizar la descarga. Con este método se fuerza al navegador a cambiar de página haciendo que se inicie la descarga. En los servidores corriendo alguna clase de Linux debe dejarse a su valor por defecto ```false``` para que no se utilice este método.

## Expiry delay (days) (7)

Número máximo de días que tiene el cliente para realizar la descarga del producto. Es un valor que se sugiere por defecto en el formulario de alta de productos. En el propio formulario se permite modificarlo para establecerlo de forma individual para cada producto.

## Maximum number of downloads (5)

Número máximo de intentos que tiene el cliente para realizar la descarga del producto. Es un valor que se sugiere por defecto en el formulario de alta de productos. En el propio formulario se permite modificarlo para establecerlo de forma individual para cada producto.
