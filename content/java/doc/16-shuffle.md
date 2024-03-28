# 16. Java: Recorridos Aleatorios

_07-06-2018_ _Juan Mellado_

La forma habitual de desordenar una lista en Java es utilizar el método ```Collections.shuffle```. El inconveniente de este método es que modifica la lista original, un efecto lateral no deseable en el mundo actual donde predominan las colecciones inmutables. La alternativa más obvia es crear una nueva lista, a costa lógicamente de duplicar el consumo de memoria.

Un enfoque distinto es recorrer la lista de forma desordenada. La premisa de tal tipo de recorrido es que se retornen todos los elementos de la lista de manera aleatoria y que cada uno de ellos sólo se retorne una única vez. La condición es que la lista sea de tamaño fijo, preferentemente inmutable; lo contrario podría dar lugar a implementaciones con comportamientos indefinidos no predecibles. Y la restricción es que la operación se ejecute de forma secuencial, no paralela, ya que realizar particiones de la lista no permitiría acceder a todos los elementos de manera aleatoria.

Una implementación de tal tipo de recorrido implica generar un número aleatorio en cada iteración que indique el índice del elemento a retornar. Además de almacenar todos los índices generados para evitar duplicados. Para ello, para una lista de tamaño ```n```, se puede crear un array de ```n``` índices, inicializar dicho _array_ secuencialmente con valores de ```0``` a ```n```, en cada iteración ```i``` generar un número aleatorio ```r``` de ```i``` a ```n - 1```, intercambiar en el _array_ el valor en la posición ```i``` por el valor en la posición ```r```, y retornar el elemento de la lista que se encuentra en la posición indicada por el valor almacenado en la posición ```i``` del _array_. Es decir, sustituir el elemento de la iteración en curso por otro de forma aleatoria y que aún no se haya retornado.

Partiendo de que en la actualidad se prefiere consumir colecciones con _streams_, en vez de con bucles ```for```, la solución pasa por la implementación de un iterador, y de forma más concreta de un ```Spliterator<T>```. Esta clase se introdujo con Java 8 para facilitar la construcción de iteradores complejos que pudieran consumir elementos tanto de forma secuencial como paralela, además de tener la capacidad de describirse a ellos mismos indicando cuales son sus características.

```SplitIterator<T>``` es una interface que requiere implementar varios métodos, pero para la mayoría de los casos de uso, que no requieran soportar paralelismo, se puede simplemente extender de ```Spliterators.AbstractSpliterator<T>```. Esta clase sólo requiere implementar el método ```boolean tryAdvance(Consumer<? super T> action)```. Un método que es equivalente a los clásicos métodos ```hasNext``` y ```next``` de los iteradores de Java, ya que a un mismo tiempo consume el elemento en curso e indica si quedan más elementos por consumir.

Una posible implementación de tal algoritmo podría ser la siguiente:

```java
public static class RandomListSpliterator<T> extends Spliterators.AbstractSpliterator<T> {

  private final SplittableRandom random = new SplittableRandom();
  private final List<T> elements;
  private final int size;
  private final int[] indexes;
  private int current = 0;

  public RandomListSpliterator(List<T> elements, int additionalCharacteristics) {
    super(elements.size(), additionalCharacteristics);

    this.elements = elements;
    this.size = elements.size();

    // Genera el array de índices inicial
    this.indexes = IntStream.range(0, this.size).toArray();
  }

  @Override
  public boolean tryAdvance(Consumer<? super T> consumer) {

    // Retorna inmediatamente si ya se ha recorrido la lista completa
    if (current == size) {
      return false;
    }

    // Obtiene el siguiente elemento de la lista de forma aleatoria
    final var element = next();

    // Consume el elemento obtenido
    consumer.accept(element);

    return true;
  }

  protected T next() {

    // Genera un índice sobre la lista de forma aleatoria
    final var rand = random.nextInt(current, size);

    // Intercambia el índice en la posición actual por el de la posición aleatoria
    final var index = indexes[rand];
    indexes[rand] = indexes[current];

    // Avanza a la siguiente posición del array para la siguiente iteración
    current ++;

    // Retorna el elemento que se encuentra en el índice de la posición aleatoria
    return elements.get(index);
  }
}
```

En el código, ```elements``` almacena la lista original y ```size``` su tamaño, ```indexes``` es el array de índices recorridos y ```current``` el número de la iteración en curso. El método ```tryAdvance``` consume el elemento en curso, y el método ```next``` realiza el proceso de generación del número aleatorio y actualización del _array_ de índices. El código debería resultar bastante sencillo de entender con los comentarios.

