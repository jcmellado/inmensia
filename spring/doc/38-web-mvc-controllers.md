# 38. Spring: Web MVC (3) - Controladores

_30-04-2012_ _Juan Mellado_

Los controladores son las clases que reciben las peticiones de los usuarios, llaman al servicio que implementa la lógica de negocio, y deciden la vista a aplicar para mostrar la información resultante. Son equivalentes a los clásicos _servlets_, pero que gracias a Spring pueden escribirse de una forma mucho más intuituiva.

## 38.1. Model y View

Las clases ```Model``` y ```View``` son los elementos básicos con los que trabaja un controlador. El modelo almacena los datos en proceso y la vista decide su representación.

Un modelo en Spring es simplemente un _array_ asociativo. Es decir, una colección de pares (nombre, valor). Lo que en Java se corresponde con el tipo ```Map```. Esto permite una gran flexibilidad, ya que dicho modelo se puede convertir fácilmente a cualquier otra clase según la tecnología última que se quiera utilizar, como por ejemplo al formato de atributos que espera una página JSP.

La vista en Spring es un cadena de texto con un nombre. El mecanismo de resolución de dicho nombre es totalmente configurable, y la salida puede ser el resultado de aplicar una plantilla JSP, o una salida personalizada utilizando XML, JSON, o cualquier otro tipo de formato.

## 38.2. Anotaciones

Spring ofrece una forma muy conveniente de utilizar anotaciones para definir controladores. Pero como de costumbre, es necesario recordar activar su procesamiento en la configuración:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
           ...
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">

  <mvc:annotation-driven />
...
```

## 38.3. @Controller

Esta anotación se utiliza para indicar que una clase actúa como controlador:

```java
@Controller
public class FarmaciaController {
...
```

Una clase de este tipo no tiene que heredar ni implementar ninguna interface específica.

## 38.4. @RequestMapping

Esta anotación se utiliza para configurar la URL a la que tiene que atender una clase o un método. Si se aplica a una clase, entonces las URLs de sus métodos son relativas a la indicada en la clase. Un método que tenga esta anotación no tiene que seguir ningún patrón específico:

```java
@Controller
@RequestMapping("/recetas")
public class RecetasController {

  @RequestMapping(method = RequestMethod.GET)
  public void handler() {
...

  @RequestMapping(value="/nueva", method = RequestMethod.GET)
  public void nuevaHandler() {
...
```

Como se observa, la anotación permite utilizar algunos parámetros adicionales, como el método HTTP concreto. Sólo si la petición HTTP es del tipo indicado se llamará al método.

Otros parámetros de la anotación permiten indicar el formato aceptado según el contenido de la cabecera ```Content-Type``` (```consumes="application/json"```), el formato generado según la cabecera ```Accept``` (```produces="text/plain"```), los parámetros presentes en la URL (```params="mode=online"```), o la cabecera HTTP (```headers="cabecera=personalizada"```).

Una característica interesante es que la expresión de los atributos también se pueden negar para excluir condiciones en vez de incluirlas (```consumes="!text/plain"```).

## 38.5. @PathVariable

Las URLs que se configuran para los controladores son en realidad URI Templates. Esta nomenclatura permite definir variables que pueden ser extraídas e inyectadas directamente en los parámetros de los métodos.

Por ejemplo, las URLs de tipo ```farmacia/cliente/cantinflas``` pueden definirse con una URI Template de la forma ```farmacia/cliente/{clienteId}```. Y asociar la variable a un parámetro de entrada utilizando la anotación ```@PathVariable```:

```java
@RequestMapping(value="/cliente/{clienteId}")
public void handler(@PathVariable("clienteId") Long clienteId) {
...
```

Pueden especificarse varias variables, e incluso combinar las de los métodos con los de la clase:

```java
@Controller
@RequestMapping(value="/cliente/{clienteId}"
public class RecetasController {

  @RequestMapping(value="/receta/{recetaId}")
  public void handler(
    @PathVariable("clienteId") Long clienteId,
    @PathVariable("recetaId") Long recetaId) {
...
```

## 38.6. @RequestParam

Esta anotación permite acceder a los parámetros de una petición HTTP:

```java
@RequestMapping(value="/consulta")
public void handler(@RequestParam("recetaId") Long recetaId) {
...
```

Para una petición de la forma ```consulta?recetaId=123```, el ejemplo anterior cargaría directamente el valor ```123``` en el parámetro. Por otra parte, si el parámetro no fuera obligatorio, se podría indicar con ```required=false```.

## 38.7. @RequestHeader

Esta anotación permite acceder al valor de una cabecera HTTP:

```java
@RequestMapping(value="/cabeza")
public void handler(@RequestHeader("Accept-Language") String cabecera) {
...
```

## 38.8. @CookieValue

Esta anotación permite acceder al valor de una _cookie_:

```java
@RequestMapping(value="/galleta")
public void handler(@CookieValue("JSESSIONID") String cookie) {
...
```

## 38.9. @RequestBody

Esta anotación permite acceder al cuerpo de una petición HTTP:

```java
@RequestMapping(value="/cuerpo/de/infarto")
public void handler(@RequestBody String cuerpo) {
...
```
