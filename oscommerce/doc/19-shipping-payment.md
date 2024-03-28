# 19. osCommerce: Módulos de Envío, Pago y Totalización

_03-04-2007_ _Juan Mellado_

Los módulos de osCommerce permiten definir los métodos de pago ofrecidos por la tienda (contra-reembolso, transferencia, cheque bancario, tarjeta de crédito, PayPal, etc), los gastos en función de los métodos de envío (tarifa única, por artículo, tabla de tarifas, etc), y el orden en que se deben totalizar los importes para obtener el total definitivo a pagar por el cliente.

Los módulos que trae por defecto osCommerce deberían bastar para cubrir con creces las necesidades básicas de cualquier tienda, no obstante, si es preciso, es posible añadir nuevos métodos de pago, o formas de envío, a través de "contribuciones" externas realizadas por desarrolladores independientes.

## 19.1. Envío

Los módulos de envío definen la forma en que se deben calcular los gastos de envío de los pedidos realizados por los clientes, siendo posible tener activos varios de ellos a un mismo tiempo. Por defecto osCommerce viene con cinco módulos de este tipo, los cuales se pueden instalar y activar según nuestra conveniencia, aunque es importante entender en este punto que, si bien los módulos se pueden "instalar" en cualquier momento, no surtirán ningún efecto en el proceso de compra hasta que se "activen".

- **Tarifa Única**: Este es el único módulo que viene por defecto instalado y activo con la instalación básica de osCommerce. Su objetivo es el de aplicar un incremento fijo en concepto de gastos de envío al precio final de todos y cada uno de los pedidos, independientemente de la cantidad de artículos o peso de los mismos. Es decir, si se fija un recargo ```R```, entonces todo pedido verá incrementado su precio en una cantidad ```R```.

- **Por Artículo**: Este módulo aplica un recargo fijo en concepto de gastos de envío en función del número de artículos de cada pedido. Es decir, si se fija un recargo ```R```, entonces un pedido de ```N``` artículos verá incrementado su precio en una cantidad igual a ```R*N```.

- **Tabla de Tarifas**: Este módulo permite definir una tabla de tarifas ajustada a nuestras necesidades particulares. La forma en que se define la tabla es escribiendo una secuencia de pares de valores separados por una coma, y con cada par separado a su vez por un punto y coma. Por ejemplo, la secuencia ```25:8.50,50:5.50,10000:0.00``` indica que hasta ```25``` se aplique un recargo de ```8.5```, que a partir de ```25```, y hasta ```50```, se aplique ```5.5```, y que a partir de ```10000``` se aplique ```0```. Una característica importante de este módulo es que permite que el primer valor de cada par pueda referirse al peso o al precio final del pedido.

- **United States Postal Service**: Con este módulo se permite que los clientes puedan elegir USPS (_United States Postal Service_) para realizar el envío de sus pedidos. Para poder utilizarlo se debe ser un cliente registrado de USPS y asegurarse que el peso de los productos se indique siempre en libras.

- **Tarifa por Zona**: Este módulo es muy similar al de la tabla de tarifas visto anteriormente. La única diferencia es que permite indicar que el recargo por peso se aplique sólo a determinadas zonas fiscales.

Cada módulo tiene una serie de opciones propias que hay configurar en detalle según las necesidades particulares de cada método de envío, referidas la mayoría de ellas a la aplicación del método en función de la zona fiscal del destinatario de los envíos. Adicionalmente todos los módulos tienen un atributo que permite definir en qué orden se deben de ofrecer las distintas formas de envío al usuario cuando realice un pedido.

## 19.2. Pago

Los métodos de pago definen la forma en las que los clientes podrán realizar el pago de los pedidos que hagan. Por defecto osCommerce trae diez módulos instalados con diversas formas de pago, incluyendo bastantes plataformas de pago electrónicas de las que sinceramente no había oído nunca hablar.

