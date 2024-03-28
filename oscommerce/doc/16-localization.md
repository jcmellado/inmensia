# 16. osCommerce: Localización

_26-03-2007_ _Juan Mellado_

Una vez instalado el _software_, y hecha una revisión del bloque de configuración principal, es tiempo de dedicarse a examinar las opciones de "localización". Estas opciones incluyen la gestión de idiomas, monedas, y las traducciones de algunos textos específicos, como los estados por los que pasan los pedidos.

## 16.1. Idiomas

osCommerce incluye por defecto tres idiomas: inglés, alemán y español. La posibilidad de cambiar a cualquiera de ellos en cualquier momento es una opción francamente interesante que hace que el _software_ gane muchos enteros. Es más, osCommerce permite establecer un idioma como predeterminado, e incluso dar de alta otros nuevos aparte de los que trae pre-instalados.

Si se va a crear una tienda exclusivamente en español, lo más normal será ir a la opción de idiomas, editar "Español", marcarlo como predeterminado, y borrar el resto. Si se necesita añadir un idioma nuevo lo más recomendables es visitar la página oficial de "contribuciones" de idiomas de osCommerce donde existen una gran cantidad de traducciones listas para instalar.

A cada idioma es posible asignarle una serie de datos, como el nombre del mismo, su código o abreviatura internacional, una imagen con la bandera del país más representativo donde se habla, el directorio en el que se encuentra instalado (más sobre esto en el siguiente párrafo), y el orden en que debe aparecer dentro de los listados de selección de idiomas.

Desde un punto de vista técnico, la forma en que gestiona osCommerce los textos en distintos idiomas es sencilla. En el directorio ```catalog/includes/languages``` se encuentra un fichero y un directorio por cada idioma. En el fichero se encuentran definidos los textos de carácter general para el idioma en cuestión, y en el directorio hay ficheros con los textos de cada página en particular. Todos los ficheros tienen extensión ```.php``` debido que contienen código PHP válido. Es decir, son ejecutados cuando se construye una página. Abriendo cualquiera de ellos se puede ver que constan sólo de una serie de sentencias ```define``` que establecen el valor de constantes con los textos para cada idioma.

Normalmente no habría que tocar ninguno de estos ficheros, pero ocurre que algunas traducciones no son todo lo afortunadas que debieran ser y es necesario cambiarlas. Para ello hay una opción dentro del propio menú de configuración de osCommerce, concretamente en la sección "_Herramientas -> Definir Idiomas_". Lo que hace esta opción es permitirnos editar los ficheros directamente desde el navegador, sin tener que acceder al servidor _web_. Hay que tener cuidado y modificar sólo los textos, respetando la sintaxis del código PHP, y utilizando los códigos adecuados para las vocales acentuadas o caracteres especiales, tal y como se requieren normalmente para que se vean correctamente en las páginas HTML.

Una última cosa, el hecho de borrar un idioma a través de la interface de osCommerce no implica que se borren los ficheros físicamente del servidor, lo que permite volver a añadir los idiomas quitados en cualquier momento.

## 16.2. Monedas

Otra baza importante a considerar para la elección de osCommerce es el soporte que ofrece para la utilización de distintas monedas a un mismo tiempo, incluyendo la posibilidad de realizar automáticamente la conversión de los precios de una moneda a otra. Aunque esta última facilidad ha de manejarse con cuidado, ya que requiere que el tipo de cambio esté siempre correctamente actualizado.

Por defecto osCommerce ofrece dos monedas pre-instaladas: dólar (EEUU) y euro. Y al igual que con los idiomas, permite hacer una de ellas predeterminada, e incluso añadir nuevas si es necesario. Por cada moneda se pueden definir una serie de parámetros, como su nombre, abreviatura, indicación de si la abreviatura debe aparecer a la izquierda o derecha de las cantidades, separador de decimales, separador de miles, número de decimales, y un factor de conversión con respecto a la moneda marcada como predeterminada.

Siguiendo con el ejemplo del apartado anterior, y suponiendo que se va a montar una página exclusivamente para algún país de la zona euro, bastaría con marcar "Euro" como predeterminado, establecer el cambio a valor ```1```, y eliminar la moneda "Dólar".

Si se requiere trabajar con varias monedas a un mismo tiempo entonces será necesario tener el tipo de cambio correctamente actualizado en cada momento. osCommerce tiene un botón en la propia opción de mantenimiento de monedas que lo actualiza _on-line_. Concretamente lo que hace es conectarse con [https://www.oanda.com](https://www.oanda.com) para recuperar el cambio actualizado a través de un servicio gratuito que ofrece esa _web_. Alternativamente, si falla la conexión con ese servidor, lo reintenta con la _web_ de [https://www.xe.com](https://www.xe.com) que ofrece un servicio similar. Naturalmente lo mejor es programar este proceso de actualización mediante el ```cron``` (o equivalente) del servidor para evitar recordar tener que hacerlo manualmente. Lo único importante a este respecto es que el código (abreviatura) de las monedas deben corresponderse con los códigos utilizados internacionalmente para que los servicios _webs_ puedan identificar de forma correcta las monedas.

Por último, a nivel estrictamente técnico, hay que prever que el servidor _web_ donde esté instalada la tienda tenga acceso a las dos _webs_ anteriormente mencionadas, y que esté debidamente configurado para permitir recuperar un fichero a través de una URL. Esto último es necesario porque lo que hacen los servicios de conversión de monedas es entregar un fichero en formato de texto del que osCommerce extrae los tipos de cambio. Para más detalles lo mejor es ver las dos sencillas funciones que se encuentran en ```catalog/admin/includes/functions/localization.php```.

## 16.3. Estado Pedidos

En esta última opción del menú de localización se ofrece la posibilidad de cambiar el nombre de los estados por defecto por los que pasan los pedidos: pendiente, proceso y entregado. Teniendo claro que lo único que se hace a través de esta opción es cambiar las descripciones para cada idioma. Internamente osCommerce seguirá funcionando de la misma forma, independientemente del nombre que se les ponga a los estados.

El estado marcado como predeterminado será el que se asigne a todos los nuevos pedidos que se hagan a través de la _web_.
