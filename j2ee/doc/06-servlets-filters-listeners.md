# 6. Servlets (6) - Filters y Listeners

_14-05-2012_ _Juan Mellado_

## 6.1. Filters

Los _filters_ pueden verse como un tipo especial de _servlets_, en la medida que se configuran en el fichero ```web.xml```, y los invoca el servidor _web_. No obstante, su propósito es bastante distinto. No actúan como componentes de una aplicación que responden a una petición, sino como componentes _proxy_ que monitorizan el tráfico con el propósito de auditarlo o modificarlo. En este caso el término tráfico abarca tanto al contenido desplegado de forma estática como al generado dinámicamente.

Un _filter_ es una clase Java que implementa la interface ```javax.servlet.Filter```:

```java
public class HelloFilter implements Filter {

  @Override
  public void init(FilterConfig filterConfig) throws ServletException {
  ...

  @Override
  public void doFilter(
    ServletRequest request,
    ServletResponse response,
    FilterChain chain
  ) throws IOException, ServletException {
  ...

  @Override
  public void destroy() {
  ...
```

Los _filters_ deben declararse en el fichero ```web.xml```, para que el servidor _web_ sepa que tiene que instanciarlos:

```xml
<filter>
  <filter-name>HelloFilter</filter-name>
  <filter-class>com.company.filter.HelloFilter</filter-class>
</filter>
```

Y de forma similar a los _servlets_, para cada _filter_ debe indicarse las URLs para las que tienen que invocarse:

```xml
<filter-mapping>
  <filter-name>HelloFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

Como alternativa a la etiqueta ```url-pattern```, que mapea un conjunto de URLs, puede utilizarse la etiqueta ```servlet-name```, que permite indicar el nombre de un _servlet_ específico. De igual forma, se puede utilizar la etiqueta ```dispatcher``` para restringir la activación del _filter_ a una operación de ```forward``` o ```include```. Y adicionalmente se pueden indicar parámetros de inicialización específicos para un _filter_ utilizando la etiqueta ```init-param```.

El servidor _web_ instancia todos los _filters_ después del despliegue, antes de atender ninguna petición, y siempre teniendo en cuenta que en entornos distribuidos los _filters_ se instancian por máquina virtual.

El método ```init``` es llamado después de instanciar el _filter_. El método ```destroy``` es llamado antes de destruir la instancia del _filter_. Y el método ```doFilter``` es llamado cuando se recibe una petición que casa con su configuración, pasándole como argumentos los objetos petición y respuesta.

Los _filters_ pueden ser usados como _proxies_ a modo de _wrappers_ sobre los objetos petición y respuesta, con el objetivo de añadirles funcionalidad o modificar su contenido. Como añadir cabeceras HTTP a las respuestas por ejemplo:

```java
chain.doFilter(request, response);

HttpServletResponse modified = (HttpServletResponse) response;

modified.addHeader("hello", "world");
```

La lista de _filters_ forman una cadena, las peticiones viajan a través de todos ellos de manera ordenada, uno detrás de otro. Es responsabilidad de un _filter_ llamar al siguiente de la cadena. Si no se hace, se interrumpe la secuencia normal de procesamiento de la petición en curso, lo que en algunos casos puede ser conveniente, como por ejemplo para implementar algún tipo de control de seguridad.

## 6.2. Listeners

Los _listeners_ son componentes _web_ que permiten a los desarrolladores monitorizar el estado de los objetos de tipo ```ServletContext```, ```HttpSession``` y ```ServletRequest```. Un ejemplo de uso típico es monitorizar la creación del contexto principal, que en cierta forma equivale al "arranque" de una aplicación _web_, con el objeto de establecer una conexión con una base de datos, y guardar dicha conexión dentro del propio contexto.

Los _listeners_ se declaran dentro del fichero ```web.xml```, de forma similar a otros componentes _web_:

```xml
<listener>
  <listener-class>com.company.listener.HelloListener</listener-class>
</listener>
```

Los _listeners_ son instanciados al arrancar el servidor _web_, y destruidos al pararlo. En entornos distribuidos se crea un _listener_ por máquina virtual.

Un _listener_ es una clase Java que implemente una o más de las interfaces especializadas que existen para cada uno de los tipos de objeto soportados, como por ejemplo ```ServletContextListener```:

```java
public class HelloListener implements ServletContextListener {

  @Override
  public void contextInitialized(ServletContextEvent sce) {
  ...

  @Override
  public void contextDestroyed(ServletContextEvent sce) {
  ...
```

La interface ```ServletContextListener``` permite monitorizar el ciclo de vida de los objetos de tipo ```ServletContext```, cuando han sido creados y están listos para recibir la primera petición, y cuando están a punto de ser destruidos.

La interface ```ServletContextAttributeListener``` permite monitorizar el cambio en los atributos de los objetos de tipo ```ServletContext```, cuando han sido añadidos, modificados, o borrados.

La interface ```HttpSessionListener``` permite monitorizar el ciclo de vida de los objetos de tipo ```HttpSession```, cuando han sido creados, invalidados, o expirados por _timeout_.

La interface ```HttpSessionAttributeListener``` permite monitorizar el cambio de atributos de los objetos de tipo ```HttpSession```, cuando han sido añadidos, modificados, o borrados.

La interface ```HttpSessionActivationListener``` permite monitorizar la migración de sesiones en entornos distribuidos de los objetos de tipo ```HttpSession```. Es decir, cuando se mueven de un servidor a otro.

La interface ```HttpSessionBindingListener``` permite monitorizar el cambio de las propiedades de los objetos almacenados dentro de los objetos de tipo ```HttpSession```, siempre y cuando implementen la interface que proporciona las facilidades de _binding_.

La interface ```ServletRequestListener``` permite monitorizar el ciclo de vida de los objetos de tipo ```ServletRequest```, en particular cuando una petición está lista para ser procesada.

La interface ```ServletRequestAttributeListener``` permite monitorizar el cambio en los atributos de los objetos de tipo ```ServletRequest```, cuando han sido añadidos, modificados, o borrados.
