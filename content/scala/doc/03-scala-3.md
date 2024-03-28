# 3. Scala (3)

_24-11-2018_ _Juan Mellado_

Más construcciones propias de Scala.

## 3.1. Tuples

Una tupla representa un conjunto de valores, ya sean del mismo o distinto tipo.

```scala
val currency = ("Euro" , "€", 166.386)
```

Es particularmente útil cuando se debe retornar más de un valor desde una función.

Para acceder a un valor de una tupla se utiliza el ordinal de su posición precedido por un guión bajo.

```scala
println(currency._1)
println(currency._2)
println(currency._3)
```

Aunque también se puede desestructurar la tupla para acceder por nombre.

```scala
val (name, symbol, exchange) = currency
```

Si se necesita indicar el tipo de forma explícita entonces se deben utilizar los tipos ```Tuple2```, ```Tuple3```, ... y así sucesivamente hasta ```Tuple22```.

```scala
val currency = ("Euro", "€", 166.386): Tuple3[String, String, Double]
```

## 3.2. Mixins

Los _mixins_ son estructuras que permiten extender una clase con distintos comportamientos sin necesidad de utilizar herencia múltiple.

Una clase sólo puede extender de una clase padre, pero de tantos _mixins_ como se quiera. La única condición es que la clase padre y los _mixins_ extiendan a su vez de la misma clase.

```scala
trait Artist

trait Writer extends Artist {
  def write(): Unit = println("Call me Ishmael...")
}

trait Musician extends Artist {
  def sing(): Unit = println("Imagine there's no heaven...")
}

class Genius extends Artist with Musician with Writer
```

El uso de _mixins_ favorece la composición frente a la herencia y mejora la intencionalidad del código.

```scala
class Widget extends Window with Scrollable with Resizable
```

## 3.3. Higher-order Functions

Una función de orden superior es aquella que admite una función como parámetro o retorna una función como resultado.

Un ejemplo de método que admite una función como parámetro es ```map```. Este método aplica una función dada a todos los elementos de una secuencia.

```scala
val numbers = Seq(1, 2, 3)

val double = (x: Int) => x * 2

val result = numbers.map(double) // 2, 4, 6
```

La función pasada como parámetro puede ser una _lambda_ anónima embebida.

```scala
val result = numbers.map(x => x * 2) // 2, 4, 6
```

E incluso es posible utilizar una notación abreviada de Scala que utiliza el símbolo ```_``` para identificar los parámetros.

```scala
val result = numbers.map(_ * 2) // 2, 4, 6
```

De forma general, los métodos que admiten funciones o retornan funciones se definen utilizando la "_signatura_" de dichas funciones. Es decir, el tipo de los parámetros que admiten y el tipo de su resultado.

```scala
class Converter[A, B](_converter: A => B) {
  def convert(value: A): B = _converter(value)

  def converter(): A => B = _converter
}
```

## 3.4. Currying

Scala permite que un método tenga múltiple lista de parámetros.

Un ejemplo de este tipo de métodos es ```foldLeft```, que aplica un operador binario ```op``` a una secuencia de valores, de izquierda a derecha, empezando por un valor inicial ```z```.

```scala
def foldLeft[B](z: B)(op: (B, A) => B): B
```

Este método permite por ejemplo sumar todos los números de una secuencia dada.

```scala
val numbers = List(1, 2, 3)
val result = numbers.foldLeft(0)((lh, rh) => lh + rh)
print(result) // 6
```

Y gracias a que el método tiene varias listas de parámetros, se puede utilizar la notación abreviada de Scala utilizando el símbolo ```_``` para representar los parámetros.

```scala
val result = numbers.foldLeft(0)(_ + _)
```

Esto puede resultar un poco confuso al principio, pero tiene la ventaja de que permite definir un método a partir de otro prefijando una o más de sus listas de parámetros.

```scala
val fold = numbers.foldLeft(0)_

val plus = fold(_ + _)
val minus = fold(_ - _)

println(plus) // 6
println(minus) // -6
```

Esta forma de escribir los métodos se asimila con la técnica conocida como _currying_. Esta técnica transforma métodos que admiten múltiples argumentos en una secuencia de métodos que admiten un único argumento cada una de ellos. Por ejemplo, se puede tomar un método que toma dos valores de tipo X e Y y retorna un valor de tipo Z, y transformarlo en un método que toma un valor de tipo X y retorna una función que convierte un valor de tipo Y en un valor de tipo Z.

