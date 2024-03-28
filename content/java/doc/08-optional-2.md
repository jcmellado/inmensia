# 8. Java: Optional (y 2)

_27-04-2018_ _Juan Mellado_

La clase ```Optional``` en Java fue creada para ser utilizada como tipo retornado por métodos que puedan devolver un valor de forma opcional. Por ejemplo, el contrato del siguiente método establece que puede retornar una cadena de texto, o no:

```java
public Optional<String> find() { ...
```

Este tipo de contratos se implementan tradicionalmente en Java retornando ```null``` desde el método para indicar que en realidad no devuelve ningún valor. Lo que eleva ```null``` a la categoría de caso especial, fuerza el uso de determinados patrones repetitivos de comprobación de nulos, impiden el uso del estilo _fluent_ basado en concatenar llamadas en cascada de varios métodos seguidos, y dificultan el uso de _streams_. Utilizando la clase ```Optional``` se puede reducir la mayoría de estos problemas.

El método ```orElse``` retorna el valor si está presente, en caso contrario un valor dado:

```java
var valor = opcional.orElse("abc");

// var valor = opcional == null ? "abc" : opcional;
```

El método ```orElseGet``` retorna el valor si está presente, en caso contrario el resultado de la función dada:

```java
var valor = opcional.orElseGet(() -> "abc");

// var valor = opcional == null ? calculaAbc() : opcional;
```

El método ```orElseThrow```, introducido en Java 10, retorna el valor si está presente, en caso contrario eleva ```NoSuchElementException```:

```java
var valor = opcional.orElseThrow();

// if (opcional == null) throw new NoSuchElementException();
// var valor = opcional;
```

El método ```orElseThrow``` está sobrecargado y admite un parámetro con el que puede indicarse el tipo concreto de excepción a elevar:

```java
var valor = opcional.orElseThrow(RuntimeException::new);
```

El método ```or```, introducido en Java 9, retorna el propio ```Optional``` si valor está presente, en caso contrario un nuevo ```Optional``` resultado de la función dada:

```java
var resultado = opcional.or(() -> Optional.of("abc"));

// var resultado = opcional == null ? calculaAbc() : opcional;
```

El método ```map``` retorna un nuevo ```Optional``` con el resultado de aplicar la función dada al valor si está presente, en caso contrario un ```Optional``` vacío:

```java
var resultado = opcional.map(String::toUpperCase);

// var resultado = opcional == null ? null : opcional.toUpperCase();
```

El método ```flatMap``` retorna el ```Optional``` resultado de aplicar la función dada al valor si está presente, en caso contrario un ```Optional``` vacío.

```java
var resultado = opcional.flatMap(x -> Optional.of(x.toUpperCase()));

// var resultado = opcional == null ? null : opcional.toUpperCase();
```

Estos dos últimos métodos abren la posibilidad a concatenar varias llamadas en cascada cuando todos los valores retornados son de tipo ```Optional```:

```java
var valor = coche.flatMap(Coche::getAireAcondicionado)
  .flatMap(AireAcondicionado::getIonizador)
  .map(Ionizador::getMarca);

// var valor = null;
// if (coche != null) {
//   var aire = coche.getAireCondicionado();
//   if (aire != null) {
//     var ionizador = aire.getIonizador();
//     if (ionizador != null) {
//       valor = ionizador.getMarca();
//     }
//   }
// }
```

El método ```stream```, introducido en Java 9, retorna un ```Stream``` con el valor si está presente, en caso contrario un ```Stream``` vacío:

```java
var stream = opcional.stream();

// var stream = opcional == null ? Stream.empty() : Stream.of(opcional);
```

El método ```filter``` retorna un nuevo ```Optional``` si el predicado dado se cumple para el valor si está presente, en caso contrario un ```Optional``` vacío:

```java
var resultado = opcional.filter(x -> !x.isEmpty());

// var resultado = (opcional == null || opcional.isEmpty()) ? null : opcional;
```

El método ```ifPresent``` ejecuta un método con el valor si está presente, en caso contrario no hace nada:

```java
opcional.ifPresent(System.out::println);

// if (opcional != null) System.out.println(opcional);
```

El método ```ifPresentOrElse```, introducido en Java 9, ejecuta un método con el valor si está presente, en caso contrario ejecuta una acción alternativa sin ningún valor:

```java
opcional.ifPresentOrElse(System.out::println, () -> System.out.println("else"));

// if (opcional == null) {
//   System.out.println(“else”);
// } else {
//   System.out.println(opcional);
// }
```

La propia clase ```Stream``` de Java utiliza el tipo ```Optional``` como resultado de algunos de sus métodos evitando las comprobaciones de valores nulos. Y de hecho es con estos casos de uso en mente para los que realmente se diseñó ```Optional```. Los métodos ```findAny```, ```findFirst```, ```max```, ```min``` y ```reduce``` de la clase ```Stream``` retornan todos ellos un ```Optional```. De esta forma, una búsqueda, de por ejemplo un máximo con el método ```max```, no interrumpe la cadena de procesamiento. No es necesario cortar la cadena y comprobar si el método retornó ```null```. Se puede seguir con la cadena y aplicar sobre el resultado cualquiera de los métodos vistos para la clase ```Optional``` que cubren la mayoría de las casuísticas.
