# 7. osCommerce: Configuración - Shipping/Packaging

_23-03-2007_ _Juan Mellado_

Los parámetros de configuración de este apartado establecen valores a considerar para el empaquetado y envío físico de los productos, como por ejemplo el peso máximo que la tienda está dispuesta a enviar a un cliente. Estos valores son usados por los módulos de envío instalados en la tienda para calcular los gastos de envío.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso, y consideraciones a tener en cuenta para su modificación.

## Country of Origin (United States)

País origen de los envíos. El país seleccionado se toma como referencia para calcular los gastos de envío aplicables a los pedidos.

**Nota:** Este parámetro, junto al resto de este apartado de configuración, se usan en los módulos de envíos. Estos módulos tienen a su vez otros parámetros de configuración que hay que ajustar adecuadamente para que el cálculo de los gastos de envío se realice correctamente.

## Postal Code (NONE)

Código postal origen de los envíos. El código postal seleccionado se toma como referencia para calcular los gastos de envío aplicables a los pedidos.

## Enter the Maximum Package Weight you will ship (50)

Este parámetro permite especificar el peso máximo por envío que la tienda está dispuesta a mandar a un cliente. Es un peso máximo para todos los envíos, independientemente del método de envío que se escoja.

## Package Tare weight (3)

Peso medio del embalaje. Esta cantidad es añadida al peso de los envíos para calcular su peso total.

**Nota:** Este parámetro se utiliza junto con el siguiente para establecer el peso total de los envíos de gran tamaño.

## Larger packages - percentage increase (10)

Este parámetro establece junto con el anterior la forma en que se calcula el peso total de los envíos de gran tamaño. El valor de este parámetro es un porcentaje, de forma que el valor de ```10``` por defecto indica un 10%. En función de esto, el peso final de un envío se calcula considerando cual de las siguientes dos cantidades es mayor: el peso medio del embalaje, o el 10% del peso total del envío (sin embalaje, claro). Si el 10% es mayor entonces se suma al envío un 10% de su peso, en caso contrario se le suma el peso medio del embalaje. Naturalmente se puede cambiar el porcentaje por defecto y establecer otro valor.
