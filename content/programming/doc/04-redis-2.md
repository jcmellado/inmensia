# 4. Redis (y 2)

_03-08-2018_ _Juan Mellado_

Continuando con el repaso de las características básicas de Redis, en este artículo se revisan los tipos que representan estructuras agregadas de elementos.

## 4.1. Hash

Este tipo almacena pares claves-valor asociados a un nombre. Se comporta como los _arrays_ asociativos presentes en la mayoría de lenguajes de programación.

El caso de uso más habitual que implementan es almacenar información relacionada con una entidad, a modo de atributos de un objeto.

```text
> HSET user:1000 name Bob
(integer) 1

> HSET user:1000 age 24
(integer) 1

> HKEYS user:1000
1) "name"
2) "age"

> HVALS user:1000
1) "Bob"
2) "24"

> HMSET user:1000 gender male surname Williams
OK

> HMGET user:1000 name surname
1) "Bob"
2) "Williams"
```

Los comandos disponibles para trabajar con ```hashes``` siguen la notación característica de Redis para estructuras, donde cada comando empieza con una letra que hace referencia al tipo con el que operan. Por ejemplo, los comandos que empiezan por H operan con ```hashes```.

Comandos específicos para el tipo ```hash```:

- ```HSET```, establece el valor de un campo

- ```HSETNX```, establece el valor de un campo, pero sólo si no existía

- ```HMSET```, establece el valor de una serie de campos

- ```HGET```, obtiene el valor de un campo

- ```HGETALL```, obtiene todos los campos y valores

- ```HMGET```, obtiene los valores de los campos dados

- ```HDEL```, borra un campo

- ```HINCRBY```, incrementa el valor entero de un campo por un valor dado

- ```HINCRBYFLOAT```, incrementa el valor decimal de un campo por un valor dado

- ```HSTRLEN```, obtiene la longitud de un valor almacenado en un campo

- ```HEXISTS```, comprueba si un campo existe

- ```HLEN```, obtiene el número de campos

- ```HKEYS```, obtiene todos los nombres de campos

- ```HVALS```, obtiene todos los valores

- ```HSCAN```, itera por los campos

Los ```hashes``` de Redis de tamaño reducido realizan un uso muy eficiente de la memoria y se recomienda su uso cuando el número de claves, y sus valores, sean de reducido tamaño.

## 4.2. List

Las listas en Redis están implementadas como listas enlazadas, lo que quiere decir que la inserción y borrado son operaciones rápidas que se ejecutan en un tiempo constante, frente el acceso a los elementos por su índice que es una operación más costosa.

El caso de uso más habitual es almacenar las últimas N ocurrencias de un determinado evento o tipo de objeto. Por ejemplo, se pueden utilizar listas para almacenar los IDs de los últimos artículos consultados en la _web_ de una tienda, de forma que puedan recuperarse muy rápidamente sin necesidad de realizar una consulta costosa sobre una base de datos relacional.

```text
> LPUSH user:1000:latest 1 2 3
(integer) 3

> LRANGE user:1000:latest 0 -1
1) "3"
2) "2"
3) "1"

> LPUSH user:1000:latest 4
(integer) 4

> LTRIM user:1000:latest 0 2
OK

> LRANGE user:1000:latest 0 -1
1) "4"
2) "3"
3) "2"
```

Comandos específicos para el tipo ```list```:

- ```LPUSH```, inserta elementos por la cabecera

- ```LPUSHX```, inserta un elemento en la cabecera, pero sólo si la lista existe

- ```RPUSH```, inserta elementos por la cola

- ```RPUSHX```, inserta un elemento por la cola, pero sólo si la lista existe

- ```LPOP```, obtiene y borra el elemento cabecera

- ```RPOP```, obtiene y borra el elemento de la cola

- ```RPOPLPUSH```, mueve el elemento de la cola a otra lista

- ```LINSERT```, inserta un elemento en un índice dado

- ```LSET```, establece el valor de un elemento en un índice dado

- ```LINDEX```, obtiene el elemento de un índice dado

- ```LRANGE```, obtiene los elementos de un rango dado

- ```LREM```, elimina elementos

- ```LTRIM```, elimina elementos en un rango dado

