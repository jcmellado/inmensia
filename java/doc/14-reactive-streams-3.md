# 14. Java: Reactive Streams (y 3)

_23-05-2018_ _Juan Mellado_

Implementar un _stream_ que cumpla todos los requerimientos exigidos por la especificación de _Reactive Streams_ requiere bastante atención a los detalles. Puede llegar a ser complicado verificar que todas y cada una de las reglas de la especificación se cumplen. Afortunadamente, el mismo grupo de trabajo que desarrolla la especificación también ofrece _Reactive Streams Technology Compatibility Kit_ (TCK), una herramienta que prueba implementaciones de _reactive streams_ y verifica si cumplen las reglas.

TCK prueba prácticamente todas las reglas, pero no todas, ya que hay unas pocas reglas que no pueden probarse de forma automatizada debido a la naturaleza de las mismas, como por ejemplo comprobar que un productor realmente genera infinitos elementos. En todo caso, TCK prueba todas las reglas importantes y es de gran ayuda para los desarrolladores.

TCK es una librería de código abierto desarrollada en Java disponible en Maven. No obstante, hay que tener en cuenta que existen dos versiones de la librería, una anterior a Java 9 y otra posterior. La anterior permite probar _streams_ que implementen las interfaces propuestas originalmente por el grupo de trabajo que desarrolla la especificación. La posterior permite probar las nuevas interfaces del paquete ```java.util.concurrent.Flow``` ofrecidas de forma nativa por Java 9 y lógicamente es la que se recomienda utilizar hoy en día.

Para incluir TCK en un proyecto basta con añadir la dependencia de Maven correspondiente en el fichero ```pom.xml``` de la forma acostumbrada:

```xml
<dependency>
  <groupId>org.reactivestreams</groupId>
  <artifactId>reactive-streams-tck-flow</artifactId>
  <version>1.0.2</version>
  <scope>test</scope>
</dependency>
```

TCK ofrece cuatro clases base de las que se puede extender para escribir pruebas basadas en TestNG. La clase ```PublisherVerification<T>``` se debe utilizar para probar publicadores. Las clases ```SubscriberWhiteboxVerification<T>``` y ```SubscriberBlackboxVerification<T>``` se deben utilizar para probar suscriptores, siendo recomendable utilizar la primera, aunque implique más esfuerzo, ya que es capaz de probar algunos casos que la segunda no puede. Y por último, la clase ```IdentityProcessorVerification<T>``` que debe utilizarse para probar procesadores. La documentación es detallada y cubre bastantes aspectos del uso de la librería.

Como ejemplo, partamos de la siguiente implementación de un publicador que no hace absolutamente nada:

```java
public class TestPublisher implements Publisher<Integer> {

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
  }
}
```

Para probar el publicador se debe escribir una clase que extienda de ```PublisherVerification<T>```. En el constructor se debe inicializar el entorno de ejecución creando una instancia de ```TestEnviroment```, que permite especificar algunos parámetros, como por ejemplo _timeouts_ o el nivel de detalle del _log_. Y se debe implementar dos métodos, ```createPublisher```, para crear una instancia del publicador que se quiere probar, y ```createFailedPublisher```, para crear una instancia del publicador que se quiere probar en condiciones específicas de error, aunque esto último se puede ignorar retornando ```null``` desde el método.

```java
@Test
public class TestPublisherTest extends FlowPublisherVerification<Integer> {

  public TestPublisherTest() {
    super(new TestEnvironment());
  }

  @Override
  public Publisher<Integer> createFlowPublisher(long elements) {
    return new TestPublisher();
  }

  @Override
  public Publisher<Integer> createFailedFlowPublisher() {
    return null;
  }
}
```

La forma de ejecutar las pruebas con TestNG depende del entorno en el que se esté trabajando. Desde Eclipse basta con instalar un _plugin_ desde el Marketplace, que añade opciones que permiten ejecutar directamente las clases de prueba. No obstante, TCK depende de una versión muy antigua de TestNG, por lo que es necesario excluirla y añadir una versión más moderna para que el _plugin_ funcione:

