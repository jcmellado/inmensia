# 36. Spring: Web MVC (1) - Configuración

_29-04-2012_ _Juan Mellado_

La capa de presentación basada en _web_ es ofrecida por Spring a través de _Web MVC framework_, sin que ello implique renunciar al uso de otras tecnologías populares como Struts o JSF.

## 36.1. Proyecto

Evidentemente, uno de los cambios importantes que hay que tener en cuenta con respecto a los ejemplos vistos hasta ahora, es que una aplicación _web_ no se ejecuta con un tradicional ```main```, por lo que la forma de instanciar el ```ApplicationContext``` a partir de un fichero XML tiene que cambiarse.

De hecho, también tiene que cambiarse el tipo de proyecto creado desde Eclipse. Más concretamente, debe crearse un proyecto de tipo _web_ (_File / New / Project... / Web / Dynamic Web Project / No target runtime / Finish_). Esto determina la creación de la estructura de directorios adecuada, y sobre todo que el modo de empaquetamiento sea de tipo _war_.

Para seguir utilizando Maven es necesario modificar el proyecto para añadirle esa faceta (_Configure / Convert to Maven Project_). Y por comodidad, es mejor unificar la política de ubicación del directorio raíz de despliegue a ```src/main/webapp```, y de la inclusión de las dependencias con Maven dentro de ```WEB-INF/lib``` (_Properties / Deployment Assembly_).

Una aplicación _web_ se tiene que ejecutar dentro de algún tipo de servidor. Como el popular servidor de aplicaciones Tomcat, que puede descargarse e instalarse directamente desde el propio Eclipse. (_Windows / Show View / Other... / Server / Servers / new server wizard... / Apache / Tomcat v7.0 Server / Next / Download and Install / Next / Finish_).

Un último destalle a tener en cuenta es que es necesario cambiar el puerto por defecto de Tomcat, ya que es el mismo que utiliza la base de datos Oracle XE que se está usando para las pruebas (_Directorio instalación / conf / server.xml / Connector port = ..._).

Finalmente, si todo hay ido correctamente, se podrá gestionar el arranque y parada del servidor _web_, e incluso el propio despliegue y prueba de la aplicación, directamente desde Eclipse.

Posiblemente esta no sea la forma más sencilla, ni desde luego la única, pero a mi personalmente me sirve de referencia para controlar esos pequeños detalles a tener en cuenta en caso de surgir algún problema.

## 36.2. Dependencias

Como de costumbre, para utilizar una característica concreta de Spring hay que añadir la dependencia correspondiente en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
```

Notar que se añade también una dependencia con ```servlet-api```, que contiene el API necesario para poder desarrollar _servlets_. Aunque sólo es necesaria durante la compilación, ya que es una dependencia que el servidor proveerá, y de ahí que se marque con el valor ```provided``` en su atributo ```scope```.

## 36.3. Configuración

Spring ofrece dos clases para cargar un fichero XML con el que instanciar el contexto en una aplicación _web_. La primera es ```ContextLoaderListener```, y la segunda ```ContextLoaderServlet```. La única diferencia real entre ambas es que la segunda sólo se puede ejecutar en servidores que cumplan con la especificación Servlet 2.4. Aunque en la práctica, con la primera debería ser suficiente para cualquier aplicación.

Estas clases actúan como listeners, y se inicializan justo después de que el servidor cree el contexto para la ejecución de _servlets_, momento perfecto para instanciar el objeto ```ApplicationContext``` de Spring.

Al tratarse de un proyecto _web_, la configuración se realiza en el fichero ```web.xml```:

```xml
<listener>
  <listener-class>
    org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>
```

Esta clase intenta leer el fichero ```/WEB-INF/applicationContext.xml``` por defecto. Y si no lo encuentra eleva una excepción. No obstante, se puede también configurar para que utilice uno o varios ficheros distintos:

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>
```

Y de igual forma que lo visto en artículos anteriores, las aplicaciones _web_ también pueden utilizar clases marcadas con la anotación ```@Configuration``` para definir el contexto usando los parámetros de configuración adecuados:

```xml
<context-param>
  <param-name>contextClass</param-name>
  <param-value>
    org.springframework.web.context.support.AnnotationConfigWebApplicationContext
   </param-value>
</context-param>

<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>com.empresa.AppConfig</param-value>
</context-param>
```

Si la configuración es correcta, Spring creará un contexto de tipo ```WebApplicationContext```, que es una clase especializada de contexto que añade características como la gestión de temas (_themes_) y, por supuesto, de _servlets_.

Una cosa importante a tener en cuenta es que la clase ```ContextLoaderListener``` no está relacionada con Web MVC, sólo sirve para inicializar el contexto de una forma conveniente, por lo que resulta útil también para trabajar con Spring cuando se usan otros _frameworks_ de presentación distintos de Web MVC.
