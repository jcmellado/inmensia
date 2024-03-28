# 4. Spring Cloud Netflix (4)

_03-07-2018_ _Juan Mellado_

Continuando con el ejemplo práctico de uso de Spring Cloud Netflix, el siguiente paso es montar los proyectos que implementen los microservicios que contengan la lógica de negocio propia de la aplicación.

## 4.1. cloud-greeter-service

Este proyecto es un microservicio construido con Spring Boot que utiliza el cliente del servidor de configuración y el cliente de Eureka.

La estructura del proyecto es idéntica que la de los anteriores. En el fichero ```pom.xml``` únicamente cambian las dependencias:

```xml
<!-- Spring Boot -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- Spring Cloud -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

<!-- Springfox -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
</dependency>
```

Las dependencias específicas de Spring Boot configuran el proyecto como una aplicación web, con un servidor Undertow embebido, con las utilidades ofrecidas por defecto por el Actuator de Spring Boot.

Las dependencias específicas de Spring Cloud configuran el proyecto para hacer uso del cliente del servidor de configuración y del cliente de Eureka.

Las dependencias específicas de Springfox configuran el proyecto para utilizar Swagger, que es una herramienta utilizada para describir los servicios REST de una aplicación mediante anotaciones Java que se aplican sobre las propias clases que implementan dichos servicios. La dependencia ```springfox-swagger2``` ofrece el API con las anotaciones. La dependencia ```springfox-swagger-ui``` añade una interface gráfica a la aplicación en forma de página web que lista los servicios REST disponibles en la aplicación, y que además permite invocarlos directamente desde el navegador. Es una forma fácil de documentar los servicios REST y permite poder probarlos rápidamente.

El proyecto se compone de varias clases organizadas por paquetes. El punto de entrada a la aplicación es la clase ```Application``` anotada con ```@SpringBootApplication``` para configurarla como una aplicación de Spring Boot, con la anotación ```@EnableEurekaClient``` para hacer que utilice el cliente de Eureka, y con la clásica anotación ```@ComponentScan``` de Spring para indicar el paquete donde localizar las clases que debe gestionar el propio Spring para implementar patrones como la inyección de dependencias.

```java
@EnableEurekaClient
@SpringBootApplication
@ComponentScan(basePackages = { "com.inmensia.greeter" })
public class Application {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

Dentro del paquete ```config``` se encuentra el detalle de la configuración específica para Swagger. Dentro del paquete ```controller``` el servicio REST que expone la lógica de negocio. Y dentro del paquete ```model``` se encuentran las clases POJO de entrada y salida del servicio REST.

La clase más destacada es el controlador que implementa el servicio REST:

```java
@Api
@RestController
@RequestMapping("/api/v1")
public class GreetingController {

  private static final Logger LOGGER =
    LoggerFactory.getLogger(GreetingController.class);

  @RequestMapping(
    method = RequestMethod.POST,
    value = "/greeting")
  @ApiOperation(value = "greeting", 
    response = GreetingResponse.class)
  @ApiResponses(value = { @ApiResponse(
    code = 200,
    message = "Success", 
    response = GreetingResponse.class) })
  public GreetingResponse greeting(
    @ApiParam(value = "request", required = true)
    @RequestBody(required = true)
    final GreetingRequest request,
    final HttpServletRequest httpRequest) {

    LOGGER.debug("Received {}", request);

    return new GreetingResponse(
      httpRequest.getServerPort(), 
      "Hello " + request.getName() + "!");
  }
}
```

Sobre esta clase llama la atención el gran número de anotaciones utilizadas, la mitad de ellas de Spring, para definir el servicio, y la otra mitad de Swagger, para describir el servicio, y aunque este último cumple su función, en determinados casos puede resultar bastante intrusivo y repetitivo.

El servicio se limita a recoger un JSON con un campo de tipo texto y responder otro JSON con otro campo de tipo de texto. Recibe un nombre y retorna un texto en forma de saludo para el nombre dado. Adicionalmente en el JSON de salida retorna el número del puerto en el que está levantado el servicio, dato utilizado sólo a modo de ejemplo, para comprobar el balanceado de carga de una forma sencilla, en una aplicación real no sería necesario.

El fichero de configuración de la aplicación es ```cloud-greeter-service.yml``` y se encuentra en el repositorio del proyecto ```cloud-greeter-config```:

```yaml
port: ${CLOUD_GREETER_SERVICE_PORT}

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

