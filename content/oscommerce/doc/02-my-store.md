# 2. osCommerce: Configuración - My Store

_20-03-2007_ _Juan Mellado_

Este conjunto de parámetros establecen valores de configuración globales de carácter general que se utilizarán a lo largo y ancho de toda la tienda virtual, es decir, son un pequeño "cajón de sastre".

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Store Name (osCommerce)

Este parámetro almacena el nombre de la tienda. Se utiliza principalmente como título de todas las páginas de la _web_. También se muestra como parte del mensaje de _copyright_ que aparece por defecto en el pie de página. Es importante, y fácil, establecer el valor correcto. Dejar el valor por defecto hace que la _web_ resulte muy poco "profesional".

## Store Owner (Harald Ponce de Leon)

Nombre del propietario de la tienda. Lo más normal es poner el nombre de la empresa propietaria de la tienda, no de una persona física. El valor que se establezca en este parámetro será el que se envíe dentro del campo "de parte de" (_from_) en los correos enviados por la tienda, por ejemplo cuando un cliente realice una compra.

## E-mail Address (root@localhost)

Dirección de correo principal de la tienda. Lo habitual cuando se monta una _web_ es tener una serie de cuentas asociadas a dicha _web_ (```contacto@tienda.com```, ```soporte@tienda.com```, ```pedidos@tienda.com```, etc). En este parámetro se debe establecer la cuenta de correo principal a través de la que se gestionará la tienda. Por ejemplo, será la cuenta en la que se reciban los correos avisando de la realización de nuevos pedidos. Hoy en día una de las formas más populares es la de ```info@tienda.com```, aunque los expertos recomiendan usar direcciones distintas para asuntos distintos para personalizar el trato con los clientes.

## E-mail From (osCommerce <root@localhost>)

Cuenta de correo que se utilizará cuando se envíe un correo desde la tienda. Como se observa, se indican dos valores en este sólo parámetro. El primero es el nombre que aparecerá como origen del correo (```osCommerce```), y el segundo la cuenta de correo a utilizar (```<root@localhost>```). El nombre y la cuenta deben separarse por un espacio en blanco, y la cuenta escribirse encerrada entre ```<``` y ```>```. Por ejemplo: ```Tienda <info@tienda.com>```.

## Country (United States)

El país donde se encuentra la tienda. Tratándose de una tienda virtual, ha de tenerse en cuenta que en realidad nuestro lugar de residencia no tiene porque coincidir con el lugar físico en el que se encuentren los servidores en los que se aloja la tienda. De hecho, lo más normal será seleccionar para este parámetro el país donde desarrollamos nuestra actividad principal. Es importante establecer el valor correcto, ya que algunas opciones, como la de gestión de impuestos por ejemplo, toman este parámetro como referencia para los visitantes no registrados que navegan por la tienda.

**Nota:** Si se cambia este parámetro entonces también habrá que cambiar el siguiente.

## Zone (Florida)

Zona, estado, o ciudad, en la que se encuentra la tienda dentro del país actual seleccionado. Lo dicho para el parámetro anterior aplica también a este.

## Expected Sort Order (desc)

Este parámetro indica en que orden se deben mostrar los productos dentro del bloque de próximos productos. El orden más habitual es mostrar las novedades primero, de ahí que la opción por defecto sea ```desc```, es decir, ordenados por fecha de forma descendente. Primero los más modernos, y luego los más antiguos. Se puede cambiar a valor ```asc``` para que aparezcan en orden inverso, por ejemplo para promocionar un producto "clásico" y bien conocido, en perjuicio de otro más "moderno" pero con menor aceptación.

**Nota:** El parámetro siguiente permite indicar que el orden no se realice por fecha.

## Expected Sort Field (date_expected)

Este parámetro complementa al anterior y permite indicar cómo se quieren ordenar realmente los productos. Permite seleccionar entre ```date_expected``` o ```product_name```. Es decir, entre fecha o nombre de producto respectivamente. Normalmente bastará con dejar este parámetro y el anterior con sus valores por defecto.

## Switch To Default Language Currency (false)

Este parámetro indica si se quiere que la tienda muestre automáticamente los precios de los productos en la moneda asociada al idioma seleccionado por el visitante de la _web_. El valor por defecto es ```false```, lo que quiere decir que los precios siempre se mostrarán en la moneda con la que se definieron, sin hacer ningún tipo de conversión. Yo lo veo como el valor más lógico, si se cambia el valor y se establece a ```true``` entonces habremos de asegurarnos de tener siempre el valor del tipo de cambio entre monedas siempre debidamente actualizado o podemos llevarnos desagradables sorpresas.

## Send Extra Order Emails To ()

Lista de cuentas a las que enviar un correo cuando se produzca un pedido. Este parámetro es bastante útil, ya que permite enviar las notificaciones de la entrada de nuevos pedidos a cualquier cuenta que queramos, sin tener que restringirnos a la cuenta principal de la tienda. Para indicar la lista de cuentas hay que escribirlas una detrás de otra separadas por coma. Por ejemplo: ```Pedidos <pedidos@tienda.com>, Luis <luis@gmail.com>```.

