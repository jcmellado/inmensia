# 12. Reactive Streams en Java (1)

_15-05-2018_ _Juan Mellado_

Java 9 introdujo la clase ```java.util.concurrent.Flow``` en un intento de estandarizar de forma nativa la implementación de _reactive streams_. Una propuesta de diseño que propugna la creación de _streams_ asíncronos y no bloqueantes, prestando especial atención a que la fuente origen de la información, potencialmente infinita, no sature al destinatario.

Los _reactive streams_ asemejan al patrón ```Observer```, con un modelo de publicadores y suscriptores. Los publicadores generan una secuencia de elementos que envían a los suscriptores, pero teniendo además estos últimos la capacidad de limitar el número de elementos que quieren recibir.

Java ha añadido unas interfaces con el mínimo número de métodos necesarios para implementar _reactive streams_, pero no una vasta colección de implementaciones de dichas interfaces. Lo que es importante, es que con la creación de todas estas interfaces dentro del propio JDK se unifican por parte de Java las distintas propuestas existentes actualmente para la implementación de _reactive streams_. Es de esperar que en un futuro todas las librerías de terceros utilicen estas interfaces en vez de crear las suyas propias.

## 12.1. Flow.Publisher\<T>

La interface funcional ```Flow.Publisher<T>``` representa los publicadores.

```java
@FunctionalInterface
public static interface Publisher<T> {

  public void subscribe(Subscriber<? super T> subscriber);
}
```

El método ```subscribe``` permite que los suscriptores se suscriban a un publicador. El método recibe una instancia de ```Subscriber``` a través de la que el publicador se comunicará con el suscriptor. El publicador debe elevar una excepción de tipo ```NullPointerException``` si la instancia de ```Subscriber``` pasada como parámetro es nula.

## 12.2. Flow.Subscriber\<T>

La interface ```Flow.Subscriber<T>``` representa los suscriptores. Es utilizada por los publicadores para notificar eventos a los suscriptores.

```java
public static interface Subscriber<T> {

  public void onSubscribe(Subscription subscription);

  public void onNext(T item);

  public void onError(Throwable throwable);

  public void onComplete();
}
```

El método ```onSubscribe``` es llamado por el publicador después de que el suscriptor llame al método suscribe del publicador y este último acepte la suscripción. Recibe una instancia de ```Subscription``` a través de la que el suscriptor podrá controlar la suscripción.

El método ```onNext``` es llamado por el publicador para pasar al suscriptor los elementos generados por la suscripción.

El método ```onComplete``` es llamado por el publicador cuando la suscripción termina, ya sea porque se ha consumido el número de elementos solicitado por el suscriptor, o el publicador ha consumido totalmente los elementos de su fuente origen de la información.

El método ```onError``` es llamado por el publicador cuando se produce un error. Después de llamar a este método la suscripción se considera terminada y ningún otro método será invocado. Notar que no se lanza una excepción en caso de error, sino que el publicador pasa una instancia de una excepción al suscriptor. Por ejemplo, este método es invocado con una excepción de tipo ```IllegalStateException``` cuando un suscriptor intenta suscribirse y ya está suscrito, o se produce un error de cualquier otro tipo durante el proceso de suscripción.

De forma general el comportamiento es indefinido si se produce un error y se eleva una excepción durante la invocación de cualquiera de estos métodos por parte del suscriptor.

## 12.3. Flow.Subscription

La interface ```Flow.Subscription``` representa las suscripciones. Es utilizada por los suscriptores para controlar la suscripción.

```java
public static interface Subscription {

  public void request(long n);

  public void cancel();
}
```

El método ```request``` es llamado por el suscriptor para notificar al publicador que está en disposición de recibir elementos de la suscripción. Se puede llamar tantas veces como se quiera. El parámetro indica el número de elementos que está en disposición de recibir además de los ya solicitados. Si se solicita cero o menos elementos el publicador llamará al método ```onError``` del suscriptor con una instancia de la excepción ```IllegalArgumentException```. Si se solicita ```Long.MAX_VALUE``` el publicador interpretará que el suscriptor quiere recibir todos los elementos generados por la suscripción sin ningún tipo de restricción en cuanto a su volumen, potencialmente infinito.

El método ```cancel``` es llamado por el suscriptor para solicitar al publicador que cancele la suscripción. Después de llamar este método, ni ```onComplete``` ni ```onError``` serán invocados por el publicador, pero ```onNext``` si puede llegar a ser invocado por el publicador debido a la naturaleza asíncrona de todo el proceso.

## 12.4. Flow.Processor\<T, R>

La interface ```Flow.Processor<T, R>``` representa un componente que se comporta tanto como un publicador como un suscriptor.

```java
public static interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

El propósito principal de esta interface es la de ser implementada por componentes intermedios que se dediquen a realizar algún tipo de transformación sobre los datos recibidos de un publicador y publicarlos a su vez transformados para otros componentes.

Para terminar, es interesante notar el cuidado diseño de todas estas interfaces. Todas ellas establecen mecanismos de comunicación entre dos componentes en una única dirección. Todos los métodos tienen ```void``` como tipo retornado y admiten a lo sumo un parámetro. Y los objetos se intercambian entre los distintos componentes mediante invocaciones a métodos.
