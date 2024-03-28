# 20. osCommerce: Clientes, Pedidos e Informes

_04-04-2007_ _Juan Mellado_

Llegados a este punto en el proceso de instalación y configuración de osCommerce, ya es posible que los clientes se registren en la _web_ y empiecen a realizar pedidos. En las secciones "_Clientes_" e "_Informes_" de osCommerce se pueden gestionar los clientes registrados y los pedidos realizados, así como consultar distintos informes que proporcionan detalles del rendimiento de la _web_ desde el punto de vista de los productos expuestos en ella.

Probablemente estas sean las secciones de la _web_ que más visiten los propietarios de las tiendas, ya que, aparte del catálogo de productos, las secciones vistas hasta son en su mayoría de carácter técnico, además de que una vez establecidos los valores correctos en dichas secciones difícilmente se volverán a cambiar.

## 20.1. Clientes

El listado de clientes proporciona una visión del volumen de usuarios registrados en la _web_, no de "clientes" como indica el nombre del listado, sino de "usuarios", ya que un usuario es simplemente cualquier visitante de la _web_ que se ha registrado en la tienda, sin necesidad por ello de haber realizado alguna vez algún pedido.

Por cada usuario registrado es posible ver y modificar el detalle de los datos que utilizó para registrarse. Recordando que los campos que se solicitan en el formulario de alta de usuarios son configurables y podemos variarlos según nuestras necesidades.

Los propios usuarios por supuesto también pueden acceder a sus datos y modificarlos en todo momento. Es más, cada usuario tiene la opción de añadir nuevas direcciones de entrega o facturación, así como la de consultar el detalle de todos los pedidos que ha realizado a través de la tienda, y decidir si quiere recibir notificaciones por correo electrónico con los boletines informativos que pueden publicarse desde la propia tienda, o cuando se produzca alguna variación en determinados productos que él mismo puede seleccionar.

Otras opciones disponibles en el listado de usuarios, para el administrador o dueño de la tienda, incluyen enviar un correo electrónico a un usuario desde la propia tienda, consultar los pedidos realizados por un usuario, o eliminar un usuario completamente de la base de datos.

## 20.2. Pedidos

En el listado de pedidos se muestran todas las compras realizadas por los clientes a través de la tienda. Por cada línea del listado se puede consultar el detalle de cada pedido y modificar su estado para indicar que se encuentra en proceso, o que ya ha sido entregado según aplique en cada caso concreto.

Al cambiar el estado de un pedido se puede indicar si se quiere notificar al cliente correspondiente dicho cambio de estado mediante un correo electrónico, con la posibilidad además de incluir algún tipo de comentario.

Otras opciones que se permiten en este listado son la impresión de albarán y factura, aunque no es realmente una impresión, sino la generación de páginas _web_ que pueden imprimirse según nuestro gusto y conveniencia.

La única opción que resta por comentar de este apartado es la de borrar pedido, que permite eliminar un pedido de la base de datos y devolver, si así lo decidimos, los artículos contenidos en el pedido al _stock_ de la tienda.

## 20.3. Informes

Este sencillo apartado permite acceder a tres listados:

- **Los Más Vistos**: Muestra la lista de todos los productos ofrecidos en la _web_, junto con un contador de las veces en las que al menos un visitante de la _web_ ha entrado a ver el detalle de cada producto en concreto.

- **Los Más Comprados**: En este listado se pueden ver los productos junto con un contador del número total de unidades vendidas de cada producto en concreto. A diferencia del anterior, en este listado sólo se muestran los productos para los que al menos se haya producido un pedido.

- **Total por Cliente**: Este tercer listado muestra la suma del importe total de todos los pedidos realizados por cada cliente individualmente en la tienda. Sólo se muestran los clientes que al menos hayan realizado alguna vez un pedido.
