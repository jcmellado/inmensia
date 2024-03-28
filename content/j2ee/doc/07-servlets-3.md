# 7. Servlets (7) - Servlets 3.0

_14-05-2012_ _Juan Mellado_

Todo lo expuesto hasta ahora sobre _servlets_ corresponde a la versión 2.4 del API, una de las más populares debido a que era la versión vigente en los años que se popularizó el desarrollo de aplicaciones _web_.

La versión 2.5 que apareció posteriormente introdujo algunos cambios menores en el descriptor de despliegue y en el API, pero sobre todo la necesidad de utilizar Java 5, ya que obligaba a los servidores _web_ a tener en cuenta las anotaciones que pudieran tener declaradas las clases. La posibilidad de inyectar meta-información directamente en las clases aportaba la ventaja de que el código y la configuración estuvieran en un mismo lugar en vez de en ficheros separados.

Las anotaciones que se introdujeron en aquel entonces trataban de permitir a una aplicación _web_ acceder fácilmente a toda la gama de recursos disponibles dentro de un contenedor J2EE de la versión 5 de Java, como por ejemplo ```@PersistenceContext```, ```@PersistenceUnit```, ```@WebServiceRef```, ```@EJB```, y ```@Resource```. Así como de controlar de una forma más sencilla el ciclo de vida de los _servlets_ con ```@PostConstruct``` y ```@PreDestroy```.

La versión 3.0 posterior añadió sobre todo más anotaciones, la introducción del concepto de _web fragments_, la ejecución asíncrona de _servlets_, y un nuevo API que posibilitaba crear _servlets_, _filters_ y _listeners_ en tiempo de ejecución por parte de una aplicación, aunque esto último más orientado a ser usado por _frameworks_.

## 7.1. Configuración

Para utilizar el nuevo API hay que actualizar la dependencia correspondiente en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.0.1</version>
  <scope>provided</scope>
</dependency>
```

Y utilizar la etiqueta ```metadata-complete``` en el descriptor de despliegue para indicar al servidor _web_ que es necesario que revise las clases contenidas dentro de un _war_ en busca de anotaciones:

```xml
<web-app ...
  id="WebApp_ID"
  version="3.0"
  metadata-complete="false">
...
```

## 7.2. Anotaciones

Una vez realizados los cambios se puede empezar a utilizar las nuevas anotaciones, como ```@WebServlet```, ```@WebFilter``` y ```@WebListener```, que eliminan la necesidad de configurar el fichero ```web.xml```, ya que la configuración se realiza directamente sobre las clases:

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
...

@WebFilter("/*")
public class HelloFilter implements Filter {
...

@WebListener
public class HelloListener implements ServletContextListener {
...
```

## 7.3. Web Fragments

Otro de los cambios realizados en la nueva versión de la especificación es la posibilidad de que las librerías incluidas dentro del directorio ```libs``` de un _war_ definan sus propios _servlets_, _filters_, o _listeners_, mediante un fichero de configuración local llamado ```web-fragment.xml```, que debe encontrarse dentro de su directorio ```META-INF```, evitando así tener que polucionar el fichero ```web.xml``` global.

## 7.4. Sevlets Asíncronos

La posibilidad de ejecutar _servlets_ de forma asíncrona se consigue añadiendo el parámetro ```asyncSupported``` en la configuración del _servlet_, o _filter_, que se quiera poder ejecutar asíncronamente, y usando una instancia de una nueva clase ```AsyncContext``` que puede obtenerse llamando al método ```startAsync``` del objeto petición.

Al indicar que una petición se va a ejecutar de forma asíncrona se puede lanzar un _thread_ y terminar el proceso del _servlet_. El cliente se quedará esperando de la misma forma que si el _servlet_ siguiera ejecutándose. La diferencia es que será el nuevo _thread_ el que se quede esperando, en vez de el _thread_ del _servlet_ impidiendo que atienda nuevas peticiones. Lógicamente el _thread lanzado_ debe ser un objeto de la clase ```Runnable```, y para que pueda terminar el proceso de la petición original se le debe pasar una referencia a la instancia del objeto ```AsyncContext```. Cuando el _thread_ termine su proceso llamará al método ```complete``` si se encarga además de generar la respuesta, o al método ```forward``` para delegar en otro _servlet_.

```java
@WebServlet(value="/hello", asyncSupported=true)
public class HelloServlet extends HttpServlet {

  @Override
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    AsyncContext async = request.startAsync();

    HelloRunnable runnable = new HelloRunnable(async);

    new Thread(runnable).start();
  }

  public class HelloRunnable implements Runnable {
    private AsyncContext async;

    public HelloRunnable(AsyncContext async) {
      this.async = async;
    }

    @Override
    public void run() {
      try {
        Thread.sleep(5 * 1000);

        PrintWriter out = async.getResponse().getWriter();
        out.write("<html><head></head><body>Hello Servlet!</body></html>");

      } catch (Exception exception) {
        exception.printStackTrace();
      } finally {
        async.complete();
      }
    }
  }
}
```

Un detalle muy importante a tener en cuenta es que todos los _filters_ y _servlets_ ejecutados dentro de la cadena por la que pasa una petición deben soportar la ejecución asíncrona. Si alguno no la soporta se eleva una excepción de tipo ```IllegalStateException```.
