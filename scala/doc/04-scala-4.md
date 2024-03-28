# 4.4 Scala (4)

_03-12-2018_ _Juan Mellado_

Más cosas de Scala.

## 4.1. For Comprehensions

Scala implementa el concepto de generador, presente en otros lenguajes de programación, mediante la combinación de las palabras reservadas ```for``` y ```yield```.

El uso más sencillo que se puede hacer de esta característica es el de procesar secuencias de valores y transformarlos en otros valores, de forma similar a como funciona el método ```map``` sobre una colección.

```scala
def squares(values: List[Int]): List[Int] =
  for (value <- values) yield value * value

val values = List(1, 2, 3)

println(squares(values)) // 1, 4, 9
```

Dentro del ```for``` puede haber varias expresiones separadas por punto y coma, llamadas enumeradores, y pueden ser tanto generadores como filtros con guardas.

```scala
def odds(values: List[Int]): List[Int] =
  for (
    value <- values if (value & 1) == 1
  ) yield value

val values = List(1, 2, 3, 4, 5, 6)

println(odds(values)) // 1, 3, 5
```

Otra forma de utilizar esta expresión es con la palabra reservada ```until```, que hace que se comporte como un bucle for de una forma más tradicional.

```scala
case class Point(val x: Int, val y: Int)

def grid(n: Int): Seq[Point] =
  for (
    x <- 0 until n;
    y <- 0 until n
  ) yield Point(x, y)

println(grid(2)) // Point(0,0), Point(0,1), Point(1,0), Point(1,1)
```

Esta construcción es más importante de lo que a priori pueda parecer, y Scala permite aplicarla sobre cualquier tipo que implemente los métodos ```withFilter```, ```map``` y ```flatMap```.

## 4.2. Generic Classes

Una clase genérica es aquella que admite un tipo o más como parámetro. El concepto en Scala es el mismo que se encuentra presente en Java desde su versión 5, cuando se introdujeron los _Generics_.

El caso de uso más habitual es el de la colecciones de elementos, pero es aplicable a cualquier tipo abstracto de datos, e incluso a funciones o métodos.

```scala
class Stack[A] {
  private var elements: List[A] = Nil
  def push(x: A) { elements = x :: elements }
  def peek: A = elements.head
  def pop(): A = {
    val currentTop = peek
    elements = elements.tail
    currentTop
  }
}
```

En el código de ejemplo, copiado directamente de la documentación de Scala, se observa que el tipo parametrizado A se indica utilizando corchetes. Puede indicarse más de uno utilizando la coma como separador.

```scala
val stack = new Stack[Int]

stack.push(1)
stack.push(2)

println(stack.pop) // 2
println(stack.pop) // 1
```

## 4.3. Variances

El concepto de varianza se utiliza en Scala para describir las relaciones entre los tipos de una jerarquía de tipos, e indicar cuales de ellos son aceptables cuando se utilizan genéricos.

Scala soporta invarianza, covarianza y contravarianza. El símbolo ```+``` delante del tipo indica covarianza, el simbolo ```-``` contravarianza, y la ausencia de símbolo indica invarianza.

```scala
class Invariant[A]

class Covariant[+A]

class Contravariant[-A]
```

La invarianza quiere decir que los únicos elementos aceptados son aquellos que son exactamente del tipo ```A```. Este es el comportamiento por defecto en Scala.

```scala
trait Operator[A] {
  def identity(value: A): A = value
}

object IntOperator extends Operator[Int]

println(IntOperator.identity(123))
println(IntOperator.identity("abc")) // Error de compilación, String no es Int
```

La covarianza quiere decir que los únicos elementos aceptados son aquellos que son de tipo ```A``` o cualquier subtipo de ```A```. Por ejemplo, las listas en Scala son covariantes.

```scala
abstract class Vehicle

case class Car() extends Vehicle

case class Motorcycle() extends Vehicle

val vehicles: List<Vehicle> = List(Car(), Motorcycle())
```

La contravarianza quiere decir que los únicos elementos aceptados son aquellos que son de tipo ```A``` o cualquier supertipo de ```A```. Esta forma de varianza es la que resulta habitualmente menos intuitiva. Lo que pretende es forzar que se cumpla la regla de que un tipo ```T``` es un subtipo de ```A``` si se puede utilizar un valor de tipo ```T``` cuando se requiere un valor de tipo ```A```, que es lo que se conoce como "_Principio de Sustitución de Liskov_".

