# 26. Spring: Testing (3) - JUnit

_25-04-2012_ _Juan Mellado_

Además de las anotaciones propias de Spring vistas en el capítulo anterior, cuando se utiliza el _framework_ en conjunción con JUnit se pueden utilizar otras anotaciones que proporcionan un mayor control sobre la definición y ejecución de las pruebas.

## 26.1. Spring y JUnit

El _framework_ de pruebas de Spring está completamente integrado con JUnit, de forma que se pueden implementar clases de prueba de la forma habitual con JUnit, y además sacar partido de las capacidades de Spring para gestionar contextos, dependencias, transacciones y demás características.

La configuración mínima que hay que realizar para utilizar el _framework_ de pruebas de Spring con JUnit es aplicar la anotación ```@RunWith(SpringJUnit4ClassRunner.class)``` sobre las clases de prueba. Y la anotación ```@TestExecutionListeners``` con una lista vacía:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@TestExecutionListeners({})
public class Tests {
...
```

La lista vacía es necesaria para evitar que se eleve una excepción, ya que Spring intenta cargar un contexto para configurar los _listeners_ por defecto, lo que requiere que exista una configuración para dicho contexto, que no existe. Aunque si lo que se quiere es utilizar un contexto en concreto, se puede especificar con la anotación ```@ContextConfiguration``` vista anteriormente.

Otra forma conveniente de configurar las clases de pruebas es heredar de algunas de las clases de ayuda que ofrece Spring, como ```AbstractJUnit4SpringContextTests```:

```java
public abstract class Tests extends AbstractJUnit4SpringContextTests {
...
```

La ventaja de esta aproximación es que proporciona una propiedad ```applicationContext``` con la que se puede acceder directamente al contexto. La desventaja es que añade una dependencia por código con Spring.

Una clase similar a la anterior, de la que también se puede heredar, es ```AbstractTransactionalJUnit4SpringContextTests```, que además de acceso al contexto proporciona una propiedad ```simpleJdbcTemplate``` para la ejecución de sentencias SQL.

## 26.2. @IfProfileValue

Esta anotación permite condicionar la ejecución de pruebas en función de los valores concretos de las variables del sistema.

Por ejemplo, se puede condicionar que una determinada prueba se ejecute sólo si realiza en una máquina con Windows 7:

```java
@Test
@IfProfileValue(name="os.name", value="Windows 7")
public void test() {
...
```

## 26.3. @ProfileValueSourceConfiguration

Esta anotación se utiliza para modificar el comportamiento por defecto de la anterior. Sirve para especificar de donde deben tomarse los valores utilizados en la anotación ```@IfProfileValue```.

Por defecto los valores se toman usando la clase ```SystemProfileValueSource```, que toma los valores de las propiedades del sistema, pero se puede indicar que se utilice cualquier otra clase:

```java
@ProfileValueSourceConfiguration(Valores.class)
public class Tests {
...
```

Una clase que quiera ser el origen de los valores debe implementar la interface ```ProfileValueSource```:

```java
public interface ProfileValueSource {

  String get(String key);
}
```

## 26.4. @Repeat

Esta anotación permite especificar el número de veces que debe ejecutarse un método de prueba:

```java
@Test
@Repeat(value=10)
public void test() {
...
```

## 26.5. @Timed

Esta anotación permite indicar el tiempo máximo dentro del que debería completarse una prueba, considerándose que la prueba falla si se supera dicho tiempo máximo.

```java
@Test
@Timed(millis=50)
public void test() {
...
```

En la documentación oficial de referencia se indica que la anotación ```@Timed``` propia de Spring tiene un criterio distinto a la de JUnit. Concretamente, la anotación de Spring considera el tiempo total en realizar la prueba incluida todas sus repeticiones, mientras que la de JUnit considera el tiempo en realizar cada repetición de forma individual. Además, la anotación de Spring espera a que la prueba termine su ejecución por completo antes de indicar que ha fallado por _timeout_ en vez de abortarla.
