# 15. Java: Benchmarks con JMH

_30-05-2018_ _Juan Mellado_

JMH (_Java Microbenchmark Harness_) es una herramienta para realizar _benchmarks_ en Java. Se desarrolla como parte de OpenJDK y basa su funcionamiento en Maven. Su propósito es el de medir dos implementaciones distintas de un mismo diseño. Lo que en la práctica quiere decir que se utiliza para escribir dos métodos y ver cuál de ellos es más rápido. JMH tiene en cuenta muchos aspectos del funcionamiento interno de la máquina virtual de Java y se asegura de que el código probado se pruebe correctamente, algo no tan sencillo como a priori pudiera parecer.

Hacer un bucle que ejecute un millón de veces un método e imprimir al principio y final la hora para ver el tiempo transcurrido no sirve para nada. La máquina virtual de Java puede decidir optimizar el código y eliminarlo completamente de la ejecución si concluye que realmente no realiza ninguna operación efectiva. El tiempo medido en ese supuesto sería extremadamente pequeño, pero dentro del contexto de un programa real, donde el código sí que se ejecutase, podría ser muy elevado. Herramientas como JMH hacen todo lo posible para que las pruebas sean fiables.

Para realizar pruebas con JMH hay que generar un proyecto con Maven, preferentemente desde línea de comandos, aunque también se pueden generar desde un IDE. Lo que si se recomienda es utilizar la línea de comandos para ejecutarlos y no añadir ningún componente externo que pueda provocar efectos laterales durante la ejecución de las pruebas.

El comando básico de creación de un proyecto a partir del arquetipo de Maven para JMH es el siguiente:

```text
mvn archetype:generate
  -DarchetypeGroupId=org.openjdk.jmh
  -DarchetypeArtifactId=jmh-java-benchmark-archetype
  -DinteractiveMode=false 
  -DgroupId=<GROUP_ID>
  -DartifactId=<ACTIFACT_ID>
  -Dversion=<VERSION>
```

Como resultado de la ejecución se crea un proyecto con dos ficheros dentro de un nuevo directorio. El primero de ellos es el clásico ```pom.xml``` con la configuración del proyecto Maven, y el segundo una clase de ejemplo que puede utilizarse como plantilla para crear _benchmarks_. El proyecto generado se puede compilar para crear un _jar_ y ejecutarlo con los siguientes comandos dentro del nuevo directorio creado:

```text
mvn clean install

java -jar target/benchmarks.jar
```

Si se está utilizando Java 9 o superior puede resultar en un error ```java.lang.NoClassDefFoundError: javax/annotation/Generated``` debido al nuevo sistema de módulos de Java. Este error se puede resolver configurando un parámetro en la máquina virtual de Java que ejecuta Maven, más concretamente indicando que cargue el módulo que contiene la clase que no se encuentra:

```text
set MAVEN_OPTS=--add-modules java.xml.ws.annotation
```

Como resultado de la ejecución del _benchmark_ se muestra por consola el progreso de la prueba, que consiste en varias ejecuciones de un proceso de precalentamiento de la máquina virtual, necesario para llevarla a un estado lo más predecible posible, varias ejecuciones de la clase de prueba, y finalmente las estadísticas resultantes.

Se pueden consultar todas las opciones disponibles para la ejecución del _benchmark_ añadiendo el clásico parámetro ```-help``` en la línea de comandos con ```java -jar target/benchmarks.jar -help```. Y es recomendable hacerlo las primeras veces para hacerse una idea de cómo se puede personalizar la ejecución.

Lógicamente la ejecución anterior con las opciones por defecto de una clase de ejemplo que se limita a probar un método vacío no aporta ningún valor. La idea es que se añada dentro del ```pom.xml``` la dependencia con el código que se quiera probar y se escriban clases que prueben dicho código. JMH proporciona en su documentación ejemplos de cómo escribir clases de prueba.

La escritura de clases de prueba para JMH se realiza mediante el uso de anotaciones de forma similar a como se realiza en otros _frameworks_.

- ```@Benchmark``` es la anotación básica que debe aplicarse sobre los métodos públicos que se quieran probar.

- ```@BenchmarkMode``` permite especificar el modo de realizar la prueba, pudiéndose elegir entre ```Throughput```, que llama continuamente a los métodos probados y mide el rendimiento total, ```AverageTime```, que llama continuamente a los métodos probados y mide el tiempo medio de ejecución, ```SampleTime```, que llama continuamente a los métodos probados y mide el tiempo de ejecución tomando muestras aleatorias de la duración de los mismos, ```SingleShotTime```, que llama una sola vez a los métodos probados y mide el tiempo de ejecución sin enmascarar la penalización por el precalentamiento, y ```All```, que ejecuta todos los modos.

