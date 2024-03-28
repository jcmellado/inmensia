# 13. JSP (6) - Tag Library

_16-05-2012_ _Juan Mellado_

La tecnología que permite utilizar acciones dentro de un JSP está diseñada para ser extensible, lo que quiere decir que permite crear acciones para uso propio dentro de una aplicación, o de propósito general para todo de tipo de aplicaciones, característica esta última ampliamente explotada por los _frameworks_.

Una _tag library_ es una librería de acciones, y el propósito principal de las acciones es encapsular código reutilizable dentro de los JSP. En resumen, es un forma más de añadir código evitando el uso de _scriptlets_.

El proceso de construcción de las librerías varía en función de la complejidad de las acciones que se quieran incluir dentro de las mismas. Las más sencillas pueden construirse con un único fichero que contenga código estático HTML. Las más complejas pueden requerir además crear clases Java que implementen determinadas interfaces del API de JSP.

## 13.1. Tag File

Un _tag file_ es un fichero de texto, con extensión ```.tag``` o ```.tagx```, que contiene un fragmento de código JSP, y se ubica en el directorio ```/WEB-INF/tags```, o dentro de alguno de sus subdirectorios.

Un sencillo ejemplo sería el del siguiente fichero ```hello.tag```:

```xml
<h1>¡Hola Tag!</h1>
```

Para utilizarlo dentro de un JSP bastaría con importarlo e invocar la acción de la siguiente forma:

```jsp
<%@ taglib tagdir="/WEB-INF/tags" prefix="t" %>
...
<t:hello/>
```

O utilizando la nomenclatura XML, que requiere utilizar el prefijo ```urn:jsptagdir```:

```xml
<... xmlns:t="urn:jsptagdir:/WEB-INF/tags">
...
<t:hello/>
```

Un ejemplo un poco más elaborado sería el siguiente, donde se añade al fichero ```hello.tag``` una directiva para declarar un atributo que es utilizado dentro de una expresión EL, permitiendo de esta forma generar contenido de forma dinámica:

```jsp
<%@ attribute name="nombre" required="true" %>

<h1>¡Hola ${nombre}!</h1>
```

El atributo estaría disponible automáticamente dentro de los JSP que utilicen la acción, y de hecho, sería obligatorio introducirlo:

```xml
<t:hello nombre="mundo"/>
```

Los _tag files_ admiten las directivas ```tag```, ```attribute``` y ```variable```, además de las directivas ```taglib``` e ```include``` vistas anteriormente. Cada directiva permite indicar una gran cantidad de parámetros, por lo que es recomendable leer la documentación de referencia para entender el significado y alcance de cada uno de ellos.

## 13.2. Tag Library Descriptor

Un _tag library descriptor_ (TLD) es un fichero de texto en formato XML, con extensión ```.tld```, que describe una librería. Las librerías sencillas como la del ejemplo anterior no requieren ningún descriptor, pero las librería más complejas, o aquellas que se quieren redistribuir para ser usadas por otros, sí lo necesitan.

Un sencillo ejemplo sería el siguiente ```hello.tld```, ubicado en el directorio ```/WEB-INF/tags```, que muestra algunos de los atributos básicos que admite un descriptor:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation=
          "http://java.sun.com/xml/ns/javaee web-jsptaglibrary_2_0.xsd"
        version="2.0">

  <short-name>hello</short-name>
  <tlib-version>1.0</tlib-version>

  <tag-file>
    <name>hello</name>
    <path>/WEB-INF/tags/hello.tag</path>
  </tag-file>

</taglib>
```

El descriptor establece algunos parámetros puramente informativos y se limita a hacer referencia al fichero ```hello.tag``` definido anteriormente. Evidentemente, donde realmente toma sentido un descriptor es cuando se declaran varias acciones que se exponen de forma pública para ser reutilizadas.

Para que un servidor _web_ tenga constancia de un descriptor, se debe hacer referencia al mismo dentro del fichero ```web.xml``` de forma similar a la siguiente:

```xml
<jsp-config>
  <taglib>
    <taglib-uri>hello</taglib-uri>
    <taglib-location>/WEB-INF/tags/hello.tld</taglib-location>
  </taglib>
</jsp-config>
```

De igual forma que antes, se pueden definir una gran cantidad de parámetros en un descriptor, por lo que es recomendable leer la documentación de referencia para entender el significado y alcance de cada uno de ellos.

## 13.3. Tag Handler

Un _tag handler_ es una clase Java que implementa la interface ```Tag```, ```IterationTag```, o ```BodyTag```. Es la forma de añadir código más elaborado a una acción.

La siguiente clase ```HelloTag``` representa un ejemplo sencillo, que por comodidad además se hace heredar de ```SimpleTagSupport```, para evitar tener que implementar completamente una de las interfaces:

```java
public class HelloTag extends SimpleTagSupport {

  @Override
  public void doTag() throws JspException, IOException {
    getJspContext().getOut().write("¡Hola Tag Handler!");
  }
}
```

Para poder compilar el código es necesario añadir la dependencia correspondiente al fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>javax.servlet.jsp</groupId>
  <artifactId>jsp-api</artifactId>
  <version>2.0</version>
  <scope>provided</scope>
</dependency>
```

Y por último, modificar el descriptor ```hello.tld``` para que en vez de hacer referencia al fichero ```hello.tag``` haga referencia a la nueva clase:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee web-jsptaglibrary_2_0.xsd"
version="2.0">
 
  <short-name>hello</short-name>
  <tlib-version>1.0</tlib-version>

  <tag>
    <name>saludo</name>
    <tag-class>com.company.tag.HelloTag</tag-class>
    <body-content>scriptless</body-content>
  </tag>

</taglib>
```

De esta forma, dentro de un JSP se puede incluir una referencia a la librería, sin tener que especificar la ruta completa, y utilizar sus acciones:

```jsp
<%@ taglib uri="hello" prefix="h" %>
...
<h:saludo/>
O utilizando la sintaxis XML:

<... xmlns:h="hello">
...
<h:saludo/>
```

Un ejemplo un poco más avanzado consistiría en añadir un atributo a la declaración de la acción:

```xml
...
  <tag>
    <name>saludo</name>
    <tag-class>com.company.tag.HelloTag</tag-class>
    <body-content>scriptless</body-content>
   
    <attribute>
      <name>nombre</name>
      <required>true</required>
      <rtexprvalue>true</rtexprvalue>
    </attribute>
  </tag>
...
```

Y el correspondiente atributo en la clase Java:

```java
public class HelloTag extends SimpleTagSupport {

  private String nombre;

  public void setNombre(String nombre) {
    this.nombre = nombre;
  }

  @Override
  public void doTag() throws JspException, IOException {
    getJspContext().getOut().write("¡Hola " + this.nombre + "!");
  }
...
```

De forma que desde un JSP que utilice la acción se le pueda pasar un valor:

```jsp
<h:saludo nombre="mundo"/>
```

La construcción de librerías de acciones es un tema un tanto avanzado, se suelen desarrollar dentro de _frameworks_, y no de aplicaciones _web_ convencionales, por lo que es conveniente leer la documentación de referencia para conocer los detalles concretos de implementación.
