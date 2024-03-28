# 2. Spring Boot 2 (2)

_16-07-2018_ _Juan Mellado_

Hace un tiempo elaboré una breve introducción a Spring Boot con el objetivo de que pudiera ser utilizada como referencia dentro de una empresa en la que trabajaba. Con el tiempo los fuentes correspondientes acabaron publicados, y ahora con el cambio de versión de Spring Boot han quedado un poco desactualizados. Afortunadamente la documentación de Spring Boot es bastante detallada y es la mejor referencia disponible. El objetivo de este artículo y sucesivos es enmendar esas notas introductorias utilizando Spring Boot 2 para construir una serie de aplicaciones básicas usando versiones más recientes de software, como Java 10, Eclipse Photon o Tomcat 9.

## 2.1. Configuración Automática

Una de las ventajas de usar Spring Boot es que crear un proyecto con Maven o Gradle es muy sencillo si se utiliza Spring Initializr, una _web_ que genera esqueletos de proyectos para Spring Boot. La _web_ ofrece una interface muy simple que permite seleccionar el tipo de proyecto, el lenguaje de programación, la versión de Spring Boot, las dependencias con librerías de terceros, y genera automáticamente un fichero zip que contiene el proyecto listo para ser compilado y ejecutado.

## 2.2. Configuración Manual

Para crear un proyecto de una forma más manual con Maven hay que crear un fichero ```pom.xml``` básico como el siguiente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>{{GRUPO}}</groupId>
  <artifactId>{{NOMBRE}}</artifactId>
  <version>{{VERSION}}</version>
  <packaging>jar</packaging>

  <properties>
    <java.version>10</java.version>

    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>

</project>
```

A partir de ese fichero básico ya se puede empezar a añadir o quitar entradas para construir un tipo u otro de proyecto. Por ejemplo, para conseguir que el proyecto sea un proyecto de Spring Boot es necesario añadir ```org.springframework.boot:spring-boot-starter-parent``` como proyecto padre en el fichero ```pom.xml```:

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.0.3.RELEASE</version>
</parent>
```

Haciendo que ```org.springframework.boot:spring-boot-starter-parent``` sea el padre de un proyecto se consigue que de forma automática se haga referencia a una serie concreta de versiones de librerías, tanto de Spring como de terceros, que funcionan correctamente de forma conjunta con la versión usada de Spring Boot, evitando así a los proyectos tener que configurar las dependencias de manera individual.

Lo que es importante en este punto es entender que al configurar el proyecto padre no se añaden las dependencias (librerías) al proyecto, sólo se indican las versiones de las mismas que se quieren utilizar. De hecho, es incluso posible configurar el proyecto sin tener como padre a Spring Boot, importando directamente en el fichero ```pom.xml``` el conjunto de referencias de Spring Boot:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.0.3.RELEASE</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Las dos opciones vistas, utilizar el proyecto padre o importar las referencias, son válidas, pudiéndose utilizar una u otra dependiendo de cada caso en concreto. Para el ejemplo se continuará con la primera opción, lo importante es conocer que existen distintas opciones.

## 2.3. Aplicación de Consola

Aunque tradicionalmente Spring se ha asociado a la construcción de aplicaciones _web_, y Spring Boot a la construcción de microservicios REST, en realidad el núcleo de Spring siempre se ha podido utilizar para aplicaciones de línea de comandos.

Spring Boot facilita la creación de distintos tipos de aplicación mediante módulos Maven llamados "_starters_". Así, para crear una aplicación de consola, el primer paso es añadir la dependencia ```org.springframework.boot:spring-boot-starter``` en el fichero ```pom.xml```:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>
</dependencies>
```

El siguiente paso es añadir una clase que sirva como punto de entrada a la aplicación de Spring Boot. Para ello basta con crear en ```/src/main/java/{paquete}``` una clase anotada con ```@SpringBootApplication``` y con un método estático ```main``` estándar de Java:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

La anotación ```@SpringBootApplication``` es equivalente, entre otras, a las anotaciones ```@Configuration``` y ```@ComponentScan``` de Spring.

El último paso es compilar y ejecutar la aplicación utilizando el propio _plugin_ de Maven de Spring Boot:

```text
mvn spring-boot:run
```

Si todo funciona correctamente, arrancará la aplicación de Spring Boot mostrando por consola el _log_ de ejecución de la aplicación, donde, como mínimo, deberá verse el famoso _ascii-art_ de Spring Boot (¡que en Spring Boot 2 se puede animar!) y la versión concreta utilizada:

```tet
Spring Boot :: (v2.0.3.RELEASE)
```

Lógicamente, la aplicación de consola arrancará y terminará inmediatamente, ya que no hace nada, pero cumple su función de hacer entender como construir un proyecto básico desde cero.

## 2.4. Aplicación web

Para construir una aplicación _web_ con Spring Boot basada en Spring MVC hay que añadir el _starter_ para _web_ al fichero ```pom.xml```:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

Crear en ```/src/main/java/{paquete}``` una clase anotada con ```@EnableAutoConfiguration```:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class Application {

  public static void main(String[] args) {
  SpringApplication.run(Application.class, args);
  }
}
```