Notar el uso de la clase ```SplittableRandom``` para la generación de números aleatorios. Esta clase se introdujo en Java 8, junto con ```SecureRandom``` cuando la aleatoriedad de la secuencia de números generados deba ser más estricta y cumplir criterios más estrictos como los requeridos en el mundo de la criptografía.

El iterador del código de ejemplo se puede probar creando una lista, una instancia del iterador, y procesando el conjunto como un _stream_ con ```StreamSupport.stream```:

```java
var elements = IntStream.range(50, 60)
  .boxed()
  .collect(Collectors.toUnmodifiableList());

var characteristics = Spliterator.IMMUTABLE | 
  Spliterator.SIZED | 
  Spliterator.NONNULL;

var spliterator = new RandomListSpliterator<>(elements, characteristics);

StreamSupport.stream(spliterator, false)
  .forEach(System.out::println);
```

En cada ejecución la salida será distinta, devolviéndose los elementos de la lista original en un orden aleatorio cada vez.

La variable ```characteristics``` establece las características de la lista para facilitar a los clientes del iterador la toma de decisiones. ```IMMUTABLE``` indica que es inmutable, ```SIZED``` que tiene un tamaño prederminado, y ```NONNULL``` que no contiene nulos.  De la forma acostumbrada, es recomendable revisar la documentación para ver todas las opciones disponibles. En la práctica sería recomendable diseñar el iterador para un determinado caso de uso evitando tener que pasar dichos parámetros cada vez.

De cualquier forma, la implementación propuesta funciona, pero es muy básica. Por ejemplo, el _array_ de índices se inicializa a costa de recorrerlo completamente, algo que puede evitarse tratando el cero como caso especial, ya que ese el valor que por defecto asigna Java a las variables de tipo ```int```. Si un índice es cero entonces se puede interpretar como que no se ha visitado todavía y que su valor es la propia posición que ocupa, ya que ese es el valor con el que se debería haber inicializado originalmente:

```java
public static class RandomListSpliterator<T> extends Spliterators.AbstractSpliterator<T> {

  private final SplittableRandom random = new SplittableRandom();
  private final List<T> elements;
  private final int size;
  private final int[] indexes;
  private int current = 0;

  public RandomListSpliterator(List<T> elements, int additionalCharacteristics) {
    super(elements.size(), additionalCharacteristics);

    this.elements = elements;
    this.size = elements.size();

    // Crea el array de índices sin inicializarlo
    this.indexes = new int[this.size];
  }

  @Override
  public boolean tryAdvance(Consumer<? super T> consumer) {
    if (current == size) {
      return false;
    }

    final var element = next();

    consumer.accept(element);

    return true;
  }

  protected T next() {

    // Genera un índice aleatorio
    final var rand = random.nextInt(current, size);

    // Obtiene el índice sobre la posición actual y la aleatoria
    final var src = indexes[current ++];
    final var dst = indexes[rand];

    // Si el índice es cero entonces no se ha visitado y se usa la posición actual
    indexes[rand] = src == 0 ? current : src;

    // Si el índice es cero entonces no se ha visitado y se usa la posición aleatoria
    return elements.get(dst == 0 ? rand: dst - 1);
  }
}
```

De igual forma sería interesante pensar en reutilizar los _arrays_ de índices cuando se procesen múltiples listas del mismo tamaño de forma secuencial, para evitar la reserva de memoria necesaria para crear dichos _arrays_ por cada lista. Para ello se podría utilizar una variable que almacenase el valor del índice utilizado como caso especial, en vez utilizar el valor constante cero. Al guardar un índice en el _array_ se tendría que almacenar sumándole el valor de dicha variable y compararlo también contra dicho valor. Así, en el primer uso del _array_ el valor de la variable sería cero y el funcionamiento sería el mismo que el del código anterior. Pero al terminar el primer recorrido, y posteriores, se le sumaría a la variable el tamaño de la lista recorrida. En el segundo recorrido, y posteriores, los índices se almacenarían sumándoles dicho valor y se compararían contra dicho valor. Si el índice es menor que dicho valor entonces se consideraría que no se ha visitado. Por seguridad, cada cierto número de recorridos se podría inicializar el _array_ y la variable a cero.

Por último, comentar que en el caso de realmente querer recorrer una lista de forma aleatoria en paralelo, una posible solución sería generar primero el _array_ con los índices desordenados, y luego tratar en paralelo este _array_ de forma particionada, en vez de particionar la lista como normalmente se suele hacer.
