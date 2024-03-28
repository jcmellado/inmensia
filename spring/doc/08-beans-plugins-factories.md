# 8. Spring: Beans (6) - Plugins y Factorías

_21-04-2012_ _Juan Mellado_

De igual forma que Spring proporciona mecanismos para acceder a las operaciones que se realizan durante el ciclo de vida de cada _bean_ de forma individual, permite también acceder a las operaciones que se realizan sobre la infraestructura que los gestiona.

## 8.1. Plugins

Spring ofrece lo que se conoce como "puntos de extensión", que son los puntos oportunos dentro de su código a través de los cuales se pueden conectar clases propias que actúen a la manera de los tradicionales _plugins_. A esos puntos de extensión se accede implementando determinadas interfaces y dejando que Spring las invoque a modo de _callbacks_.

Una de estas interfaces es ```BeanFactoryPostProcessor```. Ofrece un método que se llama antes de instanciar cualquier _bean_, y que recibe como parámetro la configuración de _beans_ de Spring. Importante fijarse en que recibe la "configuración" de _beans_, no las instancias creadas a partir de dicha configuración.

La implementación mínima que se puede crear es la siguiente, que no hace realmente nada:

```java
public class Plugin implements BeanFactoryPostProcessor {

  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
    throws BeansException {
  }
}
```

En el fichero ```applicationContext.xml``` basta con declarar el _bean_ de la forma habitual:

```xml
<bean id="plugin" class="com.empresa.Plugin"/>
```

No hace falta obtener ninguna instancia del _bean_ mediante código, Spring automáticamente detecta que el _bean_ implementa la interface, y lo añade a su lista de _plugins_ a llamar. Se pueden definir tantos _beans_ de este tipo como se quiera, siendo su uso más habitual el cambiar la configuración original recibida añadiendo algún tipo de atributo propio. Es decir, esta inferface permite personalizar la configuración.

Otra interface que ofrece acceso a los puntos de extensión de Spring es ```BeanPostProcessor```, que tiene dos métodos. El primer método que se llama cuando se crea una instancia de un _bean_, justo antes de llamar a sus métodos de inicialización. Y el segundo método que se llama justo después de llamar a dichos métodos de inicialización.

La implementación mínima que se puede crear es una que se limita simplemente a devolver el _bean_ que recibe:

```java
public class Plugin implements BeanPostProcessor {

  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    return bean;
  }

  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName)
      throws BeansException {
    return bean;
  }
}
```

De igual forma que antes, no hace falta obtener ninguna instancia del _bean_ mediante código, basta con añadirlo al fichero ```applicationContext.xml```, siendo su uso más habitual el cambiar la instancia del _bean_ original recibido por algún tipo de _proxy_. Es decir, esta interface permite personalizar los _beans_.

## 8.2. Aware

Si se necesita acceder al contexto desde dentro de un _bean_ se puede hacer que la clase del _bean_ implemente la interface ```ApplicationContextAware```:

```java
public interface ApplicationContextAware {

  void setApplicationContext(ApplicationContext applicationContext)
      throws BeansException;
}
```

Spring llama al método ```setApplicationContext``` para dar la oportunidad al _bean_ de acceder al contexto que lo contiene y que pueda realizar las operaciones que necesite con él, como acceder a otros _beans_ mediante código, por ejemplo.

Spring soporta otras interfaces más de este tipo (```Aware```), que tienen todas ellas la característica en común de permitir acceder a ciertas partes de la "infraestructura" de Spring. De forma general, el propósito de cada interface es inyectar en el _bean_ una dependencia distinta. Así, ```BeanClassLoaderAware``` permite acceder al _class loader_, ```BeanFactoryAware``` al _bean factory_, ```ResourceLoaderAware``` al resource _loader_, y así sucesivamente.

## 8.3. Factorías

Si lo que se quiere es un mayor control sobre como se instancian los objetos, se pueden incluso crear clases propias que implementen la interface ```FactoryBean```.

Supongamos que queremos montar una fábrica de pepinillos con las siguientes clases:

```java
public class Pepinillo {

  public String toString() {
    return "¡Soy un pepinillo!";
  }
}

public class Fabrica implements FactoryBean<Pepinillo> {

  @Override
  public Class<Pepinillo> getObjectType() {
    return Pepinillo.class;
  }

  @Override
  public boolean isSingleton() {
    return false;
  }

  @Override
  public Pepinillo getObject() throws Exception {
    return new Pepinillo();
  }
}
```

La interface ```FactoryBean``` tiene tres métodos que hay que implementar. ```getObjectType``` para obtener el tipo del objeto que produce la fábrica, ```isSingleton``` para indicar si el objeto devuelto es un _Singleton_, y ```getObject``` para devolver los objetos creados por la fábrica.

En el fichero ```applicationContext.xml``` basta con declarar el _bean_ de la forma habitual:

```xml
<bean id="fabrica" class="com.empresa.Fabrica"/>
```

Lo interesante viene ahora, al ejecutar la siguiente línea de código:

```java
System.out.println( context.getBean("fabrica") );
```

Al contrario de lo que la intuición sugiere, debido al nombre que tiene el _bean_, el resultado no es una referencia a la fábrica, sino un pepinillo:

```text
¡Soy un pepinillo!
```

Es decir, ¡la fábrica produce pepinillos!. Aunque quizás una manera mejor de entender el ejemplo es tener en cuenta que el código anterior es equivalente al siguiente:

```java
System.out.println( context.getBean("fabrica", Pepinillo.class) );
```

Por su parte, para acceder a la fábrica en si misma, hay que anteponer un _ampersand_ al nombre (como con los punteros en C):

```java
System.out.println( context.getBean("&fabrica") );
```

En todo caso, lo interesante es darse cuenta de que Spring detecta automáticamente que la clase implementa la interface ```FactoryBean``` y la trata como una factoría.
