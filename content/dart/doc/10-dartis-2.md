# 10. dartis (y 2)

_13-09-2018_ _Juan Mellado_

En este artículo se continúa la revisión de las características ofrecidas por dartis, un cliente para Redis escrito en Dart que publiqué hace unos días.

## 10.1. Pub/Sub

Redis permite que los clientes entren en un modo de funcionamiento especial en el que se implementa el patrón _Publish/Subscribe_. En este modo los clientes pueden suscribirse a canales y recibir mensajes a través de ellos. Es decir, que Redis se comporta como un sistema de mensajería. En este modo los clientes sólo pueden ejecutar los comandos ```SUBSCRIBE```, ```UNSUBSCRIBE```, ```PSUBSCRIBE```, ```PUNSUBSCRIBE```, ```PING``` y ```QUIT```.

El servidor envía, tanto las respuestas a las suscripciones a canales, como los propios mensajes publicados en dichos canales, en un _stream_ de eventos con una estructura de mensajes distinta a la que se recibe cuando el cliente se encuentra operando en el modo normal, por lo que se debe conocer en todo momento en que modo se encuentra trabajando el cliente para poder procesar correctamente las respuestas recibidas del servidor.

La conexión a este modo de funcionamiento se realiza de forma explícita en dartis a través de una factoría estática. Y los eventos recibidos se publican a través de un ```Stream```.

```dart
final pubsub = await PubSub.connect<String, String>('redis://localhost:6379');

pubsub.subscribe(channel: 'dev.dart');

pubsub.stream.listen(print, onError: print);
```

Tener una clase especializada para implementar este modo permite reducir la complejidad del código. Máxime cuando Redis sólo permite a los clientes en este modo ejecutar un conjunto de comandos muy concretos y salir terminando la conexión con el servidor. Aunque impide que el cliente ejecute otros comandos antes de entrar en este modo, como por ejemplo validarse contra el servidor utilizando una clave.

## 10.2. Monitor

Redis permite también que los clientes trabajen en modo monitor. En este modo no pueden ejecutar ningún comando, se limitan a recibir mensajes del servidor. Los mensajes recibidos contienen un texto con detalles acerca de todos los comandos ejecutados por todos los clientes contra el servidor. Es decir, que cada vez que un cliente ejecuta un comando, el servidor envía un mensaje informando de tal acción a todos los clientes que se encuentran en modo monitor. Esto resulta de utilidad para depurar aplicaciones de forma no intrusiva.

La conexión a este modo de funcionamiento se realiza de forma explícita en dartis a través de una factoría estática. Y los eventos recibidos se publican a través de un ```Stream```.

```dart
final monitor = await Monitor.connect('redis://localhost:6379');

monitor.start();

monitor.stream.listen(print);
```

De igual forma que en el modo del apartado anterior, disponer de una clase especializada reduce la complejidad de la implementación, pero no permite ejecutar ningún comando previo a la entrada en este modo.

## 10.3. Inline Commands

Redis permite la ejecución de comandos sin tener que implementar el protocolo RESP. En este modo los comandos se envían como cadenas de texto plano al servidor. Es similar a como se haría a través de una sesión Telnet, escribiendo directamente en una consola de texto.

```dart
final terminal = await Terminal.connect('redis://localhost:6379');

terminal.run('PING\r\n'.codeUnits);

terminal.stream.listen(print);
```

Las respuestas recibidas desde el servidor en este modo están formateadas siguiendo el protocolo RESP, por lo que este modo es adecuado para comprobar la respuesta exacta del servidor ante un determinado comando, evitando cualquier tipo de agente intermedio que la procese o altere de alguna forma.

## 10.4. Transacciones

Redis permite ejecutar varios comandos dentro del contexto de una transacción. Aunque no es una acción de "todo o nada" como habitualmente se sobreentiende que funciona una transacción con la mayoría de software existente.

Una transacción empieza en Redis ejecutando el comando ```MULTI``` y termina ejecutando el comando ```EXEC```. Todos los comandos que se ejecutan después de ```MULTI``` se encolan y se ejecutan cuando se llama a ```EXEC```. Si se produce un error encolando un comando la transacción se aborta en el momento que se llama a ```EXEC```, no en el momento que se produce el error encolando el comando. Y si se produce un error durante la ejecución de un comando encolado se continúa con el siguiente comando encolado, no se produce un _rollback_ de lo ejecutado hasta el momento.

```EXEC``` retorna el resultado de la ejecución de todos los comandos ejecutados en el contexto de la transacción, tanto los terminados con éxito como los terminados con error, para dar la oportunidad al cliente de actuar en función del resultado individual de cada comando.

```dart
await commands.multi();

commands.set(key, 1).then(print);
commands.incr(key).then(print);

await commands.exec();
```