- **Cheque/Transferencia Bancaria**: Este módulo viene instalado y activo por defecto. Su forma de funcionamiento es bien conocida, consiste en avisar a los clientes de que sus pedidos no les serán enviados hasta que no abonen por adelantado los importes correspondientes en la cuenta habilitada a tal efecto por la tienda. Con este módulo la responsabilidad del proceso recae en el administrador de la tienda que ha de comprobar periódicamente los ingresos en cuenta y gestionar el estado de los envíos.

- **Contra Reembolso**: Este módulo también viene instalado y activo por defecto. Como es bien sabido, con esta modalidad de pago los clientes deben abonar los importes en destino, cuando reciben sus pedidos.

- **Tarjeta de Crédito**: Este es el tercer y último módulo de pago que se encuentra instalado y activo por defecto. Con este método de pago los importes de los pedidos se ingresan en la cuenta habilitada a tal efecto por la tienda mediante el abono por adelantado por parte del cliente a través de una tarjeta de crédito. Sin embargo, el hecho de tener instalado este módulo no implica que se puedan realizar pagos con tarjeta de crédito en la tienda. Por desgracia, no es tan fácil y sencillo. Si queremos montar una tienda y queremos que se pueda comprar con tarjeta de crédito directamente en ella debemos dirigirnos a un banco y contratar con ellos un servicio de TPV (_Terminal de Punto de Venta_) virtual. Una vez contratado el TPV virtual tendremos que configurar nuestro servidor _web_ y osCommerce para que operen a través de él. La instalación y configuración dependerán del banco con que se opere y la pasarela de pago concreta utilizada en cada caso.

- **PayPal**: Este módulo permite realizar pagos a través de PayPal, una de las _webs_ más populares hoy en día para la realización de pagos electrónicos: [https://www.paypal.com](https://www.paypal.com).

- **Authorize.net**: Módulo de plataforma de pago electrónico: [https://www.authorize.net](https://www.authorize.net).

- ~~**iPayment**: Módulo de plataforma de pago electrónico.~~

- **NOCHEX**: Módulo de plataforma de pago electrónico: [https://www.nochex.com](https://www.nochex.com).

- **2CheckOut**: Módulo de plataforma de pago electrónico: [https://2checkout.com](https://2checkout.com).

- **PSiGate**: Módulo de plataforma de pago electrónico: [https://psigate.com](https://psigate.com).

- ~~**SECPay**: Módulo de plataforma de pago electrónico.~~

Al igual que ocurría con los módulos de envío, la mayoría de los módulos de pago requieren que se introduzcan parámetros de configuración propios, como usuario y clave, por ejemplo. Adicionalmente todos los módulos tienen un atributo que permite definir en qué orden se deben de ofrecer los distintos tipos de pago disponibles al usuario cuando realice un pedido.

## Totalización

Los módulos de totalización establecen los totales y subtotales que los clientes ven cuando realizan un pedido, permitiendo realizar un mayor control sobre el importe total final de algunos tipos de conceptos. A través de estas opciones de configuración se pueden instalar, configurar, y activar los cinco módulos que trae osCommerce por defecto.

- **Cargo por Pedido Mínimo**: Este módulo permite añadir un recargo fijo a los pedidos si el importe total de los mismos no supera un mínimo.

- **Gastos de Envío**: Con este módulo se permite realizar un control final sobre el total a pagar por el concepto de gastos de envío, pudiendo especificar si no se debe cobrar por gastos de envíos si el importe total del pedido no supera un mínimo.

- **Subtotal**: Este módulo permite indicar si se deben mostrar los subtotales.

- **Impuestos**: Este módulo permite indicar si se deben mostrar los impuestos.

- **Total**: Este módulo permite indicar si se deben mostrar los totales.

Algunos módulos tienen atributos propios que deben establecerse con el valor correcto para cada caso concreto, aunque la mayoría son referidos a zonas fiscales. Adicionalmente, todos los módulos tienen un atributo que permite definir en qué orden se deben de mostrar los totales al usuario cuando realice un pedido.
