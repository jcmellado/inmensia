# 3. Redis (1)

_30-07-2018_ _Juan Mellado_

La introducción de un nuevo tipo de datos en la futura versión 5 de Redis supone una buena oportunidad para escribir un par de artículos sobre este veterano _software_.

Aunque tradicionalmente vista como una base de datos NoSQL de tipo clave-valor en memoria, útil sólo como _cache_, en realidad Redis ofrece soluciones para un mayor número de casos de usos a través de sus componentes básicos: tipos y comandos.

Los tipos soportados por Redis son las cadenas de textos, _arrays_ de _bits_, _hashes_, listas, conjuntos, conjuntos con peso, e _HyperLogLogs_ (una estructura creada específicamente para estimar el número de elementos distintos de una colección). La futura versión 5 de Redis añadirá además los _streams_, un tipo especializado para la gestión de series temporal.

Los comandos de Redis son las herramientas a través de la que se construyen estructuras con los tipos soportados y se manipulan sus elementos. La integridad de los resultados está garantizada porque los comandos se ejecutan de forma atómica, pudiendo incluso utilizarse transacciones para ejecutar de forma atómica un conjunto de comandos y hasta ejecutar scripts en Lua.

La documentación de Redis es bastante detallada y su lectura obligada. Una característica de Redis a este respecto es que junto a la definición de los comandos se describen sus casos de uso más habituales, así como los posibles usos que se le puede dar a la herramienta, como por ejemplo su utilización como _broker_ de mensajes mediante colas.

## 3.1. Instalación

No existe una distribución oficial de Redis para Windows, pero se pueden encontrar distribuciones no oficiales, o más simplemente utilizar una imagen de Docker.

En el caso de las distribuciones no oficiales hay que descomprimir el fichero descargado en un directorio y ejecutar ```redis-server.exe```. El fichero de configuración por defecto es ```redis.conf```. Y la forma más sencilla de probar la instalación es usando el cliente de línea de comandos ejecutando ```redis-cli.exe```.

Comentar que Redis soporta de forma nativa su despliegue en una configuración distribuida en _cluster_ con dos modos de persistencia, el modo RDB (_Redis Database File_), que funciona creando _snapshots_ cada N unidades de tiempo, siempre y cuando se hayan modificado M claves en dicho periodo, y el modo AOF (_Append Only File_), que funciona almacenando todas las operaciones a fichero de forma inmediata garantizando la integridad de la información a costa de un peor rendimiento.

## 3.2. Claves

Redis almacena todos los datos asociados a una clave única, como las claves primarias en las base de datos relacionales, sólo que las claves en Redis son siempre cadenas.

No hay una forma estándar de escribir claves, sólo unas recomendaciones generales basadas en el uso de _namespaces_ separados por el carácter de dos puntos. Por ejemplo, ```user:1000:views``` podría utilizarse para almacenar el número de veces que se ha visto el perfil del usuario con ```id``` 1000, pero no deja de ser una recomendación, no una obligación seguir dicho formato.

Todas las claves se tratan siempre en función de su representación a nivel binario. Es decir, no tiene sentido hablar de ningún tipo de codificación, como UTF-8, o juego de caracteres, como ISO-8859-1. Las claves se comparan siempre en base a la secuencia de bytes que las representan.

Comandos específicos para claves:

- ```KEYS```, obtiene las claves que siguen un patrón

- ```SCAN```, itera por todas las claves de la base de datos actual seleccionada

- ```RANDOMKEY```, obtiene una clave de forma aleatoria

- ```EXISTS```, comprueba si una clave existe

- ```RENAME```, renombra una clave

- ```RENAMENX```, renombra una clave, pero sólo la nueva clave no existe

- ```DEL```, borra una clave

- ```UNLINK```, borra una clave de forma no bloqueante

- ```TYPE```, obtiene el tipo del valor almacenado bajo una clave

- ```SORT```, ordena los elementos de una lista, conjunto o conjunto con pesos

- ```EXPIRE```, establece el tiempo de vida de una clave en segundos

- ```EXPIREAT```, establece el tiempo de vida de una clave con un timestamp

- ```TTL```, obtiene el tiempo de vida de una clave

- ```PEXPIRE```, establece el tiempo de vida de una clave en milisegundos

- ```PEXPIREAT```, establece el tiempo de vida de una clave con un timestamp en milisegundos

- ```PTTL```, obtiene el tiempo de vida de una clave en milisegundos

