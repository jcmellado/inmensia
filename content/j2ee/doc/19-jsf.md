# 19. JSF (1)

_02-06-2012_ _Juan Mellado_

JSF (Java Server Faces) es un _framework_ estándar para la construcción de aplicaciones _web_. Ofrece algunas de las características básicas no incluidas dentro de JSP que tradicionalmente han obligado a los desarrolladores a utilizar _frameworks_ externos para obtenerlas.

## 19.1. Dependencias

Para construir aplicaciones utilizando JSF es necesario añadir las siguientes dependencias, o similares, en el fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>com.sun.faces</groupId>
  <artifactId>jsf-api</artifactId>
  <version>2.1.9</version>
</dependency>

<dependency>
  <groupId>com.sun.faces</groupId>
  <artifactId>jsf-impl</artifactId>
  <version>2.1.9</version>
</dependency>
```

Algunos servidores proveen las librerías, otros no, y en algunos casos hay incompatibilidades, por lo que se tiene que consultar la documentación de cada servidor en cada caso. Las dependencias indicadas funcionaron correctamente durante las pruebas en un servidor Tomcat 7.0.

## 19.2. Configuración

Como el resto de tecnologías J2EE para la _web_, JSF se basa en el uso de _servlets_. Su filosofía de funcionamiento consiste en colocar un único _servlet_ que recoja todas las peticiones de la aplicación, en vez de obligar a los desarrolladores a definir un _servlet_ por cada tipo de petición.

El _servlet_, de la clase ```javax.faces.webapp.FacesServlet```, se tiene que definir en el fichero ```web.xml``` de la aplicación, y configurar además el tipo de peticiones que debe procesar:

```xml
<servlet>
  <servlet-name>Faces Servlet</servlet-name>
  <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>Faces Servlet</servlet-name>
  <url-pattern>*.xhtml</url-pattern>
</servlet-mapping>
```

El _servlet_ de JSF implementa un ciclo de vida bien definido, de forma que a cada paso de dicho ciclo se invoca a los distintos componentes de la aplicación, dándoles la oportunidad de realizar algún tipo de proceso en cada una de las fases. Esta forma de trabajar es una constante en todos los _frameworks_ de este tipo, y tiene la ventaja añadida de que no impide que se definan otros _servlets_, _listeners_, o _filters_ propios.

## 19.3. faces-config.xml

Aunque no es estrictamente necesario, puede crearse un fichero con el nombre ```faces-config.xml``` dentro del directorio ```WEB-INF``` del _war_, o dentro del directorio ```META-INF``` de un _jar_ incluido en ```WEB-INF/lib```. Este fichero contiene configuración más específica referida a JSF, que preferentemente se define utilizando anotaciones en las clases:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<faces-config xmlns="http://java.sun.com/xml/ns/javaee" 
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                 http://java.sun.com/xml/ns/javaee/web-facesconfig_2_0.xsd"
              version="2.0">

</faces-config>
```

Para evitar conflictos debidos al orden de carga de los ficheros de configuración, JSF permite indicar un orden de carga concreto. Para ello, cada fichero recibe un nombre, y utilizando la etiqueta ```absolute-ordering``` se puede indicar un orden de forma absoluta haciendo referencia a dichos nombres:

```xml
<name>principal</name>
<absolute-ordering>
  <name>primero</name>
  <name>segundo</name>
  <name>tercero</name>
  <others/>
</absolute-ordering>
```

Además, si fuera necesario, se puede indicar un orden de carga relativo utilizando las etiquetas ```ordering```, ```before``` y ```after```:

```xml
<name>primero</name>
<ordering>
  <before>
    <name>segundo</name>
  </before>
</ordering>
```

Una forma alternativa de incluir ficheros de configuración es definirlos dentro de ```web.xml```, añadiendo una propiedad ```javax.faces.CONFIG_FILES``` al contexto con la lista de ficheros separados por coma:

```xml
<context-param>
  <param-name>javax.faces.CONFIG_FILES</param-name>
  <param-value>primero,segundo,tercero</param-value>
</context-param>
```
