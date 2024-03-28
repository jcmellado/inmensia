# 5. Spring Cloud Netflix (y 5)

09-07-2018 Juan Mellado

Continuando con el ejemplo práctico de uso de Spring Cloud Netflix, el último paso es montar los proyectos que arranquen los servicios de enrutamiento y monitorización.

## 5.1. cloud-greeter-zuul

Este proyecto es un servidor de enrutamiento basado en Zuul de Netflix OSS. Es la puerta de entrada a los servicios desde el exterior.

La estructura del proyecto es igual que en los anteriores. En el fichero ```pom.xml``` únicamente cambia la dependencia principal:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

La única clase del proyecto es el punto de entrada a la aplicación, sobre la que se añade la anotación ```@EnableZuulProxy``` para configurar el servidor Zuul:

```java
@EnableZuulProxy
@SpringBootApplication
public class Application {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

El fichero de configuración del servidor es proporcionado por el servidor de configuración. El fichero es similar a los anteriores. Configura el puerto en que se levanta el servidor, las rutas a las que atiende, la configuración para localizar el servidor Eureka y la ubicación de los ficheros de log:

```yaml
server:
  port: 8103

zuul:
  routes:
    greeting:
      path: /greeter/**
      serviceId: cloud-greeter-api

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8101/eureka/

logging:
  file: logs/${spring.application.name}.log
  level:
    org.springframework.cloud: DEBUG
    com.inmensia: DEBUG
```

La entrada ```zuul.routes``` del fichero define las rutas que atiende Zuul. En el ejemplo, una petición HTTP a la ruta ```/greeter``` es redirigida al servicio de nombre ```cloud-greeter-api```. La resolución del nombre es realizada por Eureka.

El servidor se levanta desde línea de comandos de la forma acostumbrada:

```text
mvn spring-boot:run
```

El servidor estará disponible a través de ```http://localhost:8103```. Y para comprobar que funciona correctamente el enrutamiento se puede invocar el servicio utilizando la ruta virtual y verificar que invoca realmente al servicio solicitado:

```text
http://localhost:8103/greeter/api/v1/greeting?name=World
```

El camino que sigue la petición consiste de varios pasos. El primero lo realiza Zuul, resolviendo la petición a través de Eureka, localizando el servicio solicitado e invocándolo. Lo que en nuestro ejemplo supone la invocación del servicio externo que actúa como gateway. El siguiente paso es la invocación del servicio interno por parte del gateway, resuelto también a través de Eureka. Y el último paso es la ejecución efectiva del servicio interno, retornándose el resultado a través de toda la cadena de servicios por la que pasa la petición.

En este punto es interesante notar que los componentes de Netflix OSS ofrecen muchas más características que las básicas contempladas en este proyecto de ejemplo. Por ejemplo, Zuul permite definir filtros, a la manera de los filtros habituales de los servidores web, permitiendo programar toda una serie de procesos totalmente personalizables sobre las peticiones, como balanceos de carga o estadísticas a medida.

## 5.2. cloud-greeter-turbine

Este proyecto es un servidor de monitorización basado en Turbine de Netflix OSS. Técnicamente es un agregador de streams de Server-Sent Events (SSE) en formato JSON. Lo que quiere decir que recoge eventos generados por los servidores y los agrega dejándolos disponibles para su uso, principalmente por cuadros de mando. El caso de uso principal es su uso con Hystrix, que genera eventos tales como el número de éxitos, errores y timeouts.

La estructura del proyecto es igual que en los anteriores. En el fichero ```pom.xml``` únicamente cambian las dependencias principales:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-turbine</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

La única clase del proyecto es el punto de entrada a la aplicación, sobre la que se añade la anotación ```@EnableTurbine``` para configurar el servidor Turbine y ```@EnableHystrixDashboard``` para agregar eventos de Hystrix y añadir la aplicación web con el cuadro de mando:

```java
@EnableTurbine
@EnableHystrixDashboard
@SpringBootApplication
public class Application {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

El fichero de configuración del servidor es proporcionado por el servidor de configuración. El fichero es similar a los anteriores. Configura el puerto en que se levanta el servidor, la información a agregar, la configuración para localizar el servidor Eureka y la ubicación de los ficheros de log:

```yaml
server:
  port: 8102

turbine:
  aggregator:
    clusterConfig: CLOUD-GREETER-API
    appConfig: cloud-greeter-api

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8101/eureka/

logging:
  file: logs/${spring.application.name}.log
  level:
    org.springframework.cloud: DEBUG
    com.inmensia: DEBUG
```

La entrada ```turbine.appConfig``` del fichero define el nombre de la aplicación de la que se quiere agregar eventos. La resolución del nombre es realizada por Eureka.

El servidor se levanta desde línea de comandos de la forma acostumbrada:

```text
mvn spring-boot:run
```

El servidor estará disponible a través de ```http://localhost:8102```. Y para comprobar que funciona correctamente el agregador se pueden invocar la aplicación web con el cuadro de mando de Hystrix:

```text
http://localhost:8102/hystrix/monitor?stream=http%3A%2F%2Flocalhost%3A8300%2Fhystrix.stream
http://localhost:8102/hystrix/monitor?stream=http%3A%2F%2Flocalhost%3A8102%2Fturbine.stream%3Fcluster%3DCLOUD-GREETER-API
```

Llegado este punto se tiene un entorno configurado completo y en ejecución con Spring Cloud Netflix.

¡Feliz computación en la nube!
