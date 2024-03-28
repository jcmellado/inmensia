# 1. Drupal: Instalación

_14-09-2005_ _Juan Mellado_

Drupal es un gestor de contenidos. Un tipo de _software_ que se ha popularizado con las siglas inglesas CMS (_Content Management System_).

Dentro de la amplia oferta de CMS disponibles, Drupal destaca por la calidad de su código fuente, la facilidad con la que pueden añadirse extensiones al mismo, y el hecho de generar páginas HTML que cumplen con los estándares propuestos por el consorcio W3C. Además, es _software_ libre distribuido bajo licencia GPL. Su descarga y uso es gratuito.

Es la primera vez que utilizo Drupal, y de hecho, es la primera vez que instalo un _software_ de este tipo en un servidor, por lo que este texto no es más que un breve relato de mi experiencia personal instalando y configurando la versión 4.6.3 de Drupal, no una explicación completa y exhaustiva del proceso de instalación y configuración.

Antes de proceder a la instalación de Drupal deberían consultarse los ficheros de texto que se incluyen junto al _software_, que contienen la licencia de uso y guía de instalación, así como la documentación que se encuentra disponible en la _web_ oficial [https://drupal.org](https://drupal.org).

## 1.1. Requerimientos

Los requerimientos para la versión 4.6.3 de Drupal se corresponden con los de una arquitectura LAMP (**L**inux **A**pache **M**ySQL **P**HP). Como servidor _web_ se recomienda Apache 1.3 o superior. Como base de datos MySQL 3.23.17 o superior, incluyendo MySQL 4. Y PHP 4.3.3 o superior, incluyendo PHP 5.

También se puede instalar sobre IIS y otra base de datos de tipo PEAR, como PostgreSQL. Pero este _software_ es más difícil de encontrar en las ofertas habituales de los servicios de _hosting_.

## 1.2. Instalación

Para instalar Drupal 4.6.3 hay que realizar los siguientes pasos:

- Descargar el fichero ```drupal-4.6.3.tar.gz```, de menos de 500 Kb, desde la página oficial de Drupal.
- Descomprimir el fichero en un directorio de nuestro servidor _web_.
- Ejecutar un fichero de _script_ que crea el esquema de la base de datos.
- Modificar un fichero ```.php``` para establecer los valores de dos variables de configuración.
- Acceder a nuestro sitio _web_ para crear la primera cuenta de usuario, y configurar el sistema.

El fichero ```drupal-4.6.3.tar.gz``` contiene una distribución de Drupal que incluye la licencia de uso, la guía de instalación, el _log_ de cambios, los _scripts_ de creación de la base de datos, el núcleo (_core_) del sistema, varios módulos (_module_), tres temas (_theme_), un motor de temas (_theme engine_), un fichero de configuración para un sitio por defecto, un fichero ```.htaccess```, y un ```favicon.ico``` con el logotipo de Drupal.

El fichero se debe descomprimir y copiar a un directorio del servidor _web_, incluido el directorio raíz.

Dentro del subdirectorio ```database``` se encuentran los ficheros de _scripts_ para la creación del esquema de la base de datos, tanto para MySQL como para PostgreSQL.

En el caso de utilizar MySQL, como en mi caso, y suponiendo que se tenga ya creada una base de datos, algo que generalmente se hace desde el panel de control del servicio de _hosting_, el esquema se puede crear ejecutando el siguiente comando desde línea de comandos con los valores adecuados para nuestra base de datos:

```shell
mysql -u usuario -p basedatos < database/database.mysql
```

Posteriormente, desde la propia línea de comandos, o mediante una aplicación como phpMyAdmin, se puede comprobar que se han creado las 55 tablas que conforman el esquema utilizado en esta distribución de Drupal.

A continuación se debe abrir el fichero de configuración del sitio por defecto, ```sites/default/settings.php```, y localizar y cambiar el valor de las variables ```$db_url``` y ```$base_url``` por los valores adecuados para nuestra _web_:

```php
<?php
$db_url = "mysql://usuario:clave@host/basedatos";

$base_url = "http://www.dominio.com";
?>
```

Una vez hecho esto, en la guía de instalación se recomienda crear un subdirectorio llamado ```files``` con permisos de lectura y escritura. En dicho directorio se almacenarán los ficheros propios del sitio, como logotipos, avatares, y cualquier otro tipo de archivo de este tipo. Posteriormente, a través de las opciones del menú de administración de Drupal, se puede cambiar el nombre y ubicación de este subdirectorio.

Llegado este punto ya podemos acceder a nuestra _web_, que nos recibe con una página en la que nos solicita que creemos la primera cuenta de usuario. Ese primer usuario poseerá derechos de administración total sobre el sitio.

En la guía oficial de instalación puede encontrarse más información, como la gestión de múltiples sitios en un mismo servidor, la programación de procesos mediante _cron_, o el proceso de actualización a nuevas versiones de Drupal.

## 1.3. Cambio de Idioma

El idioma por defecto de Drupal es el inglés. Para cambiarlo al castellano es necesario descargar el fichero ```es-4.6.0.tar.gz``` con los textos en nuestro idioma desde la página oficial de Drupal. Descomprimirlo en un directorio de nuestra máquina local. Y cargarlo desde las opciones del menú de administración de Drupal.

El fichero ```es-4.6.0.tar.gz``` contiene la licencia de uso, la guía de instalación, y un fichero de nombre ```es.po``` con los textos en español.

La instalación del nuevo idioma requiere la ejecución de dos pasos.

El primer paso consiste en activar el módulo ```locale``` que se encuentra en "_administer->modules_". Al hacerlo, veremos que aparece una nueva opción de menú llamada "_localization_" junto al resto de opciones de administración.

El segundo paso consiste en cargar el fichero ```es.po``` a través de la opción "_import_" del módulo "_localization_". Hay que localizar el fichero, seleccionar el idioma "_Spanish_", e importar. Al hacerlo, veremos que aparece el idioma "_Spanish_" en la lista de idiomas disponibles. Por lo que sólo resta habilitarlo, y ponerlo por defecto para que a partir de ahora todos los textos aparezcan siempre en castellano.

La traducción tiene un nivel alto bastante aceptable. Aunque se pueden encontrar algunas traducciones bastante literales, como ocurre con la palabra inglesa _access_, que se ha traducido como _accesar_.

## 1.4. Configuración Básica

Mientras estemos terminando de montar la _web_, es buena idea impedir que los visitantes puedan darse de alta libremente como usuarios en el sistema. Para ello se debe acceder al menú "_administrar->usuarios->configurar_" y marcar la opción "_Sólo los administradores pueden crear cuentas para usuarios nuevos_".

En el módulo "_administrar->opciones_" existen una gran cantidad de opciones con las que se puede personalizar el aspecto de la _web_, incluyendo el nombre del sitio, la dirección de correo electrónico de contacto, la descripción del objetivo del sitio, leyendas para la cabecera y pie de página, y el nombre dado por defecto en los comentarios a los usuarios anónimos. También se permite indicar la página de inicio y el tipo de URLs a utilizar, temas de los que trataré en el siguiente artículo.

Otros grupos de opciones se refieren a la gestión de errores, _cache_, y sistemas de archivos que podemos dejar con los valores por defecto. O modificarlos a nuestro gusto si sabemos lo que estamos haciendo.

El último grupo de opciones permite establecer la zona horaria y el formato de las fechas.

## 1.5. Conclusiones

La instalación de Drupal es sencilla, pero está pensada para ser realizada por un usuario con unos conocimientos medios, ya que requiere acceder al servidor para copiar el _software_, saber como crear una base de datos, conectarse a la misma para crear el esquema, y tener cierta familiaridad con PHP para modificar el fichero de configuración.

Una instalación _on-click_ a través del panel de control del servicio de _hosting_ sería una buena alternativa para los usuarios sin conocimientos que quieran utilizar este _software_.

Drupal ocupa muy poco espacio en disco, ya que el núcleo del sistema es muy pequeño, lo conforman muy pocos ficheros de tamaño reducido. Y más si cabe teniendo en cuenta que la mayoría de módulos que trae la distribución no son necesarios, ya que no están activos por defecto, y no haría falta ni tan siquiera tenerlos en el servidor.

El resto del grueso de ficheros instalados corresponde a los tres temas que trae la distribución, incluido uno llamado "_bluemarine_" que es el que se instala por defecto.

Una vez instalado, Drupal nos deja una _web_ accesible y configurada para comenzar a dar de alta contenido y personalizar el aspecto de la presentación del mismo.
