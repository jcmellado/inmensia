# 5. osCommerce: Configuración - Images

_23-03-2007_ _Juan Mellado_

Estos parámetros establecen los valores de configuración que fijan el tratamiento que realizará osCommerce de las imágenes que se muestran en la tienda.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso, y consideraciones a tener en cuenta para su modificación.

## Small Image Width (100)

Anchura en pixels de las imágenes de los productos cuando se muestran a tamaño reducido (_thumbnails_). Este valor, junto al del siguiente parámetro, establecen el tamaño de las imágenes de los productos cuando se muestran en los listados, o en los bloques de novedades u ofertas por ejemplo.

## Small Image Height (80)

Altura en pixels de las imágenes de los productos cuando se muestran a tamaño reducido (_thumbnails_). Usado en conjunción con el anterior parámetro.

## Heading Image Width (57)

Anchura en pixels de las imágenes que se muestran en los encabezados de los listados, junto al título. Este valor, junto al del siguiente parámetro, establecen el tamaño de las imágenes que se muestran en la esquina superior derecha de los listados, como el de categorías, novedades, u ofertas por ejemplo.

## Heading Image Height (40)

Altura en pixels de las imágenes que se muestran en los encabezados de los listados, junto al título. Usado en conjunción con el anterior parámetro.

## Subcategory Image Width (100)

Anchura en pixels de las imágenes de las sub-categorías. Este valor, junto al del siguiente parámetro, establecen el tamaño de las imágenes que se muestran en el listado de categorías para acceder a una sub-categoría.

## Subcategory Image Height (57)

Altura en pixels de las imágenes de las sub-categorías. Usado en conjunción con el anterior parámetro.

## Calculate Image Size (true)

Este parámetro indica si realmente osCommerce debe ajustar el tamaño de las imágenes según los valores establecidos en los parámetros anteriores. El valor por defecto ```true``` hace que efectivamente las imágenes se muestren con los tamaños previamente configurados, cambiando el valor a ```false``` las imágenes se mostrarán con su tamaño original. En buena lógica es mejor dejar el valor por defecto, y que todas las imágenes se muestren con un mismo tamaño a lo largo y ancho de toda la tienda.

## Image Required (true)

Este parámetro establece que debe hacer osCommerce cuando una imagen no se encuentra. El valor por defecto ```true``` indica que se intenten mostrar todas las imágenes sin comprobar su existencia o no, cambiándolo a ```false``` se indica que no se muestren las imágenes que no existan.
