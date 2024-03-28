# 3. Spring Boot 2 (3)

_19-07-2018_ _Juan Mellado_

Continuando con el ejemplo de construcción de aplicaciones básicas con Spring Boot 2, lo siguiente es añadir un servicio REST a la aplicación, ya que hoy en día es la tecnología más utilizada para construir microservicios. Tarea que puede acometerse utilizando Spring MVC, Spring WebFlux, o una implementación de JAX-RS, como Jersey o Apache CXF.

## 3.1. Spring MVC

Para crear un servicio REST con Spring MVC hay que añadir el _starter web_ de Spring Boot en el fichero ```pom.xml```:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

Esta dependencia incluye las librerías de Spring MVC en el proyecto y permite construir aplicaciones _web_ basadas en _servlets_.

Como ejemplo de servicio REST vamos a escribir un controlador que reciba un parámetro de tipo texto y retorne un JSON con un campo que contenga dicho parámetro. Es decir, el típico "_saludador_" (_greeter_), que recibe un nombre y retorna ```"Hello {{nombre}}!"```.

Lo primero es crear en ```/src/main/java/{paquete}``` la clase que se quiere retornar como resultado de la ejecución del servicio. Es decir, el típico DTO:

```java
public class Hello {
  private final String message;

  public Hello(String message) {
    this.message = message;
  }

  public String getMessage() {
    return this.message;
  }
}
```

Lo siguiente es crear en ```src/main/java/{paquete}``` la clase con el controlador, utilizando las anotaciones de Spring MVC para indicar el método, ruta y nombre del parámetro HTTP a través de los que se atenderán las peticiones de servicio:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

  @GetMapping("/hello")
  public Hello hello(@RequestParam String name) {
    return new Hello("Hello " + name + "!");
  }
}
```

La anotación ```@RestController``` hace que la respuesta del método del controlador se convierta automáticamente en un objeto de tipo JSON. La anotación ```@GetMapping``` hace que el método atienda peticiones HTTP de tipo GET. Y la anotación ```@RequestParam``` hace que el valor del parámetro HTTP con nombre ```name``` se asigne automáticamente al parámetro de entrada del método.

El último paso es compilar y ejecutar la aplicación utilizando el propio _plugin_ de Maven de Spring Boot:

```text
mvn spring-boot:run
```

Si todo funciona correctamente arrancará la aplicación mostrando por consola el _log_ de ejecución, en el que debe verse la ruta mapeada por el servicio y el servidor _web_ embebido utilizado:

```text
Mapped "{[/hello],methods=[GET]}" onto public <paquete>.Hello <paquete>.HelloController.hello(java.lang.String)
...
Tomcat started on port(s): 9999 (http) with context path ''
```

El servicio arrancado de forma local se puede invocar desde un navegador a través del puerto en el que se encuentre levantado el servidor _web_ embebido:

```text
http://localhost:9999/hello?name=World
```

La respuesta será un JSON con un mensaje que contendrá el nombre pasado como parámetro:

```json
{"message":"Hello World!"}
```

## 3.2. Spring WebFlux

Para crear un servicio REST con Spring WebFlux hay que añadir el _starter_ de WebFlux de Spring Boot en el fichero ```pom.xml```:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
  </dependency>
</dependencies>
```

Esta dependencia incluye las librerías de Spring WebFlux en el proyecto y permite construir aplicaciones asíncronas no bloqueantes basadas en el paradigma reactivo.

Para convertir el ejemplo del apartado anterior en una aplicación de WebFlux sólo hay que modificar la clase del controlador, encapsulando con ```Mono<T>``` el tipo retornado. Esta clase es proporcionada por la librería Reactor y actúa como un _publisher_ de un _reactive stream_ que se resuelve con un único elemento o un error.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Mono;

@RestController
public class HelloController {