- ```LLEN```, obtiene el número de elementos

- ```BLPOP```, obtiene y borra el elemento cabecera de forma bloqueante

- ```BRPOP```, obtiene y borra el elemento de la cola de forma bloqueante

- ```BRPOPLPUSH```, mueve el elemento de la cola a otra lista de forma bloqueante

Los comandos permiten tratar las listas como pilas y colas añadiendo y eliminando elementos por los dos extremos, operaciones ambas muy rápidas al estar implementadas como listas enlazadas. Los nombres de los comandos utilizan la letra L (_left_) para indicar que la operación afecta a la cabecera de la lista y R (_right_) para indicar que afecta a la cola.

Notar además que los comandos que acceden a un rango de elementos utilizan el cero como índice para hacer referencia al primer elemento, y un índice negativo para hacer referenciar a los elementos por el final.

Un caso de uso especial de las listas es su utilización como colas siguiendo un modelo de suscriptores/consumidores. Los comando ```BLPOP``` y ```BRPOP``` extraen un elemento de una lista, bloqueando al cliente hasta que haya un elemento disponible, si la lista está vacía, o hasta que se sobrepase un tiempo de espera indicado como parámetro de la operación. Usando 0 como tiempo de espera el cliente espera indefinidamente.

Para probar el funcionamiento de Redis como cola de mensajes bloqueante basta con abrir un segundo cliente desde línea de comandos y ejecutar el comando ```BLPOP``` o ```BRPOP``` sobre una lista creada desde el primer cliente. Si la lista tiene elementos el segundo cliente retornará inmediatamente, pero si no lo tiene se bloqueará hasta que el primer cliente añada un nuevo elemento o se supere el tiempo de espera. Es realmente sencillo e inmediato. Para casos de usos más sofisticados se puede utilizar el comando ```BRPOPLPUSH```, que extrae un elemento de una lista y lo inserta en otra, lo que puede resultar de utilidad para evitar la pérdida de mensajes, almacenándolos en una lista de trabajo temporal.

## 4.3. Set

Los conjuntos en Redis son colecciones desordenadas de elementos. Un mismo elemento sólo puede añadirse una única vez a un mismo conjunto.

El caso de uso habitual para los conjuntos es su utilización como almacén de referencias de unas entidades relacionadas con otras. Como las _foreign keys_ en una base de datos relacional. Por ejemplo, para almacenar los _ids_ de las etiquetas (_tags_) de las entradas (_posts_) de un _blog_.

```text
> SADD post:1000:tags java spring redis
(integer) 3

> SMEMBERS post:1000:tags
1) "redis"
2) "spring"
3) "java"

> SISMEMBER post:1000:tags spring
(integer) 1

> SCARD post:1000:tags
(integer) 3

> SADD post:1001:tags java spring mongodb
(integer) 3

> SINTER post:1000:tags post:1001:tags
1) "spring"
2) "java"
```

Comandos disponibles con el tipo ```list```:

- ```SADD```, añade un elemento

- ```SMEMBERS```, obtiene todos los elementos

- ```SISMEMBER```, comprueba si un elemento pertenece a un conjunto

- ```SRANDMEMBER```, obtiene elementos de forma aleatoria

- ```SPOP```, obtiene y borra elementos de forma aleatoria

- ```SMOVE```, mueve un elemento a otro conjunto

- ```SREM```, elimina elementos

- ```SUNION```, obtiene la unión de una serie de conjuntos

- ```SUNIONSTORE```, obtiene la unión de una serie de conjuntos y la almacena en otro

- ```SINTER```, obtiene la intersección de una serie de conjuntos

- ```SINTERSTORE```, obtiene la intersección de una serie de conjuntos y la almacena en otro

- ```SDIFF```, obtiene la diferencia de un conjunto contra una serie de conjuntos

- ```SDIFFSTORE```, obtiene la diferencia de un conjunto contra una serie de conjuntos y la almacena en otro

- ```SCARD```, obtiene el número de elementos

- ```SSCAN```, itera por los elementos

Los casos de uso de los conjuntos a veces parecen un poco forzados, como si se hubieran implementado y luego buscado una utilidad para los mismos. En la práctica se pueden utilizar para realizar uniones, intersecciones y diferencias entre conjuntos. Así como para obtener elementos aleatorios, incluso con repetición.

