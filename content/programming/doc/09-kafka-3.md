# 9. Apache Kafka (3)

_27-09-2018_ _Juan Mellado_

En este artículo se muestran un par de ejemplos de uso de la librería cliente para Java de Apache Kafka. El objetivo es crear clientes básicos que actúen como productores o consumidores, que son dos de los casos de uso más habituales. Los ejemplos de la documentación oficial proporcionan un punto de partida bastante sólido, por lo que este artículo sigue el guión propuesto en los mismos.

Para ejecutar los ejemplos se debe tener levantado el _cluster_ creado en el artículo anterior con un tópico de nombre ```prueba```.

## 9.1. Maven

La librería cliente de Kafka está distribuida en Maven bajo las siguientes coordernadas:

```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>2.0.0</version>
</dependency>
```

## 9.2. Producer API

Este API permite crear clientes que publiquen registros.

```java
import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class Productor {

  public static void main(String[] args) {
    var props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("acks", "all");
    props.put("retries", 0);
    props.put("batch.size", 16384);
    props.put("buffer.memory", 33554432);
    props.put("linger.ms", 1);
    props.put("key.serializer",
      "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer",
      "org.apache.kafka.common.serialization.StringSerializer");

    var producer = new KafkaProducer<String, String>(props);

    producer.send(new ProducerRecord<>("prueba", "Sherlock", "Holmes"));
    producer.send(new ProducerRecord<>("prueba", "Hercule", "Poirot"));

    producer.close();
  }
}
```

El código de ejemplo anterior crea un productor que genera dos registros que publica en el tópico ```prueba```. Cada registro compuesto de una clave y valor, ambos de tipo ```String```.

La clase principal es ```KafkaProducer```. Una clase _thread safe_ reutilizable que admite una configuración con una gran cantidad de opciones, por lo que el método preferido para instanciarla es pasarle el clásico objeto de tipo ```Properties``` de Java en el constructor con todos los parámetros.

El productor funciona de forma asíncrona no bloqueante. La llamada al método ```send``` no envía los registros al servidor de forma inmediata, los almacena de forma local. Los registros se envían al servidor en un _thread_ aparte en función de la configuración. Esta estrategia permite reducir el tráfico de red, enviando varios registros en un solo paquete, y actuando acorde al caso de uso habitual de los productores, que notifican eventos de forma masiva al servidor. No obstante, un punto importante de esta forma de funcionamiento es recordar llamar al método ```close```, o ```flush```, para garantizar que todos los registros almacenados de forma local se envían al servidor antes de terminar la conexión.

El método ```send``` retorna un ```Future``` que se completa con información acerca del envío. En dicho ```Future``` se incluye el tópico, el _timestamp_ y el _offset_ asignado al mensaje. Si se quiere esperar de forma síncrona bloqueante a que se complete el envío se puede llamar al métogo ```get``` del ```Future```. Otra opción es pasar una función _callback_ como parámetro al método ```send```. En este último caso el envío será síncrono y la función _callback_ se invocará cuando se complete el envío.

En cuanto a los parámetros de configuración, en el ejemplo se utilizan algunos de ellos. ```acks``` establece si el productor necesita que se confirme la escritura efectiva del registro en el tópico, el valor ```0``` indica que no se requiere ninguna confirmación, el valor ```1``` indica que se debe garantizar que el _broker_ líder del _cluster_ lo persiste, y ```all``` indica que se debe garantizar la escritura efectiva por todos los _brokers_ del cluster. ```retries``` indica el número de reintentos a realizar en caso de pérdida de conexión, se utiliza cero porque un número mayor puede dar lugar a registros duplicados en determinados escenarios como se verá más adelante. ```batch.size``` indica el número máximo de registros que se pueden acumular en local antes de su envío para cada partición, a más registros más memoria. ```buffer.memory``` indica el tamaño máximo de memoria que se debe utilizar para almacenar los registros de manera local, si se alcanza dicho máximo el método ```send``` se bloqueará hasta que vuelva a haber memoria disponible. ```linger.ms``` establece un tiempo de espera para enviar registros al servidor, lo que permite que se almacenen registros de forma local para enviarlos todos en un solo paquete de forma más eficiente. ```key.serializer``` y ```value.serializer``` indican las clases que deben utilizarse para serializar los valores de las claves y los valores en los registros.

