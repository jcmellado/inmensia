# 3. Java: Máquinas Virtuales Personalizadas con jlink

_10-04-2018_ _Juan Mellado_

Tener un _runtime_ o máquina virtual de tamaño reducido ha sido tradicionalmente una característica bastante apreciada, ya que normalmente se traduce en un menor almacenamiento, memoria y tiempo de carga. Las tendencias tecnológicas actuales de microservicios, contenedores, e _Internet de las Cosas_ (IOT) no han variado ni un ápice esta apreciación.

Java se ha caracterizado siempre por tener una máquina virtual de un tamaño considerable y pocas opciones de personalización. Afortunadamente, con la versión 9 de Java se cambió a un diseño más modular y se publicó la herramienta ```jlink```. El diseño modular introdujo el concepto de módulo, tanto para el código propio de la máquina virtual como para el de las aplicaciones, y la herramienta ```jlink``` dotó a los desarrolladores de una forma de crear distribuciones de Java personalizadas, en las que incluyesen tan solo los módulos que fueran necesarios para ejecutar sus aplicaciones.

Dentro del subdirectorio ```jmods``` donde se encuentre instalado Java están los distintos módulos en los que se estructura la máquina virtual. Siendo el módulo ```java.base``` el que contiene las clases básicas fundamentales necesarias para ejecutar la máquina virtual, y del que todos los demás módulos dependen, ya sea de forma directa o indirecta.

Para crear una imagen con una distribución personalizada de Java que contenga tan sólo el módulo base se debe ejecutar el siguiente comando:

```text
jlink --module-path $JAVA_HOME/jmods --add-modules java.base --output ./imagen
```

Con el parámetro ```--module-path``` se indica donde se encuentran los módulos, con ```--add-modules``` la lista de módulos que se quieren incluir en la imagen, y con ```--output``` el nombre del directorio donde crear la imagen personalizada.

Como resultado de la ejecución del comando anterior se creará un directorio de nombre ```imagen``` con la misma estructura de subdirectorios que en la distribución original de Java, pero que tan sólo contendrá los archivos del módulo ```java.base```, lo que se traduce en una ocupación de unos 36,5 MB frente a los 474 MB del JDK original.

Es posible reducir el tamaño de la imagen resultante a unos 20 MB con diversos parámetros como ```--compress```, ```--no-man-pages```, ```--no-header-files```, y otros. Las opciones disponibles se muestran en la ayuda que se lista con el clásico parámetro ```--help```.

La máquina virtual personalizada se ejecuta de igual forma que la original:

```text
.\imagen\bin\java
```

En todos los aspectos es una máquina virtual de Java, y se puede aplicar sobre ella por ejemplo la técnica de _Class Data Sharing_ para generar un fichero de clases precompiladas con ```.\imagen\bin\java -Xshare:dump```, aunque a costa de doblar el tamaño de la distribución.

De forma general, para comprobar qué módulos se encuentran en una distribución de Java se tiene que ejecutar el siguiente comando:

```text
java --list-modules
```

Ejecutado sobre la distribución original de Java lista todos los módulos, ejecutado sobre nuestra distribución personalizada lista sólo el módulo básico incluido en ella:

```text
    java.base@10
```

Una distribución personalizada de Java cobra mayor sentido si se crea para ejecutar una aplicación concreta, de forma que la imagen contenga sólo los módulos que utiliza dicha aplicación. Algo sencillo de ejemplificar con una aplicación muy simple que dependa de un módulo cualquiera, como la compuesta por los siguientes dos ficheros:

- ```modulo/paquete/Clase.java```:

```java
package paquete;

import java.util.prefs.Preferences;

public class Clase{
  public static void main(String[] args) {
    System.out.println(Preferences.systemRoot());
  }
}
```

- ```modulo/module-info.java```:

```java
module modulo {
  requires java.prefs;

  exports paquete;
}
```

La clase ```paquete.Clase``` simplemente invoca un método estático cualquiera de una clase de un paquete del módulo ```java.prefs```. El fichero ```module-info``` define el módulo modulo, indica la dependencia explícita con el módulo ```java.prefs```, y exporta el paquete paquete que contiene la clase principal de la aplicación.

La aplicación se puede compilar, empaquetar en un _jar_, y probar de la forma acostumbrada con los siguientes comandos:

```text
cd modulo

javac ./module-info.java ./paquete/Clase.java

jar cf app.jar ./module-info.class ./paquete/Clase.class

java -cp app.jar paquete.Clase
```

La imagen a partir del módulo de dicha aplicación se puede crear con el siguiente comando:

```text
jlink --module-path . --add-modules modulo --output ./imagen
```

Y se puede comprobar qué módulos se han incluido dentro la imagen con el siguiente comando:

```text
.\imagen\bin\java --list-modules
```

El resultado mostrará cuatro módulos:

```text
    java.prefs@10
    java.xml@10
    java.base@10
    modulo
```

El primer módulo es ```java.pref```, requerido por la aplicación de forma explícita. Los dos siguientes son dependencias directas y transitivas, ya que ```java.prefs``` depende de ```java.xml```, y ```java.xml``` depende a su vez de ```java.base```. El último módulo es el propio de la aplicación.

El último paso es verificar que la aplicación se ejecuta correctamente con la imagen:

```text
.\imagen\bin\java paquete.Clase
```

Como se observa en el último comando, para ejecutar la aplicación no es necesario pasar como parámetro ni el módulo ni el _jar_, la clase ```paquete.Clase``` es parte de la imagen.
