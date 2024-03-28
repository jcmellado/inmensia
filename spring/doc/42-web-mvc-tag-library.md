# 42. Spring: Web MVC (7) - Tag Library

_02-05-2012_ _Juan Mellado_

Spring define sus propias librerías de _tags_ de forma similar a las ofrecidas por JSTL.

## 42.1. Dependencias

Para los ejemplos de esta parte se utilizará JSTL, por lo que es necesario incluir la correspondiente dependencia en el fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>jstl</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

## 42.2. Tag Library

Spring define actualmente dos librerías de _tags_. La primera es de uso general, para el manejo de error, soporte para _themes_, internacionalización, y _binding_, principalmente:

```jsp
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
```

La segunda es más específica para la gestión de formularios:

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

## 42.3. spring:message

Este _tag_ proporciona el texto de un mensaje a partir de su código, resuelto en base a la configuración de internacionalización:

```xml
<spring:message code='saludo'/>
```

## 42.3. form:form

Este _tag_ contiene a todos los demás _tags_ de formularios, ofreciendo sobre todo facilidades para el _binding_.

Consideremos por ejemplo la siguiente clase POJO:

```java
public class Cliente {

  private String nombre;

  private String apellidos;
...
```

Un manejador puede devolver un objeto de este tipo en el modelo:

```java
public ModelAndView handler() {
  ModelAndView mv = new ModelAndView("formulario");

  Cliente cliente = new Cliente();
  cliente.setNombre("Cristobal");
  cliente.setApellidos("Colon");

  mv.addObject("cliente", cliente);

  return mv;
}
```

De forma que la página JSP correspondiente puede utilizarlo directamente:

```xml
<form:form modelAttribute="cliente" action="doIt" method="post">
  <div>
    Nombre: <form:input path="nombre"/>
    <br/>
    Apellidos: <form:input path="apellidos"/>
  </div>
  <input type="submit" value="Enviar"/>
</form:form>
```

El atributo ```modelAttribute``` establece el nombre del objeto dentro del modelo que contiene los datos en proceso. Por defecto su valor es ```command```, por compatibilidad con JSP, pero puede cambiarse utilizando ```commandName```.

Los atributos de tipo ```path``` permiten indicar una propiedad dentro del objeto. De forma que ```path="nombre"``` se resuelve como ```cliente.getNombre()```.

En el formulario de ejemplo, cuando se pulsase al botón de enviar, el manejador recibiría un objeto de la clase ```Cliente```:

```java
@RequestMapping(value="/doIt", method=RequestMethod.POST)
public String handler(Cliente cliente) {
...
```

## 42.4. Controles

Dentro del _tag_ de formulario se pueden incluir los siguientes controles:

- ```form:checkbox```
- ```form:checkboxes```
- ```form:radiobutton```
- ```form:radiobuttons```
- ```form:password```
- ```form:select```
- ```form:option```
- ```form:options```
- ```form:textarea```
- ```form:hidden```
- ```form:errors```

Todos son equivalentes a sus controles homónimos en HTML, pero con las capacidades de _binding_ añadidas a través del atributo ```path```. Además, los controles que tienen el nombre en plural admiten un atributo ```items``` que facilita renderizar directamente los valores de una colección:

```xml
<form:select path="aficiones" items="${aficiones}"/>
```
