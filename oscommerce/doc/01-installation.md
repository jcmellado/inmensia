# 1. osCommerce: Descarga e Instalación

_20-03-2007_ _Juan Mellado_

osCommerce es un programa que permite crear tiendas virtuales en Internet. Posiblemente sea el _software_ más utilizado hoy en día para montar este tipo de _webs_. Su principal ventaja es que es _software_ libre, gratuito, de código abierto, distribuido bajo la licencia pública general GNU, y que cuenta con una gran comunidad de usuarios.

Esta serie de artículos constituye un breve relato de mi experiencia instalando y configurando la versión 2.2 de osCommerce. Esta serie no pretende ser en ningún caso una guía detallada y exhaustiva de instalación, configuración, personalización y ampliación. Antes de proceder a la instalación de oscCommerce deberían consultarse los ficheros que se incluyen junto al _software_, que contienen la licencia de uso y guía de instalación, así como la documentación que se encuentra disponible en la _web_ oficial [https://www.oscommerce.com](https://www.oscommerce.com).

## 1.1. Requerimientos

Los requerimientos de osCommerce se podrían considerar como el "estándar de los requerimientos" de las aplicaciones _web_ de hoy en día. Requiere un servidor de páginas _web_, preferiblemente Apache, soporte para PHP 4.0 o superior, y una base de datos MySQL. El sistema operativo se podría decir que es casi indiferente, ya que se ejecuta indistintamente en plataformas Linux, Unix, BSD, Mac OS X, y Windows.

La ventaja de soportar toda esta plétora de sistemas es que nos permite empezar instalándolo localmente en un portátil, o un PC de sobremesa, para hacer todas las pruebas que creamos necesarias con él hasta que decidamos llevarlo a un entorno de producción con conexión real a Internet. Además, prácticamente cualquier empresa que ofrece servicios de _hosting_ hoy en día ofrece por un buen precio alojamiento con los requerimientos que solicita osCommerce, ya que son los típicos de un sistema englobado dentro de la generación LAMP (**L**inux **A**pache **M**ySQL **P**HP).

## 1.2. Descarga

El _software_ de osCommerce se puede descargar directamente desde su página oficial. Concretamente desde la siguiente dirección: [https://www.oscommerce.com/Products](https://www.oscommerce.com/Products).

En el momento de escribir este artículo la última versión disponible de osCommerce es la 2.2 Milestone 2 Update 060817, y el fichero que contiene el _software_ para Linux, que es el sistema operativo que he utilizado para la instalación, tiene por nombre ```oscommerce-2.2ms2-060817.tgz```.

## 1.3. Antes de la Instalación

Antes de proceder a la instalación de osCommerce en un servidor _web_ se debe tener claro que se necesitan un mínimo de conocimientos técnicos. Se debe saber como subir el contenido del fichero descargado al servidor _web_ a través de FTP, se debe saber como cambiar los permisos de directorios o ficheros en el servidor _web_, y se debe saber como crear una nueva base de datos en MySQL, o acceder a alguna creada previamente.

Las buenas noticias es que hoy en día hay disponibles muchos clientes FTP gratuitos que nos simplifican el proceso de subir ficheros a un servidor _web_, como por ejemplo FileZilla. Incluso es posible que nuestra empresa suministradora de _hosting_ tenga habilitada alguna opción que nos permita subir ficheros a través de un servicio _webftp_, es decir, a través de una página _web_. Es más, desde estos propios clientes FTP es bastante probable que podamos cambiar fácilmente los permisos de directorios y ficheros.

En cuanto al proceso de creación de una nueva base de datos MySQL, lo más seguro es que nuestra empresa de _hosting_ también nos ofrezca a través de algún tipo de panel de control de nuestra cuenta la posibilidad de crearla con un par de _clicks_.

Los detalles concretos dependerán de cada caso concreto, pero lo que debe quedar claro es que sin conocer todos estos detalles no se podrá realizar la instalación por uno mismo. Con un poco de suerte, incluso puede que el servidor de _hosting_ ofrezca el programa ya instalado y listo para usar.

## 1.4. Instalación

El primer paso para realizar la instalación con el fichero descargado consiste en copiar su contenido al servidor _web_ donde queremos instalar la tienda virtual. Más concretamente, se debe copiar el directorio ```catalog``` completo. A continuación cambiar a ```777``` los permisos del fichero ```catalog/includes/configure.php```, y por último, mediante un navegador, acceder a la URL ```catalog/install/index.php``` de nuestro servidor _web_. A partir de ahí, si todo ha ido bien, se muestra un asistente que nos guía pidiéndonos datos y realizando por sí solo la instalación.

He de confesar que la instalación de osCommerce es muy sencilla y me sorprendió bastante por su limpieza el asistente. La documentación oficial en la que se describe todo el proceso es muy concisa, merece la pena echarle un vistazo antes de instalar el _software_ para hacerse una idea de cómo transcurrirá este y de qué información se necesitará.

Después de la primera página _web_ que muestra el asistente, que nos permite elegir entre una instalación nueva o un _upgrade_ de una versión previa a una versión más moderna, se nos abre otra que nos pregunta si se quiere que se importe automáticamente la estructura de base de datos y se cargue una configuración por defecto. Naturalmente la opción más lógica al instalar por primera vez es dejar estas dos opciones marcadas por defecto. A continuación se nos pide los parámetros de acceso a la base de datos: _host_, usuario, clave, nombre, etc. Después se accede a un par de páginas en las que se muestra el resultado de la importación de tablas y carga de datos. A continuación se accede a una nueva página en la que se solicitan los parámetros del servidor _web_: URL, _path_, dominio de las _cookies_, etc. Y después de una nueva página en la que se nos informa de la corrección, o no, de los datos introducidos anteriormente, se accede a una última página en la que se solicitan los parámetros del servidor _web_ seguro para las conexiones HTTPS, aunque esta página sólo se muestra si escogemos que determinadas transacciones se realicen a través de conexiones SSL, un tipo de conexiones que requieren un certificado emitido por una empresa especializada. No obstante, disponer de un servidor seguro con certificado no es obligatorio para instalar osCommerce.

## 1.5. Después de la instalación

Una vez ejecutado con éxito el asistente de instalación se deben realizar una tareas finales de limpieza y seguridad. Concretamente estos cinco pasos que se describen en la documentación oficial:

1. Cambiar el nombre del directorio ```catalog/install```, o borrarlo completamente. Una vez hecha la instalación ya no se utilizará para nada más.

2. Cambiar los permisos del fichero ```catalog/includes/configure.php``` a ```644```. Aunque en algunos servidores puede que sea necesario cambiarlos a ```444``` si aparece algún mensaje de advertencia relacionado con temas de seguridad.

3. Cambiar los permisos del directorio ```catalog/images``` a ```777```.

4. Cambiar los permisos del directorio ```admin/images/graphs``` a ```777```.

5. Crear un nuevo directorio ```admin/backups``` con permisos ```777```. En este directorio se guardarán por defecto las copias de seguridad que se hagan de la base de datos.

En este momento ya se podría acceder a la tienda virtual a través de la dirección ```catalog/index.php``` del servidor _web_, y a las opciones de administración a través de ```catalog/admin/index.php```. El problema es que, al igual que nosotros, podría acceder cualquiera, por lo que es recomendable proteger esos directorios mediante usuario y contraseña a través de un fichero ```.htaccess``` o similar. La mayoría de servicios de _hosting_ ofrecen la posibilidad de proteger directorios de una forma sencilla, normalmente a través de alguna opción del panel de control de la cuenta del usuario.

Llegados hasta aquí podría darse por concluida la instalación en si misma de osCommerce, restando ahora las tareas de configuración y personalización, temas de próximos artículos.

## 1.6. Conclusiones

La instalación de osCommerce no es para principiantes absolutos, aunque si se tiene algo de conocimientos técnicos, o se ha instalado anteriormente otro _software_ basado en _web_ con características similares, entonces no debería presentar mayores complicaciones. Personalmente opino que la clave está en conocer los valores correctos que hay que poner en cada parámetro que nos solicita el asistente, saber cual es el nombre del _host_, de la base de datos, los usuarios, claves, y demás.

Después de la instalación se puede acceder a la tienda virtual cargada con unos artículos de ejemplo. Está bien tenerlos de guía, para ver como se definen, pero una vez vistos lo más seguro es que los borremos de forma casi inmediata. Por su parte, en el área de administración, impresiona un poco la enorme cantidad de parámetros que se pueden configurar. En los próximos artículos de esta serie entraré en el detalle de cada uno de ellos para ver su uso y significado.
