# 25. Spring: Testing (2) - Tests de Integración

_25-04-2012_ _Juan Mellado_

Spring proporciona un conjunto de anotaciones para poder realizar tests de integración, con el objeto de poder probar una aplicación sin tener que desplegarla, ni tener que establecer conexiones reales con otros sistemas, como una base de datos por ejemplo.

## 25.1. @ContextConfiguration

Esta anotación permite especificar de donde obtener la configuración del contexto concreto a utilizar para una clase de pruebas.

Como atributo se puede indicar la ubicación de ficheros XML:

```java
@ContextConfiguration(locations="classpath:testConfig.xml")
public class Tests {
...
```

O clases que tengan el atributo ```@Configuration```:

```java
@ContextConfiguration(classes=TestConfig.class)
public class Tests {
...
```

Las anotaciones en las clases son heredadas, por lo que se pueden construir jerarquías de clases para realizar configuraciones especializadas.

## 25.2. @ActiveProfiles

Esta anotación permite especificar qué perfil tienen que tener los _beans_ a utilizar por una clase de pruebas:

```java
@ActiveProfiles("dev")
public class Tests {
...
```

Los perfiles se definen utilizando la etiqueta ```profile``` en los ficheros de configuración XML, o la anotación ```@Profile``` directamente sobre las clases.

Supongamos por ejemplo el siguiente sencillo esquema de clases:

```java
public interface Vehiculo {
}

public class Camion implements Vehiculo {
}

public class Camioneta implements Vehiculo {
}
```

En el fichero de configuración se podría indicar que la clase ```Camion``` se quiere utilizar en entornos de producción, y la clase ```Camioneta``` en entornos de desarrollo:

```xml
<beans profile="production">
  <bean id="vehiculo" class="com.empresa.Camion"/>
</beans>

<beans profile="dev">
  <bean id="vehiculo" class="com.empresa.Camioneta"/>
</beans>
```

El código equivalente manipulando directamente el contexto sería similar al siguiente:

```java
context.getEnvironment().setActiveProfiles("dev");
context.refresh();
```

## 25.3. @DirtiesContext

Esta anotación sirve para indicar que el contexto utilizado por una clase de prueba se va a modificar o corromper, teniendo que ser cerrado y descartado al terminar la prueba:

```java
@DirtiesContext
public class Tests {
...
```

La anotación admite un parámetro que permite establecer cuando se tiene que descartar el contexto, por ejemplo cada vez que se ejecuta un método de la clase:

```java
@DirtiesContext(classMode=ClassMode.AFTER_EACH_TEST_METHOD)
public class Tests{
...
```

Spring mantiene internamente una _cache_ de contextos, para evitar tener que instanciarlos por cada prueba, por lo que esta anotación resulta de utilidad para eliminar contextos de la cache.

## 25.4. @TestExecutionListeners

Esta anotación permite configurar _listeners_ para un contexto de pruebas:

```java
@TestExecutionListeners(TestListener.class)
public class Tests{
...
```

Una clase que quiera actuar como _listener_ tiene que implementar la interface ```TestExecutionListener```:

```java
public interface TestExecutionListener {

  void beforeTestClass(TestContext testContext) throws Exception;

  void prepareTestInstance(TestContext testContext) throws Exception;

  void beforeTestMethod(TestContext testContext) throws Exception;

  void afterTestMethod(TestContext testContext) throws Exception;

  void afterTestClass(TestContext testContext) throws Exception;
}
```

## 25.5. @TransactionConfiguration

Esta anotación se utiliza para indicar el nombre del _bean_ que debe actuar como gestor de transacciones:

```java
@TransactionConfiguration(transactionManager="gestorTransacciones")
public class Tests {
...
```

Sólo tiene sentido si se quiere utilizar un _bean_ distinto al que se proporciona por defecto y que tiene por nombre ```transactionManager```.

## 25.6. @Rollback

Esta anotación sirve para modificar la acción realizada por defecto con las transacciones al terminar un método, que consiste en ejecutar un _rollback_ para deshacerlas.

Se puede utilizar un argumento en la anotación para que no se ejecute el _rollback_:

```java
@Test
@Rollback(false)
public void test() {
...
```

## 25.7. @BeforeTransaction

Esta anotación permite marcar un método para que sea ejecutado antes de crear la transacción utilizada para la ejecución de las pruebas de la clase:

```java
@BeforeTransaction
public void antes() {
...
```

## 25.8. @AfterTransaction

Esta anotación permite marcar un método para que sea ejecutado después de terminar la transacción utilizada para la ejecución de las pruebas de la clase:

```java
@AfterTransaction
public void despues() {
...
```
