# 2. Scala (2)

_23-11-2018_ _Juan Mellado_

Después de la instalación del entorno de trabajo es hora de dar los primeros pasitos con Scala. Lo más básico es conocer las estructuras más sencillas y aprender como definir valores, variables, funciones, métodos, clases, ... sin entrar en profundidad en todas sus capacidades.

## 2.1. Values

Un valor, según la definición formal de Scala, es un nombre que se le asigna al resultado de una expresión.

```scala
val x = 1 + 1
```

Una vez asignado un valor no se puede cambiar, de forma similar a un variable de tipo ```final``` en Java.

```scala
x = 2 // error
```

Scala infiere automáticamente el tipo de los valores, aunque también permite indicarlos de forma explícita.

```scala
val x: Int = 1 + 1
```

## 2.2. Variables

Las variables son como los valores, pero se pueden reasignar. Es decir, son como las clásicas variables presentes en la mayoría de lenguajes de programación.

```scala
var x = 1

x = 2 // correcto
```

De igual forma que con los valores, Scala infiere el tipo en función de la asignación, pero se puede indicar de forma explícita.

```scala
var x: Int = 1
```

## 2.3. Blocks

Un bloque es una secuencia de expresiones, de forma que el resultado del bloque es el resultado de la última expresión.

```scala
println({
  val x = 1
  x + 1
}) // 2
```

En Scala existe la palabra reservada ```return```, pero no suele utilizarse.

## 2.4. Functions

Las funciones son expresiones que admiten cero o más parámetros.

Pueden ser anónimas, como las expresiones lambda presentes en algunos lenguajes, o estar asociadas a un nombre.

```scala
(x: Int) => x + 1

val incr = (x: Int) => x + 1

println(incr(1)) // 2
```

Si el cuerpo de una función necesita más de una línea entonces se utiliza un bloque.

## 2.5. Methods

Los métodos son similares a las funciones, pero declaran un tipo de retorno.

```scala
def incr(x: Int): Int = x + 1

println(incr(1)) // 2
```

La lista de parámetros es opcional, e incluso se pueden indicar varias listas de parámetros.

```scala
def answer: Int = 42;

def add(x: Int)(y: Int): Int = x + y;
```

## 2.6. Classes (1)

La forma de definir una clase en Scala es similar a la forma en que se hace en otros lenguajes de programación. Una característica interesante es que junto con el propio nombre de la clase se definen los parámetros del constructor.

```scala
class Greeter(prefix: String, suffix: String) {

  def greet(name: String): Unit =
    println(prefix + name + suffix)
}
```

En el ejemplo, copiado directamente de la documentación de Scala, se observa que se define un método que retorna el tipo ```Unit```. Este tipo se utiliza de forma similar a como se utiliza ```void``` en Java, para indicar que no se quiere retornar ningún resultado. Pero en la práctica es una instancia _singleton_ de tipo ```Unit```, ya que Scala requiere que todas las expresiones evalúen a un valor.

Las clases se instancian utilizando la palabra reservada ```new```.

```scala
val greeter = new Greeter("¡Hola, ", "!")

greeter.greet("mundo") // ¡Hola, mundo!
```

## 2.7. Case Classes

Las clases de tipo case permiten definir objetos inmutables.

```scala
case class Point(x: Int, y: Int)
```

Se instancian sin utilizar la palabra reservada ```new```.

```scala
val point = Point(1, 2)
```

Y se caracterizan porque se comparan por valor.

```scala
if (Point(1, 2) == Point(1, 2)) { // true
```

Lo que quiere decir que dos objetos inmutables creados con los mismos valores son equivalentes, representan dos áreas de memoria inicializadas con los mismos valores, y por tanto intercambiables.

Las clases de este tipo implementan el método ```copy``` que permite crear copias de objetos ya existentes variando uno o más de sus valores miembro.

```scala
val point1 = Point(1, 2)
val point2 = point1.copy(y = 3)
```

Este método es importante desde el punto de vista de la programación funcional, que favorece el uso de objetos inmutables que carecen de estado interno, y recomienda que se creen nuevos objetos clonando los originales cuando se tenga que modificar alguno de sus valores.

## 2.8. Objects

Un ```object``` en Scala es a la vez la definición de una clase y un objeto singleton de dicha clase.

```scala
object Sequence {
  private var current = 0

  def next(): Int = {
    current += 1
    current
  }
}
```