- ```PERSIST```, elimina el tiempo de vida de una clave

- ```DUMP```, obtiene una cadena resultado de serializar una clave

- ```RESTORE```, restaura una clave a partir de una cadena obtenida con ```DUMP```

- ```OBJECT```, obtiene un volcado de características internas de una clave

- ```TOUCH```, modifica la fecha de último acceso a una clave

- ```MOVE```, mueve una clave a otra base de datos

- ```MIGRATE```, mueve una clave a otra instancia de Redis

Una característica interesante de Redis es que se puede programar un tiempo de expiración para una clave de forma individual, de forma que pasado ese tiempo Redis la eliminará de memoria automáticamente.

## 3.3. String

Las cadenas de texto son el tipo más básico de Redis. Como ya se ha comentado, es el tipo utilizado para almacenar las claves.

Su caso de uso más habitual es almacenar información a modo de _cache_. Por ejemplo, para almacenar _tokens_, _cookies_, fragmentos estáticos de HTML, o una página completa, utilizando su URL como clave y el HTML como valor.

```text
> SET html:login <html><body>Hello!</body></html>
OK

> EXISTS html:login
(integer) 1

> GET html:login
"<html><body>Hello!</body></html>"
```

Comandos específicos para el tipo ```string```:

- ```SET```, establece el valor de una clave

- ```SETNX```, establece el valor de una clave, pero sólo si no existe

- ```SETEX```, establece el valor y tiempo de vida de una clave en segundos

- ```PSETEX```, establece el valor y tiempo de vida de una clave en milisegundos

- ```GET```, obtiene el valor de una clave

- ```GETSET```, establece el valor de una clave y retorna el valor anterior

- ```MSET```, establece el valor de una serie de claves

- ```MSETNX```, estable el valor de una serie de claves, pero sólo si ninguna existe

- ```MGET```, obtiene el valor de una serie de claves

- ```INCR```, incrementa el valor entero de una clave

- ```INCRBY```, incrementa el valor entero de una clave por un valor dado

- ```INCRBYFLOAT```, incrementa el valor decimal de una clave por un valor dado

- ```DECR```, decrementa el valor entero de una clave

- ```DECRBY```, decrementa el valor entero de una clave por un valor dado

- ```STRLEN```, obtiene la longitud del valor de una clave

- ```GETRANGE```, obtiene una subcadena del valor de una clave

- ```SETRANGE```, establece una subcadena del valor de una clave

- ```APPEND```, añade un valor al valor de una clave

- ```BITFIELD```, ejecuta comandos a nivel de campos de _bits_ sobre el valor de una clave

Comentar que los valores de tipo ```string``` en Redis pueden tener una longitud de hasta 512 MB.

## 3.4. Bitmap

Redis no considera ```bitmap``` como un tipo, sino como un conjunto de comandos que permiten realizar operaciones a nivel de _bit_ sobre valores del tipo ```string```. Lo que cobra sentido cuando se recuerda que Redis trata las cadenas como secuencias de _bytes_, no como secuencias de caracteres.

El caso de uso habitual es marcar con un _bit_ a 1 si alguna condición se cumple. Por ejemplo para indicar que el usuario de una aplicación _web_ ha leído y aceptado las condiciones de uso de la misma. O para indicar si quiere recibir correos electrónicos periódicos con las novedades de la misma.

```text
> SETBIT user:1000:flags 4 1
(integer) 0

> SETBIT user:1000:flags 6 1
(integer) 0

> GETBIT user:1000:flags 4
(integer) 1

> GETBIT user:1000:flags 5
(integer) 0
```

Comandos específicos para el tipo ```bitmap```:

- ```SETBIT```, establece el valor de un _bit_

- ```GETBIT```, obtiene el valor de un _bit_

- ```BITOP```, realiza operaciones lógicas a nivel de _bit_

- ```BITCOUNT```, cuenta el número de _bits_ a 1

- ```BITPOS```, obtiene el primer _bit_ a cero o uno

Los ```bitmaps``` son adecuados además para señalizar eventos que ocurren con una periodicidad determinada y obtener estadísticas sobre ellos. Por ejemplo, para almacenar si un usuario visita una _web_ cada día, utilizando un _bit_ por día. Siendo lo más habitual distribuir esta información entre varias claves. Por ejemplo, guardando un ```bitmap``` por cada año, mes, o periodos más pequeños en función de cada evento concreto.
