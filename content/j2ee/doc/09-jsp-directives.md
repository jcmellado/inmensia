# 9. JSP (2) - Directivas

_16-05-2012_ _Juan Mellado_

Las directivas se utilizan para informar al servidor _web_ de características concretas de los JSP a la hora de convertirlos en clases. Hay tres directivas disponibles: ```page```, ```taglib``` e ```include```. Y de forma general se expresan con la siguiente sintaxis:

```jsp
<%@ directiva { atributo=”valor” }* %>
```

## 9.1. Directiva "page"

La directiva ```page``` define atributos que sirven para construir el _servlet_, o indicar algunos parámetros necesarios para generar la respuesta, como el tipo de contenido por ejemplo, que en un _servlet_ tradicional se indica llamando al método ```setContentType```.

Por ejemplo:

```jsp
<%@ page info="Mi primer JSP" %>
```

O con la correspondiente sintaxis XML:

```xml
<... xmlns:jsp="http://java.sun.com/JSP/Page">

<jsp:directive.page info="Mi primer JSP"/>
```

Los atributos que admite la directiva ```page``` son los siguientes:

- ```language="scriptingLanguage"```: Lenguaje de programación utilizado dentro de los _scripts_. El único valor admitido es ```java```, que además es el valor por defecto.

- ```extends="className"```: Nombre completo de la clase de la que debería heredar la clase resultante de la compilación de la página JSP, lo que permite utilizar una implementación propia si fuera preciso.

- ```import="importList"```: Lista de tipos, separados por coma, a importar en la clase resultante de la compilación de la página JSP. La lista por defecto incluye ```java.lang.*```, ```javax.servlet.*```, ```javax.servlet.jsp.*``` y ```javax.servlet.http.*```.

- ```session="true|false"```: Indica si la página interviene en una sesión. El valor por defecto es ```true```, lo que quiere decir que existirá una variable instanciada con el nombre ```session``` y de tipo ```HttpSession``` disponible para los _scripts_. Si se cambia a ```false``` la variable no existirá.

- ```buffer="none|sizekb"```: Tamaño a utilizar, en kilobytes, para el _buffer_ que contendrá la respuesta generada por el _servlet_. Es equivalente a llamar al método ```setBufferSize``` sobre el objeto respuesta.

- ```autoFlush="true|false"```: Indica cómo se debe gestionar el _buffer_ que contiene la respuesta. El valor por defecto es ```true```, lo que quiere decir que se envía al cliente cuando se llena.

- ```isThreadSafe="true|false"```: Indica si el _servlet_ resultante debe implementar la interface ```SingleThreadModel```. El valor por defecto es ```false```, y no debería cambiarse ya que el uso de la interface está deprecado.

- ```info="info_text"```: Una cadena de texto libre que se añade a la declaración del _servlet_ y que puede recuperarse con el método ```getServletInfo```.

- ```isErrorPage="true|false"```: Indica si el JSP se utiliza como página de error. Por defecto es ```false```, si se cambia a ```true``` entonces existirá una variable instanciada con nombre ```exception``` y de tipo ```Throwable``` disponible para los _scripts_.

- ```errorPage="error_url"```: URL a la que redirigir el flujo del proceso en caso de producirse una excepción.

- ```contentType="ctinfo"```: Tipo de contenido de la respuesta y codificación utilizada. Es equivalente a llamar al método ```setContentType``` sobre el objeto respuesta. Para las páginas JSP el valor por defecto es ```text/html```, y para los documentos JSP es ```text/xml```.

- ```pageEncoding="peinfo"```: Codificación utilizada para la respuesta. Es equivalente a llamar al método ```setCharacterEnconding``` sobre el objeto respuesta. El valor por defecto no está prefijado, sigue unas reglas basadas en analizar el contenido para obtener el valor correspondiente.

- ```isELIgnored="true|false"```: Indica si el Expression Language (EL) debe estar habilitado mientras se procesa el JSP y las expresiones deben evaluarse. El valor por defecto varía según la versión de la implementación, actualmente se puede asumir que es ```false```. EL se introduce en un artículo posterior.

## 9.2. Directiva "taglib"

JSP permite a las aplicaciones definir sus propias librería de acciones, llamadas _tag libraries_, en vez de limitarlas a utilizar sólo las estándar. La directiva ```taglib``` permite indicar las librerías que utiliza una página JSP, así como el prefijo utilizado para hacer referencia a las mismas.

```jsp
<%@ taglib (uri="tagLibraryURI" | tagdir="tagDir") prefix="tagPrefix" %>
```

La directiva ```taglib``` admite los siguientes parámetros:

- ```uri```: URI de la librería.

- ```tagdir```: Prefijo de la librería que se encuentra dentro del directorio ```/WEB-INF/tags```. Debe comenzar obligatoriamente por ```/WEB-INF/tags/```.

- ```prefix```: Prefijo por el que se hace referencia a la librería. No puede estar vacío, ni tener por nombre ```jsp```, ```jspx```, ```java```, ```javax```, ```servlet```, ```sun``` o ```sunw```.

## 9.3. Directiva "include"

La directiva ```include``` permite incluir un recurso dentro de un JSP, normalmente otro JSP.

Sigue la siguiente sintaxis general:

```jsp
<%@ include file="url" %>
```

O la correspondiente XML:

```xml
<... xmlns:jsp="http://java.sun.com/JSP/Page">

<jsp:directive.include file="url”/>
```
