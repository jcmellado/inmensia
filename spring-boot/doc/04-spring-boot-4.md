# 4. Spring Boot 2 (4)

_22-07-2018_ _Juan Mellado_

Una parte importante de todo desarrollo moderno es la elaboración y ejecución automatizada de pruebas. Y a este respecto Spring Boot hereda todas las capacidades que ofrece el _framework_ de Spring. Tanto para pruebas de integración, es decir, aquellas que requieren el auxilio de recursos externos a la propia aplicación, como pruebas unitarias, que pueden realizarse de forma independiente.

Para nuestro caso de uso, la construcción de servicios REST con Spring Boot 2, tiene sentido hacer pruebas para comprobar que el servidor _web_ se levanta, mapea las rutas, y retorna un JSON que cumple con el formato esperado.

## 4.1. Caso de Uso

Como ejemplo utilizaremos una versión modificada de uno de los servicios REST construidos con Spring MVC en artículos anteriores, el que retorna una lista de números desde el 1 a un número dado. Teniendo en cuenta siempre que lo que interesa es probar que se cumple el contrato del servicio, no la implementación del mismo.

El primer paso es incluir las dependencias de Spring MVC en el fichero ```pom.xml``` mediante el _starter_ de Spring Boot:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

El segundo paso es crear en ```/src/main/java/{package}``` la clase DTO que retornará el servicio:

```java
import java.util.List;

public class Count {

  private List<Integer> count;

  public Count() {
  }

  public Count(List<Integer> count) {
    this.count = count;
  }

  public List<Integer> getCount() {
    return this.count;
  }
}
```

El tercer paso es crear en ```/src/main/java/{package}``` la clase que implementa el controlador del servicio:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CounterController {

  @GetMapping("/counter")
  public Count count(@RequestParam(name="count", defaultValue="0") Integer end) {
    var count = IntStream.range(1, end + 1)
      .boxed()
      .collect(Collectors.toUnmodifiableList());

    return new Count(count);
  }
}
```

Y el último paso es compilar y ejecutar la aplicación con Maven:

```text
mvn spring-boot:run
```

Invocando a ```http://localhost:9999/counter?count=3``` desde un navegador debería retornarse un JSON con un campo que contenga el _array_ con la secuencia de números:

```json
{"count":[1,2,3]}
```

Notar que el ejemplo está cambiado con respecto a artículos anteriores con objeto de que no falle si no se proporciona el parámetro de entrada, y para que retorne un JSON en vez de un _array_. Aún así, si se le pasa un valor que no represente un número entero, el servicio fallará con una excepción de tipo ```NumberFormatException```, mostrándose la página de error por defecto de Spring Boot. En un sistema real explotado en producción, y expuesto de forma pública en Internet, los parámetros deben validarse siempre y este tipo de información no debería exponerse nunca por motivos de seguridad.

## 4.2. SpringBootTest

El primer paso para probar el servicio de ejemplo es añadir al fichero ```pom.xml``` las dependencias de Spring Test a través del _starter_ de Spring Boot:

```xml
<dependencies>
...
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

Esta dependencia añade alguna de las librerías más populares utilizadas para la elaboración de pruebas. En particular JUnit, Hamcrest, Mockito, AssertJ y JSONassert, entre otras. Esto permite implementar las pruebas directamente con JUnit. Lo que es una ventaja en la medida que la mayoría de IDEs se integran con este _framework_ y permiten lanzar las pruebas sin necesidad de pasar por Maven.

El segundo paso es crear en ```/src/test/java/{package}``` una clase con un caso de prueba:

```java
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class CounterControllerTest {

  @Autowired
  private TestRestTemplate restTemplate;

  @Test
  public void testEmptyCount() {
    var body = restTemplate.getForObject("/counter", String.class);

    assertThat(body).isEqualTo("{\"count\":[]}");
  }
}
```

La anotación ```@RunWith``` sirve para indicar el runner que se quiere utilizar con JUnit. La anotación ```@SpringBootTest``` sirve para indicar que la clase ejecuta casos de prueba de una aplicación de Spring Boot. Y el parámetro ```WebEnvironment```.```RANDOM_PORT``` indica que se quiere arrancar el servidor en un puerto aleatorio, para evitar conflictos en caso de lanzar varias pruebas en paralelo.

La instancia inyectada de la clase ```TestRestTemplate``` es registrada automáticamente como resultado de aplicar la anotación ```@SpringBootTest``` sobre la clase, y tiene la ventaja de que todas las llamadas HTTP que realice las hará contra el servidor _web_ embebido arrancado para la prueba. Con la versión 5 de Spring se prefiere el uso de la clase ```WebTestClient``` en vez de ```TestRestTemplate```, ya que su API sigue un estilo _fluent_ para las aserciones, pero en la _release_ actual sólo se registra automáticamente cuando se utiliza WebFlux y no Spring MVC.

Cuando se ejecute la prueba se arrancará el servidor _web_ embebido, se invocará al servicio a través de ```restTemplate```, y se comprobará que por defecto devuelve una cadena de texto que representa un JSON con un array vacío.

En un IDE moderno, como Eclipse Photon, las pruebas con JUnit se pueden ejecutar de varias formas de una manera extremadamente sencilla. Por ejemplo, pulsando el botón derecho sobre el método de la prueba y ejecutándolo a través del menú contextual.

## 4.3. WebMvcTest

En algunos casos tener que arrancar un servidor _web_ puede ser un inconveniente, sobre todo cuando lo que se quiere probar es sólo un controlador de la forma más rápida posible. Para evitar esto es posible reescribir la prueba anterior utilizando las facilidades de prueba de Spring MVC, en vez de las de Spring Boot. Lo que quiere decir que se puede invocar al controlador, a la manera de Spring MCV, sin tener que levantar el servidor, que es la manera de Spring Boot.

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(CounterController.class)
public class CounterControllerTest {

  @Autowired
  private MockMvc mvc;

  @Test
  public void testEmptyCount() throws Exception {
    mvc.perform(get("/counter"))
      .andExpect(status().isOk())
      .andExpect(content().string("{\"count\":[]}"));
  }
}
```

