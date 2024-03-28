# 1. Servlets (1) - WAR, Servlets y Mappings

_14-05-2012_ _Juan Mellado_

Los _servlets_ nacieron de la necesidad de permitir a los desarrolladores crear contenido de forma dinámica para la _web_, sin tener que dejar de utilizar Java, y como alternativa a otras tecnologías disponibles en aquel momento, como era CGI.

Los _servlets_ son como los ladrillos de una aplicación _web_. Son los componentes más básicos con los que se trabaja. Cuando un cliente realiza una petición HTTP, el servidor _web_ se encarga de llamar al _servlet_ adecuado según la configuración proporcionada por la aplicación.

## 1.1. WAR

Las librerías y aplicaciones de escritorio Java tradicionales se empaquetan en un fichero con extensión ```.jar``` (**J**ava **AR**chive). Las aplicaciones _web_ en cambio se empaquetan en un fichero ```.war``` (**W**eb **AR**chive). En esencia son lo mismo, pero con la extensión cambiada para distinguirlos.

Un fichero _war_ característico suele tener al menos un directorio ```WEB-INF``` y un fichero ```web.xml``` dentro de él. Un ejemplo más real contendrá más directorios y ficheros, como en la siguiente estructura de ejemplo:

```text
/META-INF
/WEB-INF
  /classes
  /lib
  web.xml
/images
index.html
```

El directorio ```classes``` es donde se ubicarán las clases que formen parte de la aplicación _web_, y en el directorio ```lib``` las librerías (_jars_) que necesite para su ejecución.

El fichero ```web.xml``` es un descriptor de despliegue, ya que "desplegar" es el nombre con el que se conoce a la acción de instalar una aplicación en un servidor _web_. Un ejemplo básico de este tipo de ficheros sería el siguiente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                             http://java.sun.com/xml/ns/javaee/web-app_2_4.xsd"
         id="WebApp_ID"
         version="2.4">

  <display-name>hello</display-name>
 
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
  </welcome-file-list>

</web-app>
```

La etiqueta ```display-name``` asigna un nombre a la aplicación, que será el que se utilice para diferenciarla del resto de aplicaciones dentro de un servidor.

La etiqueta ```welcome-file-list``` contiene el nombre del primer fichero que debe tratar de servirse cuando se acceda al directorio raíz de la aplicación. Aunque en realidad es una lista de ficheros, si no existe el primero se intenta con el segundo y así sucesivamente.

Para desplegar una aplicación _web_ normalmente basta con dejar el fichero _war_ dentro de un directorio del servidor _web_, aunque existen otras posibilidades. Lo normal es que dentro de un proyecto todos los desarrolladores trabajen de una misma manera, marcado por algún tipo de procedimiento, o simplemente por la costumbre.

## 1.2. Servlets

Un _servlet_ es una clase Java que implementa la interface ```javax.servlet.Servlet```. Sin embargo, para la mayoría de los desarrollos basta con crear una clase que herede de ```HttpServlet```, que ya implementa dicha interface:

```java
public class HelloServlet extends HttpServlet {
...
```

La clase ```HttpServlet``` se encuentra dentro del paquete ```javax.servlet.http```, no incluido dentro del JDK estándar, por lo que es necesario añadir la dependencia con la librería ```servlet-api```.

Si se está utilizando Maven, algo muy recomendable, la dependencia puede añadirse directamente en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>2.4</version>
  <scope>provided</scope>
</dependency>
```

La librería la tiene que proporcionar el servidor _web_ en tiempo de ejecución, de ahí que se utilice el valor ```provided``` en la etiqueta ```scope```. Como versión se utiliza la 2.4, que a pesar de ser bastante antigua aún se encuentra en muchos entornos de producción.

La clase ```HttpServlet``` implementa una serie de métodos que se pueden sobrescribir por parte de una aplicación _web_ con el objetivo de generar contenido propio de manera dinámica. Estos métodos se ejecutan en respuesta a las peticiones HTTP de tipo GET, POST, PUT, DELETE, HEAD, OPTIONS y TRACE, y tienen el apropiado nombre de ```doGet```, ```doPost```, ```doPut```, ```doDelete```, ```doHead```, ```doOptions``` y ```doTrace```. Todos ellos reciben un objeto con la petición realizada desde el cliente y un objeto respuesta donde dejar el resultado.

En las aplicaciones tradicionales sólo se suelen implementar los métodos usados más habitualmente, como ```doGet``` y ```doPost```:

```java
public class HelloServlet extends HttpServlet {

  @Override
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
  ...

  @Override
  public void doPost(HttpServletRequest request, HttpServletResponse response) {
  ...
```

El método ```doHead``` es una forma especializada de GET que sólo retorna las cabeceras HTTP, de igual forma que las retornaría si la petición la procesara el método GET. El método ```doOptions``` se utiliza para retornar el tipo de peticiones HTTP que soporta el _servlet_. Y el método ```doTrace``` retorna por defecto las mismas cabeceras que recibe, lo que resulta útil para depurar algunas opciones avanzadas.

Adicionalmente, un _servlet_ puede implementar también el método ```getLastModified``` para responder a peticiones de acceso a recursos de forma eficiente en función de su fecha de última modificación.

## 1.3. Mapping

Los _servlets_ de una aplicación se deben listar en el fichero ```web.xml``` para que el servidor _web_ donde se realice el despliegue tenga conocimiento de ellos y los pueda gestionar adecuadamente:

```xml
<servlet>
  <servlet-name>HelloServlet</servlet-name>
  <servlet-class>com.company.servlet.HelloServlet</servlet-class>
</servlet>
```

La etiqueta ```servlet-name``` asigna un nombre al _servlet_, y la etiqueta ```servlet-class``` indica el nombre completo de la clase que lo implementa.

En una aplicación de escritorio tradicional se utiliza un menú, u otro tipo de control, como botones por ejemplo, para ejecutar las distintas opciones. En una aplicación _web_ en cambio se utiliza un navegador en el cliente, y se accede a las distintas opciones a través de distintas URLs. Para completar la definición de un _servlet_ se debe indicar las URLs concretas en el fichero ```web.xml```:

```xml
<servlet-mapping>
  <servlet-name>HelloServlet</servlet-name>
  <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

En el ejemplo, el servidor _web_ invocará al _servlet_ ```HelloServlet``` cuando desde el navegador se intente acceder a la URL ```http(s)://<servidor>:<puerto>/<context-root>/hello```.

El ```context-root``` es habitualmente el nombre de la aplicación. Los servidores _web_ lo toman del nombre del _war_ cuando se despliega. Aunque en la práctica es realmente el nombre del directorio donde se encuentran desplegada la aplicación. Puede ser un tanto confuso a veces, sobre todo porque algunos servidores permiten redefinir el nombre por configuración.

En la configuración se pueden utilizar astesricos como comodines (```*```), de forma que un mismo _servlet_ puede utilizarse para atender más de una URL. La regla que sigue un servidor _web_ para encontrar el _servlet_ adecuado es buscar primero uno que tenga configurado exactamente la URL pedida. Si no lo encuentra expande las expresiones con comodines, y busca la que case con la URL pedida que tenga el _path_ más largo contando los caracteres separadores (```/```). Si no lo encuentra busca un _servlet_ que tenga configurada la extensión del fichero de la URL, contando a partir del último punto (```.```). Si no lo encuentra utiliza el _servlet_ por defecto, que es aquel que tiene la raíz (```/```) configurada como URL. Y por último, si no lo encuentra, eleva un error.
