# 16. JSTL (3) - i18n

_17-05-2012_ _Juan Mellado_

La librería ```i18n``` proporciona facilidades para la creación de aplicaciones que soporten varios idiomas y configuraciones regionales distintas.

## 16.1. Configuración

Para que esta librería funcione correctamente es necesario realizar una configuración previa. Aunque también se puede omitir y hacerla en tiempo de ejecución utilizando las acciones.

Los mensajes de texto que se quieran mostrar en idiomas distintos es necesario almacenarlos en ficheros ```.properties``` y ubicarlos en la carpeta ```WEB-INF/classes``` dentro del _war_ de la aplicación. Un fichero distinto para cada idioma, con el mismo nombre base para todos ellos, como ```messages.properties``` por ejemplo, pero con el código del idioma añadido para distinguirlos, como ```messages_es.properties```.

```properties
saludo=¡Hola mundo!
bienvenida=Bienvenido {0}
```

El nombre base se tiene que configurar en el fichero ```web.xml``` de la aplicación, añadiendo la variable ```javax.servlet.jsp.jstl.fmt.localizationContext``` al contexto:

```xml
<context-param>
  <param-name>javax.servlet.jsp.jstl.fmt.localizationContext</param-name>
  <param-value>messages</param-value>
</context-param>
```

Otras variables disponibles permiten establecer un idioma fijo, independientemente del utilizado por el cliente en cada petición (```javax.servlet.jsp.jstl.fmt.locale```), un idioma por defecto por si ninguno de los idiomas disponibles coincide con el utilizado por el cliente (```javax.servlet.jsp.jstl.fmt.fallbackLocale```), y una zona horaria por defecto (```javax.servlet.jsp.jstl.fmt.timeZone```).

## 16.2. Acciones

Las funcionalidad ofrecida por la librería admite muchos parámetros en cada una de sus acciones, por lo que es recomendable leer la documentación de referencia. Aunque como mínimo, es necesario saber, que de forma general, cuando no se indica el nombre de una variable como destino del resultado de una acción, este se añade a la respuesta.

- ```fmt:message```: Añade una mensaje de texto a la respuesta de la petición.

    ```xml
    <fmt:message key="saludo"/>
    ```

- ```fmt:param```: Permite añadir los valores de los parámetros en los mensajes.

    ```xml
    <fmt:message key="bienvenida">
      <fmt:param value="al convento"/>
    </fmt:message>
    ```

- ```fmt:parseNumber```: Admite una cadena de texto con un número y devuelve un objeto de tipo ```java.lang.Number```.

    ```xml
    <fmt:parseNumber value="12.3"/>
    ```

- ```fmt:formatNumber```: Admite un objeto de tipo ```java.lang.Number``` y devuelve una cadena de texto con el número.

    ```xml
    <fmt:formatNumber value="12.3" pattern="#,##0.00"/>
    ```

- ```fmt:parseDate```: Admite una cadena de texto con una fecha y devuelve un objeto de tipo ```java.util.Date```.

    ```xml
    <fmt:parseDate value="20120517" pattern="yyyymmdd"/>
    ```

- ```fmt:formatDate```: Admite un objeto de tipo ```java.util.Date``` y devuelve una cadena de texto con la fecha.

    ```xml
    <fmt:parseDate value="20120517" pattern="yyyymmdd" var="fecha"/>

    <fmt:formatDate value="${fecha}" pattern="dd/mm/yyyy"/>
    ```

- ```fmt:bundle```: Establece el nombre base con el que debe trabajarse dentro de un bloque de un JSP.

    ```xml
    <fmt:bundle basename="messages">
    ...
    </fmt:bundle>
    ```

- ```fmt:setBundle```: Almacena en la variable indicada un objeto de tipo ```java.util.ResourceBundle``` con el nombre base dado. Si no se especifica variable destino, se almacena el nombre base en la variable ```javax.servlet.jsp.jstl.fmt.localizationContext```.

    ```xml
    <fmt:setBundle basename="messages"/>
    ```

- ```fmt:setLocale```: Establece la configuración regional con la que debe trabajarse.

    ```xml
    <fmt:setLocale value="es"/>
    ```

- ```fmt:timeZone```: Establece la zona horaria con la que debe trabajarse dentro de un bloque de un JSP.

    ```xml
    <fmt:timeZone value="GMT+1">
    ...
    </fmt:timeZone>
    ```

- ```fmt:setTimeZone```: Almacena en la variable indicada un objeto de tipo ```java.util.TimeZone``` con la zona horaria dada. Si no se especifica variable destino, se almacena la zona horaria en la variable ```javax.servlet.jsp.jstl.fmt.timeZone```.

    ```xml
    <fmt:setTimeZone value="GMT+1"/>
    ```

- ```fmt:requestEncoding```: Establece el método de codificación, de igual forma que se hace en un _servlet_ tradicional llamando al método ```setCharacterEncoding```.

    ```xml
    <fmt:requestEncoding value="UTF-8"/>
    ```
