# 2. Spring Cloud Netflix (2)

_26-06-2018_ _Juan Mellado_

Hace ya un tiempo desarrollé un ejemplo mostrando un caso de uso sencillo de Spring Cloud Netflix. El proyecto acabó sirviendo de referencia e introducción a esta tecnología para los equipos de desarrollo de una empresa para la que trabajaba y el código fuente publicado en GitHub en dos repositorios, uno con los fuentes de los servicios y otro con los ficheros de configuración:

- [https://github.com/jcmellado/cloud-greeter](https://github.com/jcmellado/cloud-greeter)

- [https://github.com/jcmellado/cloud-greeter-config](https://github.com/jcmellado/cloud-greeter-config)

El ejemplo se compone de una estructura muy sencilla, sin distracciones, con la intención de fomentar las buenas prácticas en el uso de Maven para la configuración de proyectos con múltiples módulos, de forma que las dependencias comunes están declaradas en un proyecto padre del que todos heredan y cada proyecto sólo incluye las dependencias propias.

El objetivo del proyecto es mostrar las piezas mínimas necesarias para realizar una instalación con Spring Cloud Netflix, desarrollando tanto los servicios propios, que ejecutan la lógica de negocio, como los servicios de la plataforma, necesarios para que los servicios propios se puedan configurar de forma remota, se puedan invocar directamente entre ellos, o de forma externa a través de una ruta pública, se puedan ejecutar de forma balanceada, y se puedan monitorizar.

El directorio raíz del proyecto contiene el fichero ```pom.xml``` que incluye todos los módulos en los que se encuentra estructurado:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/maven-v400.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.inmensia</groupId>
  <artifactId>cloud-greeter</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <modules>
    <module>modules/cloud-greeter-parent</module>
    <module>modules/cloud-greeter-config</module>
    <module>modules/cloud-greeter-eureka</module>
    <module>modules/cloud-greeter-service</module>
    <module>modules/cloud-greeter-api</module>
    <module>modules/cloud-greeter-turbine</module>
    <module>modules/cloud-greeter-zuul</module>
  </modules>
</project>
```

- ```cloud-greeter-parent```: Proyecto padre del que todos heredan. Contiene la configuración común y declara las dependencias compartidas por todos.

- ```cloud-greeter-config```: Proyecto correspondiente al servicio de configuración. Arranca una instancia del servidor Spring Cloud Config que todos los servicios utilizan para obtener su configuración de forma remota.

- ```cloud-greeter-eureka```: Proyecto que arranca el servidor Eureka utilizado para el registro y descubrimiento de servicios.

- ```cloud-greeter-service```: Este proyecto contiene un servicio de ejemplo. Expone un servicio REST que recibe un JSON con un nombre y retorna un JSON con una cadena de texto que contiene el nombre recibido dentro del típico "Hello \<nombre>!".

- ```cloud-greeter-api```: Este proyecto contiene un servicio que invoca al servicio del proyecto anterior. La idea es de que este servicio es expuesto de forma pública mientras el anterior sólo se puede invocar de forma interna.

- ```cloud-greeter-zuul```: Proyecto que arranca el servidor Zuul que actúa como enrutador. Recibe las peticiones de llamadas a los servicios y las enruta hacia los servicios levantados aplicando balanceo de carga.

- ```cloud-greeter-turbine```: Proyecto que arranca el servidor Turbine que expone una aplicación web a través de la que se puede monitorizar el estado de los servicios.

## 2.1. cloud-greeter-parent

Como ya se ha comentado, este proyecto contiene las partes comunes, un fichero ```pom.xml``` con la configuración del proyecto y las dependencias compartidas:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.inmensia</groupId>
  <artifactId>cloud-greeter-parent</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>

    <spring-boot.version>1.5.3.RELEASE</spring-boot.version>
    <spring-cloud.version>Dalston.SR1</spring-cloud.version>
    <springfox.version>2.7.0</springfox.version>

    <maven-compiler-plugin.version>3.6.1</maven-compiler-plugin.version>
  </properties>

  <dependencyManagement>
    <dependencies>

    <!-- Spring Boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>${spring-boot.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>

    <!-- Spring Cloud -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>

    <!-- Springfox -->
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>${springfox.version}</version>
    </dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
      <version>${springfox.version}</version>
    </dependency>

    </dependencies>
  </dependencyManagement>
  
  <build>
    <pluginManagement>
      <plugins>

        <!-- Forces the JDK version -->
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>${maven-compiler-plugin.version}</version>
          <configuration>
            <source>${java.version}</source>
            <target>${java.version}</target>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

</project>
```

Las únicas dependencias declaradas son Spring Boot, Spring Cloud y Springfox. Spring Boot facilita la creación de microservicios, Spring Cloud la creación de servicios en la nube, y Springfox la documentación de los servicios REST.

Springfox quizás sea menos conocido, pero es simplemente una capa que se coloca por encima de Swagger y tiene el añadido de que es capaz de recuperar información de las anotaciones de librerías como Spring o Jackson, en vez de forzar a los desarrolladores a utilizar las anotaciones de Swagger, evitando así la dependencia y la duplicación de esfuerzos.
