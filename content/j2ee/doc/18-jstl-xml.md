# 18. JSTL (5) - XML

_17-05-2012_ _Juan Mellado_

La librería ```xml``` permite parsear documentos XML, iterar por los elementos que los componen, y aplicar transformaciones XSLT (eXtensible Stylesheet Language for Transformations).

## 18.1. XPath

Para referenciar a partes concretas dentro de un documento XML se utiliza el estándar de rutas XPath (XML Path Language).

Para poder hacer referencia a las variables de los distintos ámbitos que ofrece una aplicación _web_ se utiliza ```$<variable>```, ```$param:```, ```$header:```, ```$cookie:```, ```$initParam:```, ```$pageScope:```, ```$requestScope:```, ```$sessionScope:``` y ```$applicationScope:```.

```xpath
$applicationScope:libros/libro[1]
```

La expresión de ejemplo anterior hace referencia a un nodo XML almacenado en la variable de aplicación ```libros```, y sobre la que se aplica la ruta XPath ```libros/libro[1]:```.

## 18.2. Acciones

De forma general, las acciones para las que no se especifica una variable donde almacenar el resultado lo añaden a la respuesta de la petición en curso a través del ```Writer```.

- ```x:parse```: Parsea un documento XML indicado en un atributo, o en el cuerpo de la acción. El resultado del proceso se puede indicar que se devuelva en un atributo ```varDom``` o ```var```. Cuando se utiliza el primero el resultado es de tipo ```org.w3c.dom.Document```, mientras que para el segundo depende de la implementación del _parser_ utilizado.

    ```xml
    <c:import url="agenda.xml" var="xml"/>

    <x:parse doc="${xml}" var="agenda"/>
    ```

- ```x:out```: Admite una expresión XPath, la evalúa, y añade el resultado a la respuesta de la petición en curso.

    ```xml
    <x:out select="$agenda/contactos" escapeXml="true"/>
    ```

- ```x:set```: Admite una expresión XPath, la evalúa, y devuelve el resultado en una variable.

    ```xml
    <x:set select="$agenda/contactos/contacto[1]" var="contacto"/>
    ```

- ```x:if```: Admite una condición sobre una expresión XPath, la evalúa, y comprueba si la condición se cumple.

    ```xml
    <x:if select="$contacto/ciudad='Paris'">
      Parisino
    </x:if>
    ```

- ```x:choose```: Admite un conjunto de condiciones sobre expresiones XPath, las evalúa, y procesa el cuerpo de la condición que se cumple, o una por defecto si existe.

    ```xml
    <x:choose>
      <x:when select="$contacto/ciudad='Paris'">
        Parisino
      </x:when>
      <x:when select="$contacto/ciudad='Madrid'">
        Madrileño
      </x:when>
      <x:otherwise>
        Desconocido
      </x:otherwise>
    </x:choose>
    ```

- ```x:forEach```: Itera por los elementos devueltos por la evaluación de una expresión XPath dada. Su funcionamiento es similar a la acción ```if``` de la librería ```core```.

    ```xml
    <table>
    <x:forEach select="$agenda/contactos/contacto" var="contacto">
      <tr><td><x:out select="$contacto/nombre"/></td></tr>
    </x:forEach>
    </table>
    ```

- ```x:transform```: Permite aplicar una plantilla de transformación sobre un documento XML. Esta acción admite bastantes parámetros y es recomendable leer la documentación de referencia para sacar un mayor provecho de la misma.

    ```xml
    <c:import url="documento.xml” var="doc"/>
    <c:import url="plantilla.xsl" var="xslt"/>

    <x:transform doc="${doc}" xslt="${xslt}"/>
    ```
