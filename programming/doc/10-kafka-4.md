# 10. Apache Kafka (y 4)

_02-10-2018_ _Juan Mellado_

En este artículo se muestra un ejemplo de construcción de un cliente Java para Apache Kafka que utiliza _Kafka Streams_. Al igual que en artículos anteriores de esta serie, la documentación oficial representa un buen punto de partida, por lo que sigue el guión propuesto en la misma.

## 10.1. Preparación

Para la ejecución de este ejemplo se presupone que se encuentra arrancado un servidor local de Kafka con la configuración por defecto según lo explicado en artículos anteriores:

```text
bin\windows\zookeeper-server-start.bat config\zookeeper.properties

bin\windows\kafka-server-start.bat config\server.properties
```

El ejemplo leerá de un tópico de entrada y escribirá en un tópico de salida. Más concretamente leerá de un stream de entrada líneas de texto plano y escribirá en un stream de salida las distintas palabras utilizadas junto con el número total de ocurrencias de cada una de ellas.

El tópico de entrada se llamará ```streams-plaintext-input``` y puede crearse desde línea de comandos con la siguiente instrucción:

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 
  --replication-factor 1 --partitions 1 --topic streams-plaintext-input
```

El tópico de salida se llamará ```streams-wordcount-output``` y puede crearse desde línea de comandos con la siguiente instrucción:

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181
  --replication-factor 1 --partitions 1 --topic streams-wordcount-output
  --config cleanup.policy=compact
```

Lo único reseñable en este paso es que el segundo tópico se crea con una política de compactación. Lo que quiere decir que Kafka garantiza que el último valor asignado a cada clave estará disponible, pero no necesariamente el histórico de todos los valores asignados a cada clave. Entendiendo que en este caso concreto que las claves serán las distintas palabras encontradas en el texto, y los valores el número de ocurrencias de las mismas.

## 10.2. Maven

El API de _Kafka Streams_ está publicado en Maven bajo las siguientes coordenadas:

```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-streams</artifactId>
  <version>2.0.0</version>
</dependency>
```

## 10.2. Kafka Streams

El API de _Kafka Streams_ permite crear clientes que procesen _streams_.

```java
import java.util.Arrays;
import java.util.Locale;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.state.KeyValueStore;

public class WordCountDemo {

  public static void main(String[] args) throws Exception {
    // Define la propiedades de conexión
    var props = new Properties();
    props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-pipe");
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG,
      Serdes.String().getClass());
    props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG,
      Serdes.String().getClass());

    // Conecta al tópico de entrada
    final StreamsBuilder builder = new StreamsBuilder();
    KStream<String, String> source = builder.stream("streams-plaintext-input");

    // Define el proceso para agrupar la entrada y alimentar la salida
    source.flatMapValues(
      value -> Arrays.asList(
        value.toLowerCase(Locale.getDefault()).split("\\W+")))

      .groupBy((key, value) -> value)

      .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as(
        "counts-store")).toStream()

      .to("streams-wordcount-output", 
        Produced.with(Serdes.String(), Serdes.Long()));

    final var topology = builder.build();
    final var streams = new KafkaStreams(topology, props);

    // Espera Ctrl+C para terminar en orden
    final var latch = new CountDownLatch(1);

    Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
      @Override
      public void run() {
        streams.close();
        latch.countDown();
      }
    });

    // Inicia el proceso
    try {
      streams.start();
      latch.await();
    } catch (Throwable e) {
      System.exit(1);
    }

    System.exit(0);
  }
}
```

El código del ejemplo, un poco más extenso de lo habitual, define las propiedades de la conexión, se conecta al tópico de entrada, define la operación a realizar para alimentar el tópico de salida, arranca el proceso, y se queda a la espera hasta que se pulsa ```Ctrl+C``` para terminar en orden. Lo interesante es notar que se utiliza el estilo del API nativo de Java para la manipulación de ```Streams```. El resto del API de Kafka apenas consiste en llamar a los métodos ```start``` y ```close```.

Este ejemplo se incluye dentro de la propia distribución de Kafka, así que puede arrancarse directamente desde línea de comandos con la siguiente instrucción:

```text
bin\windows\kafka-run-class.bat
  org.apache.kafka.streams.examples.wordcount.WordCountDemo
```

La aplicación se queda a la espera de que se alimente el tópico de entrada, algo que puede hacerse creando un productor con la siguiente línea de comandos:

```text
bin\windows\kafka-console-producer.bat --broker-list localhost:9092
  --topic streams-plaintext-input
```

El productor se queda esperando a que se introduzca algún texto por teclado, de forma que todas las líneas introducidas por teclado se envían al tópico ```streams-plaintext-input```.

El último paso es crear un consumidor que examine el tópico de salida, algo que puede hacerse con la siguiente línea de comandos:

```text
bin\windows\kafka-console-consumer.bat
  --bootstrap-server localhost:9092
  --topic streams-wordcount-output
  --from-beginning
  --formatter kafka.tools.DefaultMessageFormatter
  --property print.key=true
  --property print.value=true
  --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
  --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

El consumidor muestra por pantalla el contenido del tópico ```streams-wordcount-output``` a medida que se va almacenando información en el mismo.

Si todo ha ido correctamente, cuando se introduzca algún texto en el _stream_ de entrada, deberá verse el conteo de palabras en el _stream_ de salida actualizado.
