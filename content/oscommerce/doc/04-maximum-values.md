# 4. osCommerce: Configuración - Maximum Values

_22-03-2007_ _Juan Mellado_

El objetivo de estos parámetros es establecer valores de configuración máximos para una serie de opciones, como por ejemplo el número máximo de productos que se deben mostrar en algunos listados que pueden verse en los distintos apartados de la tienda.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Address Book Entries (5)

Número máximo de entradas que un cliente registrado puede tener en su libreta de direcciones. La utilidad de la libreta de direcciones es que un mismo cliente puede enviar o facturar en distintas direcciones.

## Search Results (20)

Número máximo de productos a listar de una sola vez en los listados resultados de búsquedas o filtros. Si el servidor donde está instalado osCommerce no es muy rápido despachando páginas _web_ se puede bajar su valor, a ```10``` por ejemplo.

## Page Links (5)

Este parámetro es un poco complicado de explicar con una sola frase. Hace referencia a cuando se está en algún apartado de la tienda que está mostrando un listado muy grande y que aparece cortado en varias páginas. Normalmente en el pie de página aparecen una serie de enlaces para ir a la página siguiente, a la anterior, o saltar a una página concreta. Este parámetro controla el número máximo de enlaces que deben aparecer para saltar a una página concreta. Si se prevé que los listados de productos van a ser muy grandes tal vez interese aumentarlo, a ```10``` por ejemplo.

## Special Products (9)

Los productos "especiales" son lo que normalmente denominamos "destacados". Este parámetro controla el número máximo de productos a mostrar de una sola vez en el listado de destacados. Tal vez resulte raro que este parámetro tenga por defecto un valor impar, pero la explicación está en que el listado de destacados muestra por defecto ```3``` productos en una misma fila. Un valor de ```9``` en realidad indica que se muestre a lo sumo una matriz de ```3x3``` productos.

**Nota:** Existe otro parámetro que controla el número de productos a mostrar en cada fila del listado.

## New Products Module (9)

Este parámetro controla el número máximo de productos a mostrar de una sola vez en el listado de destacados. Lo dicho para el anterior parámetro acerca de porqué tiene por defecto un valor impar aplica también a este.

## Products Expected (10)

Número máximo de productos a listar de una sola vez en el listado de próximos productos.

## Manufacturers List (0)

Este parámetro controla la forma en que se muestra el listado de fabricantes. Si el número de fabricantes a mostrar supera la cantidad dada en este parámetro entonces se muestra en forma de un desplegable. Un valor de ```0``` hace que siempre se muestre en forma de un desplegable.

**Nota:** Este parámetro debe utilizarse de forma conjunta con el siguiente.

## Manufacturers Select Size (1)

Este parámetro controla, junto con el anterior, el listado de fabricantes. Si este parámetro vale ```1``` entonces el listado se mostrará en forma de un desplegable, en caso contrario se mostrará un listado con un número de filas igual al valor de este parámetro.

## Length of Manufacturers Name (15)

Número máximo de caracteres a mostrar en el nombre de cada fabricante en el listado de fabricantes.

## New Reviews (6)

Número máximo de comentarios a mostrar de una sola vez en el listado de comentarios.

## Selection of Random Reviews (10)

Número máximo de comentarios a considerar entre los que escoger uno para el bloque que muestra un comentario de forma aleatoria.

## Selection of Random New Products (10)

Número máximo de productos a considerar entre los que escoger uno para el bloque que muestra una novedad de forma aleatoria.

## Selection of Products on Special (10)

Número máximo de productos a considerar entre los que escoger uno para el bloque que muestra una oferta de forma aleatoria.

## Categories To List Per Row (3)

Este parámetro determina el número de productos a mostrar por fila en los listados por categoría. En esta clase de listados sólo se muestra la imagen del producto, su nombre y precio, motivo por el que caben varios productos en una sola fila.

**Nota:** Existen otros parámetros que controlan el número total de productos a mostrar de una sola vez en el listado. Si se cambia este parámetro también debería considerarse modificar estos otros.

## New Products Listing (10)

Número máximo de productos a mostrar de una sola vez en el listado de novedades.

## Best Sellers (10)

Número máximo de productos a mostrar de una sola vez en el listado de productos mejor vendidos.

## Also Purchased (6)

Número máximo de productos a mostrar de una sola vez en el listado de otros productos comprados también por un cliente.

## Customer Order History Box (6)

Número máximo de productos a mostrar de una sola vez en el listado de productos comprados por un cliente.

## Order History (10)

Número máximo de pedidos a mostrar de una sola vez en el listado de pedidos.