En este fichero se define el servidor Eureka a utilizar. La definición a base de regiones, zonas y demás es herencia de la nomenclatura utilizada por Amazon, la plataforma utilizada por Netflix.

De igual forma que con los anteriores, el servicio se puede arrancar como cualquier otra aplicación de Spring Boot desde línea de comandos. En este caso especificando como parámetro de entrada el puerto en que debe levantarse el servidor de aplicaciones, ya que lo normal es que se quiera levantar más de una instancia de un mismo servicio para garantizar una alta disponibilidad y tolerancia a fallos.

```text
mvn spring-boot:run "-DCLOUD_GREETER_SERVICE_PORT=8200"
mvn spring-boot:run "-DCLOUD_GREETER_SERVICE_PORT=8201"
```

Un servidor arrancará en el puerto ```8200``` y otro en el puerto ```8201```, y estarán disponibles a través de la rutas ```http://localhost:8200``` y ```http://localhost:8201``` respectivamente.

Accediendo a las rutas ```http://localhost:8200/swagger-ui.html``` y ```http://localhost:8201/swagger-ui.html``` se mostrarán las interfaces gráficas de Swagger para cada uno de los servidores.

Cada servidor, haciendo uso del cliente Eureka embebido, se comunicará automáticamente con el servidor Eureka para registrarse como servicio.

La última clase de proyecto está dentro del paquete de pruebas y hace uso de las capacidades de Spring Boot para probar servicios REST.

## 4.2. cloud-greeter-api

Este proyecto es un microservicio construido con Spring Boot que invoca al microservicio del proyecto anterior. El propósito de este proyecto es ilustrar la creación de un gateway. Es decir, un servicio que se limita a invocar a otros. Útil para exponer servicios de forma pública, como un API REST por ejemplo, evitando exponer los servicios que acceden a los recursos privados, como por ejemplo bases de datos.

La estructura del proyecto es idéntica que la del anterior, solo que en el fichero ```pom.xml``` se añade una nueva dependencia con Hystrix, otra librería que forma parte de Netflix OSS. Hystrix ayuda a controlar las llamadas entre los distintos servicios de un sistema. Se sitúa como una capa intermedia entre los servicios, mide la latencia, controla los errores, y permite a los servicios definir la lógica que debe ejecutarse en caso de error.

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

El fichero de configuración es proporcionado por el servidor de configuración de igual forma que en los proyectos anteriores.

```yaml
server:
  port: ${CLOUD_GREETER_API_PORT}

eureka:
  client:
  serviceUrl:
  defaultZone: http://localhost:8101/eureka/

logging:
  file: logs/${spring.application.name}.log
  level:
    org.springframework.cloud: DEBUG
    com.inmensia: DEBUG

cloud:
  greeter:
    api:
    greeterServiceUrl: http://cloud-greeter-service/api/v1/greeting
```

La última entrada del fichero de configuración define la ruta donde se encuentra el servicio que invocará el gateway. Como se observa, la ruta es virtual, es el nombre del servicio con el que se encuentra registrado en Eureka. La cadena "cloud-greeter-service" se sustituye por la URL real cuando se invoca el servicio.

La clase principal del proyecto que define el punto de entrada a la aplicación configura una aplicación Spring Boot que hace uso del cliente Eureka igual que en el anterior proyecto. La única diferencia es la anotación ```@EnableCircuitBreaker```. El nombre de la anotación deriva del nombre del patrón Circuit Breaker que implementa Hystrix.

La clase del controlador que implementa el servicio REST público expuesto por el gateway tiene la misma interface que el microservicio privado del proyecto anterior. Los parámetros que recibe el público se le pasan tal cual al privado, y el resultado que se recibe del privado se publica tal cual por el público.