Como se ha comentado anteriormente, en escenarios concretos de error o sobrecarga en el tráfico de red puede ocurrir que se dupliquen registros si se establece un número de reintentos mayor de cero. Para evitar este efecto no deseado, el productor puede configurarse estableciendo el parámetro ```enable.idempotence``` a ```true``` para trabajar en modo idempotente, un modo que garantiza que los registros no se duplican. Pero tampoco es la panacea, la garantía sólo se establece para una conexión concreta de un productor concreto. El funcionamiento de este modo se basa en la generación de un ID único para cada productor y un secuencial con el número de registros publicados. El servidor puede ignorar mensajes con secuenciales menores al último persistido con éxito, evitando así que se vuelva a publicar un mensaje ya publicado antes de una pérdida de conexión.

El productor también puede utilizarse para generar eventos dentro de una transacción, garantizando que se escriben todos, o ninguno, de los registros publicados dentro de los límites de la transacción. Aunque este modo de funcionamiento establece algunas restricciones sobre la configuración de los tópicos, que deben tener un factor de replicación de tres o más, y exigen un mínimo de dos réplicas sincronizadas. Y de igual forma exige que los consumidores se configuren para consumir exclusivamente registros completamente persistidos y sincronizados en el _cluster_. El API para este modo es síncrono y eleva excepciones en caso de error.

```java
import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.KafkaException;
import org.apache.kafka.common.errors.AuthorizationException;
import org.apache.kafka.common.errors.OutOfOrderSequenceException;
import org.apache.kafka.common.errors.ProducerFencedException;

public class ProductorTransaccional {

  public static void main(String[] args) {
    var props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("transactional.id", "my-transactional-id");
    props.put("key.serializer",
      "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer",
      "org.apache.kafka.common.serialization.StringSerializer");

    var producer = new KafkaProducer<String, String>(props);

    producer.initTransactions();

    try {
      producer.beginTransaction();

      producer.send(new ProducerRecord<>("prueba", "Miss", "Marple"));
      producer.send(new ProducerRecord<>("prueba", "Philip", "Marlowe"));

      producer.commitTransaction();
    } catch (ProducerFencedException | 
             OutOfOrderSequenceException | 
             AuthorizationException e) {
      producer.close();
    } catch (KafkaException e) {
      producer.abortTransaction();
    }

    producer.close();
  }
}
```

En el código de ejemplo anterior se crea un productor que publica dos registros dentro de una transacción. El patrón que sigue es el uso de los clásicos métodos ```begin``` y ```commit``` para delimitar la transacción. El método ```initTransactions``` garantiza que no se inicia una transacción nueva sin haber terminado la anterior. Y el método ```abortTransaction``` se utiliza para abortar la transacción en curso en caso de error.

## 9.3. Consumer API

Este API permite crear clientes que consuman registros.

```java
import java.time.Duration;
import java.util.List;
import java.util.Properties;

import org.apache.kafka.clients.consumer.KafkaConsumer;

public class Consumidor {

  public static void main(String[] args) {
    var props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("group.id", "grupo-prueba");
    props.put("enable.auto.commit", "true");
    props.put("auto.commit.interval.ms", "1000");
    props.put("key.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer");

    var consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Collections.singletonList("prueba"));

    while (true) {
      var records = consumer.poll(Duration.ofMillis(1000));

      for (var record : records) {
        System.out.printf("offset = %d, key = %s, value = %s%n", 
          record.offset(), record.key(), record.value());
      }
    }
  }
}
```

El código de ejemplo anterior crea un consumidor que consume los registros publicados en el tópico ```prueba```. La clase principal es ```KafkaConsumer```, que a diferencia de la clase del productor, NO es _thread safe_.

Un aspecto importante a tener en cuenta para la programación de consumidores es la noción de _offset_. Como ya se explicó en artículos anteriores, un _offset_ es un secuencial que indica la posición de un mensaje dentro de un tópico. Los consumidores mantienen dos _offsets_, por una parte el _offset_ del próximo mensaje que esperan recibir, y por otra parte el _offset_ del último mensaje consumido de forma efectiva. Es decir, que el cliente puede decidir e indicar al servidor el _offset_ del último mensaje que considera consumido. Más concretamente, llamando de forma manual al método ```commitSync``` o ```commitAsync```. En caso de caída del consumidor, el servidor volverá a enviarle mensajes, cuando se vuelva a levantar, a partir del último _offset_ confirmado.

