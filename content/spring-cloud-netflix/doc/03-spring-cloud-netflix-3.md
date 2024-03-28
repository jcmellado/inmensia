# 3. Spring Cloud Netflix (3)

29-06-2018 Juan Mellado

Continuando con el ejemplo práctico de uso de Spring Cloud Netflix, el siguiente paso es montar los proyectos que arranquen el servidor de configuración y el de descubrimiento de servicios.

## 3.1. cloud-greeter-config

Este proyecto es un servidor de configuración basado en Spring Cloud Config. Suministra la configuración al resto de servicios de una forma centralizada.

Cuando un servicio arranca se conecta con este servidor y le pide sus propiedades de configuración. Las propiedades se pueden configurar para obtenerse desde distintas fuentes, como por ejemplo desde un servidor git, lo que permite tener un control de versiones sobre los ficheros de configuración de igual forma que se tiene habitualmente sobre el código fuente.

El servidor se implementa como una aplicación de Spring Boot. Y de hecho esto es una constante para todos los módulos. Cada módulo es una aplicación de Spring Boot. Lo que quiere decir que cada uno de ellos es una aplicación independiente que contiene embebido su propio servidor de aplicaciones. Una vez compilados se pueden arrancar y parar de forma individual, de forma que al final se tendrán tantos servidores de aplicaciones levantados como servidores y servicios se tengan.

El fichero ```pom.xml``` del proyecto se limita a heredar del proyecto padre y añadir las dependencias de Spring Boot y Spring Cloud Config:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>com.inmensia</groupId>
    <artifactId>cloud-greeter-parent</artifactId>
    <version>0.1.0-SNAPSHOT</version>
    <relativePath>../cloud-greeter-parent</relativePath>
  </parent>

  <artifactId>cloud-greeter-config</artifactId>
  <packaging>jar</packaging>

  <dependencies>

    <!-- Spring Cloud -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <!-- Spring Boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-undertow</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

  </dependencies>

  <build>
    <plugins>

      <!-- Spring Boot -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>${spring-boot.version}</version>
      </plugin>

    </plugins>

  </build>

