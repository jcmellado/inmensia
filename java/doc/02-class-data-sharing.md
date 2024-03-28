# 2. Java: Class Data Sharing

_06-04-2018_ _Juan Mellado_

A fin de optimizar el tiempo de arranque de la máquina virtual de Java y reducir el uso de memoria se puede crear un fichero que contenga un conjunto de clases precompiladas. Este fichero almacena las clases en un formato de representación interna que puede cargarse directamente en memoria, y que además, por ser de sólo lectura y mapeado en memoria, es automáticamente compartido por las distintas máquinas virtuales que se encuentren levantadas a un mismo tiempo. Esta opción se llama _Class Data Sharing_ (CDS).

Un fichero de este tipo se crea automáticamente cuando se instala Java desde el programa de instalación en Windows con las clases básicas de Java. Pero si la instalación se realiza descomprimiendo el programa de instalación manualmente, o copiándola de un directorio ya existente, entonces el fichero hay que crearlo con el siguiente comando:

```text
java -Xshare:dump
```

Como resultado de la ejecución del comando se muestra un _log_ muy detallado que en la mayoría de los casos se puede ignorar, aunque hay algún dato informativo útil, como por ejemplo el número de clases añadidas al fichero:

```text
    Number of classes 1298
```

El fichero generado se llama ```classes.jsa```  y se crea en el subdirectorio ```/bin/server```. Si ya existe se sobreescribe.

La máquina virtual de Java tiene un comportamiento definido por defecto con respecto al uso del fichero. No obstante, existe un parámetro que puede utilizarse para deshabilitar o forzar su uso. ```-Xshare:off``` fuerza que no se utilice el fichero, ```-Xshare:on``` fuerza que se utilice, y ```-Xshare:auto``` aplica el comportamiento por defecto.

Debido al que el formato del fichero depende de la arquitectura de cada máquina en concreto, se puede utilizar ```-Xshare:on``` para comprobar que el fichero es válido, y no se ha copiado por ejemplo desde una máquina con Windows a otra con Linux. Con ```-Xshare:on``` Java no arrancará si el fichero no es válido. Con el comportamiento por defecto Java ignorará el fichero si no es válido.

Además de las clases básicas de Java añadidas al fichero, Java 10 permite crear ficheros personalizados con las clases precompiladas que se quieran. Opción a la que Oracle ha llamado _Application Class-Data Sharing_ (ApsCDS), y que se implementa utilizando un fichero que contenga un listado con el nombre de las clases que se quieran precompilar.

No obstante, antes de crearlo es interesante entender como probar y verificar el origen de cada clase en Java. Para ello se puede utilizar el parámetro que activa el _log_ en la máquina virtual de Java. Por ejemplo, para ver desde donde se cargan las clases si no se utiliza CDS se puede ejecutar el siguiente comando:

```text
java -Xshare:off -Xlog:class+load
```

En la salida se mostrará cada clase cargada y su origen. Y puede verse que la clase ```java.lang.Object```, y todas las demás, se cargan desde el _jar_ del _runtime_:

```text
    java.lang.Object source: jrt:/java.base
```

En cambio, si se fuerza el uso de CDS:

```text
java -Xshare:on -Xlog:class+load
```

en la salida puede comprobarse que ahora ```java.lang.Object```, y todas las demás, se cargan desde el fichero compartido:

```text
    java.lang.Object source: shared objects file
```

Una vez realizada esta comprobación, ya se puede crear una aplicación de prueba con objetivo de añadir sus clases a un fichero compartido que la máquina virtual pueda precargar:

```java
package test;

public class Test {

 public static void main(String[] args) {
   System.out.println("test");
 }
}
```

Aplicación que se puede compilar, empaquetar en un _jar_, y probar de la forma acostumbrada con los siguientes comandos:

```text
javac ./test/Test.java

jar cf Test.jar ./test/Test.class

java -Xshare:on -Xlog:class+load -cp Test.jar test.Test
```

De forma que en la salida puede verse que la clase ```test.Test``` se carga desde ```Test.jar```:

```text
    test.Test source: file:<path>/test/Test.jar
```

Para obtener un fichero con las clases cargadas por la aplicación de prueba se debe ejecutar el siguiente comando:

```text
java
  -XX:+UnlockCommercialFeatures
  -Xshare:off
  -XX:+UseAppCDS
  -XX:DumpLoadedClassList=clases.lst
  -cp test.jar test.Test
```

El parámetro ```-XX:+UnlockCommercialFeatures``` es necesario sólo si se está utilizando la máquina virtual de Oracle, y no requiere ningún tipo de licencia en contra de lo que el nombre parece sugerir. ```-XX:+UseAppCDS``` activa la opción de AppCDS. Y ```-XX:DumpLoadedClassList``` especifica el fichero donde se deben de volcar los nombres de las clases cargadas por la aplicación. El fichero generado es un fichero de texto plano con una línea por cada clase.

El siguiente paso es crear un fichero con las clases precompiladas a partir del fichero de clases generado en el paso anterior:

```text
java
  -XX:+UnlockCommercialFeatures
  -Xshare:dump
  -XX:+UseAppCDS
  -XX:SharedClassListFile=clases.lst
  -XX:SharedArchiveFile=test.jsa
  -cp test.jar
```

El parámetro ```-XX:SharedClassListFile``` indica el fichero con el listado de clases a utilizar, y ```-XX:SharedArchiveFile``` especifica el fichero con las clases precompiladas a crear.

El último paso es volver a lanzar la aplicación de prueba forzando que se utilice el fichero con las clases compartidas creado en el paso anterior:

```text
java
  -XX:+UnlockCommercialFeatures
  -Xshare:on
  -XX:+UseAppCDS
  -XX:SharedArchiveFile=test.jsa
  -Xlog:class+load
  -cp test.jar test.Test
```

En la salida puede verse que ahora la clase de la aplicación de pruebas se carga desde el fichero compartido:

```text
    test.Test source: shared objects file
```

Este procedimiento debería realizarse, y automatizarse, para las aplicaciones y servidores de aplicaciones, con el objetivo de disminuir el tiempo de arranque y la memoria utilizada cuando se encuentren en ejecución varias máquinas virtuales al mismo tiempo.
