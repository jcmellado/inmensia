# 5. Servlets (5) - Ciclo de Vida y Dispatchers

_14-05-2012_ _Juan Mellado_

Lo más habitual es que un servidor _web_ mantenga una única instancia de un _servlet_ concreto. O uno por cada máquina virtual en entornos distribuidos.

Cuando llega una petición, el servidor resuelve que _servlet_ concreto tiene que atenderla, y si la instancia de dicho _servlet_ no existe entonces la crea. No obstante, se puede forzar a que se instancie un determinado _servlet_ al desplegar la aplicación utilizando la etiqueta ```load-on-startup``` en el fichero ```web.xml```:

```xml
<servlet>
  <servlet-name>HelloServlet</servlet-name>
  <servlet-class>com.company.servlet.HelloServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>
```

El valor numérico de la etiqueta indica el orden en que tiene que instanciarse. Un valor negativo indica que no tiene que instanciarse automáticamente al realizar el despliegue.

## 5.1. Ciclo de Vida

El ciclo de vida de un _servlet_ está definido por la interface ```javax.servlet.Servlet``` que todos los _servlets_ implementan. Cuando se crea una instancia se llama al método ```init``` para dar la oportunidad a que se inicialice. Cuando se recibe una petición que tiene que atender se llama al método ```service``` para que la atienda. Y cuando se va a destruir la instancia se llama al método ```destroy``` para que pueda terminar en orden.

```java
public class HelloServlet extends HttpServlet {

  @Override
  public void init() {
  ...

  @Override
  public void destroy() {
  ...
```

El método ```init``` tiene otra forma en la que recibe un parámetro de tipo ```ServletConfig``` con detalles de la configuración del entorno de ejecución. Un _servlet_ puede decidir que no debe instanciarse por código dentro del método ```init``` elevando una excepción de tipo ```ServletException``` o ```UnavailableException```. Aunque el servidor _web_ puede tratar de instanciarlo de nuevo más adelante. En el caso de ```UnavailableException``` se permite indicar un tiempo (en segundos) durante el cual no se debe intentar volver a instanciarlo.

El método ```service``` no se suele sobrecargar cuando se hereda de ```HttpServlet```, ya que la clase base lo que hace es analizar la petición y llamar a los métodos ```doGet```, ```doPost```, ...

El tiempo que está instanciado en memoria un _servlet_ es responsabilidad del servidor _web_, que puede decidir mantenerlo unos pocos milisegundos, o durante todo el tiempo que esté la aplicación desplegada.

Un mismo _servlet_ puede ser invocado de forma concurrente por varias peticiones simultáneas, por lo que el código debe ser seguro en ese aspecto, evitando el uso de variables de instancia y sincronizando el acceso a recursos si es necesario. El uso de ```synchronized``` sobre los métodos está desaconsejado, ya que impide la ejecución en paralelo obligando a serializar las llamadas.

## 5.2. Dispatchers

Dentro de una aplicación _web_ a veces es necesario tener que redirigir el flujo de proceso de un _servlet_ a otro, o tener que añadir al resultado generado por un _servlet_ el generado por otro. Los _dispatchers_ son las clases que permiten realizar este tipo de tareas.

A los _dispatchers_ se accede a través de dos métodos del _Servlet Context_. El método ```getRequestDispatcher``` devuelve un objeto de tipo ```RequestDispatcher``` correspondiente a un _path_ dado, que necesariamente tiene que empezar con el carácter separador ```/```. El método ```getNamedDispatcher``` devuelve un objeto de tipo ```RequestDispatcher``` correspondiente a un nombre de _servlet_ dado.

```java
RequestDispatcher dispatcher = context.getRequestDispatcher("/otra/url");
```

Una vez obtenida una instancia del _dispatcher_ se puede llamar a sus métodos. El método ```forward``` redirige el flujo de proceso, aunque es necesario que se llame antes de haber enviado ninguna información al cliente y con el _buffer_ de respuesta limpio. El método ```include``` permite que se añada información al _buffer_ de respuesta, aunque en ese caso el uso del objeto respuesta es limitado, no pudiéndose añadir cabeceras HTTP por ejemplo.

```java
dispatcher.forward(request, response);
```
