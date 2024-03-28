# 40. Spring: Web MVC (5) - Modelos

_30-04-2012_ _Juan Mellado_

Un modelo es un objeto genérico de Spring utilizado para almacenar los datos en proceso durante una petición _web_. En la práctica es un objeto ```Map``` que almacena los valores asociados a una cadena de texto con su nombre.

## 40.1. Model

La forma más sencilla de acceder al modelo dentro de un _handler_ es declarar un argumento de tipo ```Model```, ```ModelMap``` o ```Map```.

```java
@RequestMapping(value="/modelo")
public void handler(Model model) {
...
  model.addAttribute("pi", 3.14159);
...
  model.addAttribute(calculadora);
...
```

Como se observa, los atributos pueden añadirse indicando su nombre, o directamente, sin especificarlo. En este segundo caso Spring asigna nombres automáticamente según los siguientes criterios:

- Un valor nulo elevará una excepción

- Un objeto de tipo ```com.empresa.Receta``` tendrá como nombre ```receta```

- Un objeto de tipo ```HashMap``` tendrá como nombre ```hashmap```

- Un _array_ ```Receta[]``` con cero o más elementos tendrá como nombre ```recetaList```

- Un ```ArrayList``` vacío no se añadirá al modelo, pero un ```ArrayList``` de recetas con uno o más elementos tendrá como nombre ```recetaList```

## 40.2. @SessionAttributes

Esta anotación sirve para acceder o establecer las variable de sesión. Las variables se declaran en la clase, y se accede a ellas a través del modelo:

```java
@Controller
@RequestMapping("/recetas")
@SessionAttributes("clienteId")
public class RecetasController {

  @RequestMapping(value="/cuenta")
  public void handler(Model model) {
    model.addAttribute("clienteId", 666);
...
```

## 40.3. @ModelAttribute

Cuando se utiliza esta anotación en un método sirve para indicar que el resultado es un atributo que debe añadirse al modelo:

```java
@RequestMapping(value="/consulta/receta/{recetaId}")
@ModelAttribute("receta")
public Receta handler(@PathVariable("recetaId") Long recetaId) {
    return recetarioService.getReceta(recetaId);
}
```

Cuando se utiliza en un argumento de un método sirve para indicar que el atributo debe recuperarse del modelo:

```java
@RequestMapping(value="/elabora/receta/{recetaId}", method = RequestMethod.POST)
public void handler(@ModelAttribute("receta") Receta receta, BindingResult result) {
...
```

El valor que se obtiene puede provenir de varias fuentes. En primer lugar puede venir de un valor almacenado en las variables de sesión. En segundo lugar puede venir de un valor que ya se encuentre presente en el modelo, añadido por otro _handler_ del mismo controlador. En tercer lugar puede venir recuperado a partir de un valor presente en la URL, explicado a continuación. Y por último, puede venir instanciado por su constructor por defecto.

Los valores recuperados a partir de un valor presente en la URL se basan en el uso de las facilidades de conversión de tipos de Spring. En el ejemplo, si llega un identificador de receta en forma de ```String```, y se define un conversor del tipo ```String``` al tipo ```Receta```, entonces se instanciará un objeto automáticamente a través del conversor. Los tipos básicos por su parte son automáticamente convertidos.

## 40.4. @InitBinder

Esta anotación sirve para declarar un conversor de tipos dentro de un controlador. Se aplica a un método que admite un argumento de tipo ```WebDataBinder```, además de casi cualquier combinación de los mismos tipos soportados por los métodos marcados con ```@RequestMapping```.

La clase ```WebDataBinder``` permite configurar conversores a medida de una aplicación. Sirva de ejemplo el conversor de fechas que aparece en la documentación oficial de referencia:

```java
@Controller
public class MyFormController {

  @InitBinder
  public void initBinder(WebDataBinder binder) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    dateFormat.setLenient(false);
    binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
...
```

Alternativamente puede declararse mediante configuración sobreescribiendo el comportamiento por defecto:

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="cacheSeconds" value="0" />
    <property name="webBindingInitializer">
        <bean class="com.empresa.FarmaciaBindingInitializer" />
    </property>
</bean>
```