```java
@Api
@RestController
@RequestMapping("/api/v1")
public class ApiController {

  private static final Logger LOGGER =
    LoggerFactory.getLogger(ApiController.class);

  private final GreeterService greeterService;

  @Autowired
  public ApiController(final GreeterService greeterService) {
    this.greeterService = greeterService;
  }

  @ApiOperation(
    value = "greeting",
    response = ApiResponse.class)
  @ApiResponses(value = { @ApiResponse(
    code = 200,
    message = "Success", 
    response = GreetingResponse.class) })
  @RequestMapping(
    method = RequestMethod.GET, 
    value = "/greeting")
  public GreetingResponse greeting(
    @ApiParam(value = "name", required = true) 
    @RequestParam(name = "name") final String name) {

    LOGGER.debug("Name: {}", name);

    final GreetingResponse response = greeterService.greeting(name);

    LOGGER.debug("Port: {}", response.getPort());
    LOGGER.debug("Message: {}", response.getMessage());

    return response;
  }
}
```

La invocación del servicio privado se realiza mediante una clase de servicio que utiliza la conocida clase ```RestTemplate``` de Spring. Una alternativa sería utilizar Ribbon, otra librería de Netflix OSS, que funciona mediante anotaciones en interfaces, sin implementación, de forma similar a los repositorios de Spring Data.

```java
@ConfigurationProperties(prefix = "cloud.greeter.api")
@Service
public class GreeterServiceImpl implements GreeterService {

  private static final Logger LOGGER =
    LoggerFactory.getLogger(GreeterServiceImpl.class);

  private final RestTemplate restTemplate;

  private String greeterServiceUrl;

  @Autowired
  public GreeterServiceImpl(final RestTemplate restTemplate) {
    this.restTemplate = restTemplate;
  }

  public void setGreeterServiceUrl(final String greeterServiceUrl) {
    this.greeterServiceUrl = greeterServiceUrl;
  }

  @Override
  @HystrixCommand(fallbackMethod = "fallbackGreeting")
  public GreetingResponse greeting(final String name) {
    LOGGER.debug("Name: {}", name);

    if ("Hell".equals(name)) {
      throw new NullPointerException();
    }

    final GreetingRequest request = new GreetingRequest(name);

    final GreetingResponse response = restTemplate.postForObject(
      greeterServiceUrl,  request, GreetingResponse.class);

    LOGGER.debug("Port: {}", response.getPort());
    LOGGER.debug("Message: {}", response.getMessage());

    return response;
  }

  public GreetingResponse fallbackGreeting(final String name) {
    LOGGER.debug("Failed");

    return new GreetingResponse(null, "Failed!");
  }
}
```

Para demostrar el uso de Hystrix, el método que implementa el servicio está anotado con ```@HystrixCommand(fallbackMethod = "fallbackGreeting")```. Lo que instruye a Hystrix a invocar el método ```fallbackGreeting``` de la clase cuando se produzca un error. Esto da la oportunidad a los servicios de ejecutar un comportamiento alternativo, o un comando compensatorio si es necesario deshacer alguna operación.

Para comprobar el funcionamiento de Hystrix, el método que implementa el servicio eleva un ```NullPointerException``` si se le pasa como nombre el valor "```Hell```". En esos casos el servicio fallará y Hystrix llamará al método configurado en la anotación.

De la forma acostumbrada, uno o más servicios se pueden ejecutar desde línea de comandos indicando el puerto en que se quieren levantar:

```text
mvn spring-boot:run "-DCLOUD_GREETER_API_PORT=8300"
mvn spring-boot:run "-DCLOUD_GREETER_API_PORT=8301"
```

Los servicios levantados serán accesibles desde los puertos indicados, así como sus aplicaciones de Swagger correspondientes.

Para terminar, comentar que el cliente Eureka recupera la lista de servicios levantados y la guarda en local, de forma que no interroga al servidor Eureka cada vez que se invoca a un servicio, lo que mejora el rendimiento, y tiene el beneficio añadido de que, si se produce una caída del servidor Eureka, el cliente aún tendrá en local la ubicación de los servicios, lo que aumenta la disponibilidad y la tolerancia a fallos.
