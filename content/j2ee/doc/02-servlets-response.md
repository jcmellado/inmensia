# 2. Servlets (2) - Response

_14-05-2012_ _Juan Mellado_

El parámetro de tipo ```HttpServletResponse``` que reciben los métodos de un _servlet_ sirve para encapsular toda la información que el _servlet_ retorna al cliente. Los objetos de este tipo sólo son válidos durante la ejecución de los métodos, los servidores _web_ los reutilizan por razones de eficiencia, por lo que no se deben guardar referencias a ellos.

Al tratarse de peticiones HTTP, la información devuelta al cliente se divide en dos, por una parte las cabeceras y por otra parte el cuerpo.

## 2.1. Body

El mecanismo utilizado para almacenar el cuerpo de la respuesta es utilizar un ```Writer``` sobre un _buffer_ expuesto a través del método ```getOutputStream``` o ```getWriter```:

```java
@Override
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
  PrintWriter out = response.getWriter();

  out.write("<html><head></head><body>Hello Servlet!</body></html>");
}
```

En el ejemplo se observa como se escribe directamente el contenido de una página _web_ en formato de texto plano.

Por razones de eficiencia, los servidores _web_ habitualmente no envían la información del _buffer_ a medida que se va introduciendo, sino que esperan para enviarla toda de una sola vez. No obstante, los _servlets_ pueden controlar algunos detalles de este proceso.

El método ```getBufferSize``` devuelve el tamaño de _buffer_ utilizado, o cero, si no se está utilizando _buffering_. El método ```setBufferSize``` permite especificar un tamaño deseado de _buffer_, aunque un servidor _web_ puede reservar un tamaño más grande al solicitado para gestionar de manera más eficiente los bloques de memoria. Una vez escrito algo al _buffer_ es ilegal intentar sugerir otro tamaño, hacerlo provoca que se eleve una excepción de tipo ```IllegalStateException```.

```java
response.setBufferSize(16384);
```

Por otra parte, el método ```isCommitted``` permite averiguar si ya se ha devuelto información al cliente. El método ```flushBuffer``` permite forzar el envío de información al cliente. Y los métodos ```reset``` y ```resetBuffer``` limpian la respuesta, el primero de forma completa, incluyendo las cabeceras y el estado, y el segundo de forma parcial, sólo la información contenida en el _buffer_. Es ilegal llamar a un método de limpieza una vez la información ha sido enviada.

## 2.2. Headers

Si se tiene que establecer o modificar las cabeceras HTTP se dispone de dos métodos. El método ```addHeader``` permite añadir una cabecera con un nombre y un valor. El método ```setHeader``` permite modificar el valor de una cabecera ya existente.

Los dos métodos anteriores trabajan sólo con cadenas de texto, así que por conveniencia existen además los métodos ```addIntHeader```, ```addDateHeader```, ```setIntHeader```, y ```setDateHeader```, que trabajan con enteros y fechas respectivamente.

De igual forma, por conveniencia, existe el método ```setLocale```, que permite especificar un código de país estándar para la configuración regional, y ```setContentType```, que permite especificar el tipo de contenido y juego de caracteres utilizado, aunque sólo es efectivo si se llama antes de obtener una referencia al ```Writer```:

```java
response.setContentType("text/html;charset=UTF-8");
```

En caso de utilizarse ```setContentType```, es responsabilidad del desarrollador que el tipo especificado en la cabecera corresponda con el contenido del cuerpo. El servidor no realiza ningún tipo de conversión del contenido automáticamente.

## 2.3. Redirecciones

El resultado de una petición puede ser una redirección a otra página, o un código de error generado de forma intencionada por código. El método ```sendRedirect``` permite indicar una URL a la que debe redigirse el navegador. El método ```sendError``` permite indicar un código de error.

Cuando un cliente realiza una petición a un servidor, y se produce una redirección, lo que hace el servidor es enviar una URL al cliente, y el cliente realiza una nueva petición al servidor a la URL indicada:

```java
response.sendRedirect("otro.html");
```

En el caso de los errores, es normal tener dispuestas algunas páginas estáticas informativas para mostrárselas a los clientes, como la clásica de "_página no encontrada_". Estas páginas se configuran en el fichero ```web.xml```:

```xml
<error-page>
  <error-code>404</error-code>
  <location>/404.html</location>
</error-page>
```

El valor de la etiqueta ```error-page``` es un código de error HTTP, y el de la etiqueta ```location``` el de la URL que debe servirse en caso de que se produzca el error.

De forma análoga, se puede utilizar la etiqueta ```exception-type``` para capturar excepciones, indicando el nombre completo de la clase, en vez códigos de error HTTP:

```xml
<error-page>
  <exception-type>java.lang.NullPointerException</exception-type>
  <location>/nulo.html</location>
</error-page>
```