  @GetMapping("/hello")
  public Mono<Hello> hello(@RequestParam String name) {
    var hello = new Hello("Hello " + name + "!");

    return Mono.just(hello);
  }
}
```

La nueva aplicación se puede compilar y ejecutar utilizando el _plugin_ de Maven de Spring Boot de igual forma que en el apartado anterior:

```text
mvn spring-boot:run
```

Si todo funciona correctamente arrancará la aplicación mostrando por consola el _log_ de ejecución, en el que debe verse la nueva ruta mapeada y el servidor embebido, que por defecto es Netty:

```text
Mapped "{[/hello],methods=[GET]}" onto public reactor.core.publisher.Mono<<paquete>.Hello> <paquete>.HelloController.hello(java.lang.String)

Netty started on port(s): 9999
```

Y de igual forma que en el apartado anterior, el servicio arrancado de forma local se puede invocar desde un navegador a través del puerto en el que se encuentre levantado el servidor _web_ embebido:

```text
http://localhost:9999/hello?name=World
```

La respuesta será un JSON con un mensaje que contendrá el nombre pasado como parámetro:

```json
{"message":"Hello World!"}
```

En el caso de que el servicio REST tuviera que retornar una lista de objetos el resultado habría que encapsularlo con ```Flux<T>```, otra clase proporcionada por la librería Reactor y que representa un publicador de _reactive streams_ que se resuelve con 0, n elementos o error. Como ejemplo vamos a construir un nuevo controlador que admite un número como parámetro y retorna una lista de números desde 1 al número dado.

El primer paso es crear en ```/src/main/java/{paquete}``` una nueva clase para el DTO:

```java
public class Counter {

  private Integer count;

  public Counter(Integer count) {
    this.count = count;
  }

  public Integer getCount() {
    return this.count;
  }
}
```

El segundo paso es crear en ```/src/main/java/{paquete}``` una nueva clase para el controlador:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;

@RestController
public class CounterController {

  @GetMapping(“/counter”)
  public Flux<Counter> count(@RequestParam Integer count) {
    return Flux.range(1, count).map(Counter::new);
  }
}
```

Y el último paso es el mismo de siempre: compilar, ejecutar con el _plugin_ e invocar al servicio desde el navegador:

```text
http://localhost:9999/counter?count=3
```

La respuesta será un array de objetos:

```json
[{"count":1},{"count":2},{"count":3}]
```

WebFlux permite además definir las rutas (_endpoints_) sin utilizar anotaciones, de una forma más funcional, construyendo las rutas que se quiere que atiendan los servicios. Por ejemplo, el servicio contador anterior se puede escribir de forma equivalente creando _beans_ de Spring de tipo ```RouterFunction```. _Beans_ que tienen que crearse al arrancar la aplicación, como parte del proceso de configuración, por lo que tienen que definirse en una clase anotada con ```@Configuration``` como la siguiente:

```java
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

import java.util.function.Function;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.HandlerFunction;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;

import reactor.core.publisher.Flux;

@Configuration
public class CounterRouter {

  @Bean
  public RouterFunction<ServerResponse> countRouter() {
    Function<ServerRequest, Integer> count = 
      request -> Integer.valueOf(request.queryParam("count").get());

    Function<ServerRequest, Flux<Counter>> flux = 
      request -> Flux.range(1, count.apply(request)).map(Counter::new);

    HandlerFunction<ServerResponse> handler = 
      request -> ok().body(flux.apply(request), Counter.class);

    return route(GET("/counter"), handler);
 }
}
```

Como se observa, con este modelo es responsabilidad de la aplicación extraer los parámetros de la petición, construir el resultado y cuerpo de la respuesta, y mapear la ruta junto con el método. Es un enfoque más funcional evitando el uso de anotaciones y orientado a definir manejadores (_handlers_). En el ejemplo no se aprecia, ya que se hace todo dentro de un mismo método. En la práctica la responsabilidad estará repartida entre distintos componentes. El _stream_ de datos normalmente procederá de algún repositorio, el manejador estará en un servicio, y el método que define la ruta sólo contendrá la parte técnica relacionada con HTTP. Notar que en este ejemplo además resulta un poco engorroso la cantidad de clases estáticas que se requiere importar.