```scala
trait Writer[-T] {
  def write(value: T)
}
```

En este último código de ejemplo, la clase ```Writer``` fuerza a que una instancia del tipo parametrizado ```T``` sólo pueda reemplazarse por otra del mismo tipo ```T``` o algún supertipo de la misma. De esta forma, una instancia de ```Writer<String>``` no puede utilizarse como sustituta de otra instancia de tipo ```Writer<AnyRef>```, ya que una instancia que sólo soporta cadenas de texto no es intercambiable con otra que soporta referencias a cualquier tipo de objeto.

## 4.4. Upper Type Bounds

La expresión ```T <: A``` en Scala quiere decir que ```T``` es un subtipo de ```A```.

Sirve para limitar los tipos de valores admisibles en un genérico. Es equivalente a utilizar ```<? extends A>``` en Java.

## 4.5. Lower Type Bounds

La expresión ```T >:``` A en Scala quiere decir que ```T``` es un supertipo de ```A```.

Sirve para limitar los tipos de valores admisibles en un genérico. Es equivalente a utilizar ```<? super A>```  en Java.

```scala
trait Node[+B] {
  def prepend[U >: B](elem: U): Node[U]
}

case class ListNode[+B](h: B, t: Node[B]) extends Node[B] {
  def prepend[U >: B](elem: U): ListNode[U] = ListNode(elem, this)
  def head: B = h
  def tail: Node[B] = t
}

case class NilNode[+B]() extends Node[B] {
  def prepend[U >: B](elem: U): ListNode[U] = ListNode(elem, this)
}
```

En el código del ejemplo, adaptado de la documentación oficial de Scala, se observa como se utiliza ```U >: B``` en los métodos para forzar que los elementos contenidos en los nodos sean de un determinado tipo ```B```, o alguno de sus supertipos, y el valor retornado sea de dicho tipo ```B```.

## 4.6. Inner Classes

Scala permite definir clases embebidas dentro de otras clases. Java también lo permite, pero mientras que en Java la clase interna pertenece a la clase contenedora, en Scala la clase interna pertenece a las instancias de la clase contenedora.

```scala
class Stream {
  class Item()

  def produce(): Item = new Item
  def consume(item: Item): Unit = ()
}

val stream1 = new Stream
val stream2 = new Stream

val item = stream1.produce()
stream2.consume(item) // Error de compilación stream1.Item no es stream2.Item
```

En el código de ejemplo se produce un error de compilación porque la clase ```Item``` es diferente para ```stream1``` y ```stream2```. Cada variable tiene su propia definición de ```Item```.

Si el comportamiento por defecto de Scala no es el deseado, se pueden cambiar los métodos para que trabajen con una referencia a la clase interna de la clase contenedora, en vez de a la clase interna de cada instancia. Para ello hay que indicar la ruta completa a la clase interna mediante ```Stream#Item```.

```scala
class Stream {
  class Item()

  def produce(): Stream#Item = new Item
  def consume(item: Stream#Item): Unit = ()
}
```

## 4.7. Abstract Types

Scala permite definir clases con tipos abstractos. Es una característica similar a la definición de clases genéricas, pero con alguna diferencia de orden semántico. La idea es que se puedan definir tipos abstractos en una clase de igual forma en la que, por ejemplo, se declaran métodos abstractos.

```scala
abstract class Sequence {
  type U
}

class IntSequence extends Sequence {
  type U = Int
  
  private var current: U = 0
  
  def next(): U = {
    current += 1
    current
  }
}
```

Es posible incluso definir una clase abstracta en base a otra y redefinir el tipo abstracto.

## 4.8. Compound Types

Como ya se vio anteriormente, Scala soporta _mixins_, que de forma simplificada se puede decir que es la unión de varias interfaces. Lo interesante de esta característica es que su uso no está restringido a la definición de las clases, sino que se puede aplicar en otras construcciones, como por ejemplo en los tipos de los parámetros de las funciones.

```scala
def maximize(widget: Moveable with Resizable): Unit = {
  widget.move(viewport.origin);
  widget.resize(viewport.size);
}
```

Utilizando la palabra reservada ```with``` se pueden concatenar dos o más interfaces y crear un tipo compuesto. Es similar a la unión de tipos que soportan algunos lenguajes de programación.
