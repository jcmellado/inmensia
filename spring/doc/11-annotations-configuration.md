# 11. Spring: Anotaciones (3) - Configuración

_22-04-2012_ _Juan Mellado_

Los artículos anteriores de esta serie han mostrado como usar anotaciones para embeber algunos detalles de configuración en el propio código Java. En este artículo se muestra como especificar toda la configuración completa dentro de una única clase Java equivalente a un fichero de configuración XML.

Esta es la aproximación recomendada hoy en día. Declarar todos los _beans_ por código y evitar los ficheros XML.

## 11.1. @Configuration y @Bean

La anotación ```@Configuration``` indica que la clase sobre la que se encuentra aplicada debe ser usada como parte de la configuración de Spring. La anotación ```@Bean``` indica que el elemento sobre el que se encuentra aplicada define un _bean_.

Consideremos un ejemplo sencillo de una aplicación que implementa un único servicio de reparación de coches:

```java
public class Coche {
}

public interface Garage {

  void repara(Coche coche);
}

public class GarageImpl implements Garage {

  @Override
  public void repara(Coche coche) {
  }
}

@Configuration
public class AppConfig {

  @Bean
  public Garage garageBean() {
    return new GarageImpl();
  }
}
```

Lo interesante de esta definición de clases es que no requiere crear un fichero ```applicationContext.xml``` para que Spring funcione. Lo único que hay que hacer es cambiar el método ```main``` que estábamos utilizando desde el principio, para indicar que se quiere utilizar una clase de configuración en vez de un fichero:

```java
AnnotationConfigApplicationContext context =
    new AnnotationConfigApplicationContext(AppConfig.class);   

Garage garage = context.getBean(Garage.class);
```

La clase ```AppConfig``` es equivalente al siguiente fichero de configuración:

```xml
<bean id="garageBean" class="com.empresa.GarageImpl"/>
```

Los atributos que normalmente se utilizan sobre un _bean_ en un fichero de configuración pueden añadirse dentro de la anotación ```@Bean```:

```java
@Bean(name="Chapuzas S.A.", initMethod="init", destroyMethod="exit")
public Garage garageBean() {
...
```

La propia clase de configuración queda registrada como un _bean_, así como cualquier otra clase anotada con un estereotipo de componente, y sobre ellas las anotaciones de inyección de dependencias automáticas se aplican de la forma habitual.

Además, se pueden añadir tantas clases al contexto como se quiera. Ya sea inyectándolas como parámetros en el constructor, o registrándolas más tarde por código en tiempo de ejecución:

```java
context.register(Configuracion.class);
context.register(Componente.class);
context.refresh();
```

También se le puede dejar a Spring la tarea de escanear un paquete de clases, de igual forma que ocurre cuando se especifica la etiqueta ```context:component-scan``` en los ficheros XML:

```java
context.scan("com.empresa");
context.refresh();
```

## 11.2. @Import

Esta anotación permite que una clase de configuración incluya a otra. Esto evitar tener que registrarlas una a una. Basta con hacer una principal que incluya a todas las demás:

```java
@Configuration
@Import(AppConfigAuxiliar.class)
public class AppConfig {
...
```

## 11.3. @ImportResource

Esta anotación es similar a la anterior, pero permite especificar el nombre de un fichero (recurso) en vez de una clase:

```java
@Configuration
@ImportResource("classpath:beans.xml")
public class AppConfig {
...
```

No muy utilizada cuando lo que realmente se quiere es realizar toda la configuración por código.

## 11.4. @Scope

Este anotación permite especificar el tipo de un _bean_. Es decir, el alcance de su definición, de igual forma que la etiqueta ```scope``` en los ficheros XML:

```java
@Bean
@Scope("prototype")
public Coche cocheBean() {
...
```
