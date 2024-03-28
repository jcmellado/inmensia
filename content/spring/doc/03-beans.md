# 3. Spring: Beans (1) - Instanciación y Dependencias

_19-04-2012_ _Juan Mellado_

Es hora de empezar a ver algunos de los numerosos atributos que se pueden utilizar a la hora de definir _beans_, ya que es necesario recordar en todo momento que un contexto de Spring implementa un patrón _Factory_ capaz de instanciar cualquier clase que se le defina, y para poder lograrlo tiene que poder dejarnos indicarle que es exactamente lo que queremos instanciar y como lo queremos instanciar.

## 3.1. Nombres

En el ejemplo básico del artículo anterior se utilizó el atributo ```id``` para identificar a un _bean_. Los identificadores deben ser únicos en un sistema, elevándose una excepción en caso contrario. Si un identificador se omite, Spring creará uno automáticamente.

También se permite tener más de un nombre para un mismo _bean_. Ya sea utilizando el atributo ```name```, que admite una lista de nombres separados por espacios, comas, o puntos y comas. O utilizando la etiqueta ```alias```:

```xml
<bean id="zoo" name="zoologico,wildPark" class="com.empresa.zoo.ZooServiceImpl"/>

<alias name="zoo" alias="zooBean"/>
<alias name="zoologico" alias="wildParkZoo"/>
```

Con esta configuración de ejemplo el mismo _bean_ podría referenciarse como ```zoo```, ```zoologico```, ```wildPark```, ```zooBean```, o ```wildParkZoo```.

## 3.2. Instanciación

Spring puede instanciar un _bean_ de muy diversas formas. La más básica es la del ejemplo básico del artículo anterior. Es decir, usando un simple constructor sin parámetros.

Otra forma de hacerlo es proporcionándole a Spring el nombre de un método estático de la clase que implemente el patrón _Factory_:

```xml
<bean id="zoo"
      class="com.empresa.zoo.ZooServiceImpl"
      factory-method="createInstance"/>
```

Lo que permite utilizar la típica implementación del patrón _Singleton_ sin renunciar a Spring:

```java
public class ZooServiceImpl implements ZooService {

  private static final ZooService service = new ZooServiceImpl();

  public static ZooService createInstance() {
    return service;
  }
...
```

E incluso es posible indicar que se utilice otra clase (_bean_) para que sea esa la que actúe con el rol de _Factory_:

```xml
<bean id="zoo"
      factory-bean="serviceLocator"
      factory-method="createZooServiceInstance"/>

<bean id="serviceLocator" class="com.empresa.zoo.ServiceLocator"/>
```

En este último ejemplo es interesante notar que el atributo ```factory-bean``` hace referencia a un _bean_. Utilizar referencias a otros _beans_ es una constante dentro de los ficheros de configuración.

Con esta configuración podemos eliminar el método estático del ejemplo anterior e implementar una clase factoría más tradicional:

```java
public class ServiceLocator {

  private static final ZooService zooService = new ZooServiceImpl();
 
  private ServiceLocator() {
  }

  public ZooService createZooServiceInstance() {
    return zooService;
  }
}
```

No obstante, lo realmente importante es darse cuenta de que la implementación del método ```main``` original no se ha cambiado en ningún momento, la forma de acceder al _bean_ sigue siendo la misma desde el principio. Es decir, sólo es necesario cambiar la configuración de Spring para variar la forma en la que instancian los objetos dentro de nuestra aplicación. Esto permite establecer la estrategia que resulte más adecuada en cualquier momento del ciclo de vida de una aplicación, sin tener que modificar y recompilar el código.

## 3.3. Dependencias

La inyección de dependencias permite indicar, mediante configuración, no de forma explícita en el código, qué _beans_ concretos deben utilizar otros _beans_. Es decir, qué componentes concretos debe utilizar la aplicación. En la práctica significa delegar en Spring la instanciación de los parámetros de los constructores o de las propiedades de las clases.

Imaginemos que queremos montar restaurantes en nuestro zoológico y estamos barajando varias opciones, como puestos de comida rápida, cocina mediterránea, o algo más exótico. Creamos una interface, la dejamos abierta a las diversas implementaciones posibles, pero forzamos a que se pase como parámetro en el constructor:

```java
public class ZooServiceImpl implements ZooService {

  private static final FoodService foodService;

  public ZooServiceImpl(FoodService foodService) {
    this.foodService = foodService;
  }
...
```

En la configuración le indicamos a Spring el componente (_bean_) concreto que queremos utilizar con la etiqueta ```constructor-arg```. Por ejemplo, si nos decidiéramos por una cadena de restaurantes de comida mejicana podríamos utilizar la siguiente configuración:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <constructor-arg ref="mexicanFood"/>
</bean>

<bean id="mexicanFood" class="com.empresa.zoo.MexicanFoodServiceImpl"/>
```

De forma general, con este tipo de configuración, lo que hace Spring es casar los argumentos por tipo, y si la configuración resulta ambigua entonces eleva una excepción.

Si se quiere más control se puede indicar el tipo concreto de cada parámetro y su valor:

```xml
<constructor-arg type="int" value="3"/>
<constructor-arg type="java.lang.String" value="Mexican Food"/>
```

O indicar la posición concreta que ocupa cada parámetro:

```xml
<constructor-arg index="0" value="3"/>
<constructor-arg index="1" value="Mexican Food"/>
```

E incluso utilizar el nombre original de los parámetros:

```xml
<constructor-arg name="numeroRestaurantes" value="3"/>
<constructor-arg name="nombreFranquicia" value="Mexican Food"/>
```

Aunque esto último sólo funciona si se compila en modo _debug_, o se usa la anotación ```@ConstructorProperties``` del JDK.

Otra forma distinta de pasarle las dependencias a una clase es llamar a _setters_ una vez esté instanciada, en vez de utilizar parámetros en el constructor:

```java
public class ZooServiceImpl implements ZooService {

  private FoodService foodService;

  public ZooServiceImpl() {
  }

  public void setFoodService(FoodService foodService) {
    this.foodService = foodService;
  }
...
```

En este caso, para inyectar las dependencias, se utiliza la etiqueta ```property``` con el nombre del atributo de la clase correspondiente, tantas como se quieran inyectar:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService" ref="mexicanFood"/>
</bean>

<bean id="mexicanFood" class="com.empresa.zoo.MexicanFoodServiceImpl"/>
```

De forma parecida a ejemplos anteriores, el atributo ```ref``` sirve para referenciar a otro _bean_, objeto de estudio del siguiente capítulo.