Con este patrón no se puede utilizar ```await``` sobre cada comando, ya que el ```Future``` de cada comando no se completa hasta que se obtiene la respuesta del servidor, algo que no ocurre hasta ejecutar ```EXEC```.

Una transacción iniciada con ```MULTI``` puede abortarse ejecutando el command ```DISCARD``` antes de llamar a ```EXEC```. El comando ```DISCARD``` termina la transacción en curso descartando todos los comandos encolados hasta el momento.

Por último, comentar que en el contexto de una transacción no se debe utilizar el comando ```CLIENT REPLY```, ya que en determinados casos el cliente puede perder la sincronía con el servidor. Es por ello que las transacciones en Redis están documentadas como una funcionalidad a ser deprecada, prefiriéndose el uso de _scripts_ en Lua en el servidor.

## 10.5. Scripts Lua

Redis permite ejecutar _scripts_ en Lua en el servidor. Esta característica es ofrecida a través de comandos, por lo que no requiere ninguna implementación especial por parte del cliente.

```dart
await commands.eval<void>(
  'return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}',
  keys: [key1, key2],
  args: ['first', 'second']);
```

La única característica particular a tener en cuenta con esta funcionalidad es que la salida de la ejecución de un script puede ser de cualquier tipo. Lo mismo puede resultar en una primitiva, como una cadena de texto o un entero, o un _array_ de valores heterogéneos. Para lidiar con esta situación, dartis admite un parámetro que permite mapear la respuesta del servidor en cualquier tipo que se quiera.

```dart
class CustomMapper implements Mapper<List<String>> {
  @override
  List<String> map(Reply reply, RedisCodec codec) =>
    codec.decode<List<String>>(reply);
  }

...

final results = await commands.eval<List<String>>(
  'return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}',
  keys: [key1, key2],
  args: ['first', 'second'],
  mapper: CustomMapper());

print(results); // ['key1', 'key2', 'first', 'second']
```

En el ejemplo del código anterior, el _mapper_ convierte la respuesta del servidor en una lista de cadenas de texto utilizando las facilidades que ofrece dartis para crear serializadores y deserializadores personalizados.

## 10.6. Serializadores / Deserializadores

dartis tiene un sistema de conversores dinámico que permite añadir conversores nuevos o sobreescribir los existentes. Un conversor es una clase que convierte una primitiva en una lista de bytes de cara a ser enviado al servidor, y convierte una lista de respuestas del servidor o bytes en una primitiva de cara a que un cliente pueda utilizarla en una aplicación.

Por defecto hay registrados conversores para los tipos de uso más habitual como enteros, decimales, cadenas de texto y listas. Utilizando UTF-8 por defecto para las cadenas de texto.

Los conversores que transforman primitivas en listas de _bytes_ se denominan codificadores, y son clases que se crean extendiendo de ```Converter``` o ```Encoder```. Por ejemplo, el siguiente código muestra un conversor que transforma instancias del tipo DateTime de Dart en listas de _bytes_.

```dart
class DateTimeEncoder extends Encoder<DateTime> {
  @override
    List<int> convert(DateTime value, [RedisCodec codec]) =>
    utf8.encode(value.toString());
  }
}
```

Por su parte, los conversores que transforman listas de respuestas del servidor o _bytes_ en una primitiva se denominan decodificadores, y son clases que se crean extendiendo de ```Converter``` o ```Decoder```. Por ejemplo, el siguiente código muestra un conversor que transforma una respuesta del servidor en una instancia del tipo ```DateTime``` de Dart.

```dart
class DateTimeDecoder extends Decoder<SingleReply, DateTime> {
@override
DateTime convert(SingleReply value, RedisCodec codec) =>
value.bytes == null ? null : DateTime.parse(utf8.decode(value.bytes));
}
```

Todos los codificadores se registran utilizando el atributo ```codec``` del cliente.

```dart
client.codec.register(
  encoder: DateTimeEncoder(),
  decoder: DateTimeDecoder());
```

Permitir definir tipos personalizados facilita trabajar de forma sencilla con los comandos propios que pueda definir cualquier módulo para Redis. Siendo un módulo una extensión que se puede añadir a un servidor Redis, a modo de _plugin_, y que dartis soporta de manera natural.

Y estas son básicamente las principales características de la librería. En el propio repositorio del proyecto puede encontrarse más documentación y ejemplos de uso.

Para terminar, comentar que el proyecto está desarrollado utilizando Visual Studio Code como IDE con tan sólo un par de extensiones. EditorConfig para garantizar la uniformidad del formato de los ficheros, y Dart Code para la integración de las herramientas del SDK, en particular el formateador de código y el analizador de código estático. Travis como servidor de integración continua y Coveralls para el análisis de la cobertura del código. El proyecto tiene implementadas unos trescientos casos de prueba, incluyendo tanto pruebas unitarias como de integración, con un noventa por ciento de cobertura.