La anotación ```@WebMvcTest``` permite especificar el controlador que se quiere probar y tiene el efecto añadido que registra algunos beans de Spring, en particular una instancia de la clase ```MockMvc```, que se puede utilizar para invocar al controlador simulando la llamada HTTP sin tener que arrancar realmente ningún servidor _web_.

Si se ejecuta la prueba se puede comprobar en el _log_ que el servidor embebido no arranca, pero que el servicio es invocado y retorna el objeto JSON con un array vacío de igual forma que con el método del apartado anterior.

## 4.4. JSON

El inconveniente de los métodos de prueba de los apartados anteriores es que comprueban el resultado como una cadena de texto, sin ninguna estructura. Un espacio en blanco, o un orden distinto en los campos retornados, provocaría que la prueba fallase.

Para evitar estos problemas es mejor tratar el resultado como JSON y reescribir el método para que no opere con cadenas de texto.

Una forma sencilla de hacerlo es cambiar el método ```string``` por el método ```json```:

```java
.andExpect(content().json("{ \"count\" : [ ] }"));
```

La diferencia entre ambos métodos es que el primero es estricto en la comprobación que realiza entre las dos cadenas de texto, mientras que el segundo comprueba la similitud entre las dos cadenas tratándolas como si fueran representaciones en formato texto de objetos JSON. Por eso la prueba sigue ejecutándose con éxito, a pesar de los espacios añadidos a la cadena de texto utilizada para la comprobación.

Evidentemente sigue sin ser una solución correcta, ya que implica seguir escribiendo cadenas de texto, e impide utilizar las facilidades de comprobación de nombres y tipos del IDE. Una mejor solución es trabajar directamente con clases Java mapeadas como objetos JSON.

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import java.util.Collections;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.json.AutoConfigureJsonTesters;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.json.JacksonTester;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(CounterController.class)
@AutoConfigureJsonTesters
public class CounterControllerTest {

@Autowired
private MockMvc mvc;

@Autowired
private JacksonTester<Count> json;

  @Test
  public void testEmptyCount() throws Exception {
    var response = mvc.perform(get("/counter"))
      .andExpect(status().isOk())
      .andReturn().getResponse();

    var expected = json.write(new Count(Collections.emptyList())).getJson();

    assertThat(response.getContentAsString()).isEqualTo(expected);
  }
}
```

La anotación ```@AutoConfigureJsonTesters``` registra automáticamente una serie de _beans_ de Spring para trabajar con librerías como Jackson, Gson y Jsonb, existiendo además anotaciones específicas para cada una de estas librerías. La instancia inyectada de la clase ```JacksonTester``` trabaja con Jackson y se utiliza para la prueba porque es la librería que se utiliza por defecto en las aplicaciones de Spring Boot.

De esta forma se evita utilizar cadenas de texto y se pueden escribir pruebas más elaboradas.

```java
@Test
public void testCountThree() throws Exception {
  var response = mvc.perform(get("/counter?count=3"))
    .andExpect(status().isOk())
    .andReturn().getResponse();

  var expected = json.write(new Count(List.of(1, 2, 3))).getJson();

  assertThat(response.getContentAsString()).isEqualTo(expected);
}
```

Otra opción disponible es comparar el JSON devuelto por el servicio con un JSON almacenado en un fichero.

Resumiendo, Spring ofrece toda una variedad de configuraciones para la realización de pruebas, no sólo las básicas vistas en este artículo, sino otras más específicas para probar servicios que utilizan JPA, LDAP, Redis, MongoDB, junto con algunas otras que merece la pena explorar en la documentación oficial de referencia.
