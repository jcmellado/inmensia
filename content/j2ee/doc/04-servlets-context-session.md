# 4. Servlets (4) - Servlet Context y Sesiones

_14-05-2012_ _Juan Mellado_

## 4.1. Servlet Context

El objeto de tipo ```ServletContext``` permite a un _servlet_ tener acceso a la aplicación _web_ que lo contiene. Aunque pueda parecer un concepto un poco extraño al principio, se entiende mejor diciendo que es un área a la que tienen acceso todos los _servlets_ de una misma aplicación _web_, y en consecuencia, se usa para almacenar objetos "globales".

Un _servlet_ puede acceder al contexto llamando al método ```getServletContext```:

```java
ServletContext context = getServletContext();
```

Una vez recuperado se puede utilizar para llamar a los distintos métodos que implementa, aunque teniendo en cuenta que en entornos distribuidos existe un objeto de contexto distinto por cada máquina virtual.

## 4.2. Parámetros de Inicialización

Los parámetros de inicialización de una aplicación _web_ son valores propios que se definen dentro del fichero ```web.xml```:

```xml
<context-param>
  <param-name>webmaster</param-name>
  <param-value>webmaster@company.com</param-value>
</context-param>
```

El método ```getInitParameterNames``` devuelve un ```Enumeration``` con los nombres de todos los parámetros de inicialización de la aplicación _web_. El método ```getInitParameter``` devuelve el valor de un parámetro de inicialización dado su nombre.

```java
context.getInitParameter("webmaster");
```

## 4.3. Atributos

Los atributos son los objetos propios añadidos al contexto por los _servlets_. Los atributos se identifican por un nombre, y su valor es accesible por todos los _servlets_ de una aplicación _web_. En cierta forma, son objetos "globales" dentro de una aplicación _web_.

El método ```setAttribute``` añade un atributo al contexto. El método ```getAttributeNames``` devuelve un ```Enumeration``` con los nombres de todos los atributos presentes en el contexto. El método ```getAttribute``` devuelve el valor de un atributo dado su nombre. El método ```removeAttribute``` elimina un atributo del contexto dado su nombre.

```java
context.getAttribute("dao");
```

Los atributos son locales a la máquina virtual donde se crearon, por lo que no son seguros de utilizar en entornos distribuidos para compartir información.

## 4.4. Recursos

El contexto permite acceder al contenido estático de una aplicación _web_. Es decir, a los ficheros sin procesar desplegados con la aplicación, como por ejemplo las imágenes.

El método ```getResource``` devuelve un objeto URL correspondiente al nombre de recurso dado, y que necesariamente tiene que comenzar con el carácter separador ```/```. El método ```getResourceAsStream``` devuelve un ```InputStream``` para el nombre dado.

```java
context.getResource("/index.html");
```

Si se solicita un fichero con extensión JSP, o de cualquier otro tipo similar, lo que se obtiene es el fichero sin procesar, su código fuente, no el contenido generado dinámicamente por él.

Un recurso interesante al que se puede acceder es el directorio temporal que un servidor _web_ está obligado a proporcionar a cada aplicación, y que se expone a través de un atributo que tiene por nombre ```javax.servlet.context.tempdir```:

```java
context.getAttribute("javax.servlet.context.tempdir");
```

El objeto retornado es de tipo ```File```.

## 4.5. Sesiones

HTTP es un protocolo que carece de estado, las peticiones entre cliente y servidor se suceden de forma independiente entre sí. Sin embargo, para algunas aplicaciones _web_ es importante que las peticiones puedan asociarse con un determinado cliente.

Los servidores _web_ permiten el establecimiento de sesiones entre una aplicación y un cliente determinado, de forma que se pueda almacenar en ellas información de contexto relativa a cada conexión en particular. Es decir, es un área donde almacenar objetos propios relativos a un cliente en particular, a modo de variables de "instancia" por cliente.

La identificación de las sesiones HTTP se mantiene de dos formas. O bien a través de una _cookie_, es decir, como información enviada del servidor al cliente, y devuelta luego por el cliente al servidor. O bien a través de la reescritura de la URL, es decir, añadiendo información extra a la URL. En ambos casos el identificador de la sesión viaja asociado al nombre ```JSESSIONID```. Ya sea como el nombre de la _cookie_, o como un parámetro adicional añadido a la URL, como por ejemplo: ```http://www.company.com/hello?JSESSIONID=1234```.

A las sesiones se accede desde los objetos de petición y respuesta que reciben los métodos que implementan los _servlets_.

```java
HttpSession session = request.getSession();
```

Los objetos almacenados en una sesión se llaman atributos y se identifican por un nombre. El método ```setAttribute``` establece el valor de un atributo dado su nombre. El método ```getAttribute``` devuelve el valor de un atributo dado su nombre. El método ```getAttributeNames``` devuelve un ```Enumeration``` con todos los nombres de los atributos de la sesión.

```java
session.getAttribute("cliente");
```

Las sesiones tienen un tiempo de validez determinado. Si no se detecta actividad en la sesión transcurrido ese tiempo el servidor las invalida. En el fichero ```web.xml``` se puede configurar el tiempo máximo de validez en minutos, o indicar un valor de cero o negativo para que no expiren nunca:

```xml
<session-config>
  <session-timeout>60</session-timeout>
</session-config>
```

El método ```getCreationTime``` devuelve la fecha de creación de la sesión. El método ```getLastAccessedTime``` devuelve la última fecha en la que se accedió a la sesión. El método ```getMaxInactiveInterval``` devuelve el tiempo máximo que puede estar una sesión inactiva hasta de que el servidor la haga expirar automáticamente.

Algunas aplicaciones que requieran un tratamiento más avanzado de lo habitual pueden añadir objetos de tipo ```HttpSessionBindingListener``` para controlar cuando se añade, modifica o borra un atributo de una sesión.

Los sesiones son locales a la máquina virtual donde se crearon, por lo que no son seguros de utilizar en entornos distribuidos para compartir información.
