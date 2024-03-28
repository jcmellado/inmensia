# 8. JSP (1)

_15-05-2012_ _Juan Mellado_

Los _servlets_ permiten crear aplicaciones _web_ en Java que generen contenido de forma dinámica. No obstante, es un API de muy bajo nivel que se basa en concatenar cadenas de texto para componer el resultado final. Esto complica todo el proceso de codificación, ya que las cadenas de texto no se pueden validar en tiempo de compilación, además de dificultar el análisis del código, ya que una cadena de texto se puede añadir en cualquier parte del proceso.

## 8.1. JSP

JSP (JavaServer Pages) es una tecnología que permite programar _servlets_ utilizando una nomenclatura que se asemeja más al resultado final que se quiere conseguir, lo que normalmente es una página HTML. De hecho, un fichero HTML, XML, SVG, o cualquier otro de este tipo, sin más aditamentos, es una página JSP válida. Lógicamente, lo que se hace es añadir elementos propios de JSP para poder generar contenido dinámicamente.

Una página JSP es un fichero de texto, normalmente con extensión ```.jsp```, que dice como generar una respuesta a una petición dada.

Una cosa muy importante que hay que entender es que los JSP son _servlets_. Tienen el aspecto de páginas HTML "con esteroides", pero no por ello dejan de ser _servlets_. Cuando se despliega una aplicación en un servidor _web_, las páginas JSP se parsean y convierten en clases Java dentro de ficheros ```.java``` que son compilados a sus correspondientes ```.class```. Cuando se invoca a una página JSP en realidad se ejecuta dicho código compilado que implementa un _servlet_.

La secuencia que sigue un servidor _web_ cuando se invoca un JSP es comprobar la fecha de última modificación del fichero ```.jsp```. Si la fecha es posterior a la fecha del fichero ```.class``` entonces vuelve a parsear el fichero ```.jsp``` y genera un nuevo ```.class``` que será el que finalmente se ejecute.

Las páginas JSP se almacenan en ficheros que normalmente tienen extensión ```.jsp```. No obstante, JSP tiene directivas que permiten que unos ficheros incluyan a otros, por lo que la especificación recomienda utilizar ```.jspf``` para los ficheros incluidos, aunque no es muy habitual ver esta nomenclatura en la práctica. La letra "f" proviene de "JSP fragments".

## 8.2. Páginas JSP

Desde el punto de vista de un servidor _web_, la estructura de una página JSP se compone de dos partes. Por una parte la plantilla, como el contenido HTML estático, del que no se preocupa. Y por otra parte los elementos JSP que entiende y procesa, que se llaman directivas, elementos de _scripts_ y acciones.

Las directivas se declaran utilizando la siguiente nomenclatura:

```jsp
<%@ directiva ... %>
```

Los elementos de _scripts_ se dividen a su vez en declaraciones, expresiones y _scriptlets_. Las declaraciones adoptan la siguiente forma:

```jsp
<%! ... %>
```

Las expresiones esta:

```jsp
<%= ... %>
```

Y por último, los _scriptlets_:

```jsp
<% ... %>
```

Las acciones se expresan utilizando la nomenclatura XML tradicional:

```jsp
<accion atributo="valor" ... />
```

Y si es necesario, pueden añadirse comentarios con el siguiente patrón:

```jsp
<%-- ... --%>
```

## 8.3. Documentos JSP

Uno de los inconvenientes de trabajar con páginas JSP es que tienen un formato propio que requiere utilizar herramientas a medida para editarlos y procesarlos de una manera conveniente. Para permitir el uso de herramientas estándar, una página JSP se puede escribir también como un "documento JSP". A diferencia de una página JSP, un documento JSP es un fichero con formato XML:

```xml
... xmlns:jsp="http://java.sun.com/JSP/Page"
```

La acciones de JSP cumplen con el estándar XML, por lo que no hay ningún inconveniente en utilizarlas dentro de páginas o documentos JSP indistintamente. Otros elementos del lenguaje en cambio se escriben de forma distinta en función de donde se utilicen:

Elemento | Página JSP | Documento JSP
---------|------------|--------------
Directiva | ```<%@ page ... %>``` | ```<jsp:directive.page ... />```
" | ```<%@ taglib ... %>``` | ```... xmlns:prefijo="URL"```
" | ```<%@ include ... %>``` | ```<jsp:directive.include ... />```
Declaración | ```<%! ... %>``` | ```<jsp:declaration> ... </jsp:declaration>```
Expresión | ```<%= ... %>``` | ```<jsp:expression> ... </jsp:expression>```
Scriptlet | ```<% ... %>``` | ```<jsp:scriptlet> ... </jsp:scriptlet>```
Comentario | ```<%-- ... --%>``` | ```<!-- ... -->```

Los documentos JSP se suelen guardar en ficheros con extensión ```.jspx``` para distinguirlos de los tradicionales ```.jsp```. En la práctica no hay diferencia entre utilizar una u otra nomenclatura, aunque el uso de la notación en formato XML es la recomendada.
