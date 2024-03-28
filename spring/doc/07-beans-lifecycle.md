# 7- Spring: Beans (5) - Ciclos de Vida

_21-04-2012_ _Juan Mellado_

El ciclo de vida de un _bean_ es la secuencia de operaciones a las que se ve sometido desde su creación y hasta su posterior destrucción. Spring proporciona los mecanismos adecuados para que las aplicaciones puedan monitorizar y reaccionar a estos eventos a través de métodos _callback_.

## 7.1. Inicialización

Un _bean_ puede ser notificado de su inicialización por parte de Spring implementando la interface ```InitializingBean```. Esta interface tiene un método público ```afterPropertiesSet``` que es llamado cuando todas las propiedades del _bean_ han sido completamente inicializadas.

Continuando con el ejemplo de la pizzería del artículo anterior, se puede modificar la clase ```Pizzeria``` para que sea notificada cada vez que Spring instancie completamente un objeto de dicha clase:

```java
import org.springframework.beans.factory.InitializingBean;

public class Pizzeria implements PizzeriaService, InitializingBean {

  @Override
  public void afterPropertiesSet() {
  }
...
```

El inconveniente de esta solución es que el ```import``` añade una dependencia con Spring dentro del código fuente. Para evitar dicha dependencia se puede utilizar el atributo ```init-method``` dentro de la declaración del _bean_ en el fichero XML de la siguiente manera:

```xml
<bean id="pizzeria" class="..." init-method="init">
```

Esta forma de declarar el _bean_ tiene la ventaja de que elimina la dependencia del código, y además permite elegir el nombre del método que se quiere que sea llamado:

```java
public class Pizzeria {

  public void init() {
  }
...
```

## 7.2. Destrucción

De forma análoga a la inicialización, existe una interface ```DisposableBean``` que proporciona un método ```destroy```, invocado por Spring cuando se destruye la instancia de un _bean_.

No obstante, para evitar de nuevo la dependencia por código, y poder elegir el nombre del método a ser llamado, la forma preferida de uso es a través del atributo ```destroy-method```:

```xml
<bean id="pizzeria" class="..." destroy-method="exit">
```

## 7.3. Métodos por Defecto

Para evitar tener que declarar los métodos de inicialización y destrucción de forma individual para cada _bean_, Spring permite especificar unos nombres por defecto para todos los _beans_:

```xml
<beans default-init-method="init"
       default-destroy-method="exit">
...
</beans>
```

El hecho de que se declaren nombres por defecto no implica que tengan que implementarse obligatoriamente en cada clase, sólo que si existen entonces Spring los llamará.

## 7.4. Anotaciones

Si se utiliza la versión 6 de Java entonces se pueden utilizar las anotaciones estándar ```@PostConstruct``` y ```@PreDestroy```:

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public abstract class Pizzeria implements PizzeriaService {

  @PostConstruct
  public void init() {
  }

  @PreDestroy
  public void exit() {
  }
...
```

De esta forma no es necesario configurar ningún atributo específico en cada _bean_, ni establecer nombres por defecto. Sin embargo, el código tal cual está no funcionará, ¡como de costumbre!, los métodos no serán llamados. Es necesario indicar a Spring que procese las anotaciones añadiendo la etiqueta ```annotation-config``` en el fichero ```applicationContext.xml``` de la siguiente manera:

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

## 7.5. Ciclos de Vida

Spring facilita la construcción de ciclos de vida personalizados, ofreciendo interfaces que los _beans_ pueden implementar según las necesidades propias de cada aplicación.

Este tipo de interfaces están pensadas sobre todo para poder controlar los objetos que realizan algún tipo de procesamiento en segundo plano, o para aquellos en los que tiene sentido hablar de arranque y parada, como algún tipo de servicio por ejemplo. Proporcionan un mecanismo estándar automatizado de control de estado.

La interface más básica de todas ellas es ```Lifecycle```:

```java
public interface Lifecycle {

  void start();

  void stop();

  boolean isRunning();
}
```

Si una clase implementa la interface ```Lifecycle```, Spring llamará a su método ```isRunning``` antes de destruirlo. Y en caso de que este método devuelva ```true```, llamará a su método ```stop``` para darle la oportunidad de detenerse correctamente.

El metódo ```start``` no es llamado automáticamente por Spring, ya que el arranque de un componente concreto se considera responsabilidad propia de la aplicación que lo utilice. No obstante, Spring define las interfaces ```Phased``` y ```SmartLifecycle``` que permiten arrancar automáticamente los _beans_ e incluso definir el orden en que deben ejecutarse:

```java
public interface Phased {

  int getPhase();
}

public interface SmartLifecycle extends Lifecycle, Phased {

  boolean isAutoStartup();

  void stop(Runnable callback);
}
```

Si una clase implementa la interface ```SmartLifecycle``` entonces Spring llamará a su método ```isAutoStartup``` para averiguar si ha de arrancarlo automáticamente. Es decir, si ha de llamar a su método ```start```. Para controlar el orden de arranque, la implementación del método ```getPhase``` debe retornar un valor entero indicando su prioridad. Usando cero como valor por defecto, un valor negativo para mayor prioridad (primero en arrancar, último en parar) y un valor positivo para menor prioridad (último en arrancar, primero en parar).

El método ```stop``` admite un parámetro ```callback``` sobre el que se tiene que invocar su método ```run``` de manera obligatoria cuando se haya terminado con las tareas de parada del _bean_. Esto permite realizar operaciones de forma asíncrona, ya que Spring espera a que se invoque ese _callback_ antes de continuar con el proceso de parada de otro _bean_, aunque con un determinado periodo máximo de espera configurable.
