# 5. Scala (y 5)

_06-12-2018_ _Juan Mellado_

Finalizando el _tour_ de Scala.

## 5.1. Self-Type

Un _auto-tipo_ en Scala indica que un ```trait``` extiende de otro. Es similar al uso de la palabra reservada ```extends```, pero utiliza una nomenclatura distinta y no obliga a importar el ```trait``` del que se extiende.

Resulta un tanto confuso en un principio, pero por lo que podido leer al respecto, la motivación principal es permitir modelar relaciones de tipo "_requiere un_" frente a las más clásicas "_extiende de_". Algo que resulta de utilidad para implementar algunos patrones, en particular el de inyección de dependencias.

Para definir un _auto-tipo_ hay que definir un identificador, normalmente ```this```, seguido del nombre del ```trait``` del que se quiere obligar a extender, seguido del símbolo ```=>```.

```scala
trait Vehicle {
  def brand: String
}

trait Car {
  this: Vehicle =>

  def drive(): Unit = println("driving")
}

class Batmovil() extends Car with Vehicle {
  def brand = "batmovil"
}

val car = new Batmovil()
car.drive()
```

La declaración del _auto-tipo_ obliga a extender con ```with``` de ```Vehicle```, y por consiguiente, a definir el miembro ```brand```. El compilador generará un error si no se implementa dicho requerimiento.

## 5.2. Implicit Parameters

Un método puede declarar una o más listas de parámetros con la palabra reservada ```implicit```. Esto hace que dichos parámetros sean opcionales y que Scala trate de proporcionar un valor para los mismos si no se pasan de manera explícita.

```scala
def scale(x: Int)(implicit y: Int) = x * y

implicit val y: Int = 5

scale(5) // 25
```

En el código de ejemplo anterior, Scala resuelve en tiempo de ejecución el valor de y para completar la llamada a ```scale```. En este sencillo ejemplo casan tanto el nombre como el tipo, pero podría cambiarse el nombre y aún así Scala seguiría casando el valor. Si ningún valor tuviera el mismo tipo entonces Scala trataría de resolver el valor atendiendo a unas determinadas reglas de conversión de tipo y resolución de contexto.

La resolución de los valores implícitos es un tanto compleja. La documentación oficial contiene un apartado dedicado de forma específica a explicar los detalles del proceso. Es conveniente revisarlo si se quiere comprender como implementa Scala esta característica, ya que es materia para un artículo completo.

## 5.3. Implict Conversions

Las conversiones implícitas en Scala son similares a los _castings_ automáticos de algunos lenguajes de programación, pero con algunos matices propios de Scala.

Si se espera un valor de tipo ```T``` y se proporciona un valor de tipo ```S``` entonces se trata de hacer la conversión. En este sentido, Scala tiene un paquete llamado ```scala.Predef``` con un conjunto de conversores predefinidos.

```scala
import scala.language.implicitConversions

implicit def int2Integer(x: Int)= java.lang.Integer.valueOf(x)
```

En el código de ejemplo anterior se convierte un valor de tipo ```scala.Int``` en otro de tipo ```java.lang.Integer```.

El compilador avisa con _warnings_ cuando detecta alguna de estas conversiones implícitas, que se pueden silenciar importando el paquete ```scala.language.implicitConversions```.

## 5.4. Polymorphic Methods

Scala denomina métodos polimórficos a los métodos con tipos parametrizados. Estos métodos se definen y comportan de manera similar a las clases genéricas.

```scala
def repeat[A](x: A, length: Int): List[A] =
  if (length < 1) Nil else x :: repeat(x, length - 1)

println(repeat[Int](3, 4)) // 3, 3, 3, 3
println(repeat("Hi", 5)) // Hi, Hi, Hi, Hi, Hi
```

En el código de ejemplo anterior se observa que el tipo parametrizado se puede indicar de forma explícita, o dejar que sea el propio Scala el que lo determine.

## 5.5. Type Inference

En línea con lo comentado en el apartado anterior, y lo visto a lo largo de estos artículos, Scala es capaz de inferir los tipos de determinadas expresiones sin necesidad de que se indiquen de forma explícita.

Se puede omitir el tipo en la declaración de un valor o variable inicializada en su definición.

```scala
val entero = 5
```

En algunos casos se puede omitir el tipo resultante de un método o función cuando se puede inferir automáticamente del resultado.

```scala
def square (x: Int) = x * x
```

Se puede omitir el tipo parametrizado en la instanciación de clases genéricas o métodos polimórficos cuando se puede inferir del contexto.

```scala
val lista = List(1, 2, 3)
```

Scala no puede inferir los tipos de los parámetros, excepto en funciones anónimas pasadas como parámetros.

