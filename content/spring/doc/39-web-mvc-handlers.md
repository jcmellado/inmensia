# 39. Spring: Web MVC (4) - Handlers

_30-04-2012_ _Juan Mellado_

Los _handlers_ (manejadores) son los métodos dentro de los controladores que procesan las peticiones que recibe el servidor _web_. Spring no fuerza que cumplan con el contrato de una interface predeterminada, como ocurre con los _servlets_ tradicionales, ofreciendo un grado de libertad bastante grande a la hora de realizar la implementación.

## 39.1. Tipos de Argumentos

Un _handler_ puede tener cualquier número y casi cualquier tipo de argumentos, sin restricciones de orden, excepto para los de un tipo llamado ```BindingResult```.

Los tipos de argumentos soportados por un _handler_ son:

- ```ServletRequest```, ```HttpServletRequest```, ```ServletResponse```, ```HttpServletResponseInputStream```, ```OutputStream```, ```Reader``` y ```Writer```, como en los _servlets_ tradicionales.

- ```WebRequest``` o ```NativeWebRequest```, que permiten acceder a los objetos de petición y respuesta originales sin tener que utilizar las clases de los _servlets_ tradicionales.

- ```HttpSession```, con la sesión actual.

- ```Principal```, con el usuario actual autentificado.

- ```Locale```, con la configuración regional.

- Parámetros anotados con ```@PathVariable```, ```@RequestParam```, ```@RequestHeader```, ```@RequestBody``` o ```@RequestPart```, para acceder a los distintos componentes de una petición HTTP.

- ```HttpEntity<?>```, para acceder a los componentes de un petición HTTP sin usar anotaciones.

- ```Map```, ```Model``` o ```ModelMap```, para acceder a los datos de negocio intercambiados con la capa de presentación.

- ```RedirectAttributes```, para especificar los atributos a utilizar cuando se produce una redirección.

- Cualquier tipo de objeto sobre el que se deba realizar el _binding_ entre la información del usuario y el modelo de negocio de la aplicación.

- ```Errors``` o ```BindingResult```, con el resultado del proceso de _binding_ efectuado sobre el parámetro declarado justo anteriormente. Es decir, el orden es importante.

- ```SessionStatus```, para controlar el estado e inicializar variables de sesión.

- ```UriComponentsBuilder```, de utilidad para construir URLs relativas a la de la petición siendo atendida.

## 39.2. Tipos Retornados

Un _handler_ puede retornar cualquiera de los siguientes tipos:

- ```ModelAndView```, con la información de proceso y la vista, o nombre de la vista, a aplicar.

- ```Model``` o ```Map```, con la información de proceso, siendo el nombre de la vista resuelto con algún tipo de ```RequestToViewNameTranslator```, cuya implementación por defecto lo resuelve eliminando la extensión del fichero de la URL (```admin/index.html``` => ```admin/index```).

- ```void```, si el _handler_ gestiona la propia respuesta a través de un argumento de tipo ```HttpServletResponse```, o el nombre de la vista se resuelve con algún tipo de ```RequestToViewNameTranslator```.

- ```View``` o ```String```, con la vista a aplicar, estando la información de proceso determinada por los valores asignados a los argumentos del _handler_.

- ```HttpEntity<?>``` o ```ResponseEntity<?>```, con algún componente HTTP.

- Cualquier tipo, que se interpreta en función de si el _handler_ tiene aplicada la anotación ```@ResponseBody```, en cuyo caso se interpreta como el cuerpo HTTP de la respuesta, o la anotación ```@ModelAttribute```, en cuyo caso se interpreta como un dato de la información de proceso con el nombre indicado en la anotación.

## 39.3. Ejemplos

Ejemplos de _handlers_ que muestran algunas de las combinaciones permitidas:

```java
@RequestMapping(value="/tradicional")
public void handler(HttpServletRequest request, HttpServletResponse response) {
...

@RequestMapping(value="/modelo")
public String handler(Model model) {
...

@RequestMapping(value="/binding")
public void handler(@ModelAttribute("receta") Receta receta, BindingResult result) {
...

@RequestMapping(value="/idioma")
public void handler(Locale locale, @RequestParam("gps") String gps) {
...

@RequestMapping(value="/cuerpo")
@ResponseBody
public String handler(@RequestHeader("Accept-Language") String header) {
...
```
