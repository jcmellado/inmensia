# 1. Scala (1)

_19-11-2018_ _Juan Mellado_

Scala es un lenguaje de programación que se ejecuta sobre la máquina virtual de Java y aspira a sacar partido de lo mejor de los dos paradigmas actuales dominantes, es decir, de la programación orientada a objetos y de la programación funcional.

Por necesidades del guión estoy ahora revisando sus características, y como de costumbre he decidido publicar mis notas personales para futuras referencias. La idea es hacer una instalación desde cero, evitando utilizar _wizards_ al principio, y realizando una configuración de forma manual. El plan es ejecutar primero un ejemplo sencillo desde línea de comandos, a continuación instalar y configurar un IDE, y por último realizar un "_tour_" por el lenguaje siguiendo el guión propuesto en la propia página oficial de Scala.

## 1.1. JVM

Scala necesita una máquina virtual de Java para compilar y ejecutarse. La versión actual del JDK es la ```11.0.1```, que puede descargarse en formato zip y descomprimirse en cualquier directorio ```<JDK>```.

Basta con añadir el directorio ```<JDK>/bin``` al ```PATH``` del sistema para poder ejecutar Java desde línea de comandos:

```text
set PATH=<JDK>/bin

java -version

java version "11.0.1" 2018-10-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.1+13-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.1+13-LTS, mixed mode, sharing)
```

No obstante, antes de continuar, es recomendable crear el fichero de clases compartidas precompiladas de Java. Este fichero aumenta ligeramente el rendimiento, particularmente el arranque de la máquina virtual. Para crearlo hay que ejecutar la siguiente instrucción desde línea de comandos:

```text
java -Xshare:dump
```

Si la ejecución termina de forma correcta se creará un fichero ```classes.jsa``` en el directorio ```<JDK>/bin/server```.

## 1.2. Scala

La versión actual de Scala es la ```2.12.7```, que puede descargarse en formato zip y descomprimirse en cualquier directorio ```<SCALA>```.

Basta con añadir el directorio ```<SCALA>/bin``` al ```PATH``` del sistema para poder ejecutar Scala como un REPL desde línea de comandos:

```text
set PATH=<JDK>/bin;<SCALA>/bin

scala -version

Scala code runner version 2.12.7 -- Copyright 2002-2018, LAMP/EPFL and Lightbend, Inc.
```

Para compilar un programa se debe utilizar ```scalac```, que prácticamente es igual que ```javac```, el compilador de Java.

```text
¡Hola, mundo!
```

Como primer ejemplo de compilación se puede utilizar el clásico "_¡Hola, mundo!_", que sin entrar en detalles, se puede crear en un fichero ```HolaMundo.scala``` con el siguiente código:

```scala
object HolaMundo extends App {
  println("¡Hola, mundo!")
}
```

La compilación se realiza ejecutando la siguiente instrucción desde línea de comandos:

```text
scalac HolaMundo.scala
```

Como resultado de la compilación se crearán tres ficheros: ```HolaMundo.class```,  ```HolaMundo$.class``` y ```HolaMundo$delayedInit$body.class``` en el propio directorio donde se lanza la compilación, y que pueden ejecutarse con la siguiente instrucción desde línea de comandos:

```text
scala HolaMundo
```

O de forma más abreviada, compilando y ejecutando con una sola línea:

```text
scala HolaMundo.scala
```

En buena lógica, un desarrollo moderno implica el uso de un IDE. No obstante, conocer como funcionan las cosas a más bajo nivel puede ayudar a futuro a resolver problemas, o aspirar a usos más avanzados del lenguaje, su entorno y herramientas.

## 1.3. IntelliJ IDEA

La versión gratuita actual del popular IDE es la ```2018.2.5```, que puede descargarse en formato zip y descomprimirse en cualquier directorio ```<INTELLIJ>```.

Basta con añadir el directorio ```<INTELLIJ>/bin``` al PATH del sistema para poder ejecutar IntelliJ (64 bits) desde línea de comandos:

```text
idea64
```

Tras aceptar la licencia, y decidir si se quiere enviar o no informes de uso, se puede crear un nuevo proyecto.

La instalación por defecto no tiene instalado el _plugin_ de Scala. Se puede instalar desde la propia ventana de de creación de proyecto a través de "_Configure > Plugins > Install JetBrains plugin... > Scala > Install_". Después de reiniciar el IDE aparecerá la opción para crear un proyecto de tipo Scala con tres plantillas, siendo la basada en ```sbt``` la recomendada por defecto, siendo ```sbt``` una herramienta de construcción de proyectos en Scala que funciona de forma similar a como lo hacen otras más populares como Maven o Gradle.

A partir de aquí la creación de un nuevo proyecto puede realizarse seleccionando la versión de Scala y ```sbt``` que se quiere utilizar. El IDE automáticamente descargará las versiones seleccionadas y creará el proyecto en la ruta indicada con la estructura clásica de directorios de las aplicaciones Java basadas en Maven, pero cambiando el directorio java por scala. Y con esto ya se puede empezar el _tour_ básico de Scala.
