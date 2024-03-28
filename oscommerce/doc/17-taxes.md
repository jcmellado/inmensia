# 17. osCommerce: Zonas / Impuestos

_27-03-2007_ _Juan Mellado_

osCommerce permite definir todos los tipos de impuestos que sean necesarios para cada zona fiscal en la que vaya a operar la tienda. Es en este apartado de la configuración donde se declaran dichas zonas y sus impuestos asociados. Adicionalmente permite realizar el mantenimiento de los países y estados (provincias) almacenados en base de datos.

Si no quiere complicarse en exceso y su tienda sólo va a operar dentro de una zona fiscal concreta, lo mejor es obviar todo el proceso de alta de zonas fiscales e impuestos incluyendo las tasas correspondientes dentro del precio del producto (aquello de "IVA Incluido").

## 17.1. Países

osCommerce tiene por defecto dados de alta 239 países, por lo que es raro que vayamos a necesitar nunca añadir uno nuevo. Por cada país se guarda su nombre, dos códigos ISO internacionales con las abreviaturas oficiales, y el formato de dirección postal que se utiliza normalmente en dicho país.

Técnicamente, el formato de dirección postal hace referencia a cómo se escriben habitualmente las direcciones cuando se franquea algún tipo de pedido para enviarlo. Hay cinco tipos de formatos que se encuentran en la tabla ```address_format``` de la base de datos. Cada formato está almacenado como una cadena de texto formada por la concatenación de los nombres de las variables que deben aparecer en ellas y en su orden preciso. Por ejemplo, el tipo ```3```, que es el utilizado normalmente en España, se encuentra definido con la siguiente cadena:

```text
$firstname $lastname$cr$streets$cr$city$cr$postcode - $state,$country
```

Cada variable empieza por el signo del dólar, y son fáciles de identificar por su nombre en inglés, aunque la variable ```$cr``` tiene un significado especial, significa que se debe saltar una línea. Con ese formato del tipo ```3``` las direcciones tendrán el siguiente aspecto:

```text
Nombre Apellidos
Calle
Ciudad
Código Postal - Provincia, País
```

## 17.2. Provincias

Además de los países, osCommerce tiene por defecto registradas 181 zonas, estados o provincias. Por cada provincia se guarda el país al que pertenece, su nombre, y código o abreviatura oficial. En la práctica va a ser bastante raro que se necesite insertar una nueva provincia, aunque siempre se puede recorrer la lista y sustituir los códigos que trae por defecto por otros, como por ejemplo, por los dos primeros números del código postal de cada provincia.

## 17.3. Zonas Fiscales

Las zonas fiscales delimitan área concretas en las que aplican determinados tipos de impuestos. Por defecto viene sólo una dada de alta a modo de ejemplo, que además no resulta de mucha utilidad y que por tanto se puede borrar sin más.

Para dar de alta una zona fiscal hay que introducir un nombre y una descripción. Lo que nos permite por ejemplo crear la zona "_Unión Europea_". Una vez creada una zona podemos añadirle sub-zonas pinchando sobre el icono en forma de carpeta que aparece junto al nombre. Por cada sub-zona hay que seleccionar primero un país, y luego una zona dentro del mismo (o dejar la opción "_Todas las zonas por defecto_"). Lo que nos permite por ejemplo añadir "_España_", "_Portugal_", "_Francia_", ... a nuestra zona "_Unión Europea_".

## 17.4. Tipos de Impuestos

osCommerce trae como ejemplo un tipo de impuesto llamado "_Taxable Goods_", que es un tipo genérico de impuesto que se aplica normalmente a todos los productos, excluyendo a algunos pocos, como los alimentos o servicios, por ejemplo. Lo que es importante de entender es que declarar un tipo de impuesto no hace que se apliquen, ni de forma inmediata, ni de forma correcta. En este punto, un tipo de impuesto para osCommerce es sólo un nombre y una descripción, nada más.

## 17.5. Impuestos

En este apartado de la configuración es donde se asocian todas las entidades que se han dado de alta en las secciones anteriores y se asigna el porcentaje que corresponde a cada impuesto.

Cada impuesto consta de un tipo (seleccionable entre los tipos de impuestos dados de alta anteriormente), una zona fiscal de aplicación (seleccionable entre las zonas fiscales dadas de altas anteriormente), un valor porcentual en tanto por cien a aplicar sobre el precio, una descripción, y una prioridad.

La prioridad es el orden en que se debe aplicar el impuesto para el caso en que se apliquen varios impuestos sobre un mismo producto. Los impuestos con una misma prioridad se aplican a un mismo tiempo sumando sus porcentajes. Por ejemplo, si se tienen tres impuestos, dos de prioridad ```1```, y otro de prioridad ```2```, entonces se sumarán por una parte todos los porcentajes, y por otra parte se sumarán los porcentajes de los impuestos de prioridad ```1``` y se multiplicarán por el impuesto de prioridad ```2```, la suma de esas dos cantidades será la que se aplique al precio de los productos para obtener el resultado final.

Internamente osCommerce almacena los precios de cada producto con una precisión de 4 decimales y sin ningún tipo de impuesto aplicado. Cuando un cliente sin registrar navega por la tienda se muestran los impuestos del país y zona en que se encuentra la tienda declarada según los parámetros correspondientes establecidos en la configuración general. Cuando el cliente se registra entonces se muestran según el país y zona con los que se ha dado de alta. No obstante, cuando realiza una compra, en el momento de confirmar la dirección a la que debe ser enviado su pedido, se recalculan los tipos de impuestos que aplican según el país y zona escogida como destino del envío.
