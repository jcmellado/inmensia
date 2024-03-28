# 12. JSP (5) - Acciones

_16-05-2012_ _Juan Mellado_

Las acciones permiten realizar las operaciones que normalmente se realizan dentro de los _scriptlets_, pero sin tener que embeber código Java dentro del JSP. Las acciones permiten declarar y modificar objetos, así como cambiar el flujo de procesamiento de un JSP.

Las acciones se agrupan dentro de librerías llamadas _tag libraries_. JSP proporciona una librería de acciones básicas, una librería aparte de propósito general llamada JSTL, y permite además que las aplicaciones definan sus propias librerías.

## 12.1. Acciones Estándar

Las páginas JSP no necesitan incluir una directiva ```taglib``` para poder utilizar las acciones estándar, mientras que los documentos JSP en cambio tienen que incluir el siguiente namespace:

```xml
... xmlns:jsp="http://java.sun.com/JSP/Page"
```

Los servidores _web_ que soporten JSP deben proporcionar obligatoriamente las siguientes acciones estándar:

- ```jsp:root```: Todo documento XML para ser válido necesita tener una única raíz. Esta acción está pensada para ser utilizada como raíz, sobre todo en documentos JSP que son incluidos por otros.

  ```xml
  <jsp:root xmlns:jsp=”http://java.sun.com/JSP/Page” version=”2.0">
    <div>uno</div>
    <div>dos</div>
  </jsp:root>
  ```

- ```jsp:output```: Admite una serie de atributos encaminados a especificar como debe ser generado el contenido, por ejemplo si se ha de generar una cabecera XML, y el ```DOCTYPE``` a utilizar.

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <html xmlns:jsp="http://java.sun.com/JSP/Page">
  <jsp:directive.page contentType="text/html"/>
  <jsp:output doctype-root-element="html"
              doctype-public="-//W3C//DTD XHTML Basic 1.0//EN"
              doctype-system="http://www.w3.org/TR/xhtml-basic/xhtml-basic10.dtd"/>
  <body>
  Hello JSPX!
  </body>
  </html>
  ```

- ```jsp:text```: Permite embeber expresiones EL, o cualquier otra información, de forma que el resultado siga siendo un documento XML válido.

  ```xml
  <jsp:text>
    ${"evaluado"}
  </jsp:text>
  ```

- ```jsp:element```: Genera un elemento de forma dinámica dentro de un JSP y lo añade al resultado. Es equivalente a la típica cadena de texto que se crea en el código de un _servlet_ tradicional para representar un objeto mediante algún tipo de elemento HTML.

  ```xml
  <jsp:element name="td">
    <jsp:attribute name="class">celda</jsp:attribute>
    <jsp:body>dato</jsp:body>
  </jsp:element>
  ```

  El ejemplo anterior crearía el siguiente código HTML:

  ```xml
  <td class="celda">dato</td>
  ```

- ```jsp:attribute```: Se utiliza conjuntamente con ```jsp:element``` para añadir un atributo a un elemento generado de forma dinámica.

- ```jsp:body```: Se utiliza conjuntamente con ```jsp:element``` para especificar el cuerpo de un elemento generado de forma dinámica.

- ```jsp:plugin```: Añade el código HTML necesario en función del navegador del cliente para que se pueda ejecutar un _applet_.

  ```xml
  <jsp:plugin type="applet" code="Plugin.class" codebase=”/html” >
    <jsp:params>
      <jsp:param name="atributo" value="valor"/>
    </jsp:params>
    <jsp:fallback>
      <p>No se pudo ejecutar el plugin</p>
    </jsp:fallback>
  </jsp:plugin>
  ```

- ```jsp:params```: Se utiliza conjuntamente con ```jsp:plugin``` para especificar los parámetros de un _applet_ creado de forma dinámica.

- ```jsp:fallback```: Se utiliza conjuntamente con ```jsp:plugin``` para especificar el mensaje de error que debe mostrarse en caso de no poder ejecutar un _applet_ creado de forma dinámica.

- ```jsp:useBean```: Crea un objeto de un tipo dado y lo identifica con un nombre. El tipo tiene que que ser conforme a la especificación _JavaBean_, es decir, tener un constructor sin parámetros y _getters/setters_ para sus atributos. Si ya existe un objeto con el identificador dado no crea uno nuevo, retorna el existente. Si se utiliza dentro de una página JSP se crea una variable en el _servlet_, si se utiliza dentro de un documento JSP se crea una variable EL. La sintaxis que acepta es bastante flexible, por lo que es recomendable leer la documentación de referencia.

  ```xml
  <jsp:useBean id="libro" class="com.company.bean.Libro"/>
  ```

- ```jsp:setProperty```: Establece una propiedad de un _bean_.

  ```xml
  <jsp:setProperty name="libro" property="titulo" value="Don Quijote"/>
  ```

- ```jsp:getProperty```: Obtiene una propiedad de un _bean_ y añade el resultado a la respuesta, o lo asigna a una variable si se indica.

  ```xml
  <jsp:getProperty name="libro" property="titulo"/>
  ```

- ```jsp:include```: Incluye el recurso indicado dentro del proceso del JSP.

  ```xml
  <jsp:include page="copyright.html"/>
  ```

- ```jsp:forward```: Detiene el proceso del JSP y lo dirige al recurso indicado.

  ```xml
  <jsp:forward page="otro.jsp"/>
  ```

- ```jsp:param```: Se utiliza conjuntamente con ```jsp:include``` y ```jsp:forward``` para especificar parámetros.

- ```jsp:declaration```: Permite introducir una declaración escrita en Java dentro de un documento JSP de forma similar a como se hace en las páginas JSP. Su uso no está recomendado.

  ```xml
  <jsp:declaration>
    int i = 10;
  </jsp:declaration>
  ```

- ```jsp:expression```: Permite introducir una expresión escrita en Java dentro de un documento JSP de forma similar a como se hace en las páginas JSP. La expresión es evaluada y su resultado añadido a la respuesta. Su uso no está recomendado.

  ```xml
  <jsp:expression>
    i + 1
  </jsp:expression>
  ```

- ```jsp:scriptlet```: Permite introducir un bloque de código Java dentro de un documento JSP de forma similar a como se hace en las páginas JSP. Su uso no está recomendado.

  ```xml
  <jsp:scriptlet>
    if (i == 10){
  </jsp:scriptlet>
      Diez
  <jsp:scriptlet>
    }
  </jsp:scriptlet>
  ```

- ```jsp:invoke```: Esta acción está pensada para ser usada dentro de una librería de acciones, es un error utilizarla dentro de un JSP. Ejecuta un fragmento JSP y añade el resultado a la respuesta, o lo almacena en una variable si así se indica.

- ```jsp:doBody```: Esta acción está pensada para ser usada dentro de una librería de acciones, es un error utilizarla dentro de un JSP. Ejecuta el fragmento JSP contenido dentro del cuerpo de la acción y añade el resultado a la respuesta, o lo almacena en una variable si así se indica.
