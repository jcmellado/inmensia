# 4. Java: Inferencia de Tipos de Variables Locales

_12-04-2018_ _Juan Mellado_

Una de las novedades más relevantes de la versión 10 de Java, desde el punto de vista del desarrollador, es la inferencia de los tipos de las variables locales. Lo que quiere decir que ya no es necesario indicar el tipo de las variables al declararlas. El compilador calculará (inferirá) los tipos en función de cómo se inicializan las variables.

En la práctica quiere decir que el siguiente código que declara un par de variables:

```java
String texto = "abc";
List<String> lista = new ArrayList<>();
```

ahora puede escribirse sin indicar los tipos de manera explícita con el nuevo nombre de tipo reservado ```var```:

```java
var texto = "abc";
var lista = new ArrayList<String>();
```

```var``` puede utilizarse sobre variables inicializadas con literales de tipo ```String```, ```char```, ```long```, ```float```, ```double``` y ```boolean```:

```java
var texto = "abc";
var caracter = '\n';
var largo = 42L;
var flotante = 3.14f;
var doble = 3.14d;
var logico = true;
```

```var``` interpreta todos los valores numéricos enteros sin sufijo como de tipo ```int``` por defecto, por lo que las variables de tipo ```byte``` y ```short``` deben seguir siendo declaradas con su tipo de manera explícita o mediante un _cast_:

```java
byte octeto = 1; // var octeto = (byte)1;
short corto = 2; // var corto = (short)2;
var entero = 3; // int
```

```var``` interpreta los valores numéricos con decimales sin sufijo como ```double``` por defecto:

```java
var doble = 3.14; // double
```

La inicialización de variables con tipos numéricos nulables como ```Integer``` o ```Double``` puede realizarse con el método ```valueOf```:

```java
var entero = Integer.valueOf(1);
var doble = Double.valueOf(2d);
```

El uso de ```var``` simplifica la declaración de variables en los bucles de tipo ```for```, evitando tener que repetir el tipo de la colección recorrida, y minimizando el impacto de posibles refactorizaciones:

```java
for (var elemento : coleccion) {
```

El uso de ```var``` evita el clásico patrón de Java de tener que importar interfaces en una clase con objeto de declarar variables inicializadas con una implementación de dicha interface:

```java
var lista = new ArrayList<String>(); // List<String> lista = new ArrayList<>();
```

```var``` interpreta como ```Object``` el tipo de las clases genéricas cuando se utiliza el operador diamante en la inicialización, por lo que el tipo debe indicarse de forma explícita:

```java
var lista = new ArrayList<>(); // List<Object> lista = new ArrayList<>();
```

En todo caso, ```var``` no se puede utilizar sobre variables que no se inicialicen en el momento de declararse:

```java
var x; // ERROR
```

Teniendo en cuenta que Java es un lenguaje fuertemente tipado, al que siempre se ha acusado de ser extremadamente repetitivo, es un cambio importante y que continúa el esfuerzo realizado en la versión 8 con la introducción del operador diamante o las funciones _lambda_.

No se trata tanto de escribir menos código, el ahorro de caracteres netos tecleados, sino de hacerlo más conciso, legible, a la par que favorecer buenas prácticas, como la elección de nombres coherentes, o la minimización de la localidad de las variables, y simplificar algunos patrones de Java, como la declaración de variables en bucles, o la declaración de variables con una interface y su inicialización con una clase que implementa dicha interface.

Y aunque parezca algo novedoso, hay que tener en cuenta que es una característica que se encuentra presente en muchos lenguajes veteranos como C++ o recientes como Go, por citar algunos. Pero que supone un paso en la dirección correcta en todo caso.
