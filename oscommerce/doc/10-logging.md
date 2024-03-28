# 10. osCommerce: Configuración - Logging

_24-03-2007_ _Juan Mellado_

Con este conjunto de parámetros se controla una opción de osCommerce que permite almacenar en un fichero de texto el tiempo que tarda en construirse cada página que se muestra en la _web_. Proporciona información adicional acerca de las páginas visitadas y el rendimiento del servidor _web_. Este tipo de ficheros se conocen como _logs_, motivo por el que esta sección de configuración se denomina _logging_.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Store Page Parse Time (false)

Este parámetro indica si realmente se tiene que activar la generación del fichero de _log_. Por defecto tiene valor ```false```, lo que hace que no se genere, cambiándolo a ```true``` se generará. Normalmente no se necesitará, a menos que se detecte que la _web_ tiene un rendimiento muy pobre y se quiera hacer un análisis más detallado del comportamiento de la misma.

**Nota:** Si se establece el valor de este parámetro a ```false``` entonces las cuatro variables de configuración siguientes no tendrán efecto.

## Log Destination (/var/log/www/tep/page_parse_time.log)

Ubicación y nombre del fichero de _log_. Este parámetro almacena la ruta y nombre del fichero de texto donde se almacenarán los mensajes con el tiempo de generación de cada página.

## Log Date Format (%d/%m/%Y %H:%M:%S)

Este parámetro indica el formato con el que se almacenarán las fechas dentro del fichero de _log_. Su nomenclatura puede resultar un tanto extraña para los que no posean conocimientos técnicos, por lo que no recomendaría cambiarla a menos que se sepa lo que se está haciendo. Baste decir que con el valor por defecto se obtendrán fechas en formato ```DD/MM/YYYY hh24:mi:ss```, como ```21/03/2007 14:56:20``` por ejemplo.

## Display The Page Parse Time (true)

Con este parámetro se indica si se quiere mostrar en la _web_ de la tienda el tiempo que se tarda en generar cada página. Aunque tenga valor ```true``` por defecto, sólo funcionará si se establece a ```true``` también el valor del primer parámetro de configuración de este apartado. Personalmente nunca he considerado que aporte ningún valor añadido a los clientes.

## Store Database Queries (false)

Este parámetro indica si se quiere guardar las sentencias SQL ejecutadas por osCommerce contra base de datos en el fichero de _log_. Por defecto su valor es ```false```, lo que se hace que no se guarden, cambiándolo a ```true``` hará que se guarden.