```scala
List(1, 2, 3).map(x => x * x)
```

Por último, comentar que Scala no puede inferir el tipo del resultado en el caso de funciones recursivas.

```scala
def factorial(n: Int) = if (n == 0) 1 else n * factorial(n - 1) // Error
```

## 5.6. Operators

Scala permite definir operadores, de igual forma que otros lenguajes de programación, tratándolos como métodos de la clase dentro de la que se definen.

```scala
case class Vector(val x: Int, val y: Int) {
  def +(other: Vector) = Vector(this.x + other.x, this.y + other.y)
}

val vec1 = Vector(1, 2)
val vec2 = Vector(3, 4)
val vec3 = vec1 + vec2

println(vec3) // 4, 6
```

De forma alternativa, los operadores pueden escribirse utilizando la notación punto como cualquier otro método.

```scala
vec1.+(vec2)
```

## 5.7. By-name Parameters

Los parámetros por nombre son una forma que tiene Scala de permitir indicar que el valor de un parámetro debe evaluarse cada vez que se acceda a él dentro del cuerpo de un método, en vez de cuando se realice la llamada al método.

Notar que el nombre de esta característica resulta un poco confusa, y que además, aunque lo parezca, no es totalmente equivalente a una evaluación perezosa. De hecho, Scala tiene la palabra reservada lazy para evaluar perezosamente un valor.

```scala
def execute(time: Long) {
  println(time)
  Thread.sleep(1000L)
  println(time)
}

execute(System.nanoTime())
```

En el código de ejemplo anterior el valor impreso es siempre el mismo, ya que se evalúa una sola vez, justo cuando se invoca al método execute. Y aunque este es el comportamiento por defecto esperado, Scala permite modificarlo utilizando los parámetros por nombre. Es una característica curiosa del lenguaje.

Los parámetros por nombre se denotan anteponiendo el símbolo ```=>``` delante del tipo del parámetro.

```scala
def execute(time: => Long) {
  println(time)
  Thread.sleep(1000L)
  println(time)
}

execute(System.nanoTime())
```

En el nuevo código modificado el valor impreso es diferente cada vez, ya que la expresión pasada como parámetro ahora es evaluada cada vez que se accede a ella.

En buena lógica, el caso de uso habitual de esta característica es del demorar la ejecución de procesos computacionalmente costosos hasta que sea realmente necesario acceder al resultado de los mismos.

## 5.8. Annotations

Scala permite utilizar las anotaciones definidas en Java de la misma forma que se hacen en el propio código Java.

```scala
@deprecated
def old(): Unit = ()
```

La documentación de Scala menciona un conjunto de anotaciones de propósito bien conocido como ```@deprecated```, ```@deprecatedName```, ```@transient```, ```@volatile```, ```@SerialVersionUID``` y ```@throws```. Algunas más específicas como ```@scala.beans.BeanProperty``` y ```@scala.beans.BooleanBeanProperty```. Y algunas que afectan a la forma en la que compilador genera el código como ```@unchecked```, ```@uncheckedStable``` y ```@specialized```.

Respecto a la construcción de anotaciones propias, la especificación las menciona en un apartado, pero no proporciona ningún ejemplo.

## 5.9. Default Parameter Values

El _tour_ de Scala reintroduce en este punto un apartado acerca de los valores por defecto de los parámetros. Algo ya visto anteriormente.

```scala
case class Point(val x: Int = 0, val y: Int = 0)
```

## 5.10. Named Arguments

En la misma línea que en el apartado anterior, el _tour_ de Scala comenta en este punto que los parámetros pueden pasarse en cualquier orden indicando su nombre.

```scala
case class Point(val x: Int, val y: Int)

val p1 = Point(x = 1, y = 2)
val p2 = Point(y = 4, x = 3)
```

## 5.11. Packages and Imports

Las clases en Scala se organizan en paquetes de igual forma que en Java, aunque no se fuerza a que la ruta del fichero coincida con el nombre del paquete.

```scala
package com.example.services
```

Los paquetes se pueden definir de forma anidada utilizando bloques.

```scala
package com.example {
  package services {
  }
  package controllers {
  }
}
```

La sentencia ```import``` es similar a la de Java, pero puede utilizarse en cualquier punto, no sólo al principio de los ficheros, usa el caracter ```_``` en vez de ```*``` para importar todos los miembros de un paquete, permite importar varias clases en una sola línea, e incluso permite renombrar un miembro al ser importado.

```scala
import com.example.services._
import com.example.services.UserService
import com.example.services.{UserService, RoleService}
import com.example.services.{UserService => Users}
```
