# 17. Spring: SpEL (1) - Expression Language

_24-04-2012_ _Juan Mellado_

Spring ofrece un lenguaje propio llamado SpEL (_Spring Expression Language_) que permite construir expresiones complejas evitando tener que limitar la configuración de un sistema a utilizar simples literales estáticos. Se utiliza habitualmente en los ficheros XML donde se definen los _beans_, y también en conjunción con algunas anotaciones.

Es un lenguaje independiente que puede utilizarse sin necesidad de definir ningún contexto.

## 17.1. SpelExpressionParser

Aunque no es lo habitual, se puede acceder mediante código al lenguaje instanciando directamente el _parser_ de expresiones:

```java
ExpressionParser parser = new SpelExpressionParser();

Expression expression = parser.parseExpression("'Texto estático'");

System.out.println(expression.getValue(String.class) );
```

La cadena de texto estático del ejemplo no es sólo una expresión, sino que es un objeto de tipo ```String```, y sobre él se pueden aplicar todas las operaciones habituales que pueden realizarse sobre un objeto de este tipo:

```java
parser.parseExpression("'Texto estático'.concat(' con esteroides')");
```

SpEL permite invocar constructores, acceder a propiedades, y llamar a métodos. Pero el uso habitual es aplicar una expresión sobre un objeto contextual, llamado ```root```, de forma que sea evaluada sobre él:

```java
Libro libro = new Libro("Don Quijote");
   
EvaluationContext context = new StandardEvaluationContext(libro);

ExpressionParser parser = new SpelExpressionParser();

Expression expression = parser.parseExpression("titulo");

System.out.println(expression.getValue(context));
```

El lenguaje utiliza el servicio de conversión de tipos explicado en el capítulo anterior, que puede extenderse para dar soporte a tipos propios, incluyendo las propiedades basadas en alguna clase de colección:

```java
public class Biblioteca {

    public final List<Libro> libro = new ArrayList<Libro>();
}

Biblioteca biblioteca = new Biblioteca();
   
biblioteca.libros.add(new Libro("Don Quijote"));

EvaluationContext context = new StandardEvaluationContext(biblioteca);
   
ExpressionParser parser = new SpelExpressionParser();
   
Expression expression = parser.parseExpression("libros[0].paginas");
   
expression.setValue(context, 1000);
   
System.out.println(biblioteca.libros.get(0).getPaginas());
```

## 17.2. XML

SpEL se utiliza dentro de los ficheros XML de configuración en la forma ```#{ <expresión> }```:

```xml
<bean id="bienvenida" class="com.empresa.Bienvenida">
  <property name="saludo" value="#{'¡Hola '.concat(' mundo!')}"/>
</bean>
```

Y entre otras muchas opciones, permite por ejemplo acceder directamente a las propiedades del sistema:

```xml
<bean id="sistema" class="com.empresa.Sistema">
  <property name="os" value="#{systemProperties['os.name']}"/>
</bean>
```

O evaluar expresiones más complejas que impliquen invocar al API de Java:

```xml
<bean id="juego" class="com.empresa.Juego">
  <property name="aleatorio" value="#{T(java.lang.Math).random() * 100.0}"/>
</bean>
```

## 17.3. Anotaciones

Las expresiones de SpEL se pueden utilizar con la anotación ```@Value``` directamente sobre las propiedades de los _beans_, o sobre sus _setters_ correspondientes, para establecer valores por defecto:

```java
@Service
public class Sistema {
  private String os;

  @Value("#{systemProperties['os.name']}")
  public void setOs(String os) {
    this.os = os;
  }
...
```

E incluso sobre los parámetros del constructor:

```java
@Service
public class Sistema {
  private static final String os;

  @Autowired
  public Sistema(@Value("#{systemProperties['os.name']}") String os) {
    this.os = os;
  }
...
```

Aunque es un tema que ya se ha comentado anteriormente, es bueno recordar aquí que Spring necesita las directivas adecuadas en el fichero de configuración para activar el escaneo de paquetes y el procesamiento de anotaciones:

```xml
<context:component-scan base-package="com.empresa"/>

<context:annotation-config/>
```