Y compilar y ejecutar la aplicación con el _plugin_ de Maven de Spring Boot:

```text
mvn spring-boot:run
```

Si todo funciona correctamente arrancará una instancia de Tomcat 8.5 y se mostrará por consola el _log_ de ejecución, incluyendo el puerto y el contexto en que se encuentra escuchando el servidor, siendo esto último, por cierto, una novedad de Spring Boot 2:

```text
Tomcat started on port(s): 8080 (http) with context path ''
```

Una vez arrancado el servidor se puede parar con la habitual combinación de teclas ```Control+C```.

## 2.5. Cambiar Puerto de Tomcat

Para cambiar el puerto por defecto de arranque de Tomcat hay que crear un fichero de propiedades e indicar el nuevo puerto. El fichero de propiedades se debe ubicar en ```/src/main/resources```, debe llamarse ```application```, y puede estar en el formato ```.properties``` clásico de Java o en formato YAML.

Si se utiliza ```application.properties``` el fichero debe tener el siguiente contenido:

```properties
server.port = 9999
```

Si se utiliza ```application.yml``` el fichero debe tener el siguiente contenido:

```yaml
server:
  port: 9999
```

Al arrancar de nuevo el servidor, Tomcat escuchará en el nuevo puerto indicado:

```text
Tomcat started on port(s): 9999 (http) with context path ''
```

El uso del fichero de propiedades es una constante en Spring Boot. Por defecto el _framework_ utiliza una serie de valores predefinidos según el criterio del equipo de Spring, de forma que se puede arrancar una aplicación sin tener que configurar ningún valor, pero deja la puerta abierta a cambiar dichos valores por parte de los desarrolladores a través del fichero de propiedades.

En lo sucesivo se sobreentiende que se utiliza el formato YAML para los ejemplos, pero usar un formato u otro es indistinto.

## 2.6. Cambiar Versión de Tomcat

Para cambiar la versión de Tomcat hay que añadir una propiedad en el fichero ```pom.xml```:

```xml
<properties>
...
  <tomcat.version>9.0.10</tomcat.version>
...
</properties>
```

Al arrancar de nuevo el servidor, la versión de Tomcat utilizada será la nueva indicada:

```text
Starting Servlet Engine: Apache Tomcat/9.0.10
```

Notar que la propiedad se añade al fichero ```pom.xml```, y no al fichero de propiedades de la aplicación, porque es un parámetro necesario para construir la aplicación, para descargar la versión indicada de Tomcat y embeberla dentro de la aplicación, no un parámetro que se vaya a utilizar en tiempo de ejecución.

## 2.7. Cambiar Servidor de Aplicaciones

Para utilizar Jetty o Undertow en vez de Tomcat hay que excluir la dependencia con Tomcat del fichero ```pom.xml``` e incluir la dependencia con el nuevo servidor que se quiera utilizar.

Por ejemplo, para utilizar Jetty hay que realizar la siguiente modificación:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>

<!-- Excluye Tomcat -->
    <exclusions>
      <exclusion>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
      </exclusion>
    </exclusions>

  </dependency>

<!-- Incluye Jetty -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
  </dependency>

</dependencies>
```

Al volver a arrancar la aplicación se levantará una instancia de Jetty en vez de Tomcat:

```text
Jetty started on port(s) 9999 (http/1.1) with context path '/'
```

En todos los casos, accediendo a la URL ```http://localhost:9999/``` se puede ver la página de error 404 por defecto de Spring, ya que de momento no se ha añadido ningún servicio al servidor.
