# 41. Spring: Web MVC (6) - Vistas

_01-05-2012_ _Juan Mellado_

La capa de presentación _web_ se compone de vistas, donde cada vista se identifica por un nombre virtual que Spring resuelve según la tecnología usada por la aplicación, como JSP por ejemplo.

## 41.1. Vistas

Como ya se vió en artículos anteriores, una _web_ se compone de controladores que mantienen una serie de manejadores. Cada manejador debe indicar el nombre de la vista concreta que debe utilizarse como respuesta a la petición recibida. Spring permite especificar el nombre de la vista retornando una simple cadena de texto con el nombre de la vista, un objeto de la clase ```View``` o ```ModelAndView```, o dejando que se utilice un nombre por defecto construido por convención en vez de por configuración.

El siguiente ejemplo muestra un manejador que retorna el nombre virtual de vista ```saludo```:

```java
@RequestMapping(value="/bienvenida")
public String handler() {
  return "saludo";
}
```

Para resolver los nombres Spring utiliza un _bean_ que implementa la interface ```ViewResolver```. Algunas de las implementaciones disponibles son ```XmlViewResolver```, que resuelve los nombres usando un fichero XML, o ```UrlBasedViewResolver```, que los resuelve de una forma directa sin necesidad de configuración.

El siguiente ejemplo muestra la configuración de un _bean_ que resuelve los nombres de las vistas anteponiéndoles ```/WEB-INF/jsp``` y terminándolos con ```.jsp```:

```xml
<bean id="viewResolver"
      class="org.springframework.web.servlet.view.UrlBasedViewResolver">
  <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
  <property name="prefix" value="/WEB-INF/jsp/"/>
  <property name="suffix" value=".jsp"/>
</bean>
```

De esta forma, la vista ```saludo``` se resuelve a ```/WEB-INF/jsp/saludo.jsp```. Los JSP es conveniente que estén dentro del directorio ```WEB-INF``` por seguridad, ya que de esa forma no son accesibles desde fuera del servidor directamente.

Spring también permite configurar varios _beans_ para resolver los nombres de las vistas, formando una cadena. Útil cuando se necesitan distintas tecnologías en una misma aplicación. E incluso resolver hacia vistas distintas en función del tipo de contenido solicitado (HTML, atom, PDF, ...).

## 41.2. Errores y Excepciones

El hecho de utilizar Spring no implica que se tenga que renunciar a la configuración habitual en el fichero ```web.xml``` para la gestión de errores o excepciones:

```xml
<error-page>
  <error-code>404</error-code>
  <location>/noEncontrado</location>
</error-page>

<error-page>
  <exception-type>java.lang.Exception</exception-type>
  <location>/excepcion</location>
</error-page>
```

No obstante, Spring permite especificar la anotación ```@ExceptionHandler``` sobre un manejador para responder a las excepciones de un determinado tipo:

```java
@ExceptionHandler(IOException.class)
public String handler(IOException ex, HttpServletRequest request) {
...
```

## 41.3. Redirección

En el desarrollo de una aplicación _web_ a veces es necesario instruir al cliente para realizar una redirección HTTP. Lo que quiere decir que el cliente acaba solicitando una URL distinta a la de partida. El ejemplo más habitual es el del envío de los datos de un formulario con un POST, que fuerza la redirección con un GET a una página de éxito o error según el resultado. De esta forma se evita que el cliente pueda enviar varias veces los datos del formulario refrescando la página, ya que desde el punto de vista del navegador el cliente se encuentra en una página obtenida con un GET.

Spring permite realizar redirecciones a un manejador retornando un objeto de tipo ```RedirectView```, o utilizando el prefijo ```redirect:``` en el nombre de la vista:

```java
@RequestMapping(value="/bienvenida", method=RequestMethod.POST)
public String handler() {
  return "redirect:/al/convento";
}
```

De manera similar, Spring permite utilizar el prefijo ```forward:``` con el comportamiento acostumbrado. Es decir, redirigir de una página a otra, pero esta vez sin intervención del cliente, de forma que la URL aparentemente sea la misma que la original solicitada.

## 41.4. Themes

Los temas (_themes_) definen el aspecto visual de una aplicación. Spring permite definir el tema a utilizar mediante alguna clase que implemente la interface ```ThemeSource```. La implementación por defecto es ```ResourceBundleThemeSource```, que lee la configuración de un fichero ```.properties``` donde deben definirse los recursos que utiliza el tema:

```properties
estilos=/themes/clasico/estilos.css
fondo=/themes/clasico/img/fondo.png
```

Una vez definido, se indica que tema utilizar a Spring a través de una clase que implemente la interface ```ThemeResolver```. Spring proporciona varias implementaciones por defecto, que permiten recuperar el nombre del tema a aplicar de las variables de sesión, o de una _cookie_, por ejemplo.

Otras posibilidades de Spring permiten cambiar el tema de forma dinámica en cada petición mediante el uso de interceptores.
