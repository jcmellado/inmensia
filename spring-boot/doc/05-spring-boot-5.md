# 5. Spring Boot 2 (y 5)

_25-07-2018_ _Juan Mellado_

Para cerrar el ejemplo de uso básico de Spring Boot 2, en este artículo se revisan algunas formas de empaquetar una aplicación _web_ de cara a su distribución. Decidir utilizar un formato u otro depende de cada caso de uso en concreto, no existe una opción mejor que las demás. En algunos entornos/empresas se sigue requiriendo un WAR/EAR para su despliegue en un servidor de aplicaciones, en otros se requiere un único fichero JAR que se pueda ejecutar con una determinada máquina virtual, y en otros se requiere una imagen de Docker.

## 5.1. WAR

Spring Boot permite empaquetar una aplicación en un fichero WAR, eliminando el servidor embebido del empaquetado, y que se puede desplegar sobre un servidor de aplicaciones. Lo que quiere decir que lo que se obtiene es un único fichero con la aplicación, con sus dependencias incluidas dentro del propio WAR, o proporcionadas por el servidor de aplicaciones con objeto de ser compartidas por todas las aplicaciones desplegadas en el mismo.

Para generar un WAR hay que cambiar el formato de empaquetado del proyecto en el fichero ```pom.xml```.

```xml
<packaging>war</packaging>
```

Para excluir el servidor Tomcat embebido por defecto por Spring Boot dentro de la aplicación hay que añadir una entrada en el ```fichero pom.xml``` indicando que las librerías de Tomcat son proporcionadas de forma externa:

```xml
<dependencies>
  ...

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
  </dependency>

</dependencies>
```

Para hacer que la aplicación pueda reaccionar al proceso de inicialización del servidor de aplicaciones, ya que ahora será el servidor y no Spring Boot quien arranque la aplicación, se debe cambiar la clase con el punto de entrada de la aplicación para que extienda de ```SpringBootServletInitializer``` y sobreescribir su método ```configure```:

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder application)   {
    return application.sources(Application.class);
  }
}
```

Y por último, para generar el WAR, hay que ejecutar la tarea de empaquetado estándar de Maven:

```text
mvn package
```

Como resultado de la ejecución de la tarea se genera un fichero WAR en el directorio ```\target``` que puede desplegarse sobre un servidor de aplicaciones de la forma acostumbrada.

A pesar de considerarse un modelo obsoleto, esta forma de distribución se sigue utilizando en muchas empresas. Dejar un fichero en un directorio para que lo desplieguen en un servidor de aplicaciones sigue siendo bastante más habitual de lo que debería.

## 5.2. Nested JARs

Spring Boot permite crear un único fichero JAR que contenga todas las dependencias en un formato propio llamado Nested JARs. Lo que quiere decir que lo que se obtiene es un único fichero, con la aplicación y sus dependencias, y se necesita una máquina virtual para ejecutarlo.

Para generar este tipo de ficheros se debe añadir al fichero ```pom.xml``` un _plugin_ de Spring Boot:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

Y ejecutar la tarea estándar de empaquetado de Maven:

```text
mvn package
```

Como resultado de la ejecución de la tarea se genera un fichero JAR en el directorio ```\target``` que puede ejecutarse con la máquina virtual de Java de la forma acostumbrada:

```text
java -jar .\target\{{NOMBRE}}.jar
```

El fichero JAR que genera el _plugin_ de Spring Boot contiene tanto las clases de la aplicación como las librerías de terceros. Es un modelo de empaquetado similar al formato Uber JAR, en el que todas las clases de todas las librerías de terceros se incluyen dentro del JAR como si fueran parte de la aplicación. La diferencia es que Spring Boot incluye los JARs completos, en vez de sólo sus clases, y utiliza un _loader_ propio para poder realizar la carga de dichos JARs.

Es conveniente abrir el JAR generado y examinarlo para entender como está construido. Las clases en el directorio raíz son el cargador de Spring Boot, y las librerías están en un directorio propio llamado ```BOOT-INF```, de forma similar a como se encuentran en el directorio ```WEB-INF``` de una aplicación _web_.

En la práctica, sólo he visto utilizar este modelo de distribución en un proyecto, concretamente para una aplicación de escritorio que se incluía dentro de los terminales de punto de venta de una gran cadena comercial.

## 5.3. Docker

Las aplicaciones empaquetadas en un único fichero son adecuadas para su distribución en entornos _cloud_. La mayoría de plataformas de computación en la nube permiten configurar una capa por encima de las aplicaciones con la máquina virtual de Java o el servidor de aplicaciones que necesiten.

Si una aplicación se distribuye como un WAR necesita un servidor de aplicaciones y una máquina virtual de Java, pero puede haber problemas en el entorno de producción si hay diferencias entre el _software_ proporcionado por la plataforma y el utilizado en el entorno de desarrollo. De igual forma, si una aplicación se distribuye como un JAR con un servidor embebido, sólo necesita una máquina virtual de Java, pero todavía puede haber problemas si las versiones en los entornos de producción y desarrollo son distintas. Por su parte, en una imagen de Docker se empaqueta tanto la aplicación como todo el _software_ que necesita, consiguiendo que los entornos de desarrollo y producción sean lo más parecidos posibles.

La forma más directa de generar una imagen de Docker es escribir manualmente un fichero de texto plano con nombre ```Dockerfile```, tal cual, sin extensión. En este fichero se indica la imagen base que se quiere utilizar y los pasos necesarios para construir la nueva imagen a partir de la base.

Para nuestro ejemplo necesitamos una imagen que tenga una máquina virtual de Java 10. La imagen oficial proporcionada por OpenJDK en su versión _slim_ está basada en Debian y pesa casi 400Mb, pero para el ejemplo el tamaño no importa.

Spring Boot recomienda utilizar un fichero ```Dockerfile``` como el siguiente:

```Dockerfile
FROM openjdk:10.0.1-10-jdk-slim-sid
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app.jar"]
```

En el fichero se parte de la imagen del OpenJDK, se indica que se cree un directorio temporal, se copie el JAR de la aplicación, y que cuando se ejecute la imagen se levante la maquina virtual de Java para correr la aplicación desde el JAR copiado.

El directorio temporal no es estrictamente necesario, pero Spring Boot recomienda crearlo para garantizar que todas las aplicaciones de Java funcionen correctamente, ya que algunas lo necesitan. El parámetro ```java.security.egd``` se configura para hacer que Tomcat arranque más rápido, y lo que hace es definir un fuente de entropía no bloqueante para la generación de números aleatorios necesarios para las librerías de seguridad.

Una vez escrito el fichero ```Dockerfile``` lo siguiente es invocar a docker desde línea de comandos para construir la imagen de la forma acostumbrada:

```text
docker build -t {{NOMBRE}} --build-arg JAR_FILE={{NOMBRE}}.jar .
```

La nueva imagen se listará junto con el resto:

```text
docker images
```

Y se podrá ejecutar por su nombre (_tag_) haciendo público el puerto en el que se encuentra el servicio:

```text
docker run -p 9999:9999 -t {{NOMBRE}}
```

Si todo el proceso se ejecuta de forma correcta la aplicación arrancará y el servicio estará disponible en el puerto indicado.

Para terminar, comentar que existen otros modelos de distribución a parte de los tres vistos. Uno de ellos es empaquetar la aplicación como un autoejecutable, de forma que pueda ejecutarse sin necesidad de anteponer ```java -jar```, pero sólo funciona en determinados entornos UNIX.
