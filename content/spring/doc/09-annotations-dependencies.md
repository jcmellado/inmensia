# 9. Spring: Anotaciones (1) - Dependencias

_21-04-2012_ _Juan Mellado_

Usar anotaciones dentro de las clases permite que la configuración y la implementación estén en un único sitio. Spring soporta que toda la configuración esté en un fichero XML, que se haga toda en forma de anotaciones, o que se mezclen ambos formatos. Lo importante es recordar que primero se aplican las anotaciones y luego la configuración proveniente de los ficheros, pudiendo esta última sobreescribir lo establecido por las anotaciones.

## 9.1. Configuración

Como se mencionó en un artículo anterior, cuando se explicaron ```@PostConstruct``` y ```@PreDestroy```, para que Spring tenga en cuenta las anotaciones hay que añadir la etiqueta ```annotation-config``` en el fichero ```applicationContext.xml``` de la siguiente manera:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.1.xsd">

<context:annotation-config/>
...
</beans>
```

## 9.2. @Required

Esta anotación permite indicar que una dependencia debe ser resuelta obligatoriamente en tiempo de configuración. Es decir, que una determinada propiedad de un _bean_ debe tener un valor distinto de nulo al acabar de instanciarla.

Consideremos el ejemplo de un coche que necesita de su motor:

```java
import org.springframework.beans.factory.annotation.Required;

public class Coche {

  private Motor motor;

  @Required
  public void setMotor(Motor motor) {
    this.motor = motor;
  }
...
```

Si en el fichero ```applicationContext.xml``` definimos el _bean_ de esta forma:

```xml
<bean id="coche" class="com.empresa.Coche"/>
```

Al ejecutar la aplicación se elevará una excepción de tipo ```BeanCreationException```, porque la dependencia que tiene el coche con el motor no puede resolverse. Para evitarlo es necesario activar algún tipo de mecanismo de auto-descubrimiento o inyectarla explícitamente:

```xml
<bean id="coche" class="com.empresa.Coche">
  <property name="motor" ref="motor"/>
</bean>

<bean id="motor" class="com.empresa.Motor"/>
```

La anotación ```@Required``` se aplica comúnmente sobre los DAOs a los que accede un servicio.

## 9.3. @Autowired

Esta anotación permite el autodescubrimiento e inyección automática de dependencias. Resuelve el problema del apartado anterior, haciendo innecesario declarar de forma explícita las dependencias en el fichero XML de configuración, y además tiene la ventaja de que se puede aplicar sobre muchos de los elementos de una clase.

El uso más común es sobre un _setter_, de forma que Spring automáticamente busca el _bean_ que mejor casa el tipo del parámetro:

```java
@Autowired
public void setMotor(Motor motor) {
    this.motor = motor;
}
```

Pero también se puede utilizar sobre las propiedades directamente, sobre el constructor, o sobre un método, con cualquier nombre, y con cualquier número y tipo de parámetros:

```java
@Autowired
private Motor motor;

@Autowired
public Coche(Motor motor) {
  this.motor = motor;
}

@Autowired
public void montaje(Motor motor, Volante volante) {
  this.motor = motor;
  this.volante = volante;
}
```

Incluso se puede utilizar para obtener todos los _beans_ de un mismo tipo declarados en la configuración y que se almacenen en algún tipo de colección:

```java
@Autowired
private Pieza[] piezas;

@Autowired
private List<Pieza> piezas;

@Autowired
private Map<String, Pieza> piezas;
```

Si la dependencia no se puede resolver se eleva una excepción. No obstante, este comportamiento se puede modificar utilizando el parámetro ```required```:

```java
@Autowired(required=false)
private Copiloto copiloto;
```

## 9.4 @Inject

Esta anotación tiene el mismo comportamiento que ```@Autowired```, aunque carece del parámetro ```required```. La ventaja está en que pertenece a la especificación estándar de Java en vez de pertenecer a Spring.

```java
import javax.inject.Inject;

public class Coche {

  @Inject
  private Motor motor;
...
```

No obstante, para utilizar esta notación es necesario añadir la dependencia correspondiente en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>javax.inject</groupId>
  <artifactId>javax.inject</artifactId>
  <version>1</version>
</dependency>
```

## 9.5. @Resource

Esta anotación se utiliza para eliminar ambiguedades a la hora de inyectar dependencias automáticamente, sobre todo si se aplica la anotación ```@Autowired``` en propiedades que tienen como tipo una interface, ya que clases distintas pueden implementar una misma interface.

Utiliza los identificadores de los _beans_ para resolver las dependencias. Pertenece a la especificación estándar de Java, en vez de a Spring, por lo que requiere añadir la misma dependencia de Maven indicada para la anotación ```@Inject```.

```xml
<bean id="volante" class="com.empresa.Pieza"/>

<bean id="retrovisor" class="com.empresa.Pieza"/>
```

```java
@Autowired
@Resource(name="volante")
private Pieza volante;

@Autowired
@Resource(name="retrovisor")
private Pieza retrovisor;
```

Si se omite el valor del parámetro ```name``` se intenta resolver la dependencia buscando un _bean_ que tenga el mismo nombre de la propiedad a la que se ha aplicado la anotación. Y si no se encuentra un _bean_ que case se intenta resolver por tipo.

## 9.6. @Qualifier

Esta anotación tiene un comportamiento similar a la anterior ```@Resource```, pero utiliza los roles de los _beans_ en vez de sus identificadores para resolver las dependencias.

Su uso más común es aplicarla usando el valor de alguna característica semántica de los _beans_ definidos en la configuración. Vale, así dicho asusta un poco, pero la "característica semántica" es simplemente el valor de la etiqueta ```qualifier``` que tiene un _bean_ dentro del fichero XML, y que se usa para indicar el "rol" que tiene un _bean_ dentro de la aplicación.

Supongamos la siguiente configuración de ejemplo donde los _beans_ representan piezas de un coche que se clasifican en función de su posición:

```xml
<bean id="faros" class="com.empresa.Pieza">
  <qualifier value="frontal"/>
</bean>

<bean id="parachoque" class="com.empresa.Pieza">
  <qualifier value="frontal"/>
</bean>

<bean id="maletero" class="com.empresa.Pieza">
  <qualifier value="posterior"/>
</bean>
```

Utilizando la anotación ```@Qualifier``` se puede eliminar la ambiguedad a la hora de resolver las dependencias de forma automática, incluso cuando se aplican a colecciones:

```java
@Autowired
@Qualifier("posterior")
private Pieza maletero;

@Autowired
@Qualifier("frontal")
private List<Pieza> morro;
```

Las anotaciones son un mecanismo muy potente proporcionado por Java, a la vez que extensible, lo que hace incluso posible utilizar anotaciones ```@Qualifier``` propias en vez de ceñirse a las existentes por defecto.
