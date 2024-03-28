# 9. dartis (1)

_10-09-2018_ _Juan Mellado_

Hace un par de días liberé dartis, un cliente para Redis escrito en Dart. Empecé a desarrollarlo para probar algunas de las novedades de la versión 2 de Dart y al final he acabado con un cliente sencillo pero bastante completo.

- [https://github.com/jcmellado/dartis](https://github.com/jcmellado/dartis)

## 9.1. RESP

_REdis Serialization Protocol_ (RESP) es el protocolo que implementa Redis para comunicar los clientes con el servidor. Es un protocolo muy básico sobre TCP, con tan sólo cinco tipos de paquetes distintos. Los mensajes son todos de tipo texto, con un primer carácter identificador del tipo de paquete, seguido de una longitud en algunos casos, a continuación el contenido del mensaje en sí mismo, y un par de caracteres de terminación.

El hecho de que sea un protocolo de tipo texto tiene la ventaja de que resulta fácil de implementar y depurar, pero tiene el inconveniente de que hay que convertir entre binario y texto continuamente, en particular las longitudes de los mensajes. En principio he optado por utilizar las facilidades del propio lenguaje para las conversiones, sin realizar ninguna medición del rendimiento, pero podría estudiarse la posibilidad de hacer una implementación más específica.

Por otra parte, debido a que los paquetes no contienen ningún tipo de identificador, es necesario guardar la secuencia de cada comando enviado de forma ordenada y casarla contra las respuestas recibidas en el mismo orden. Con el hándicap de que es posible deshabilitar las respuestas del servidor, por lo que en algunos casos los paquetes han de enviarse sin esperar respuesta. En determinadas circunstancias muy concretas el protocolo no impide que se produzcan pérdidas de sincronía entre cliente y servidor.

## 9.2. Conexión

La conexión al servidor se realiza siguiendo el típico patrón consistente en utilizar un método estático en la clase del cliente a modo de factoría.

```dart
final client = await Client.connect('redis://localhost:6379');
...
await client.disconnect();
```

Como parámetro se admite una cadena de conexión siguiendo el formato clásico de protocolo, _host_ y puerto. Lo que posibilita una configuración muy sencilla y ampliable.

Redis permite trabajar con varias bases de datos dentro de una misma instancia, pero esta forma de trabajo no está totalmente soportada cuando se utiliza una instalación de Redis en _cluster_, por lo que es mejor no utilizar esta característica si no es absolutamente necesaria. Resulta más sencillo levantar más instancias.

Redis permite proteger de forma opcional con una clave los accesos al servidor, pero normalmente las instancias de Redis no se encuentran abiertas al mundo exterior, sino como dentro de una DMZ, por lo que no suele utilizarse, aunque ello depende en gran medida de las políticas de seguridad de cada cual.

Algunos puntos abiertos en la implementación de las conexiones con el servidor son la posibilidad de indicar un _timeout_, realizar reconexiones automáticas en caso de pérdida de conexión, y un _pool_ de conexiones.

## 9.3. Comandos

Un cliente le comunica a un servidor Redis lo que quiere hacer a través de comandos. Existen más de 200 comandos distintos. dartis implementa una interface genérica que permite ejecutar cualquier tipo de comando, a la vez que ofrece una interface más específica que expone todos los comandos ofrecidos por Redis de forma fuertemente tipada.

dartis funciona de forma totalmente asíncrona, por lo que todos los comandos retornan un ```Future``` que se completa cuando el servidor retorna el resultado.

Además, el conjunto de comandos está expuesto utilizando _Generics_, lo que permite que cada aplicación que utilice la librería pueda decidir el tipo concreto de datos que quiere utilizar, en vez de limitarse a utilizar ```String``` como hacen otras librerías.

```dart
final commands = client.asCommands<String, String>();

await commands.set(‘key, ‘value’);
final value = await commands.get(‘key’);

print(value);
```

Redis almacena secuencias de _bytes_, no cadenas de textos, por lo que puede utilizarse para gestionar cualquier tipo de información.

```dart
final commands = client.asCommands<String, List<int>();

await commands.set(‘key’, <int>[1, 2, 3]);
```

Los comandos están expuestos en forma de vista sobre el cliente, es decir, es la misma instancia pero exponiendo sólo una interface sobre la misma, por lo que pueden obtenerse distintas vistas del mismo cliente para trabajar con distintos tipos de datos.

```dart
final strings = client.asCommands<String, String>();
final bytes = client.asCommands<String, List<int>>();

String title = await strings.get('book:24902:title');
List<int> cover = await bytes.get('book:24902:cover');
```

Es seguro trabajar con varias vistas a un mismo tiempo, ya que todas utilizan el mismo cliente.

## 9.4. Pipelining

Redis permite enviar al servidor una serie de comandos a un mismo tiempo, en vez de uno a uno. Aunque en la práctica esta es más una característica propia de las comunicaciones que de Redis. El cliente puede enviar más de un comando dentro de un mismo paquete TCP. Y el servidor puede procesar paquetes TCP que contengan más de un comando.

Por defecto dartis sólo envía un comando en cada paquete TCP, deshabilitando de forma explícita el algoritmo de _Nagle_ sobre el _socket_, para garantizar que cada comando se envía de forma inmediata al servidor, pero permite activar el _pipelining_ llamando al método _pipeline_:

```dart
client.pipeline();

commands.incr('product:9238:views').then(print);
commands.incr('product:1725:views').then(print);
commands.incr('product:4560:views').then(print);

client.flush();
```

En buena lógica, con este patrón no se puede utilizar ```await``` sobre cada comando, ya que el ```Future``` de cada comando no se completa hasta que se obtiene respuesta del servidor, y los comandos no se envían al servidor hasta que se vacía el _pipeline_ con la llamada al método ```flush```.

Otra opción es utilizar la lista de ```Future```s retornada por el método ```flush``` para esperar hasta que se completen todos los comandos.

```dart
client.pipeline();

commands
..incr('product:9238:views')
..incr('product:1725:views')
..incr('product:4560:views');

final futures = client.flush();

await Future.wait<Object>(futures).then(print);
```

Esta forma de trabajo se utiliza sobre todo para realizar cargas masivas de información, o para clientes con casos de uso concretos que requieren un mayor rendimiento.

## 9.5. Fire And Forget

Redis permite que los clientes indiquen al servidor que no quieren recibir respuestas del servidor. Es decir, se pueden enviar comandos para su ejecución y desentenderse del resultado (_fire and forget_).

Esta es una característica que Redis ofrece a través del comando ```CLIENT REPLY```. El comando necesita un parámetro que admite el valor ```OFF``` para deshabilitar todas las respuestas, ```SKIP``` para deshabilitar sólo la respuesta del comando siguiente, y ```ON``` para habilitar todas las respuestas.

En este modo los comandos se completan de forma inmediata con el valor ```null``` cuando las respuestas del servidor están deshabilitadas.

```dart
await commands.clientReply(ReplyMode.off);

await commands.ping().then(print); // null
await commands.ping().then(print); // null
await commands.ping().then(print); // null
```

La implementación de esta característica no es compleja, pero requiere que el cliente detecte de manera explícita que se está ejecutando el comando ```CLIENT REPLY```, y llevar la cuenta de para que comandos se debe esperar respuesta y para cuáles no. Si el propio comando ```CLIENT REPLY``` es el que falla entonces puede llegar a perderse la sincronía entre servidor y cliente, aunque se presupone una casuística con una probabilidad muy baja de que suceda.
