# 18. osCommerce: Catálogo

_30-03-2007_ _Juan Mellado_

El catálogo de osCommerce contiene el detalle de los productos que se venden en la tienda, siendo quizás una parte muy importante de este paquete de _software_ la facilidad que ofrece para definir los atributos adicionales que se quieran para los productos. Es decir, osCommerce no ofrece una plantilla de atributos estáticos a rellenar para cada producto, sino que permite que seamos nosotros mismos los que definamos nuestros propios atributos de forma personalizada acorde al tipo de artículos que vendamos.

## 18.1. Fabricantes

La opción de mantenimiento de fabricantes permite crear y editar la lista de fabricantes de los productos ofrecidos en la tienda. Aunque al realizar el alta de un nuevo producto no es obligatorio indicar el fabricante del mismo.

A cada fabricante puede asociársele un nombre, una imagen identificativa (normalmente su logo corporativo), y un enlace a su página _web_. Los enlaces a las páginas _web_ de los fabricantes pueden introducirse de forma individual por cada idioma instalado. En función del idioma actual seleccionado por el cliente osCommerce utilizará automáticamente un enlace u otro.

## 18.2. Categorías/Productos

Esta opción es a través de la que realmente se introducen los productos que se venderán en la tienda, y en la que se clasifican en categorías y sub-categorías.

La idea que hay que tener en mente es que osCommerce permite construir un árbol de categorías personalizado para nuestra tienda, facilitando a los clientes la tarea de encontrar lo que andan buscando navegando por la _web_ a través de las distintas ramas de este árbol. Las categorías que vienen de ejemplo por defecto deberían darnos una buena idea de lo que se puede conseguir.

Para dar de alta una categoría hay que pulsar el botón de "_nueva categoría_" e introducir su nombre (por cada idioma disponible), una imagen, y el orden en que tiene que aparecer dentro del listado de categorías. Una vez dada de alta una categoría podemos entrar en ella pulsando sobre el icono con forma de carpeta que aparece a la izquierda del nombre de la misma. Dentro de una categoría puede crearse a su vez una o más categorías, y así sucesivamente.

Para dar de alta un producto hay que pulsar el botón de "_nuevo producto_" e introducir su disponibilidad, fecha de lanzamiento, fabricante, nombre (por idioma), tipo de impuesto a aplicar, precio neto, precio bruto, descripción (por idioma), cantidad disponible en _stock_, modelo, imagen, _link_ a la _web_ del fabricante (por idioma), y peso. La mayoría de campos son opcionales, por lo que el proceso de alta se puede limitar a introducir valores para los campos más básicos con los que se identifican normalmente los productos.

Los campos de precios tienen un comportamiento especial. Así, cuando se introduce un valor para el precio neto se recalcula automáticamente el precio bruto, y viceversa. De igual forma, cuando se elige el tipo de impuesto a aplicar, se recalcula el precio bruto.

El campo descripción además de texto plano admite etiquetas HTML que se llevan tal cual a las páginas donde se muestran los productos. De esta forma, para indicar un salto de línea se puede utilizar la etiqueta ````<br>````, y para destacar un texto en negrita se debe colocar entre un par de etiquetas ````<strong></strong>````.

Por último, comentar que si creamos una sub-categoría o un producto dentro de una categoría equivocada tenemos la opción de traspasarlos a su lugar correcto con el botón "_mover_" que aparece junto a los de "_editar_" y "_eliminar_". Para los productos, tenemos además la opción de copiarlos o duplicarlos en la misma categoría, u otra categoría, para facilitar la creación de nuevos productos similares a los ya existentes.

## 18.3. Atributos

La opción de mantenimiento de atributos resulta un tanto caótica la primera vez que se accede a ella en comparación con el resto de opciones de configuración. La dificultad radica en que realmente se están viendo tres mantenimientos en una sola página: opciones de productos, valores de opciones, y atributos de productos. Afortunadamente osCommerce trae por defecto dados de alta varios atributos que nos pueden servir para orientarnos.

En "_opciones de productos_" se definen las distintas características adicionales que se quiere que tengan los productos, como por ejemplo "_Color_", "_Talla_", o "_Modelo_". En "_valores de opciones_" se definen los posibles valores que pueden tener las distintas características, o dicho de otro modo, los valores que aparecerán en los desplegables de selección cuando se elija una característica concreta. Por ejemplo, para "_Color_", valores como "_Azul_", "_Verde_", o "_Rojo_". Y por último, en "_atributos de productos_" se asocia a cada producto las características adicionales que se quieran, junto un valor concreto para cada característica, y la variación del precio del producto cuando se seleccione dicha característica.

Por ejemplo, imaginemos que montamos una tienda para vender bicicletas. Cuando un cliente realice un pedido de una bicicleta ha de poder seleccionar el color de la misma sin que ello suponga una variación en el precio final. En "_opciones de productos_" creariamos la característica "_Color_". En "_valores de opciones_" creariamos los colores "_Azul_", "_Verde_", y "_Rojo_". Y en "_atributos de productos_" seleccionariamos el producto bicicleta y le asociaramos tres veces la característica "_Color_" con cada uno de los colores, pero sin indicar en ningún caso incremento de precio alguno (o estableciendo un valor de cero).

Imaginemos ahora que queremos dar la posibilidad al cliente de elegir entre montar un sistema de cambio manual o automático con el consiguiente incremento de precio si se elige la segunda opción. En "_opciones de productos_" creariamos la característica "_Cambio_". En "_valores de opciones_" creariamos "_Manual_" y "_Automático_". Y en "_atributos de productos_" seleccionariamos el producto bicicleta y le asociariamos dos veces la característica "_Cambio_" con cada uno de los sistemas disponibles. Para el cambio manual no se indicaría precio, pero para el cambio automático se indicaría el correspondiente incremento de precio ("_+29.99_" por ejemplo).

En este punto es interesante comentar que sería interesante que osCommerce permitiera asociar atributos a categorías o sub-categorías enteras en vez de tener que hacerlo por cada producto en particular.

## 18.4. Ofertas

Las opciones de este apartado permiten indicar que determinados productos se encuentran en oferta y tienen un precio más reducido que el habitual. Los productos en oferta se muestran en los bloques de ofertas de la _web_ durante la navegación ordinaria por las páginas de la tienda. El precio original aparece tachado y junto a él se muestra en color rojo el precio rebajado.

Para dar de alta una oferta se tiene que elegir un producto de la tienda y asignarle el precio rebajado. Tal y como se indica en el propio formulario de alta, el precio rebajado puede introducirse indicando directamente el nuevo precio del producto, o mediante un porcentaje a aplicar sobre el precio original.

Adicionalmente puede introducirse una fecha de caducidad de la oferta, pero se echa en falta una fecha de inicio de la oferta.

## 18.5. Próximamente

Esta opción muestra un listado con los productos que están dados de alta en el catálogo pero que aún no están disponibles para el público. La lista se confecciona tomando como referencia la fecha de lanzamiento de los productos, de forma que sólo aparecen en ella los productos cuya fecha de lanzamiento es posterior al día en curso.

## 18.6. Comentarios

En este apartado se muestra un listado con todos los comentarios introducidos por los clientes acerca de los productos de la tienda, permitiendo editarlos o borrarlos.
