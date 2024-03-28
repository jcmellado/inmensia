# 3. Servlets (3) - Request

_14-05-2012_ _Juan Mellado_

El parámetro de tipo ```HttpServletRequest``` que reciben los métodos de un _servlet_ sirve para encapsular toda la información que envía un cliente a un servidor. Los objetos de este tipo sólo son válidos durante la ejecución de los métodos, los servidores _web_ los reutilizan por razones de eficiencia, por lo que no se deben guardar referencias a ellos.

## 3.1. Parámetros HTTP

Los parámetros HTTP de una petición son valores enviados por un cliente, expresados en forma de cadenas de texto, e identificados por un nombre, aunque teniendo en cuenta que un mismo nombre puede repetirse y tener asignado varios valores distintos.

Un parámetro puede ser parte de la URL de una petición, o estar definido dentro del cuerpo de la misma, como los que se introducen en un formulario:

```xml
<form action="enviar" method="post">
  <label for="nombre">Nombre</label>
  <input id="nombre" type="text" name="nombre">
  <input type="submit" value="Enviar">
</form>
```

El método ```getParameterMap``` devuelve un ```Map``` con los parámetros de la petición. El método ```getParameterNames``` devuelve un ```Enumeration``` con los nombres de los parámetros. El método ```getParameter``` devuelve el primer valor de un parámetro dado su nombre, o ```null``` si no existe. Y el método ```getParameterValues``` devuelve un _array_ de cadenas de texto con todos los valores de un parámetro dado su nombre.

```java
request.getParameter("nombre");
```

Los parámetros accesibles a través de estos métodos son sólo los enviados en el cuerpo de las peticiones POST y los que forman parte de la URL de la petición.

## 3.2. Atributos

Los atributos son objetos propios de una aplicación que se identifican por un nombre, igual que los parámetros HTTP, pero cuyos tipos van más allá de las simples cadenas de textos, y que además sólo pueden contener un único valor. Se usan tradicionalmente para el intercambio de información entre _servlets_.

El método ```getAttributeNames``` devuelve un ```Enumeration``` con los nombres de los atributos. El método ```getAttribute``` devuelve el valor de un atributo dado su nombre. Y el método ```setAttribute``` permite establecer el valor de un atributo.

```java
request.getAttribute("cliente");
```

Los nombres de los atributos no pueden empezar por ```java.```, ```javax.``` ni ```com.sun.```.

## 3.3. Headers

Para acceder a las cabeceras HTTP de una petición se dispone de tres métodos. El método ```getHeaderNames``` devuelve un ```Enumeration``` con los nombres de todas las cabeceras. El método ```getHeader``` devuelve el valor de una cabecera dado su nombre. Y el método ```getHeaders``` devuelve un ```Enumeration``` con todos los valores de una cabecera dado su nombre.

```java
request.getHeader("accept");
```

Por conveniencia existen además los métodos ```getIntHeader``` y ```getDateHeader``` que permiten acceder directamente a los valores de las cabeceras de tipo entero y fechas respectivamente. Así como ```getLocale``` y ```getCharacterEncoding```, para acceder a cabeceras específicas.

## 3.4. Path

Para acceder a la URL de una petición atendida por un _servlet_ se dispone de tres métodos. El método ```getContextPath``` devuelve la parte inicial de la URL que sirve como prefijo de todas las peticiones. El método ```getServletPath``` devuelve la parte intermedia de la URL que casa con el mapping del _servlet_. El método ```getPathInfo``` devuelve la parte final de la URL que no forma parte de las dos anteriores.

```java
request.getContextPath();
```

Las reglas que rigen estos métodos a veces no parecen muy intuitivas. La normal general es que ```contextPath``` y ```servletPath``` no terminan nunca con el separador "/", devolviendo una cadena vacía si sólo tienen dicho carácter. Mientras que ```pathInfo``` devuelve nulo en dicho caso. De cualquier forma, siempre se cumple que la URL completa se puede obtener concatenando ```contextPath```, ```servletPath``` y ```pathInfo```.

Adicionalmente existe toda una familia de métodos para obtener el nombre del protocolo, la dirección del servidor, del cliente, el número del puerto, y así sucesivamente.

## 3.5. Cookies

El método ```getCookies``` devuelve un array de objetos de tipo ```Cookie```:

```java
request.getCookies();
```

Los únicos atributos de una _cookie_ que envía un cliente a un servidor normalmente son el nombre y su valor.

## 3.6. HTTPS

El método ```isSecure``` sirve para comprobar si una conexión se ha realizado a través de HTTPS en vez de HTTP:

```java
request.isSecure();
```

Si la conexión es a través de HTTPS entonces el servidor _web_ añade a la petición el atributo ```javax.servlet.request.cipher_suite```, con el nombre del algoritmo de cifrado, y el atributo ```javax.servlet.request.key_size```, con el tamaño de clave utilizado por el algoritmo.

Si la conexión utiliza certificados SSL entonces el servidor _web_ añade además a la petición el atributo ```javax.servlet.request.X509Certificate``` con un _array_ de objetos de tipo ```java.security.cert.X509Certificate``` que contiene toda la cadena de certificados empezando por el del cliente, luego el utilizado para verificarlo, y así sucesivamente.
