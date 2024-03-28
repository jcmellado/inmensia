# 5. HTML5: Offline Web applications (1)

_15-01-2011_ _Juan Mellado_

Bajo el nombre de _Offline Web applications_ se encuentran agrupadas una serie de tecnologías definidas por HTML5 que permiten desarrollar aplicaciones _web_ que funcionen sin que el navegador esté conectado a Internet. Lo que así dicho parece un poco absurdo, ¿una aplicación _web_ sin conexión a Internet? WTF?

En realidad, de lo que trata la especificación HTML5 es de poder realizar más tareas en la parte cliente, y hacer más tareas implica por lo general aplicaciones más grandes y complejas. Las técnicas que se describen en esta parte de la documentación oficial están orientadas a evitar tener que descargar aplicaciones enteras de gran tamaño cada vez que nos conectemos a un sitio _web_, de forma que se puedan ejecutar de forma inmediata desde una _cache_ local ubicada en nuestro propio disco duro. Y de igual forma, proporciona los medios adecuados para que se puedan realizar actualizaciones _online_ de estos programas de una forma muy sencilla.

Un problema adicional que se pretende resolver además con la especificación es la desconexión temporal a la red, facilitando los mecanismos adecuados para que las aplicaciones puedan detectar esta circunstancia, guardar los datos en local, y sincronizarlos con el servidor una vez se restaure de nuevo la conexión.

En la especificación de HTML5 se distinguen tres elementos claves para la consecución de todos estos propósitos. El "_Cache Manifest_", un fichero de texto donde se le indica al navegador los recursos que utiliza una aplicación y el uso que se requiere de ellos. El "_Application Cache API_", un objeto JavaScript que permite comprobar el estado de la _cache_ y solicitar la actualización de la misma. Y el "_Browser State_", un par de eventos JavaScript para saber cuando el navegador pierde o gana la conexión a red.

## 5.1. Cache Manifest

El _manifest_ de una aplicación es un simple fichero de texto en formato UTF-8 con líneas separadas por retorno de carro, salto de línea, o ambos seguidos de la forma acostumbrada en _Windows_, y que se tiene que colocar en el servidor _web_ de igual forma que el resto de recursos que necesita una aplicación _web_.

Una aplicación le dice al navegador donde localizar su _manifest_ mediante un nuevo atributo añadido por la especificación a la etiqueta ```html```.

```html
<html manifest="prueba.manifest">
```

Los ficheros no tienen porque seguir el patrón ```*.manifest```, pero se recomienda. Y una cosa bastante importante a tener en cuenta es que el servidor _web_ debe enviarlos al navegador con el tipo MIME ```text/cache-manifest```, algo que en Apache por ejemplo se puede conseguir añadiendo la siguiente línea al fichero ```.htaccess```:

```text
AddType text/cache-manifest .manifest
```

La primera línea de un fichero _manifest_ debe contener obligatoriamente el texto ```CACHE MANIFEST```. Las siguientes líneas, si existen, pueden contener líneas en blanco, comentarios, cabeceras de inicio de sección, o los datos correspondientes a una sección.

```text
CACHE MANIFEST
```

Como comentario se interpretará cualquier línea cuyo primer carácter distinto de un espacio blanco o tabulador sea una almohadilla (```#```).

```text
CACHE MANIFEST
# Comentario

# Otro comentario
# Otro comentario más
```

Los inicios de sección son líneas que sólo contienen el texto ```CACHE:```, ```FALLBACK:```, o ```NETWORK:```, independientemente de los espacios o tabuladores que puedan aparecer antes o después que ellos. Irónicamente, como bien apunta la especificación, ```CACHE:``` es la sección por defecto y no es necesario declararla de forma explícita. En cualquier caso, estas secciones pueden repetirse tantas veces como se quiera dentro de un manifest.

```text
CACHE MANIFEST

# Datos de la sección CACHE por defecto
index.html
images/logo.png

# Sección NETWORK para recursos que requieren forzosamente conexión
NETWORK:
http://api.twitter.com

# Sección FALLBACK para saber que hacer si no hay conexión
FALLBACK:
images/maps unavailable.png

# Otra sección CACHE, las URLs pueden ser relativas o absolutas
CACHE:
http://www.prueba.com/images/image01.png
http://www.prueba.com/images/image02.png
http://www.prueba.com/images/image03.png
```

Las secciones de tipo ```CACHE``` están formadas por líneas en la que se indica al navegador los recursos que debe guardar en su _cache_ local, incluida la propia página donde se hace referencia al _manifest_, lo que no es estrictamente necesario, pero si recomendable. Sólo puede aparecer un recurso por línea. Las URLs pueden ser relativas a la ubicación del _manifest_ o absolutas, e incluso se pueden utilizar direcciones HTTPS, aunque en este último caso todas las direcciones que se resuelvan tienen que tener el mismo origen. No hay una cantidad máxima permitida, ni una restricción sobre el tamaño total de la _cache_, aunque la cifra mágica actual es del orden de 5 MB. De igual forma no hay un tiempo máximo de estancia en _cache_, aunque si se recomienda que los navegadores implementen una política de expiración del contenido, y den a los usuarios la posibilidad de limpiar la _cache_ de igual forma que pueden eliminar el historial o las _cookies_.

Las secciones de tipo ```NETWORK``` contienen líneas con nombres de recursos que necesitan forzosamente una conexión a red para funcionar, permitiéndose el uso del carácter comodín ```*``` para indicar que cualquier otro recurso de cualquier otro sitio distinto a la _web_ de la aplicación debe ser buscado en la red.

Las secciones de tipo ```FALLBACK``` se componen de líneas con dos nombres de recursos separados por espacios y/o tabuladores que indican al navegador que recurso alternativo utilizar si el original no puede ser accedido debido a que la conexión a red no está disponible. En cualquier caso, ambos recursos deben pertenecer al mismo origen que el manifest.

Por último, indicar que algo muy importante a tener en cuenta es que la _cache_ de una aplicación sólo se refresca si su _manifest_ cambia, no si se modifica algún recurso referenciado dentro de él.
