# 7. Java: Optional (1)

_30-04-2018_ _Juan Mellado_

Java 8 introdujo el tipo ```java.util.Optional``` con el objeto de mejorar la expresividad de algunas interfaces y simplificar ciertos patrones de código. Amada y odiada por igual, la mayoría de los desarrolladores sólo se acuerdan de ella como la clase que prometía evitar el famoso ```NullPointerException```.

```Optional``` es un contenedor de valores. Pudiendo el valor estar presente o no. Así de simple. La clase está declarada de forma genérica como ```Optional<T>```, siendo ```T``` el tipo del valor contenido, y se incluyen además de forma nativa clases especializadas como ```OptionalInt```, ```OptionalLong``` y ```OptionalDouble```.

La principal utilidad que se espera obtener de esta clase es su uso como tipo retornado por un método. Un método declarado como ```Optional<T> findFirst()``` tiene un contrato más estricto que otro declarado simplemente como ```T findFirst()```. No hace falta leer la documentación del primer método para saber que el valor retornado es opcional, y que nunca retornará una referencia nula, es decir, un ```null```.

Normalmente muchos desarrolladores dejan de leer a partir del párrafo anterior. ¿El motivo? Tienen que escribir ocho caracteres más (O-p-t-i-o-n-a-l). Pero sobre todo porque nadie, ni Java, puede garantizar que ninguna de todas las posibles implementaciones de dicho método no retornará nunca un ```null```, independientemente de lo que estipule su contrato.

Una de las claves de todo el asunto es entender que una referencia a una instancia de tipo ```Optional``` siempre debe tener valor, es decir, nunca debe ser nula. Nunca debe escribirse código que inicialice una variable de tipo ```Optional``` a ```null```. NUNCA debe escribirse ```Optional variable = null;```.

La clase ```Optional``` ofrece distintos métodos estáticos a modo de factorías para construir instancias de ella misma:

```java
Optional.empty();

Optional.ofNullable(valor);

Optional.of(valor);
```

El método ```empty``` retorna una instancia vacía. El método ```ofNullable``` retorna una instancia que contiene el valor pasado como parámetro, o la instancia vacía si el valor es nulo. Y el método ```of``` retorna una instancia que contiene el valor pasado como parámetro, pero eleva ```NullPointerException``` si el valor es nulo. ¿```NullPointerException```? Aquí dejan de leer los desarrolladores que no lo hicieron hace un par de párrafos.

La clase ```Optional``` se introdujo para suplir una carencia de las referencias en Java. Una referencia no es más que un puntero a una dirección de memoria. Un puntero siempre tiene valor. No existe el concepto de puntero no inicializado. Se utiliza ```null``` a forma de número mágico. Java eleva una excepción si se intenta operar con una referencia/puntero que sea igual a dicho número mágico.

Los desarrolladores son los responsables de comprobar que su código no opera sobre referencias nulas:

```java
if (variable == null) {
```

Por su parte, la clase ```Optional``` tiene el método ```isPresent```, que permite comprobar si contiene algún valor o no:

```java
if (opcional.isPresent()) {
```

Otras de las claves de todo el asunto es entender que ```Optional``` no se añadió a Java para cambiar la comparación con ```null``` por la llamada al método ```isPresent```. Si este hubiera sido el único motivo entonces el cambio hubiera sido totalmente contraproductivo, ya que usar ```Optional``` implica gastar un poco más de memoria al tener que crear un objeto contenedor, y es un poco más lento al tener que invocar un método para comprobar si el objeto contenedor está vacío o no.

Para recuperar el valor contenido en un Optional se debe utilizar el método ```get```:

```java
var valor = opcional.get()
```

Si la variable de tipo ```Optional``` está vacía entonces el método ```get``` eleva ```NoSuchElementException```. _Wat!?_ En este punto si que ya no queda ningún desarrollador leyendo. Pero es que otra clave más de todo el asunto es entender que ```Optional``` no se añadió para sustituir la excepción ```NullPointerException``` por otra. No se debe intentar obtener el valor de un ```Optional``` si no se sabe si está presente o no. Eso sería lo mismo que intentar utilizar una referencia a un objeto sin comprobar antes que no es ```null```.

No entender los puntos claves de todo el asunto vistos hasta ahora hace que se escriba código INCORRECTO como el siguiente:

```java
if (opcional != null && opcional.get() != null)
```

Ni un variable de tipo ```Optional``` debe ser null. Ni la forma correcta de comprobar si contiene algún valor es recuperarlo y compararlo con ```null```. Una variable de tipo ```Optional``` nunca debe ser ```null```. Y debe utilizarse el método ```isPresent``` para comprobar si contiene algún valor.

Java introdujo ```Optional``` para propiciar el uso de determinadas prácticas de diseño. Para la implementación de determinados patrones, sobre todo del ámbito de la programación funcional. No se trata de sustituir todos los tipos de parámetros, variables y valores retornados por ```Optional```. De hecho no se recomienda para métodos que retornen colecciones, prefiriéndose retornar colecciones vacías inmutables en dicho caso. ```Optional``` sólo debe utilizarse como tipo retornado por un método cuando aporte alguna ventaja significativa.
