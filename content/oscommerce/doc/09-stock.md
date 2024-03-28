# 9. osCommerce: Configuración - Stock

_23-03-2007_ _Juan Mellado_

Estos parámetros establecen valores de configuración que afectan al modo en que osCommerce realiza el control de las existencias de los productos (_stock_) en la tienda.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Check stock level (true)

Este parámetro indica si realmente se tiene que comprobar el _stock_ cuando un cliente realice un pedido. El valor por defecto ```true``` indica que efectivamente debe comprobarse, si se cambia a ```false``` entonces osCommerce no realizará ninguna comprobación y se podrán hacer pedidos aunque no existan existencias de los productos. Es claro que esta opción deberá ajustarse en función de la naturaleza de los productos que venda la tienda.

**Nota:** Si se establece el valor de este parámetro a ```false``` entonces las tres variables de configuración siguientes no tendrán efecto.

## Subtract stock (true)

Este parámetro indica si tienen que descontarse los productos del _stock_ cada vez que un cliente realice un pedido. El valor por defecto ```true``` indica que efectivamente deben descontarse, si se cambia a ```false``` entonces no se descontarán.

## Allow Checkout (true)

Este parámetro indica si los clientes pueden realizar un pedido de un producto aunque no existan suficientes existencias del mismo en el _stock_. El valor por defecto ```true``` indica que efectivamente pueden realizarse pedidos aunque no haya existencias en _stock_. Si se cambia a ```false``` entonces no se podrán realizar pedidos de productos agotados en _stock_, al cliente se le mostrará un mensaje informativo cuando esté intentando hacer un pedido de un producto agotado.

## Mark product out of stock (***)

El texto que se ponga en este parámetro será el que se muestre junto a los productos agotados en _stock_. El valor por defecto es de tres asteriscos, aunque siempre se puede cambiar por algo que resulte más descriptivo.

## Stock Re-order level (5)

Este parámetro indica la cantidad mínima que debe existir de los productos en _stock_ para que sea necesario reponerlos. Esta información lógicamente sólo se muestra al propietario de la tienda.
