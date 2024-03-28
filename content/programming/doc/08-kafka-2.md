# 8. Apache Kafka (2)

_21-09-2018_ _Juan Mellado_

Continuando con la serie dedicada a Apache Kafka, en este artículo se revisa el proceso de instalación y algunas de las tareas básicas que pueden realizarse para familiarizarse con el _software_. La documentación es bastante efectiva a este respecto, por lo que este artículo se limita a seguir paso por paso el guión propuesto en la misma. El objetivo es instalar y verificar el funcionamiento de Kakfa de forma local en una máquina Windows partiendo de cero y sin utilizar Docker.

## 8.1. Descarga

La versión actual de Kafka en el momento de escribir este artículo es la ```2.0.0```. Puede descargarse desde la propia página de descargas oficial del proyecto. El descargable es un fichero comprimido en formato ```.tgz``` que puede descomprimirse en cualquier directorio.

## 8.2. Instalación

El _software_ descomprimido se distribuye en cuatro directorios. El directorio ```bin``` contiene _scripts_ para ser ejecutados desde línea de comandos, tanto para Windows en formato ```.bat``` como para UNIX en formato ```.sh```. El directorio ```config``` contiene los ficheros de configuración en formato ```.properties```. El directorio ```libs``` contiene las dependencias en forma de librerías de Java en formato ```.jar```. Y el directorio ```site-docs``` contiene toda la documentación comprimida en formato ```.tgz```.

El único requerimiento para ejecutar Kafka es tener instalada una máquina virtual de Java 1.8 o superior. Con tenerla disponible en el ```PATH``` de la máquina es suficiente.

## 8.3. Arranque

Kafka utiliza Zookeeper, por lo que es necesario tener una instancia de este _software_ arrancada. El propio Kafka proporciona el ejecutable y un _script_ para arrancar una instancia preconfigurada con valores por defecto:

```text
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

Una vez arrancado Zookeeper se puede arrancar Kafka con otro _script_ y valores por defecto:

```text
bin\windows\kafka-server-start.bat config\server.properties
```

Si todo se ejecuta de forma correcta se mostrará en ambos casos el log de ejecución de los procesos por consola.

Cada fichero ```.properties``` pasado como parámetro en la línea de comandos contiene los valores de configuración utilizados por defecto para los servidores. Cada entrada de cada fichero está documentada con bastante detalle y en su conjunto resulta sencillo de entender. La documentación proporciona un listado exhaustivo con todos los parámetros de configuración disponibles.

Uno de los parámetros importantes dentro de la configuración es el que establece el directorio donde se almacena la información. Por defecto su valor es ```/tmp/kafka-logs```. Al arrancar el servidor se puede comprobar cómo se crea el directorio y ficheros dentro del mismo.

## 8.4. Tópico

Para crear tópicos se puede utilizar el script ```kafka-topics``` desde línea de comandos con el parámetro ```-create```. Por ejemplo, para crear un tópico de nombre ```prueba``` con una partición se puede ejecutar la siguiente línea:

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 
  --replication-factor 1 --partitions 1 --topic prueba
```

Si todo se ejecuta de forma correcta, se mostrará un mensaje confirmando la creación:

```text
Created topic "prueba".
```

Adicionalmente se puede comprobar con el mismo _script_ y el parámetro ```-list```, que lista los tópicos disponibles:

```text
bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
```

Tal y como indica la documentación, Kafka se puede configurar para crear los tópicos de forma automática si no existen en vez de tener que crearlos de forma manual.

Para los más curiosos del lugar, comentar que estos _scripts_ en realidad se limitan a invocar una clase Java que se encuentra dentro de ```libs\kafka_<versión de Scala>-<versión de Kafka>.jar```.

## 8.5. Publicación/Suscripción

Para publicar mensajes en un tópico se puede utilizar un _script_ que abre un cliente que captura la entrada de teclado y publica las líneas tecleadas como mensajes:

```text
bin\windows\kafka-console-producer.bat --broker-list localhost:9092
  --topic prueba

> uno
> dos
> tres
```

Para suscribirse a un tópico y recibir los mensajes publicados se puede ejecutar otro cliente que se queda a la espera y muestra por pantalla todos los mensajes recibidos:

```text
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092
  --topic prueba --from-beginning

uno
dos
tres
```

Si se ejecuta varias veces el último _script_ se puede comprobar que los mensajes son persistidos y que se pueden volver a procesar tantas veces como se quiera.

Como nota al margen, comentar que si se ejecutan los _scripts_ sin parámetros se muestra un texto de ayuda con todas las opciones disponibles.