Se acceden a ellos por su nombre, de forma similar a como se accede a los métodos estáticos de una clase en muchos lenguajes de programación:

```scala
println(Sequence.next());
println(Sequence.next());
```

## 2.9. Traits

Un ```trait``` en Scala es una definición de tipo, lo que incluye tanto atributos como métodos.

```scala
trait Predicate {
  def test: Boolean
}
```

Una forma simple de entender esta construcción es verla como una ```interface``` de Java 8 o superior. Un ```trait``` puede heredar de otros, puede tener implementaciones de métodos por defecto y puede hacer un ```override``` de los métodos de los padres.

## 2.10. Types

Scala define el tipo ```Any``` como padre de todos los tipos, y que por su semejanza con Java, define los métodos ```equals```, ```hasCode``` y ```toString```.

Los tipos ```AnyVal``` y ```AnyRef``` son hijos directos de ```Any```. El primero representa tipos de valores no nulables, y el segundo tipos de referencias.

Los tipos de valores no nulables disponibles son ```Double```, ```Float```, ```Long```, ```Int```, ```Short```, ```Byte```, ```Char```, ```Boolean``` y ```Unit```.

El tipo de referencia ```AnyRef``` es equivalente a ```java.lang.Object``` en Java, y es el padre de todos los tipos propios definidos por los usuarios.

El tipo ```Null``` es un subtipo de todos los tipos de referencias, tiene un único valor representado por la palabra reservada ```null```, y normalmente no se utiliza dentro de los programas escritos en Scala, existe más por compatibilidad con Java.

El tipo ```Nothing``` es un subtipo de todos los tipos y se utiliza para indicar una terminación o proceso anormal, como por ejemplo el lanzamiento de una excepción o un bucle infinito.

## 2.11. Classes (2)

De vuelta a las clases, comentar que una clase puede contener valores, variables, métodos, tipos, ```objects```, ```traits```, u otras clases. La clase más sencilla que se puede definir es aquella que no contiene ningún miembro.

```scala
class Point

val point = new Point
```

Todas las clases tienen un constructor sin parámetros por defecto, aunque como ya se mencionó anteriormente, se pueden definir parámetros en la propia declaración de la clase.

```scala
class Point(var x: Int, var y: Int) {

  def move(dx: Int, dy: Int): Unit = {
    x = x + dx
    y = y + dy
  }

  override def toString: String = s"($x, $y)"
}
```

Los parámetros pueden tener valores por defecto y hacer referencia a ellos por su nombre.

```scala
class Point(var x: Int = 0, var y: Int = 0) {
...

val p1 = new Point
val p2 = new Point(1, 2)
val p3 = new Point(y = 3)
```

Para hacer miembros privados se puede utilizar la palabra reservada ```private```, siendo la norma utilizar un guión bajo como prefijo del miembro privado. Para implementar un ```getter``` la norma es utilizar un método con el mismo nombre del miembro privado pero sin guión bajo y sin lista de parámetros. Y para implementar un ```setter``` la norma es utilizar el nombre del miembro privado pero con el sufijo ```_=```.

```scala
class Point {
  private var _x = 0
  private var _y = 0

  def x = _x
  def x_= (value: Int): Unit = _x = value

  def y = _y
  def y_= (value: Int): Unit = _y = value
...
```

Para hacer privados los parámetros del constructor se debe omitir la palabra reservada ```var``` o ```val``` en su declaración.

```scala
class Point(x: Int, y: Int) {
...
```

## 2.12. Traits (2)

El ```trait``` más sencillo que puede definirse es el que no contiene ningún miembro, pero resultan de mayor utilidad cuando tienen tipos genéricos y métodos abstractos.

```scala
trait Iterator[A] {

  def hasNext: Boolean

  def next(): A
}
```

En el ejemplo, se define la clásica interface para un iterador de tipo genérico A.

Una clase implementa un ```trait``` utilizando la palabra reservada ```extends```.

```scala
class IntIterator(to: Int) extends Iterator[Int] {

  private var current = 0

  override def hasNext: Boolean = current < to

  override def next(): Int = {
    if (hasNext) {
      val t = current
      current += 1
      t
    } else 0
  }
}
```

Llegado este punto queda claro que Scala hereda muchas características de Java, pero también que añade muchas propias, en particular para reducir la cantidad de código a escribir y proporcionar algunas estructuras básicas dentro del ámbito de la programación funcional.

Queda mucho por descubrir todavía.