## Use Search-Engine Safe URLs (still in development) (false)

De forma general, a los buscadores, como el popular Google, no le gustan que las páginas de nuestras _webs_ tengan direcciones (URLs) complicadas. Por ejemplo, una página con el detalle de un producto de nuestra tienda que tuviera por nombre ```http://www.tienda.com/product_info.php?products_id=25&osCsid=11b65cdca3e06d3e375739eb7f12e588``` resultaría muy poco atractiva para los buscadores, y de hecho es bastante probable que la ignorase con el consecuente perjuicio para nosotros. El propósito de este parámetro es permitir que las URLs de nuestra tienda tengan un aspecto más atractivo. Desgraciadamente esta es una opción que está aún en desarrollo y no se encuentra completamente implementada, o por lo menos, con todas las características que personalmente creo que debería tener. La única solución válida hoy en día parece ser utilizar una "contribución" externa, como por ejemplo _Ultimate SEO URLs_.

**Nota:** Las "contribuciones" son modificaciones que se realizan sobre el _software_ básico de osCommerce para cambiar o añadir funcionalidad a osCommerce.

## Display Cart After Adding Product (true)

Este parámetro indica si se quiere que después de que un visitante añada un producto a su carrito de la compra se le muestre el contenido actual del carro (```true```) o bien se quede en la propia página en la que se encuentra el producto (```false```). Particularmente me inclino por dejar la opción por defecto, me gusta que se vea que se ha realizado la acción correctamente. Si la tienda ofreciera productos que por su naturaleza hicieran que fuera habitual comprar varios de ellos juntos entonces puede que interesase dejar al visitante en la página de productos. Es cuestión de probar lo que mejor se adapta a cada tienda en particular.

## Allow Guest To Tell A Friend (false)

Este parámetro permite que los visitantes anónimos, no registrados como clientes en la tienda, puedan enviar un correo con un enlace a la página de un producto de la tienda. Si tiene valor ```false``` no se permite, si tiene valor ```true``` si se permite. Es cuestión de gustos, a mi nunca me ha hecho gracia ese tipo de opciones, pero puede funcionar bien para determinadas tiendas dependiendo del tipo de público al que estén dirigidas.

## Default Search Operator (and)

osCommerce proporciona un cuadro que permite realizar búsquedas sobre la tienda a los visitantes. El procedimiento es sencillo, se introducen las palabras que se quieren buscar y osCommerce trata de localizarlas. Este parámetro controla si se quiere que la búsqueda muestre resultados que contengan todas las palabras introducidas, o al menos alguna de ellas. Por defecto tiene el valor ```and```, que fuerza a que los resultados tengan todas las palabras, el otro posible valor es ```or```, que muestra resultados en los que aparece al menos una de ellas. Los visitantes tienen la opción de cambiar esta opción por defecto utilizando el cuadro de búsqueda avanzada. Particularmente prefiero la opción ```or```, porque suele mostrar más resultados y puede generar alguna venta al mostrarse algún producto en el que el visitante no podía haberse fijado en un principio. Depende del tamaño del catálogo y del número de productos que tengamos, si se tiene un catálogo muy amplio puede ser frustrante que aparezca una lista muy grande de resultados, devalúa la opción de búsqueda.

## Store Address and Phone (Store Name Address Country Phone)

En este parámetro se almacenan los datos de ubicación y contacto reales de la tienda. Es la información que normalmente se imprime en cualquier tipo de documento emitido por una tienda. Incluye el nombre de la tienda, dirección, ubicación (país), y número de teléfono. Un dato por línea. Es un parámetro un tanto libre y debería rellenarse con información real. Es muy importante que sea totalmente correcta, porque es la información que se muestra a los clientes que deciden realizar pago en metálico o mediante un cheque.

## Show Category Counts (true)

Este parámetro indica si se debe mostrar el número de productos que existe para cada categoría. Si se deja el valor ```true``` por defecto se muestra, y si se cambia a ```false``` no se muestra. Hay personas que adoran esta opción y otros que la odian. El motivo de esta división de opiniones es que las tiendas con un catálogo pequeño no gustan de ver aparecer un solitario (1) junto a una categoría de productos, mientras que a los poseedores de un catálogo enorme les encanta que les recuerden que su tienda ofrece (73.594) productos distintos.

## Tax Decimal Places (0)

Número de decimales con los que debe mostrarse el valor de los impuestos aplicados al precio final de un producto. Por defecto es ```0```, pero hay que ajustarlo en función de las necesidades concretas de la moneda con la que trabaje la tienda. Para operar con euros habría que establecer ```2``` decimales, aunque si no quiere complicarse la vida en exceso lo mejor es que muestre los precios de los productos con los impuestos ya incluidos (lo típico de: "IVA incluido").

**Nota:** El siguiente parámetro permite indicar si los precios incluyen impuestos o no.

## Display Prices with Tax (false)

Este parámetro complementa en cierto sentido al anterior, permite indicar si queremos que los precios se muestren con los impuestos incluidos o no. Por defecto es ```false``` para que no incluyan los impuestos, pero se puede cambiar a ```true``` para que sí se incluyan.
