# 6. Java: Colecciones Inmutables

_20-04-2018_ _Juan Mellado_

Hubo un tiempo en que parecía que los objetos inmutables iban a salvar el mundo. En particular cuando los gurús los pusieron de moda introduciendo la programación funcional a las masas. Hoy en día sigue siendo un patrón totalmente válido y que debería utilizarse más a menudo de lo que normalmente se hace.

En algunos lenguajes todo son objetos inmutables. La idea es que una vez que se crea un objeto éste no se puede modificar. De esta manera el objeto se puede pasar de forma segura entre distintos componentes de un sistema. Un objeto que no expone ningún mecanismo para cambiar su estado interno no se puede modificar por ningún componente que consuma dicho objeto. Los componentes que actúan como clientes pueden llamar a componentes que actúan como servidores sin preocuparse de que los objetos pasados se modifiquen.

De forma muy simplista puede visualizarse como una clase POJO (_Plain Old Java Object_) en la que todos los atributos son ```final```, inicializados en el constructor, y que por tanto carecen de _mutators_ (_setters_). En la práctica se podría incluso prescindir de los _accesors_ (_getters_) y hacer los atributos ```public```, pero en algunos entornos esto es motivo de excomunión.

Los compiladores y máquinas virtuales pueden aplicar optimizaciones cuando se usan objetos inmutables. Un objeto en memoria no es más que una secuencia de _bytes_. Si se garantiza que dicha secuencia no puede cambiar, entonces es seguro pasar una referencia (puntero) a dicha secuencia a lo largo y ancho de todo un sistema. No hay que preocuparse de hacer copias o sincronizar el acceso en entornos multhilos. Y es más, si se crean varios objetos iguales entonces es seguro utilizar la misma referencia para todos ellos.

En Java los objetos inmutables han existido desde el principio, por ejemplo las instancias de las clase ```String``` son objetos inmutables. Cuando se modifica una instancia lo que se hace en realidad es crear una nueva instancia. La instancia no cambia su estado, sino que se crea una nueva instancia con un nuevo estado. Y de esta forma se eliminan los efectos laterales debidos a los cambios de estado de los objetos.

Un caso de uso habitual en el desarrollo de casi cualquier sistema es procesar una colección de objetos. Se pasa una colección a un método, y se presupone que la colección original pasada no es modificada por el método llamado. Sin embargo esta suposición es difícil de garantizar en la práctica. Se puede establecer en el contrato de la interface, pero debe ser finalmente la implementación la que asegure el cumplimiento de dicho contrato.

En Java las colecciones inmutables se han creado tradicionalmente a través de los métodos estáticos de la clase ```Collections```, como ```emptyList```, para crear listas vacías, y ```unmodifiableList```, que admite una lista como parámetro y retorna una copia inmutable de dicha lista:

```java
Collections.emptyList();
Collections.unmodifiableList(lista);
```

En el segundo caso, si la lista original pasada como parámetro se modifica, la lista inmutable no se ve afectada. Si se intenta añadir, modificar o borrar sobre la lista inmutable se eleva una excepción en tiempo de ejecución. Si la lista es de objetos inmutables entonces toda la lista es inmutable. Si se modifica un objeto de la lista original parecerá que la lista inmutable se modifica también, pero en realidad lo que se modifica es el objeto. La lista inmutable sólo almacena referencias (punteros) a los objetos que contenía la lista original en el momento de su creación.

La clase ```Collections``` contiene métodos adicionales para la creación de otros tipos de colecciones inmutables, como ```Map``` y ```Set```. Pero Java en si mismo carece de una sintaxis que permita crear colecciones _inline_ de manera sencilla.

Crear _arrays_ es sencillo:

```java
int[] array = {1, 2, 3};
```

Pero crear listas es sencillo sólo si se conoce el API, ¡el método está en la clase ```Arrays```!:

```java
var lista = Arrays.asList("uno", "dos", "tres");
```

Java 9 añadió un nuevo método estático of sobrecargado dentro de las interfaces ```List```, ```Set``` y ```Map``` para facilitar la creación de colecciones inmutables:

```java
List.of();
List.of(1);
List.of(1, 2);
List.of(1, 2, 3);

Set.of();
Set.of(1);
Set.of(1, 2);
Set.of(1, 2, 3);

Map.of();
Map.of("uno", 1);
Map.of("uno", 1, "dos", 2);
Map.of("uno", 1, "dos", 2, "tres", 3);

Map.ofEntries(Map.entry("uno", 1), Map.entry("dos", 2), Map.entry("tres", 3));
```

Teniendo en cuenta que estas factorías no admiten nulos, ni duplicados en el caso concreto de ```Set``` y ```Map```, ni claves nulas en el ```Map```, y que dos llamadas distintas pueden retornar la misma referencia si las colecciones creadas son iguales.

Por su parte Java 10 añadió un nuevo método estático ```copyOf``` dentro de las interfaces ```List```, ```Set``` y ```Map``` para facilitar la creación de colecciones inmutables a partir de una colección dada:

```java
List.copyOf(lista);
Set.copyOf(conjunto);
Map.copyOf(mapa);
```

Y métodos en ```Collectors``` para facilitar la creación de colecciones inmutables a partir de un _stream_:

```java
Collectors.toUnmodifiableList();
Collectors.toUnmodifiableSet();
Collectors.toUnmodifiableMap(keyMapper, valueMapper);
Collectors.toUnmodifiableMap(keyMapper, valueMapper, mergeFunction);
```

Be inmutable, my friend.