```xml
<dependency>
  <groupId>org.reactivestreams</groupId>
  <artifactId>reactive-streams-tck-flow</artifactId>
  <version>1.0.2</version>
  <scope>test</scope>
  <exclusions>
    <exclusion>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.testng</groupId>
  <artifactId>testng</artifactId>
  <version>6.14.3</version>
</dependency>
```

En buena lógica, el resultado de la ejecución de la prueba sobre el productor de ejemplo, que no implementa nada, retorna una gran cantidad de errores:

```text
FAILED: required_createPublisher3MustProduceAStreamOfExactly3Elements
FAILED: required_spec101_subscriptionRequestMustResultInTheCorrectNumberOfProducedElements
FAILED: required_spec102_maySignalLessThanRequestedAndTerminateSubscription
FAILED: required_spec105_mustSignalOnCompleteWhenFiniteStreamTerminates
...
Tests run: 38, Failures: 20, Skips: 16
```

Los errores devueltos por TCK son de la forma ```TYPE_spec###_DESC```. Donde ```TYPE``` puede ser ```required```, ```optional```, ```stochastic``` o ```untested```, e indica si la regla es obligatoria, opcional, no verificable, o no implementada. ```###``` es el número de regla, siendo el primer número el apartado de la especificación y los dos últimos el número de la regla dentro del apartado. Y ```DESC``` es una breve descripción de la regla.

La implementación original de nuestro productor de ejemplo se puede mejorar implementando una suscripción y retornándola cuando un suscriptor lo solicite, aunque sin implementar aún nada realmente:

```java
public class TestPublisher implements Publisher<Integer> {

  public static class TestSubscription implements Subscription {

    @Override
    public void request(long n) {
    }

    @Override
    public void cancel() {
    }
  }

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    Subscription subscription = new TestSubscription();

    subscriber.onSubscribe(subscription);
  }
}
```

Si se vuelve a ejecutar la prueba, se observa como el número de errores se reduce:

```text
Tests run: 38, Failures: 14, Skips: 14
```

Si se estudia la lista de pruebas ejecutadas con éxito se puede observar que algunas pasan de casualidad debido a algún efecto lateral. Por ejemplo, la prueba ```required_spec109_subscribeThrowNPEOnNullSubscriber``` se ejecuta con éxito porque el método utiliza directamente el suscriptor recibido, sin comprobar si es nulo, y eso hace que se eleve un ```NullPointerException``` (NPE) tal y como exige la regla. La solución obvia es comprobar que el parámetro no es nulo antes de utilizarlo:

```java
@Override
public void subscribe(Subscriber<? super Integer> subscriber) {
  Objects.requireNonNull(subscriber);
...
```

El código se puede seguir evolucionando para hacer que implemente más reglas de la especificación, como por ejemplo la relativa a la prueba ```optional_spec309_requestNegativeNumberMaySignalIllegalArgumentExceptionWithSpecificMessage``` que exige que se debe señalizar un error con una instancia de ```IllegalArgumentException``` si se solicitan cero o menos elementos:

```java
public static class TestSubscription implements Subscription {

  private final Subscriber<? super Integer> subscriber;
  
  public TestSubscription(Subscriber<? super Integer> subscriber) {
    this.subscriber = subscriber;
  }

  @Override
  public void request(long n) {
    if (n <= 0L) {
      subscriber.onError(new IllegalArgumentException("3.9");
    ...
```

Llegado este punto ya debería quedar la clara la utilidad de TCK, que no sólo se limita a facilitar las pruebas, sino que ayuda a delimitar la funcionalidad pendiente de implementar para cumplir con los requerimientos de la especificación de _Reactive Streams_. El propio repositorio de código de TCK incluye unas cuantas clases de ejemplo donde se puede observar la complejidad necesaria para cumplir con algunas de las reglas.

En líneas generales, implementar un _reactive stream_ no se considera una tarea abordable sin un estudio previo de la especificación y el conocimiento de aspectos concretos del funcionamiento de Java, como por ejemplo la sincronización de procesos concurrentes. El propio Java proporciona la clase ```SubmissionPublisher<T>``` para facilitar la creación de publicadores asíncronos, y se recomienda su utilización si cubre el caso de uso que se debe implementar en vez de realizar un desarrollo partiendo completamente de cero.