## 3.5. Pattern Matching

La técnica de _pattern matching_ permite comprobar si un valor cumple un determinado patrón. Scala ofrece una implementación de esta técnica a través de la palabra reservada ```match``` que utiliza una sintaxis similar a la sentencia ```switch``` de Java.

```scala
def test(value: Int): String = value match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}

println(test(1)) // one
println(test(2)) // two
println(test(3)) // other
```

El guión bajo se utiliza para casar cualquier valor, es equivalente a ```default``` en Java.

Además de valores constantes, Scala permite utilizar tipos como patrones.

```scala
abstract class Connector

case class Mail() extends Connector {
  def sendMessage(): Unit = {}
}

case class Chat() extends Connector {
  def sendNotification(): Unit = {}
}

def send(connector: Connector) : Unit = connector match {
  case m: Mail => m.sendMessage()
  case c: Chat => c.sendNotification()
}
```

Una forma particularmente útil de match permite desestructurar los valores de clases de tipo case.

```scala
abstract class Shape

case class Square(val length: Int) extends Shape

case class Circle(val radius: Double) extends Shape

def area(shape: Shape): Double = shape match {
  case Square(length) => length * length
  case Circle(radius) => Math.PI * (radius * radius)
}
```

Los patrones se pueden ampliar utilizando una expresión de guarda para añadir comprobaciones adicionales.

```scala
abstract class Meal

case class Pizza(ingredients: List[String]) extends Meal

case class Burger(ingredients: List[String]) extends Meal

def veganTest(value: Meal): String = value match {
  case Pizza(ingredients) if (ingredients.contains("meat"))
    => "This pizza contains meat"
  case Burger(ingredients) if (ingredients.contains("meat"))
    => "This burger contains meat"
...
```

## 3.6. Regular Expressions

Las expresiones regulares en Scala siguen el patrón habitual de otros lenguajes de programación. Están implementadas por la clase ```Regex``` y se construyen a partir de cadenas de texto que se convierten en expresiones regulares mediante el método ```r```.

```scala
import scala.util.matching.Regex

val digits: Regex = "[0-9]+".r

def isInteger(value: String): Boolean =
  digits.findFirstMatchIn(value) match {
    case Some(_) => true
    case None => false
  }

println(isInteger("123")) // true
println(isInteger("abc")) // false
```

## 3.7. Singleton Objects

Un ```object``` en Scala es a la vez la definición de una clase y un objeto _Singleton_ de dicha clase.

```scala
object Sequence {
  private var current = 0

  def next(): Int = {
    current += 1
    current
  }
}

Sequence.next() // 1
Sequence.next() // 2
```

Este tipo de objetos ya han aparecido anteriormente, pero es necesario reintroducirlos para hablar de los _companion objects_. Un _companion object_ es un objeto que tiene el mismo nombre que una clase. Extiende la clase homónima añadiendo miembros que son visibles por todas las instancias de dicha clase. Es similar a decir que permite añadir métodos o propiedades estáticas a una clase desde otra clase.

Uno de los usos preferidos de esta técnica es la de crear factorías de objetos.

```scala
case class Position(val x: Int, val y: Int, val z: Int)

object Position {
  def center: Position = Position(0, 0, 0)

  def from(other: Position): Position = other.copy()
}

val p1 = Position.center
val p2 = Position.from(p1)
```

La única limitación de esta técnica es que la clase y el _companion object_ tienen que estar definidos en el mismo fichero.

## 3.8. Extractor Objects

Sobre un ```object``` en Scala se puede definir un método de nombre ```apply``` que actúa como factoría, admite parámetros y crea un objeto. De igual forma, se puede definir un método de nombre ```unapply``` que realiza el proceso inverso, admite un objeto y retorna los parámetros que se utilizaron para construirlo.

```scala
object Customer {

  def apply(name: String, surname: String): String = name + " " + surname;

  def unapply(fullName: String): Option[(String, String)] = {
    val parts = fullName.split(' ')
    if (parts.length == 2) Some (parts(0), parts(1))
    else None
  }
}

val customer = Customer("John", "Smith")
println(customer) // John Smith 

val Customer(name, surname) = customer
println(s"$surname, $name") // Smith, John
```
