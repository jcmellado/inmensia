# 37. Spring: Web MVC (2) - Servlets

_29-04-2012_ _Juan Mellado_

Los _servlets_ son la pieza central de una aplicación _web_. Su función es implementar todo el proceso necesario para construir la respuesta adecuada a cada petición que recibe el servidor. Spring proporciona los mecanismos adecuados para evitar mezclar la capa de negocio con la de presentación dentro de un _servlet_.

## 37.1. Servlets

Antes de empezar a utilizar Spring, lo mejor es comenzar viendo un ejemplo básico de como implementar y configurar un _servlet_ "a la manera tradicional".

Un _servlet_ es una clase que hereda de ```HttpServlet```, y cuyo objetivo es procesar peticiones HTTP (GET, POST, PUT, ...). Recibe un objeto petición (_request_) y devuelve un objeto respuesta (_response_):

```java
public class FarmaciaServlet extends HttpServlet {

  @Override
  public void doGet(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
    response.getWriter().println("Hola mundo cruel");
  }
...
```

Los _servlets_ son invocados por el servidor en función de una configuración realizada en el fichero ```web.xml```. Dicha configuración establece para cada _servlet_ que peticiones concretas (URLs) deben atender.

A cada _servlet_ se le asigna un nombre, una referencia a la clase que lo implementa, un indicador de si tiene que cargarse al arrancar el servidor, y una expresión regular que define las URLs para las que se le invocará:

```xml
<servlet>
  <servlet-name>farmacia</servlet-name>
  <servlet-class>com.empresa.FarmaciaServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>farmacia</servlet-name>
  <url-pattern>/farmacia/*</url-pattern>
</servlet-mapping>
```

Debería resultar evidente que implementar una aplicación _web_ entera codificando todas las respuestas dentro de los _servlets_ no resulta nada práctico. Y ahí es donde entran en juego los _frameworks_.

## 37.2. DispatcherServlet

El funcionamiento básico de Spring Web MVC es como el de la mayoría de los _frameworks_ para la _web_. Tiene un _servlet_ principal que recibe todas las peticiones y las despacha en función de una configuración. Dicha configuración establece el mapeo entre las URLs de las peticiones _web_ recibidas y las clases Java correspondientes que deben atendar cada petición. Dichas clases, llamadas "controladores", gestionan el modelo de negocio de la aplicación y deciden la vista a aplicar.

El controlador principal (frontal) de Spring está implementado en la clase ```DispatcherServlet```, que en la práctica es un _servlet_ ordinario que hereda de la clase ```HttpServlet```, y como tal debe configurarse en el fichero ```web.xml``` para que atienda las peticiones de la aplicación _web_:

```xml
<servlet>
  <servlet-name>farmacia</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>farmacia</servlet-name>
  <url-pattern>/farmacia/*</url-pattern>
</servlet-mapping>
```

Para cada _servlet_ debe existir un fichero de configuración en el directorio ```WEB-INF``` llamado ```nombre-servlet.xml```, aunque se puede indicar que se utilice un fichero con otro nombre mediante el atributo ```contextConfigLocation```.

Para que funcione el _servlet_ del ejemplo, debe crearse un fichero ```farmacia-servlet.xml```, aunque sea vacío:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

</beans>
```

Este _servlet_ implementa toda la secuencia que se sigue desde que se recibe una petición y hasta que se construye la respuesta. Por defecto mantiene una lista de _beans_ con una implementación concreta para cada paso del proceso, aunque dichos _beans_ se pueden sustituir para utilizar otras implementaciones según las necesidades de cada aplicación.
