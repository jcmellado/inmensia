# 21. osCommerce: Herramientas

_05-04-2007_ _Juan Mellado_

El último apartado dentro de la sección de administración de osCommerce está compuesto por una serie de utilidades de muy diversa índole. El objetivo de cada una de estas herramientas es muy distinto entre sí, mezclándose las que tienen una marcada vertiente técnica, como la realización de copias de seguridad o gestión de los archivos del servidor, con las que tienen influencia en el aspecto de la tienda, como la definición de los textos en distintos idiomas o la configuración de _banners_.

## 21.1. Copia de Seguridad

Las opciones de copia de seguridad permiten exportar el contenido actual de la base de datos a un fichero en el servidor _web_, con el objetivo de servir de copia de respaldo en caso de producirse algún tipo de problema que implique restaurar dicha base de datos. Por defecto las copias de seguridad se guardan en el directorio ```catalog/admin/backups```, aunque es aconsejable que además de en este directorio se encuentren físicamente en otro lugar aparte del propio servidor _web_.

Los ficheros generados por osCommerce tienen por defecto el nombre ```db_BASEDATOS-FECHA.sql```, donde ```BASEDATOS``` es el nombre de la base de datos, y ```FECHA``` corresponde con el día y hora en el que se generó la copia de seguridad en formato ```YYYYMMDDHHMISS```.

A través de esta propia sección de administración se listan las copias de seguridad realizadas, y se permite eliminar o restaurar cualquiera de las disponibles, aunque para la opción de restauración es el propio osCommerce el que indica que es aconsejable utilizar directamente el cliente de MySQL en vez de hacerlo a través de la tienda.

## 21.2. Banners

En su aceptación más habitual, un _banner_ es simplemente un anuncio. Es decir, un texto, imagen, o animación promocional. La utilidad principal de los _banners_ dentro de una tienda es normalmente la de destacar algún tipo producto o servicio, anunciar ofertas extraordinarias, precios rebajados, o cualquier tipo de evento que resulte de interés. Son los típicos cuadros de colores chillones que encontramos en las _webs_ intentando llamar desesperadamente nuestra atención.

osCommerce permite programar la aparición de _banners_ dentro de la tienda. Para ello hay que introducir un título, la URL a la que se redirigirá al visitante si pincha en él, un grupo (más sobre esto en el siguiente párrafo), una imagen o texto en formato HTML, una fecha de lanzamiento, y una fecha de fin, o alternativamente, el número máximo de veces que debe mostrarse antes de retirarlo de la _web_.

Los grupos de _banners_ permiten, como su propio nombre indica, agrupar _banners_ con el objetivo de poder mostrar de forma aleatoria varios de ellos en una misma ubicación. Por defecto osCommerce tiene configurado un _banner_ en un grupo llamado ```468x50```. Puede verse como se gestiona su aparición en las páginas _web_ de la tienda en el fichero ```catalog/includes/footer.php```.

En el siguiente código de ejemplo puede verse como primero se solicita de forma dinámica el ID de un _banner_ del grupo ```468x50```, y como después se solicita de forma estática el _banner_ correspondiente a dicho ID:

```php
<?php
  if ($banner = tep_banner_exists('dynamic', '468x50')) {
    echo tep_display_banner('static', $banner);
  }
?>
```

## 21.3. Control de Cache

El concepto de _cache_ y las opciones de configuración disponibles se vieron en un artículo anterior de esta serie. En este apartado simplemente se listan los distintos ficheros de _cache_ generados en el servidor ofreciendo la opción de inicializarlos.

## 21.4. Definir Idiomas

Esta opción permite modificar los textos que aparecen en la _web_ para los diferentes idiomas configurados, aunque desgraciadamente esta modificación se realiza directamente sobre los ficheros PHP que componen el propio _software_ de osCommerce, lo cual es sin duda uno de los puntos más flacos de este programa.

Para cambiar un texto por otro lo único que hay que hacer es localizar el fichero que contiene el texto que se quiere cambiar y editarlo para su modificación. A la hora de modificar un fichero se debe tener en cuenta que la modificación se debe limitar al cambio controlado de una cadena de texto por otra respetando en todo momento la sintaxis del código PHP donde se encuentra embebida. Los caracteres especiales o vocales acentuadas han de introducirse utilizando sus correspondientes códigos en HTML.

Naturalmente antes de realizar cualquier tipo de modificación es recomendable realizar una copia de seguridad de los ficheros que van a modificarse.

## 21.5. Archivos

Esta opción permite la administración remota de los ficheros del servidor _web_. Es como disponer de un cliente FTP basado en _web_ dentro la propia tienda. Puede ser de ayuda para consultar algún directorio o fichero en cualquier momento sin tener que conectarse a través de Telnet o FTP. Si no se tienen conocimientos técnicos es mejor no andar trasteando con esta opción.

## 21.6. Enviar Email

Tal y como su nombre indica, esta sección permite enviar correos, pudiendo seleccionar un usuario concreto o un grupo de ellos. Poco más que decir, se selecciona destinatario, se introduce asunto y cuerpo de mensaje, y se pulsa "_enviar_".

## 21.7. Boletines

El objetivo de los boletines es mantener informados a los clientes de las novedades que se produzcan en la tienda. Son pequeños panfletos propagandísticos que pueden editarse desde la misma _web_.

Se distinguen dos tipos: "_listas de correos_" y "_notificaciones de producto_". Los primeros están orientados a comunicar novedades sobre la tienda en general a los clientes. Los segundos en cambio son notificaciones concretas que aplican sólo a determinados productos. Por ejemplo, dentro del primer tipo tendría cabida un boletín trimestral notificando la inclusión de nuevos productos o servicios dentro de la tienda, y dentro del segundo tipo tendría cabida un boletín puntual avisando de una liquidación o rebaja de una determinada serie de productos concretos por un tiempo limitado.

Para crear un boletín basta con elegir un tipo, y escribir el título y cuerpo con el contenido del mismo. Una vez creado se mostrará en la lista de boletines, permitiendo previsualizarlo y modificarlo cuantas veces sea necesario. Una vez terminada la elaboración de un boletín se debe bloquear para indicar que está listo para su envío por correo a los clientes. Los de tipo "_lista de correo_" se enviarán a todos los clientes que estén subscritos a los boletines en su cuenta de usuario. Los de tipo "_notificación de producto_" se enviarán a todos los clientes que estén subscritos a alguno de los productos asociados al boletín. Los productos se asocian a un determinado boletín en el momento de enviarlo por correo.

## 21.8. Información

Esta opción proporciona información de carácter técnico acerca del servidor _web_ y las versiones de _software_ que se están utilizando. Se lista información del sistema operativo, del servidor _web_, del servidor de base de datos, de la versión de osCommerce, y de la versión de PHP.

## 21.9. Usuarios conectados

El listado de usuarios conectados muestra el detalle de los visitantes que se encuentran actualmente navegando por la tienda, incluyendo tanto a los usuarios registrados como a los anónimos. Por cada uno de ellos se muestra el tiempo que llevan conectados, su nombre (si está registrado), su dirección IP, la hora en la que se conectó, la hora en la que pinchó por última vez sobre algún enlace de la página, la última URL visitada, y el detalle de su "_carrito de la compra_".
