# 9. Java: Streams

_30-04-2018_ _Juan Mellado_

Un _stream_ es una representación abstracta de una secuencia de elementos. Su propósito principal es facilitar la ejecución de operaciones sobre dicha secuencia. Operaciones individuales sobre cada elemento, sobre un subconjunto de ellos, o sobre la totalidad de los mismos. Ejecutadas de forma secuencial o paralela. Evaluadas normalmente de forma perezosa. Y con el propósito de modificar los elementos de la secuencia, obtener una nueva secuencia, extraer un elemento individual, o calcular un valor agregado.

Los _streams_ se diferencian de las colecciones clásicas de Java en varios aspectos:

- No son almacenes de datos. Un _stream_ no es una estructura de datos que almacena información. Obtiene un dato de una fuente y lo pasa a través de una cadena de operaciones.

- Están orientados a la programación funcional. Un _stream_ no modifica la fuente de la que obtiene datos. No basa su funcionamiento interno en efectos laterales.

- Promueven la inicialización y evaluación perezosa (_lazy_). Un _stream_ implementa todas sus operaciones de forma perezosa siempre que sea posible. Un elemento es visitado sólo una vez.

- No tienen un tamaño fijo predeterminado. Un _stream_ puede alimentarse de una fuente de datos potencialmente infinita.

Los _streams_ fueron introducidos en Java 8 con el paquete ```java.util.stream``` a través de la interface genérica ```Stream<T>```, y las interfaces especializadas ```IntStream```, ```LongStream``` y ```DoubleStream``` que operan sobre las primitivas ```int```, ```long``` y ```double``` respectivamente.

Un _stream_ puede crearse de muchas formas, por ejemplo con las factorías ```Stream.empty``` o ```Stream.of```:

```java
var stream1 = Stream.empty();

var stream2 = Stream.of("a", "b", "c");
```

O a partir de un _array_ ya existente con el método ```Arrays.stream```:

```java
var numeros = new int[] {1, 2, 3};

var stream = Arrays.stream(numeros);
```

O a partir de una colección ya existente con los métodos ```stream``` y ```parallelStream``` proporcionados por las propias colecciones:

```java
var cadenas = List.of("a", "b", "c");

var secuencial = cadenas.stream();

var paralelo = cadenas.parallelStream();
```

O con factorías que generan ellas mismas una secuencia de elementos, de un tamaño determinado como ```IntStream.rangeClosed```, o potencialmente infinita como ```Random.ints```:

```java
var stream1 = IntStream.rangeClosed(1, 3);

var stream2 = new Random().ints();
```

O con métodos de clases de dominios específicos, como ```String.chars``` o ```Files.list```:

```java
var stream1 = "abc".chars();

var path = Paths.get("/tmp");

var stream2 = Files.list(path);
```

Las operaciones a realizar sobre un _stream_ se definen de forma habitual concatenándolas en un estilo _fluent_ formando un _pipeline_ con los métodos de la propia interface ```Stream<T>```. Pudiendo estas operaciones ser intermedias o terminales. Las intermedias son siempre evaluadas de forma perezosa, se espera hasta alcanzar una terminal para evaluarlas. Las terminales provocan que se consuman elementos de la fuente del _stream_ y generan un resultado o efecto lateral.

La mayoría de los métodos de la interface ```Stream<T>``` admiten una interface funcional como parámetro. Lo que en la práctica se expresa como una función _lambda_, referencia a un método, o constructor. Por ejemplo, el método ```filter``` permite filtrar elementos de la fuente de un _stream_. Por lo que para filtrar los números pares de un _stream_ de enteros basta con crear una función que diga cuando un número se considera par:

```java
var stream = IntStream.rangeClosed(1, 10).filter(x -> x % 2 == 0);
```

El método ```filter``` es un ejemplo de operación intermedia, de igual forma que ```takewhile```, ```dropWhile```, ```skip```, o ```distinct```, por citar algunas:

```java
var stream = IntStream.rangeClosed(1, 10).skip(2).takeWhile(x -> x < 7);
```

Por su parte, un ejemplo de operación terminal es ```map```, que aplica una función sobre cada elemento del _stream_ y genera un nuevo _stream_ con los elementos resultantes, o ```flatMap```, que por cada elemento puede generar cero, uno, o más elementos:

```java
var stream1 = Stream.of(1, 2, 3)
  .map(x -> x * 2);

var stream2 = IntStream.rangeClosed(1, 5)
  .flatMap(x -> IntStream.rangeClosed(1, x));
```

Una operación similar es ```forEach```, que aplica una función sobre cada elemento del _stream_ pero no retorna nada:

```java
Stream.of(1, 2, 3).forEach(System.out::println);
```

Ejemplos de operaciones terminales que reducen un _stream_ a un único valor son ```min```, ```max```, ```count```, o ```findFirst```:

```java
var minimo = IntStream.rangeClosed(1, 100).min();

var maximo = IntStream.rangeClosed(1, 100).max();

var cuenta = IntStream.range(1, 100).count();

var primero = IntStream.range(1, 100).findFirst();
```

Otro conjunto de operaciones muy utilizadas son aquellas que reducen un _stream_ a un _array_ o una colección. La reducción a un _array_ se realiza con ```toArray```, mientras que la reducción a una colección se realiza con ```toCollect```, que utiliza una familia de funciones de la clase ```Collectors```:

```java
var array = IntStream.rangeClosed(1, 10)
  .map(x -> x * 2)
  .toArray();

var lista = Stream.of("a", "b", "c")
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

La clase ```Collectors``` tiene una gama muy amplia de métodos que permiten agrupar elementos de una forma muy variada simplificando sobremanera el código. Es altamente recomendable revisar su API, junto con la de ```Stream<T>```, para profundizar en las posibilidades ofrecidas por estas y detalles técnicos como el recorrido de colecciones ordenadas.

Utilizando _streams_ se simplifica el código, se mejora la intencionalidad de las interfaces, y se facilita el desarrollo orientado a pruebas.

```java
var libros = libreria
    .filter(libro -> libro.esOferta())
    .map(libro -> libro.getTitulo())
    .limit(10)
    .collect(Collectors.toList());
```

Los _streams_ en Java ofrecen de forma nativa muchas más opciones, como la ejecución en paralelo por ejemplo, muy útiles para la implementación del patrón _map-reduce_ sobre una cantidad elevada de datos. Desde su introducción en Java 8 han ganado mucha popularidad e introducido la programación funcional de forma natural en el ecosistema de Java.