</project>
```

La dependencia ```spring-boot-starter-undertow``` hace que se utilice Undertow como servidor de aplicaciones, un servidor embebible que rinde bien con pocos recursos, algo necesario para este proyecto que levanta muchos servicios de forma local. Aunque se podría utilizar cualquier otro servidor, como Jetty o Apache por ejemplo.

La dependencia ```spring-boot-starter-actuator``` es una utilidad de Spring Boot que expone una serie de servicios REST para la aplicación de forma automática. Estos servicios implementan tareas habituales, evitando que los desarrolladores tengan que implementarlos por su cuenta, como por ejemplo un servicio que atiende la ruta ```/health``` que se puede utilizar para comprobar si el servidor está levantado y retorna un JSON con información acerca del sistema.

La única clase del proyecto es el punto de entrada a la aplicación:

```java
@EnableConfigServer
@SpringBootApplication
public class Application {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

El servidor se arranca como una aplicación de Spring Boot ordinaria gracias a la anotación ```@SpringBootApplication```. Y el servidor de configuración se activa con la anotación ```@EnableConfigServer```. Así de simple, no hay más. No es de extrañar que Spring Boot se haya convertido en un referente para la construcción de microservicios para las empresas con equipos de desarrolladores Java que necesitan obtener resultados de una forma rápida.

El único fichero de configuración del proyecto es ```application.yml```, que es el propio fichero de configuración cargado por defecto por Spring Boot:

```yaml
server:
  port: 8100

spring:
  application:
    name: cloud-greeter-config
      profiles:
        active: native

logging:
  file: logs/${spring.application.name}.log
  level:
    org.springframework.cloud: DEBUG
    com.inmensia: DEBUG

---

spring:
  profiles: native
  cloud:
    config:
      server:
        native:
          searchLocations: ${CLOUD_GREETER_CONFIG_PATH}
```

En el fichero se configura el puerto en que se debe levantar el servidor de aplicaciones, el nombre de la aplicación, la ubicación de los ficheros de logs, y el repositorio con los ficheros de configuración de los servicios. La ubicación del repositorio se realiza mediante una variable de entorno, de forma que pueda cambiarse fácilmente. Otra forma más habitual de hacerlo es utilizar varios profiles y definir el nombre del profile a utilizar en una variable de entorno.

El servidor se puede arrancar como cualquier otra aplicación de Spring Boot. Por ejemplo desde línea de comandos, en el directorio donde se encuentre el módulo, utilizando el plugin de Spring Boot para Maven:

```text
mvn spring-boot:run "-DCLOUD_GREETER_CONFIG_PATH=../../../cloud-greeter-config"
```

El servidor arrancará en el puerto 8100 y estará disponible a través de la ruta ```http://localhost:8100```, pudiéndose comprobar su estado a través de la ruta expuesta por Spring Boot Actuator:

```text
http://localhost:8100/health
```

## 3.2. cloud-greeter-eureka

Este proyecto es un servidor de descubrimiento de servicios basado en Eureka. Mantiene el registro centralizado de los servicios disponibles y garantiza una alta disponibilidad de los mismos, o al menos una alta posibilidad de encontrar una instancia de los mismos en funcionamiento a pesar de las incidencias que puedan producirse, como cortes de red o caídas de servidores.

La estructura del proyecto es idéntica que la del anterior. En el fichero ```pom.xml``` únicamente cambian las dependencias principales:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka-server</artifactId>
  </dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

La dependencia ```spring-cloud-config-client``` es para utilizar el cliente de Spring Cloud Config. Es decir, que el servidor Eureka también recoge su configuración del servidor de configuración. De hecho, la idea es que todas las aplicaciones que se levanten, tanto los servicios que implementen lógica de negocio, como los servidores que les dan soporte, utilicen el servidor de configuración.

La única clase del proyecto es el punto de entrada a la aplicación:

```java
@EnableEurekaServer
@SpringBootApplication
public class Application {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

El servidor se arranca como una aplicación de Spring Boot ordinaria gracias a la anotación ```@SpringBootApplication```. Y el servidor Eureka se activa con la anotación ```@EnableEurekaServer```.

El único fichero de configuración del proyecto es ```bootstrap.yml```, que es el propio fichero de configuración cargado por defecto por Spring Boot cuando una aplicación se configura para obtener sus propiedades desde un servidor de configuración:

```yaml
spring:
  application:
    name: cloud-greeter-eureka
  cloud:
    config:
     failFast: true
     uri: http://localhost:8100
```

En el fichero se configura el nombre de la aplicación y la URL donde se encuentra el servidor de configuración.

El servidor Eureka obtiene su fichero de configuración del servidor de configuración. Fichero que se llama ```cloud-greeter-eureka.yml``` y se encuentra en el repositorio del proyecto ```cloud-greeter-config``` junto con el resto de ficheros de configuración. El servidor de configuración simplemente casa el nombre de la aplicación con el nombre del fichero. Si existe un fichero con el nombre de la aplicación supone que es el fichero de configuración de dicha aplicación y se lo sirve.

```yaml
server:
  port: 8101

eureka:
  instance:
    hostname: localhost
    client:
      registerWithEureka: false
      fetchRegistry: false

logging:
  file: logs/${spring.application.name}.log
  level:
    org.springframework.cloud: 'DEBUG'
    com.inmensia: 'DEBUG'
```

En el fichero se configura el puerto en que se debe levantar el servidor de aplicaciones, la ubicación de los ficheros de logs, y los parámetros propios del servidor Eureka. Notar que uno de los parámetros sirve para evitar que el propio servidor se registre contra si mismo.

El servidor se puede arrancar como cualquier otra aplicación de Spring Boot. Por ejemplo desde línea de comandos, en el directorio donde se encuentre el módulo, utilizando el plugin de Spring Boot para Maven:

```text
mvn spring-boot:run
```

El servidor arrancará en el puerto ```8101``` y estará disponible a través de la ruta ```http://localhost:8101```. Al acceder a la URL se abre un panel de control con los servicios registrados. A medida que se levanten nuevos servicios estos irán apareciendo en el listado.

En buena lógica, los servidores pueden configurarse en cluster para asegurar su disponibilidad y que no representen un punto único de fallo.
