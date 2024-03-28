# 2. Spring: Ejemplo Básico

_19-04-2012_ _Juan Mellado_

En este artículo se explica como crear un proyecto básico para dar los primeros pasos con Spring con el objetivo de entender su filosofía de funcionamiento, ya que la mejor forma de aprender a utilizar un _framework_ es precisamente utilizándolo.

Como versión de Spring se utilizará la actual 3.1.1-RELEASE sobre Java 1.6. Como entorno de trabajo Eclipse, concretamente la versión más reciente para "_Java EE Developers_" que en estos momentos es la 3.7, más conocida como Indigo. Y para evitar tener que estar descargando _jars_ sueltos a mano se utilizará Maven instalado como _plugin_ de Eclipse a través de la opción "_Help / Eclipse Marketplace / Maven Integration for Eclipse_" para no tener que salir del entorno.

Evidentemente esta serie de artículos parte del hecho de que se sabe algo de Java, Eclipse, Maven, y ese tipo de cosas. Aún así se tratará de dejar algunas miguitas de pan por el camino.

## 2.1. Proyecto

Lo primero como siempre es crear un proyecto (_File / New / Project / Maven / Maven Project_). En este caso un proyecto Maven utilizando el arquetipo por defecto que crea directamente una clase con un ```main``` y una única dependencia con JUnit.

## 2.2. pom.xml

Lo segundo es modificar el fichero ```pom.xml``` del proyecto para añadir la dependencia con Spring:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
</properties> 

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>${spring.version}</version>
</dependency>
```

Si todo ha ido bien se incluirán todos los _jars_ necesarios automáticamente: ```spring-context```, ```spring-aop```, ```aopalliance```, ```spring-beans```, ```spring-core```, ```commons-logging```, ```spring-expression``` y ```spring-asm```. ¡Un montón!

## 2.3. ApplicationContext

Spring ofrece muchas posibilidades de configuración, pero todas ellas acaban instanciando un objeto que implementa la interface ```ApplicationContext```. De hecho, lo primero que suele hacerse normalmente cuando te incorporas a un proyecto es echarle un vistazo a un fichero que casi siempre tiene por nombre ```applicationContext.xml```.

Un "contexto" en Spring es un contenedor que implementa la inversión de control. El que gestiona los _beans_. Y un _bean_ es cualquier objeto de nuestra aplicación gestionado por Spring.

Para empezar, lo más sencillo es instanciar un contexto dentro del ```main``` de una forma similar a la siguiente:

```java
AbstractApplicationContext context =
    new ClassPathXmlApplicationContext(new String[{"applicationContext.xml"});

context.registerShutdownHook();
```

La primera línea está claro que lo que hace es instanciar un contexto tomando la configuración de un fichero llamado ```applicationContext.xml``` que debe encontrarse en el ```classpath```.

La segunda línea es necesaria para que Spring pueda terminar su ejecución de forma ordenada, y sobre todo para que pueda terminar de forma ordenada también los objetos cuyos ciclos de vida gestiona.

No obstante, si se ejecuta el código tal cual está, se elevará una excepción de tipo ```java.io.FileNotFoundException``` debido a que el fichero ```applicationContext.xml``` no existe todavía.

## 2.4. applicationContext.xml

¿Qué debe contener un fichero de configuración? Pues la definición de los _beans_ que queremos utilizar en nuestra aplicación.

Para respetar la nomenclatura habitual crearemos un fichero ```applicationContext.xml``` en el directorio ```src/main/resources```. Y de hecho, crearemos el fichero de configuración más básico que se puede crear, uno vacío que no contenga ninguna definición de ningún _bean_:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
                      http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

</beans>
```

Si todo ha ido bien, al ejecutar nuevamente la aplicación ya no se elevará la excepción. ¡Enhorabuena, acaba de implementar su primera aplicación utilizando Spring!

No todos los _beans_ tienen que estar definidos en un único fichero, sino que se pueden definir en varios de ellos. Si se observa bien, se ve que el constructor del contexto admite una lista de ficheros, no sólo uno. Otra forma de hacerlo es simplemente importándolos de una manera más tradicional desde otro XML:

```xml
<beans>
  <import resource="services.xml"/>
  <import resource="daos.xml"/>
  ...
</beans>
```

## 2.5. Beans

El siguiente paso es crear un _bean_. Para ello vamos a suponer que se quiere hacer una aplicación para gestionar un zoológico y se crea una clase llamada ```ZooServiceImpl``` que implementa una interface ```ZooService```:

```java
public class Animal {
}

public interface ZooService {
  List<Animal> getAnimals();
}

public class ZooServiceImpl implements ZooService {

  public ZooServiceImpl() {
  }

  @Override
  public List<Animal> getAnimals() {
    return null;
  }
}
```

Se quiere que la clase se trate como un _Singleton_ a todo lo largo y ancho del sistema. Y para ello se deja que Spring la gestione declarando un _bean_ dentro del ```applicacionContext.xml```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl"/>
```

La sintaxis es bastante sencilla. El atributo ```id``` es un identificador del objeto (_bean_) que se quiere crear, y el atributo ```class``` es el nombre completo de la clase que se quiere instanciar.

¿Qué ocurre cuando Spring lee el fichero al arrancar la aplicación? Pues que automáticamente, haciendo uso de su potente implementación del patrón _Factory_, instancia un objeto de la clase ```ZooServiceImpl```. Y ese objeto, como todos los _beans_ por defecto, será un _Singleton_.

¿Y cómo se puede acceder a los _beans_? Pues a través del contexto. Existen muchas formas, una de las más sencillas es utilizar el identificador del _bean_ tal y como se declaró en el fichero de configuración:

```java
ZooService zoo = (ZooService)context.getBean("zoo");
```

Una vez obtenido el objeto se puede utilizar de la forma acostumbrada:

```java
zoo.getAnimals();
```

## 2.6. Recapitulando

El ejemplo de este artículo es de lo más sencillo, de hecho debe parecer bastante decepcionante, pero está hecho así a propósito. Por lo general Spring intimida bastante. Un enfoque paso a paso, haciendo un simple ```main```, entendiendo lo que se está haciendo, me parece más correcto que empezar usando una herramienta de generación automática de código. Sobre todo cuando hay problemas y es necesario saber lo que se está realmente ejecutándose por debajo.
