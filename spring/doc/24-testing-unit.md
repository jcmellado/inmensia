# 24. Spring: Testing (1) - Tests Unitarios

_25-04-2012_ _Juan Mellado_

Spring proporciona cierta infraestructura que facilita la escritura y ejecución de pruebas, sobre todo para las de tipo unitario y de integración.

Este artículo no pretende ser una guía detallada de como escribir clases de prueba, sólo presentar algunas de las características que ofrece Spring a este respecto.

## 24.1. Dependencias

Las utilidades de soporte para la realización de pruebas se encuentran en una dependencia externa que es necesario añadir al fichero ```pom.xml``` de Maven:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
</properties>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>${spring.version}</version>
  <scope>test</scope>
<dependency>
```

La etiqueta ```scope``` con el valor ```test``` limita la dependencia al proceso de pruebas.

## 24.2. JUnit

Para realizar las pruebas de ejemplo se utilizará la librería JUnit, por lo que es necesario incluir la dependencia correspondiente en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.10</version>
  <scope>test</scope>
</dependency>
```

JUnit tiene una muy buena integración con Maven y Eclipse. Para ejecutar los tests basta con ejecutar la tarea ```test``` de Maven (_Run as / Maven test_). O utilizar las opciones de la ventana JUnit de Eclipse (_Windows / Show view / Other / Java / JUnit_).

## 24.3. Ejemplo

Sirva de ejemplo una sencilla clase POJO ubicada en ```src/main/Java```:

```java
package com.empresa;

public class Libro {

  private String titulo;

  public Libro(){
  }

  public String getTitulo() {
    return titulo;
  }

  public void setTitulo(String titulo) {
    this.titulo = titulo;
  }
}
```

Y su correspondiente test unitario ubicado en ```src/test/java```:

```java
package com.empresa;

import static org.junit.Assert.assertNull;
import static org.junit.Assert.assertEquals;

import org.junit.Test;

public class LibroTest{

  @Test
  public void testBasico(){
    Libro libro = new Libro();
    assertNull(libro.getTitulo());

    libro.setTitulo("Don Quijote");
    assertEquals(libro.getTitulo(), "Don Quijote");
  }
}
```

## 24.4. Utilidades

Spring, como complemento a las facilidades ofrecidas por librerías como JUnit, proporciona una clase de utilidades ```ReflectionTestUtils``` que sirve para acceder a propiedades o invocar métodos privados de una clase.

Por ejemplo, para los casos de clases con propiedades privadas que no exponen un _getter_:

```java
public class Documento{

  private String secreto;
...
```

Utilizando la clase de utilidades es posible acceder a la propiedad de forma externa desde dentro de un método de prueba:

```java
Documento documento = new Documento();

String secreto = ReflectionTestUtils.getField(documento, "secreto");
```

De igual forma, para las aplicaciones _web_, Spring ofrece la clase ```ModelAndViewAssert```, que sirve para probar objetos de tipo ```ModelAndView```, una pieza clave en el funcionamiento de este tipo de aplicaciones basadas en Spring MVC.

## 24.5. Mocks

Los objetos _Mocks_ simulan el comportamiento de otros, como los muñecos que se utilizan para simular accidentes de coche.

Spring proporciona tres paquetes que permiten ejecutar el código de producción y el de pruebas utilizando una misma configuración:

- ```org.springframework.mock.jndi```
- ```org.springframework.mock.web```
- ```org.springframework.mock.web.portlet```

El uso de estos objetos normalmente es mejor realizarlo en conjunción con alguna librería especializada, como Mockito, o EasyMock, por ejemplo.