Otro aspecto importante, también comentado en artículos anteriores, es el concepto de grupo de consumidores. Cada consumidor dentro de un grupo es asignado por el servidor a una partición de un tópico. Es decir, que los mensajes de un tópico son consumidos por un único consumidor. Si el consumidor cae, el servidor asignará automáticamente la partición a otro consumidor del grupo de consumidores. Si todos los consumidores de un mismo tópico se adhieren al mismo grupo de consumidores entonces Kakfa se comporta como una cola, de igual forma que la mayoría de sistemas de mensajería, donde un mensaje es consumido por un único cliente. Si cada consumidor de un mismo tópico tiene un identificador de grupo de consumidores distinto entonces Kakfa se comporta como un mecanismo de publicación/suscripción, donde cada mensaje es consumido por todos los clientes.

La clase ```KafkaConsumer``` permite asignar un consumidor a una partición concreta, recibir notificaciones cuando se produzca un cambio en la asignación de particiones, y realizar otras tareas relacionadas con la gestión de grupos de consumidores.

Respecto a los parámetros de configuración, estos se establecen a través de una instancia de la clase ```Properties``` como en la clase del productor. Los parámetros más relevantes en el ejemplo son ```group.id```, que contiene el nombre del grupo de consumidores, junto con ```enable.auto.commit``` y ```auto.commit.interval.ms```, que indican que el consumidor quiere que se comunique al servidor cada N milisegundos que los mensajes recibidos se consideran completamente consumidos de forma automática, sin que el consumidor tenga que llamar manualmente a ```commitSync``` o ```commitAsync```.

El método ```poll``` consume mensajes del servidor. Si hay mensajes disponibles retorna inmediatamente dichos mensajes. Si no hay mensajes disponibles se bloquea el tiempo indicado. Si llegan mensajes durante la espera se desbloquea y retorna los mensajes recibidos. Si no llegan mensajes durante la espera retorna una lista vacía al expirar el tiempo indicado. Opcionalmente se puede indicar a través de la propiedad ```max.poll.records``` el número máximo de mensajes que el cliente quiere procesar de una sola vez. A diferencia de la mayoría de plataformas, donde el servidor tiene la responsabilidad de controlar todo lo que hacen sus clientes, Kafka utiliza una estrategia distinta, con clientes que implementan bastante lógica de control.

```java
import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;

public class ConsumidorCommit {

  public static void main(String[] args) {
    var props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("group.id", "grupo-prueba");
    props.put("enable.auto.commit", "false");
    props.put("key.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer");

    var consumer = new KafkaConsumer<String, String>(props);
    consumer.subscribe(Collections.singletonList("prueba"));

    while (true) {
      var records = consumer.poll(Duration.ofMillis(1000));

      for (var partition : records.partitions()) {
        var partitionRecords = records.records(partition);

        for (var record : partitionRecords) {
          System.out.println(record.offset() + ": " + record.value());
        }

        var lastOffset = partitionRecords.get(partitionRecords.size() - 1)
          .offset();
        consumer.commitSync(Collections.singletonMap(partition, 
          new OffsetAndMetadata(lastOffset + 1)));
      }
    }
  }
}
```

En el código de ejemplo anterior se crea un consumidor que consume mensajes y decide de forma manual cuando han sido efectivamente consumidos llamando a ```commitSync``` con el _offset_ del último mensaje consumido, o más precisamente, con el _offset_ del siguiente mensaje, ya que el método requiere sumar 1 al último _offset_ procesado. En cualquier caso, lo que hay que tener claro es que nunca se puede garantizar un 100% de consistencia. Por ejemplo, el proceso podría caer antes de llamar a ```commitSync```. En cuyo caso algunos mensajes se habrían procesado, pero no le llegaría la notificación a Kafka. Al volver a arrancar el consumidor, Kafka volvería a enviar los mismos últimos mensajes no confirmados por el consumidor.

Por último, comentar que la clase ```KafkaConsumer``` ofrece métodos como ```seek```, ```seekToBeginning``` y ```seekToEnd``` para dejar que el consumidor especifique de forma precisa la posición a partir de la cual quiere consumir mensajes, permitiendo por ejemplo volver a consumir mensajes ya procesados, o ignorar mensajes y sólo consumir los nuevos.