## 8.6. Cluster

Un ejercicio un poco más sofisticado es configurar Kafka como un _clúster_. Para el ejemplo se utilizan tres servidores ejecutándose sobre una misma máquina de manera local.

El primer paso es crear un fichero ```.properties``` para cada servidor, tomando como punto de partida el fichero por defecto utilizado para arrancar el primer servidor.

```text
copy config\server.properties config\server-1.properties
copy config\server.properties config\server-2.properties
```

El segundo paso es modificar los ficheros para configurar un identificador, puertos y directorio distintos. Para ello hay que editarlos manualmente y sustituir las siguientes entradas:

- En ```config/server-1.properties```:

  ```properties
  broker.id=1
  listeners=PLAINTEXT://:9093
  log.dirs=/tmp/kafka-logs-1
  ```

- En ```config/server-2.properties```:

  ```properties
  broker.id=2
  listeners=PLAINTEXT://:9094
  log.dirs=/tmp/kafka-logs-2
  ```

Y el tercer paso es arrancar dos nuevos servidores utilizando los nuevos ficheros de configuración:

```text
bin\windows\kafka-server-start.bat config\server-1.properties
bin\windows\kafka-server-start.bat config\server-2.properties
```

A partir de aquí ya se puede empezar a probar el funcionamiento. Por ejemplo, creando un tópico con un factor de replicación de 3:

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181
  --topic replicado --partitions 1 --replication-factor 3
```

Comprobando que efectivamente está replicado:

```text
bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181
  --topic replicado

Topic:replicado PartitionCount:1 ReplicationFactor:3 Configs:
Topic: replicado Partition: 0 Leader: 1 Replicas: 1,2,0 Isr: 1,2,0
Publicando mensajes:

bin\windows\kafka-console-producer.bat --broker-list localhost:9092
  --topic replicado

>Mechor
>Gaspar
>Baltasar
^C
```

Matando el proceso del servidor líder:

```text
wmic process where "caption = 'java.exe' and
  commandline like '%server-1.properties%'" get processid

ProcessId
NNNN

taskkill /pid NNNN /f

Correcto: se terminó el proceso con PID NNNN.
```

Y comprobando que los mensajes siguen disponibles a través de un seguidor que ahora se comporta como líder:

```text
bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181
  --topic replicado

Topic:replicado PartitionCount:1 ReplicationFactor:3 Configs:
Topic: replicado Partition: 0 Leader: 2 Replicas: 1,2,0 Isr: 2,0

bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092
  --topic replicado --from-beginning

Melchor
Gaspar
Baltasar
^C
```

## 8.7. Connectors

El Connector API de Kafka permite importar y exportar datos desde diversas fuentes externas. Por ejemplo, desde un fichero de texto plano, o desde el _commit log_ de una base de datos relacional. Es decir, que se puede configurar Kafka para que monitorice un recurso y a medida que se añada información en él se replique en tiempo real en un tópico. O alimentar en tiempo real un recurso a partir de los mensajes publicados en un tópico.

Para el ejemplo demostrativo de esta funcionalidad se configura el _cluster_ del ejemplo anterior para monitorizar el contenido de un fichero de texto plano de nombre ```test.txt```, publicar su contenido a medida que se le añade información en un tópico de nombre ```connect-test```, y generar otro fichero de nombre ```test.sink.txt``` con la información publicada en el tópico.

El primer paso es crear el fichero de texto ```test.txt``` con un par de líneas en el directorio raíz de la instalación:

```text
Quijote
Sancho
```

El segundo paso es instalar los conectores, que son los componentes _software_ que implementan todo el proceso. Se crean dos. El primero para monitorizar el fichero de entrada, y el segundo para generar el fichero de salida:

```text
bin\windows\connect-standalone.bat config\connect-standalone.properties 

config\connect-file-source.properties config\connect-file-sink.properties
```

Y el tercer y último paso es comprobar que el fichero de salida contiene las líneas del fichero de entrada:

```text
more test.sink.txt

Quijote
Sancho
```

Si se abre el fichero de entrada y se añade otra línea, automáticamente se replicará en el fichero de salida. Opcionalmente se puede comprobar que cada línea es publicada como un mensaje en el tópico:

```text
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092
  --topic connect-test --from-beginning

{"schema":{"type":"string","optional":false},"payload":"Quijote"}
{"schema":{"type":"string","optional":false},"payload":"Sancho"}
```

Notar que disponer de los mensajes publicados en un tópico permite además que puedan ser consumidos por más procesos.