- ```@OutputTimeUnit``` permite indicar la unidad de tiempo en que se quiere mostrar los resultados, pudiéndose elegir cualquier valor del enumerado ```java.util.concurrent.TimeUnit```.

- ```@Measurement``` y ```@Warmup``` permiten controlar el proceso de prueba y precalentamiento respectivamente mediante parámetros tales como el número de iteraciones a realizar o la duración de las mismas. Otras anotaciones como ```@Fork```, ```@Threads```, ```@CompilerControl``` y ```@Param``` controlan detalles de más bajo nivel referentes a la creación de hilos de ejecución y la compilación de las clases de prueba.

- ```@State``` se utiliza para crear clases que se puedan pasar como argumentos a los métodos marcados con ```@Benchmark```. En la anotación se debe indicar el ámbito de los objetos creados, pudiéndose elegir entre ```Thread```, para que todas las instancias sean distintas, ```Benchmark```, para que todas las instancias de una misma clase se compartan entre todos los threads, o ```Group```, para que todas las instancias de una misma clase se compartan entre los _threads_ de un grupo. Los grupos se definen con las anotaciones ```@Group``` y ```@GroupThread``` que permiten probar varios métodos a la vez en una misma iteración.

- ```@Setup``` y ```@Teardown``` permiten ejecutar procesos de inicialización y finalización de una clase anotada con ```@State```. De hecho, sólo se pueden utilizar sobre métodos de clases anotadas con ```@State```. Se invocan en función de un argumento de las propias anotaciones, pudiéndose elegir entre ```Trial```, para que se invoquen por cada prueba, ```Iteration```, para que se invoquen en cada iteración de las pruebas, e ```Invocation```, para que se invoquen en cada ejecución del método, aunque este último no se recomienda sin leer la documentación y entender sus implicaciones.

Muchas de las anotaciones son heredables, por lo que es sencillo escribir clases base de las que heredar el comportamiento y crear _suites_ de pruebas. O clases concretas para realizar pruebas más específicas, como en el siguiente ejemplo:

```java
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations=5, time=50, timeUnit=TimeUnit.MILLISECONDS)
@Measurement(iterations=10, time=100, timeUnit=TimeUnit.MILLISECONDS)
@Fork(value=1)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class RandomBenchmark {

  @Benchmark
  public double testMathRandom() {
    return Math.random();
  }

  @Benchmark
  public double testRandom(StatedRandom stated) {
    return stated.getRandom().nextDouble();
  }

  @Benchmark
  public double testThreadLocalRandom() {
    return ThreadLocalRandom.current().nextDouble();
  }

  @State(Scope.Benchmark)
  public static class StatedRandom {

    private Random random;

    public Random getRandom() {
      return random;
    }

    @Setup public void setup() {
      random = new Random();
    }
  } 
}
```

Como se observa, los métodos de prueba no deben contener bucles, sólo la invocación a la funcionalidad que se quiera probar. JMH ya realiza las iteraciones pertinentes. No obstante, si fuera realmente necesario por alguna razón técnica hacer un bucle que invoque a la funcionalidad que se quiere probar, se puede utilizar la anotación ```@OperationsPerInvocation(n)``` sobre el método que contiene el bucle para indicar que dicho método realiza ```n``` iteraciones.

Otro detalle a tener en cuenta es que resulta conveniente retornar algún valor desde los métodos de prueba, para evitar que la máquina virtual considere el código como muerto y lo optimice eliminando su ejecución. De igual forma, si el valor retornado por el método de prueba no depende del estado de ningún objeto y el resultado es predecible, la máquina virtual puede optimizarlo reemplazándolo por un valor constante, o eliminándolo completamente si concluye que no hay ningún proceso posterior que lo utilice. Para evitar estos y otros problemas se debe utilizar la clase ```BlackHole``` de JMH, que consume todo tipo de objetos y hace todo lo posible por garantizar que la prueba sea lo más fiable posible.

```java
@Benchmark
public void testMathRandomBh(Blackhole bh) {
  bh.consume(Math.random());
}
```

Para terminar, comentar además otras dos características interesantes que ofrece JMH. La primera de ellas es un API para Java que permite crear programas que ejecuten el propio JMH desde un ```main```. Y la segunda una serie de _profiles_, para pruebas más sofisticadas, que permiten controlar aspectos tales como el tiempo consumido por el compilador JIT (_Just In Time_) o el _Garbage Collector_, por citar sólo algunas de sus posibilidades.
