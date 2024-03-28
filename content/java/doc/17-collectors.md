# 17. Java: Collectors

_14-06-2018_ _Juan Mellado_

Las clases que implementan la interface ```Collector``` de Java son las encargadas de convertir en conjuntos finitos de información las secuencias de elementos, potencialmente infinitas, generadas por los _streams_. Son operaciones terminales de reducción dentro de la cadena de procesamiento de un _stream_. Acumulan los datos producidos y los almacenan en algún tipo de colección, o generan un único valor a partir de ellos. Se utilizan por ejemplo para crear listas a partir de _streams_, o calcular valores máximos o mínimos. En la práctica son más conocidas por "_¡Ah, eso que se pone al final! La cosa esa del ```.collect(Collectors.toList())```_" (_sic_).

Los recolectores fueron introducidos en Java 8 y han sido ampliados en las versiones 9 y 10. La interface básica es ```java.util.stream.Collector<T, A, R>```, que ofrece cuatro métodos básicos con los que se construyen los recolectores. Estos métodos hacen uso del concepto de interface funcional de Java y no implementan las acciones directamente, sino que retornan una función que es la que realmente implementa la acción. Es como ir a una ventanilla para hacer una gestión y que te respondan "_sí, es aquí, pero vaya a esa otra ventanilla_".

Las funciones básicas expuestas por los recolectores son muy genéricas, indican cómo construir un contenedor intermedio, cómo añadir elementos a dicho contenedor, cómo combinar dos contenedores para generar un resultado parcial, y cómo calcular el resultado final a partir de los parciales:

- ```Supplier<A> supplier()```: Este método retorna una función que crea un contenedor intermedio de tipo ```A```. Un contenedor puede ser de cualquier clase que se quiera. No tiene que implementar ninguna interface concreta. Puede ser una ```Collection``` clásica de Java, pero también puede ser un ```StringBuilder``` por ejemplo.

- ```BiConsumer<A, T> accumulator()```: Este método retorna la función que acumula elementos de entrada del tipo ```T``` en un contenedor intermedio de tipo ```A```.

- ```BinaryOperator<A> combiner()```: Este método retorna la función que combina dos contenedores intermedios de tipo ```A``` y crea un resultado parcial también de tipo ```A```.

- ```Function<A, R> finisher()```: Este método retorna la función que calcula el resultado final de tipo ```R``` a partir de los resultados parciales de tipo ```A```. El tipo ```R``` no tiene que implementar ninguna interface concreta.

Una forma muy directa de implementar un recolector es utilizar uno de los dos métodos estáticos ```of``` de la interface ```Collector```. Estos métodos admiten como parámetros de entrada las cuatro funciones básicas y las características del recolector. Siendo estas características una combinación de valores del enumerado ```Collector.Characteristics```. Donde el valor ```CONCURRENT``` indica que el proceso de recolección puede ejecutarse de forma concurrente, el valor ```UNORDERER``` indica que el resultado no depende del orden original de los elementos de entrada, y el valor ```IDENTITY_FINISH``` indica que la función de finalización puede omitirse retornándose directamente el contenedor intermedio. Es el mismo principio de definición de características usado para los ```Spliterators```, nombre inspirado seguramente en alguna de las películas de _Parque Jurásico_.

A modo de ejemplo, el recolector del código siguiente se crea especificando directamente las funciones como parámetros del método ```Collector.of```. Su propósito es retornar la suma de todos los elementos de tipo ```Integer``` de una fuente de entrada. La primera función crea un _array_ con un entero que se puede utilizar como contenedor intermedio, la segunda función suma a un contenedor intermedio un elemento de entrada, la tercera función suma al contenido de un contenedor el contenido de otro, y la cuarta función retorna el resultado final de un contenedor.

```java
var collector = Collector.of(

// supplier
  () -> new int[1],

// accumulator
  (int[] result, Integer element) -> result[0] += element,

// combiner
  (int[] result, int[] partial) -> {
    result[0] += partial[0];
    return result;
  },

// finisher
  (int[] result) -> result[0],

// characteristics
  Collector.Characteristics.UNORDERED
);
```

El recolector anterior puede aplicarse directamente en un _stream_ de enteros para obtener la suma de todos los elementos:

```java
var sum = Stream.of(1, 2, 3).collect(collector);
```

Como es de suponer, los recolectores se pueden crear de una manera más tradicional definiendo una clase que implementa la interface ```Collector```, sin necesidad de utilizar expresiones _lambdas_ como en el código de ejemplo anterior. No obstante, normalmente no suele ser necesario escribirlos, ya que Java proporciona de forma nativa implementaciones de una gran cantidad de recolectores de uso habitual a través de métodos estáticos de la clase ```java.util.stream.Collectors```.

Los recolectores más comunes de la clase ```Collectors``` son los que crean colecciones, como los clásicos métodos ```toList```, ```toSet``` y ```toMap``` que existen desde la versión 8 de Java, más los que se añadieron en la versión 10 como ```toUnmodifiableList```, ```toUnmodifiableSet``` y ```toUnmodifiableMap```, alguno más específico como ```toConcurrentMap```, o el más genérico ```toCollection```:

```java
stream.collect(Collectors.toList());

stream.collect(Collectors.toUnmodifiableSet());

stream.collect(Collectors.toCollection(ArrayList::new));
```

Otros recolectores de gran utilidad ofrecidos por ```Collectors``` son los que realizan reducciones, es decir, calculan un único resultado a partir de los elementos de entrada. Ejemplos de este tipo de recolectores son ```counting```, que cuenta el número de elementos, ```joining```, que concatena todos los elementos como una cadena de texto, ```maxBy``` y ```minBy```, que calculan el máximo y mínimo, ```summingInt```, ```summingLong``` y ```summingDouble```, que calculan la suma de los elementos, y ```averagingInt```, ```averagingLong``` y ```averagingDouble```, que calculan la media de los elementos. Un caso particular es ```summarizingInt```, ```summarizingLong``` y ```summarizingDouble```, que calcula todos los valores anteriores a un mismo tiempo, es decir que retorna un objeto que contiene el total de elementos, el máximo, el mínimo y la media.

```java
empleados.collect(Collectors.counting());

empleados.collect(Collectors.averagingDouble(Empleado::getSueldo));
```

Otro tipo interesante de recolectores son los de agrupación, que como su propio nombre indica permiten agrupar los elementos de entrada en base a un determinado criterio. La salida de estos recolectores es un ```Map```, donde la clave es el criterio de agrupación y el valor la colección de elementos agrupados.

```java
empleados.collect(Collectors.groupingBy(Empleado::getDepartamento));

empleados.collect(Collectors.groupingBy(
  Empleado::getDepartamento,
  Collectors.counting()));
```

La clase ```Collectors``` expone más factorías de recolectores aparte de las aquí citadas, por lo que es recomendable echar un vistazo a la documentación del API.

Como nota final, comentar que he decidido traducir _collector_ como recolector, en vez de dejarlo en el inglés original, como se suele hacer con otras palabras, como _stream_ por ejemplo, por la similitud con la palabra en español, pero evitando traducirlo directamente como colector, que para mí tiene otro sentido.

¡Feliz día de la recolección!