## 4.4. Sorted Set

Los conjuntos ordenados en Redis son colecciones de elementos a los que se puede asociar un valor o puntuación (_score_). Dicho valor se utiliza para ordenar los elementos retornados por las operaciones de consulta sobre los mismos.

El caso de uso más habitual es la implementación de listas con _ranking_ (_leader boards_). Por ejemplo, para implementar una lista de jugadores con las mayores puntuaciones, u obtener la posición en el ranking de un jugador dado.

```text
> ZADD board 567 john 324 paul 107 ringo
(integer) 3

> ZSCORE board paul
"324"

> ZRANK board ringo
(integer) 0

> ZCOUNT board 100 500
(integer) 2

> ZRANGE board 0 -1
1) "ringo"
2) "paul"
3) "john"

> ZREVRANGE board 0 -1
1) "john"
2) "paul"
3) "ringo"
```

Comandos específicos para el tipo ```sorted set```:

- ```ZADD```, añade elementos con puntuación

- ```ZINCRBY```, incrementa la puntuación de un elemento

- ```ZSCORE```, obtiene la puntuación de un elemento

- ```ZRANK```, obtiene el _ranking_ de un elemento

- ```ZREVRANK```, obtiene el _ranking_ de un elemento de forma descendente

- ```ZPOPMIN```, obtiene y elimina elementos con la menor puntuación

- ```ZPOPMAX```, obtiene y elimina elementos con la mayor puntuación

- ```ZREM```, elimina elementos

- ```ZREMRANGEBYSCORE```, elimina elementos en un rango de puntuaciones dado

- ```ZREMRANGEBYRANK```, elimina elementos en un _ranking_ dado

- ```ZREMRANGEBYLEX```, elimina elementos en un rango lexicográfico dado

- ```ZRANGE```, obtiene elementos en un rango de puntuaciones dado

- ```ZREVRANGE```, obtiene elementos en un rango de puntuaciones dado ordenados de forma descendente

- ```ZRANGEBYSCORE```, obtiene elementos en un rango de puntuaciones dado ordenados por puntuación

- ```ZREVRANGEBYSCORE```, obtiene elementos en un rango de puntuaciones dado ordenados por puntuación de forma descendente

- ```ZRANGEBYLEX```, obtiene elementos en un rango de puntuaciones dado ordenados lexicográficamente

- ```ZREVRANGEBYLEX```, obtiene elementos en un rango de puntuaciones dado ordenados lexicográficamente de forma descendente

- ```ZUNIONSTORE```, obtiene la unión de varios conjuntos ordenados y la almacena en otro

- ```ZINTERSTORE```, obtiene la intersección de varios conjuntos ordenados y la almacena en otro

- ```ZCARD```, obtiene el número de elementos

- ```ZCOUNT```, obtiene el número de elementos en un rango de puntuaciones dado

- ```ZLEXCOUNT```, obtiene el número de elementos en un rango lexicográfico dado

- ```BZPOPMIN```, obtiene y elimina elementos con la menor puntuación de forma bloqueante

- ```BZPOPMAX```, obtiene y elimina elementos con la mayor puntuación de forma bloqueante

- ```ZSCAN```, itera por los elementos

En la práctica las puntuaciones se tratan ordenadas de menor a mayor, por lo que la mayoría de los comandos que recuperan rangos de elementos tienen un comando similar pero que recupera en orden inverso. En caso de dos elementos con la misma puntuación se retorna el resultado ordenado en base al orden lexicográfico de los elementos, que en Redis son siempre cadenas de texto. El orden está garantizado en la medida que un conjunto no puede contener dos veces una misma cadena.

El tipo ```sorted set``` es utilizado para casos de uso más elaborados, como por ejemplo para obtener productos similares a los consultados por un cliente en la _web_ de una tienda, normalmente en función de las compras realizadas por otros usuarios que también consultaron o compraron los mismos productos.

## 4.5. HyperLogLogs

Este tipo está orientado a obtener el número de elementos distintos de una colección. Su peculiaridad es que no almacena realmente los elementos, por lo que no requiere mucha memoria, y no retorna un valor exacto, sino una aproximación.

