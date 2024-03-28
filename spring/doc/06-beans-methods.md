# 6. Spring: Beans (4) - Inyección de métodos

_20-04-2012_ _Juan Mellado_

En este artículo se desarrolla un ejemplo un poco más complejo con el objetivo de ver alguna característica un poco más avanzada de Spring, como es la inyección de métodos, que permite poner en práctica lo aprendido hasta ahora.

## 6.1. Proyecto

Para este ejemplo se parte de un proyecto Maven nuevo al que se le ha añadido la dependencia de Spring correspondiente en el fichero ```pom.xml```.

Esta vez se supone que la aplicación de ejemplo se utilizará en una pizzeria, por lo que hay definida una clase ```Pizzeria``` que implementa una interface ```PizzeriaService```. Su único propósito es servir pizzas, representadas por una clase POJO llamada ```Pizza```.

```java
public class Pizza {

  public Pizza() {
  }
}

public interface PizzeriaService {

  public Pizza dispatch();
}

public class Pizzeria implements PizzeriaService {

  public Pizzeria() {
  }

  @Override
  public Pizza dispatch() {
    return createPizza();
  }

  public Pizza createPizza() {
    return new Pizza();
  }
}
```

Como queremos utilizar Spring para controlar los objetos, definimos los _beans_ correspondientes en el fichero ```applicationContext.xml```, teniendo en cuenta que el _bean_ para la clase ```Pizza``` no puede ser un _Singleton_, ya que lógicamente queremos servir una pizza distinta a cada cliente, y no la misma para todo ellos. Afortunadamente eso ya sabemos como hacerlo, asignando el valor prototype al atributo ```scope```:

```xml
<bean id="pizzeria" class="com.empresa.Pizzeria"/>

<bean id="pizza" class="com.empresa.Pizza" scope="prototype"/>
```

Para probar la configuración, en el método ```main``` del programa inicializamos el contexto, recuperamos el _bean_ ```pizzeria```, y servimos un par de pizzas:

```java
AbstractApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");

context.registerShutdownHook();

Pizzeria pizzeria = context.getBean("pizzeria", Pizzeria.class);
       
Pizza pizza1 = pizzeria.dispatch();
Pizza pizza2 = pizzeria.dispatch();
```

Hasta aquí todo debería parecer funcionar correctamente. No obstante, hay algo que debería llamarnos la atención. La clase ```Pizzeria``` crea las pizzas con un simple ```new```, de esos de toda la vida. ¿Cómo se entera Spring de la creación de esos objetos? ¿Cómo aplica los detalles de configuración definidos en el ```applicationContext.xml```? ¿Son las pizzas en realidad _beans_?

Algo huele mal aquí ...

## 6.2. Prueba

Para salir de dudas haremos una modificación en la clase ```Pizza``` añadiendo una propiedad ```ingredientes```:

```java
public class Pizza {

  private List<String> ingredientes;

  public Pizza() {
  }

  public void setIngredientes(List<String> ingredientes) {
    this.ingredientes = ingredientes;
  }

  public List<String> getIngredientes() {
    return this.ingredientes;
  }
}
```

E inicializaremos todas las pizzas que se creen con un primer ingrediente, la masa, ya que todas las pizzas la llevan, a través del ```applicationContext.xml```:

```xml
<bean id="pizza" class="com.empresa.Pizza" scope="prototype">
  <property name="ingredientes">
    <list>
     <value>Masa</value>
    </list>
  </property>
</bean>
```

Y por último en el ```main``` preguntaremos cuantos ingredientes tiene una pizza:

```java
Pizza pizza = pizzeria.dispatch();

pizza.getIngredientes().size();
```

No demasiado sorprendentemente, al ejecutar la última línea de código se eleva una excepción de tipo ```java.lang.NullPointerException```. Lo que quiere decir que la lista de ingredientes ni siquiera se ha llegado a instanciar. La configuración está siendo ignorada. Spring no sabe nada de esos objetos pizza, no son _beans_.

## 6.3. Análisis

El problema es que al final resulta que Spring no hace magia, aunque a veces lo parezca. Si creamos objetos por nuestra cuenta y riesgo, esos objetos no estarán contenidos dentro del contexto de Spring. La inversión de control no tiene lugar a menos que lo indiquemos de forma explícita y utilicemos los mecanismos habilitados para ello.

La documentación de Spring se refiere al "contenedor de la inversión de control" (IoC container) para denotar al contexto como un recipiente de objetos creados y gestionados por la inversión de control. Los objetos pizza no están incluidos dentro de ese contenedor, han sido instanciados fuera de él, en ningún momento han pasado por las manos de Spring, y consecuentemente ninguna configuración indicada en el ```applicationContext.xml``` les aplica.

## 6.4. Solución

Para permitir que los objetos pizza sean gestionados por Spring vamos a aplicar la técnica de inyección de métodos, que básicamente permite que Spring inyecte código a nuestras clases para darle así la oportunidad de dejarle hacer lo que mejor sabe hacer: gestionar _beans_.

En nuestro ejemplo en particular, es el método ```createPizza``` de la clase ```Pizzeria``` el que necesita reescribirse, con la idea de dejar que sea Spring el que lo implemente y eliminar nuestro patético ```new```. Para ello, simplemente declaramos el método como abstracto y eliminamos su implementación, ya que será ahora Spring quien la proporcione:

```java
public abstract class Pizzeria implements PizzeriaService {

  public Pizzeria() {
  }

  @Override
  public Pizza dispatch() {
    return createPizza();
  }

  public abstract Pizza createPizza();
}
```

Usando la etiqueta ```lookup-method``` podemos indicar a Spring en el ```applicationContext.xml``` que implementar el método ```createPizza``` es asunto suyo, y que lo queremos que haga es retornar un _bean_ ```pizza```:

```xml
<bean id="pizzeria" class="com.empresa.Pizzeria">
  <lookup-method name="createPizza" bean="pizza"/>
</bean>
```

La nomenclatura es sencilla, el atributo ```name``` contiene el nombre del método a inyectar, y el atributo ```bean``` la referencia al _bean_ a instanciar.

No obstante, si ejecutamos ahora la aplicación se eleva una nueva excepción ```java.lang.ClassNotFoundException``` respecto a la clase ```net.sf.cglib.proxy.CallbackFilter```. ¿O es que alguien todavía piensa a estas alturas que con Spring funcionan todas las cosas a la primera?

El error es debido a que Spring realiza la inyección del método a muy bajo nivel, manipulando directamente el _bytecode_ de las clases a través de _cglib_, que es una librería que permite hacer esto precisamente.

Para resolver el error es necesario añadir la dependencia Maven correspondiente a _cglib_ en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>2.2.2</version>
</dependency>
```

Una vez realizado este cambio podemos volver a ejecutar y comprobar como todas nuestra pizzas vienen de fábrica con el ingrediente ```Masa```, señal inequívoca de que son _beans_ gestionados por Spring.

Técnicas más avanzadas de programación con Spring permiten sustituir cualquier método de una manera más genérica.
