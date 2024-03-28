# 14. JSTL (1) - Core

_17-05-2012_ _Juan Mellado_

JSTL (JavaServer Pages Standard Tag Library) es una librería estándar de acciones JSP de propósito general, y aunque se llame a sí misma "librería" (singular), en realidad está compuesta por cinco librerías de acciones:

Librería|Namespace|Prefijo
--------|---------|-------
core | <http://java.sun.com/jsp/jstl/core> | c
functions | <http://java.sun.com/jsp/jstl/functions> | f
i18n | <http://java.sun.com/jsp/jstl/fmt> | fmt
SQL | <http://java.sun.com/jsp/jstl/sql> | sql
XML | <http://java.sun.com/jsp/jstl/xml> | x

JSTL ofrece facilidades para implementar iteraciones, evaluar condiciones, capturar excepciones, formatear variables, e incluso acceder a base de datos y manipular XML, aunque el uso de estas dos últimas no están recomendadas en la práctica.

Es decir, JSTL proporciona las herramientas necesarias para realizar dentro de una página JSP las tareas que normalmente se hacen mediante código Java dentro de un _servlet_ tradicional.

## 14.1. Dependencias

Para utilizar JSTL dentro de un proyecto es necesario añadir la correspondiente dependencia en el fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>jstl</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

A pesar de ser estándar, JSTL tiene la consideración de librería externa no imprescindible. Determinados servidores, como el popular Tomcat por ejemplo, no la proporcionan, por lo que es necesario incluirla dentro del _war_ de la aplicación.

## 14.2. core

La librería ```core``` permite crear variables, realizar iteraciones, evaluar condiciones, y ofrece una serie de acciones de propósito general para facilitar la creación de URLs y acceder a recursos.

- ```c:out```: Evalúa una expresión y añade el resultado a la respuesta. Es equivalente a añadir una cadena de texto al ```Writer``` de respuesta en un _servlet_ tradicional. Permite indicar un valor por defecto, y si los caracteres especiales se tienen que codificar como XML.

    ```xml
    <c:out value="${header['accept']}" default="Nulo"/>
    ```

- ```c:set```: Establece, o crea si no existe, una variable en el ámbito dado. Si no se especifica ámbito por defecto, la visibilidad de la variable se restringe a la página.

    ```xml
    <c:set var="saludo" value="Hello!" scope="application"/>
    ```

    Admite una sintaxis alternativa que permite establecer el valor de las propiedades de un objeto:

    ```xml
    <c:set target="libro" property="titulo" value="Don Quijote"/>
    ```

- ```c:remove```: Elimina una variable del contexto.

    ```xml
    <c:remove var="temporal"/>
    ```

- ```c:if```: Evalúa una condición dada, y si el resultado es ```true``` procesa el bloque que contiene. El resultado de la evaluación se puede recoger en una variable si es necesario. Es evidente que esto permite realizar un control de flujo condicional básico, como el que se haría por código en un _servlet_ tradicional, y generar contenido de forma dinámica.

    ```xml
    <c:if test="${valor != null}">
      ¡Tiene valor!
    </c:if>
    ```

- ```c:choose```: Evalúa condiciones, y procesa el bloque de la condición que se cumple, o el bloque por defecto si existe y no se cumple ninguna condición.

    ```xml
    <c:choose>
    <c:when test="${valor == 1}">
      Uno
    </c:when>
    <c:when test="${valor == 2}">
      Dos
    </c:when>
    <c:otherwise>
      Otro
    </c:otherwise>
    </c:choose>
    ```

- ```c:forEach```: Itera variables de tipo colección. Permite especificar los índices de inicio y final, el tamaño de paso, y una variable adicional que almacena todos estos parámetros y proporciona información sobre la iteración en curso dentro del bloque.

    ```xml
    <table>
    <c:forEach items="${header}" var="variable" varStatus="status">
      <tr>
        <td>${status.count}</td>
        <td>${variable.key}</td>
        <td>${variable.value}</td>
      </tr>
    </c:forEach>
    </table>
    ```

- ```c:forTokens```: Itera variables de texto que contiene _tokens_ separados por un determinado delimitador dado.

    ```xml
    <c:forTokens items="uno,dos,tres" delims="," var="variable">
      ${variable}
    </c:forTokens>
    ```

- ```c:catch```: Permite capturar excepciones que se produzcan dentro de un bloque de un JSP, y guardar la excepción en una variable de la página, para que su valor pueda ser evaluado posteriormente y comprobar si se produjo una excepción.

    ```xml
    <c:catch var="excepcion">
    ...
    </c:catch>
    ...
    <c:if test="${excepcion != null}">
    ...
    <c:/if>
    ```

- ```c:redirect```: Aborta el proceso del JSP en curso y envía una URL al cliente para que se redirija a ella. En un _servlet_ tradicional es equivalente al llamar al método ```sendRedirect``` del objeto respuesta:

    ```xml
    <c:redirect url="otro.jsp"/>
    ```

- ```c:url```: Facilita la construcción de URLs evitando tener que realizar concatenaciones de cadenas de texto por código.

    ```xml
    <c:url value="https://www.company.com/biblioteca" var="url">
      <c:param name="autor">Cervantes</c:param>
      <c:param name="titulo">Don Quijote</c:param>
    </c:url>
    <a href="${url}">
    ```

- ```c:import```: Importa un recurso dada su URL y lo añade a la respuesta. Adicionalmente permite especificar una variable donde se almacena una referencia al recurso para que pueda ser accedido posteriormente.

    ```xml
    <c:import url="pie-de-pagina"/>
    ```