Su caso de uso habitual es obtener la cuenta de las distintas ocurrencias de una determinada entidad. Por ejemplo, para obtener el número de las distintas direcciones IP que visitan una página _web_.

```text
> pfadd hll a b c d
(integer) 1

> pfcount hll
(integer) 4
```

Comandos específicos del tipo ```HyperLogLogs```:

- ```PFADD```, añade un elemento

- ```PFCOUNT```, obtiene el número aproximado de elementos

- ```PFMERGE```, realiza la unión de varios ```HyperLogLogs``` y la almacena en otro

Para el caso de uso habitual comentado, contar las distintas IPs que visitan una _web_, lo ideal sería llevar una cuenta a base de guardar todas las direcciones, incrementando el contador en el caso de que la dirección no estuviera ya guardada. Lógicamente, si se hiciera así, los requerimientos de memoria y tiempo de proceso serían inviables. La solución de Redis es hacer un ```hash``` por cada elemento (IP) añadido al conjunto (```HyperLogLogs```) y contar los distintos ```hashes```. Esto elimina la necesidad de guardar todos los elementos y da una aproximación bastante buena de la cardinalidad (contador) del conjunto. De hecho, según la documentación, con un error de menos del 1%.

## 4.6. Streams

El tipo ```stream``` estará disponible a partir de la versión 5 de Redis. Su caso de uso más natural será el de modelar series temporales. Es decir, registrar información asociada a un determinado momento en el tiempo.

Es equivalente a un fichero de _log_, donde cada línea tiene un _timestamp_. Y aunque aún es pronto para afirmarlo, puede convertirse en el sistema de referencia para almacenar de forma local las medidas generadas por los sensores de dispositivos IoT (_Internet of Things_) antes de ser enviadas a la nube para su procesamiento.

```text
> XADD mystream * sensor-id 1234 temperature 19.8
1518951480106-0

> XADD mystream * sensor-id 9999 temperature 18.2
1518951482479-0

> XRANGE mystream - +
1) 1) 1518951480106-0
2) 1) "sensor-id"
2) "1234"
3) "temperature"
4) "19.8"
2) 1) 1518951482479-0
2) 1) "sensor-id"
2) "9999"
3) "temperature"
4) "18.2"
```

Comandos específicos del tipo ```stream```:

- ```XADD```, añade un elemento

- ```XRANGE```, obtiene elementos en un rango dado

- ```XDEVRANGE```, obtiene elementos en un rango dado en orden descendente

- ```XLEN```, obtiene el número de elementos

- ```XREAD```, obtiene elementos en un rango dado, opcionalmente de forma bloqueante

- ```XREADGROUP```, obtiene elementos en un rango para un grupo dado, opcionalmente de forma bloqueante

- ```XPENDING```, obtiene los elementos pendientes de un grupo

Los ```streams``` guardan elementos de igual forma que otras estructuras, pero su implementación está optimizada para su caso de uso más natural. Cuando se añade un elemento se genera automáticamente un ID que se compone de dos partes. La primera parte es un _timestamp_ y la segunda es un secuencial dentro de dicho _timestamp_. Opcionalmente se puede utilizar un ID propio obviando el generado automáticamente, pero no se espera que sea una opción muy utilizada.

Redis ofrece dos formas de consultar los datos almacenados en un ```stream```. La primera es mediante una consulta por rango, y la segunda es mediante un mecanismo de publicación/subscripción. Esta segunda forma se suma a los mecanismos ya existentes dentro de Redis, como los comandos bloqueantes para extraer elementos de listas, y los comandos más específicos como ```PUB/SUB``` para operar con canales. La novedad es que los clientes podrán suscribirse para obtener las entradas registradas a partir de un momento concreto en el tiempo. Con los tipos existentes anteriormente esto no era posible, cuando un cliente se desconectaba y volvía a conectarse no podía acceder a la información generada durante el periodo de desconexión.

En definitiva, un nuevo tipo añadido a Redis que aún está por venir y tiene más características por explotar, como los grupos de consumidores, que garantiza que cada mensaje es enviado a un único consumidor distinto dentro del grupo.
