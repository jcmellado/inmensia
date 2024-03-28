# 5. Multi-Release JAR Files

_17-04-2018_ _Juan Mellado_

Si se está desarrollando una librería en la versión 8 de Java, no se puede empezar a utilizar las novedades introducidas en las versiones 10 de la noche a la mañana, ya que ello obligaría a los usuarios de la librería a migrar también a las nuevas versiones de Java si quieren seguir utilizando la librería.

Para intentar mitigar en parte este problema, Java permite compilar el código de forma que funcione en una versión inferior. Por ejemplo, con Java 10 se puede generar código para Java 8, pero evidentemente no se puede utilizar ninguna característica específica de Java 10.

A partir de Java 9 se pueden utilizar ficheros _Multi-Release JAR_ (MRJAR). Un nuevo formato de fichero JAR que permite empaquetar distintas versiones de una misma clase compiladas para ser ejecutadas en distintas versiones de Java.

Supongamos que partimos de una sencilla aplicación escrita originalmente en Java 8, que se compone de las dos clases listadas a continuación, y que se limita a obtener la versión de la máquina virtual de Java que se está ejecutando leyéndola de una variable del sistema.

- ```src/Main.java```:

```java
public class Test {

  public static void main(String[] args) {
    String version = Util.version();

    System.out.println(version);
  }
}
```

- ```src/Util.java```:

```java
public class Util {

  public static String version() {
    return System.getProperty("java.version");
  }
}
```

Aplicación que puede compilarse, empaquetarse y ejecutarse de la forma habitual con los siguientes comandos:

```text
%JDK_8%/javac -d bin -sourcepath src src/Util.java src/Test.java
%JDK_8%/jar cvf test.jar -C bin .
%JDK_8%/java -cp test.jar Test
```

El resultado de la aplicación dependerá de la versión de Java utilizada para ejecutar el programa. Si se ejecuta con la máquina virtual de Java 8 (```161```) se obtendrá ```1.8.0_161```. Pero si se ejecuta con la máquina virtual de Java 10 se obtendrá ```10```.

Si la aplicación se compilase con Java 10 utilizando los mismos argumentos de compilación por defecto entonces no podría ejecutarse con Java 8, daría un error en tiempo de ejecución al arrancar la aplicación.

Para generar código que pueda ejecutarse en una versión anterior de Java se debe utilizar el parámetro ```--release``` seguido del número de la versión de Java para la que se quiere compilar. Por ejemplo, para compilar y empaquetar con Java 10 la aplicación indicando que se genere código compatible con la versión 8 se debe usar el siguiente comando:

```text
%JDK_10%/javac -d bin --release 8 -sourcepath src src/Util.java src/Test.java
%JDK_10%/jar cvf test.jar -C bin .
```

El código generado de esta manera se ejecutará correctamente con la versión 8 y superiores de Java.

El parámetro ```--release``` se introdujo nuevo con la versión 9 de Java, es similar a los parámetros ```-source``` y ```-target``` ya existentes, pero además detecta si el código utiliza alguna API pública conocida de Java que no se encuentre en la versión de Java destino de la compilación.

Veamos ahora como mejorar la aplicación de ejemplo anterior, utilizando el método estático ```java.lang.Runtime.version```, que permite a las aplicaciones obtener la versión de Java que se está ejecutando, en vez de leerla de una variable del sistema. Es un método que sólo está disponible a partir de Java 9, por lo que no se puede simplemente reemplazar el código de la clase ```Util``` original para que lo utilice si se quiere que la aplicación siga siendo compatible con Java 8.

La solución es crear una nueva clase como la listada a continuación, y empaquetarla junto con las dos anteriores en un MRJAR.

- ```src-9/Util.java```:

```java
import java.lang.Runtime;

public class Util {
  public static String version() {
    String version = Runtime.version().toString();

    return version;
  }
}
```

Clase que puede compilarse con el siguiente comando:

```text
%JDK_10%/javac -d bin-9 --release 9 -sourcepath src-9 src-9/Util.java
```

Y que puede empaquetarse junto con el resto de clases con el siguiente comando:

```text
%JDK_10%/jar cvf test.jar -C bin . --release 9 -C bin-9 .
```

El parámetro ```--release``` indica la versión de Java que se debe utilizar para una lista de clases dada, hace que se añada la entrada ```Multi-Release: true``` al fichero ```MANIFEST.MF```, y fuerza a que se cree un MRJAR con la siguiente estructura:

```text
/META-INF
  /versions
    /9
      Util.class
      MANIFEST.MF
Test.class
Util.class
```

Cuando se ejecute la aplicación con Java 8 se ejecutará la clase ```Util``` de la raíz, y cuando se ejecute con Java 9 o superior se ejecutará la clase ```Util``` de ```META-INF/versions/9```.
