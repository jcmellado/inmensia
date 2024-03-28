# 27. JSF (9) - Composite Components

_02-06-2012_ _Juan Mellado_

Los componentes compuestos son tipos especiales de plantillas que actúan, como su propio nombre indica, como componentes.

La idea general es definir elementos que implementan una determinada funcionalidad y pueden reutilizarse. La librería de acciones ```composite``` de JSF permite desarrollar componentes usando XHTML sin tener que escribir código Java.

## 27.1. Componentes

La definición de un componente se divide en dos partes. Por una parte la interface, y por otra parte la implementación.

La interface se declara utilizando la acción ```composite:interface```. En ella se declaran, entre otros, los atributos que se quiere tenga el componente. Estos atributos actúan como variables de instancia y se gestionan como un ```Map``` de pares (nombre, valor).

Por su parte, la implementación se declara utilizando la acción ```composite:implementation```. En ella se disponen los elementos que forman parte del cuerpo del componente y se puede hacer referencia a los atributos definidos en la interface.

Sirva de ejemplo el siguiente componente:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:composite="http://java.sun.com/jsf/composite"
      xmlns:h="http://java.sun.com/jsf/html">

<h:body>
  <composite:interface>
    <composite:attribute name="nombre" required="false"/>
  </composite:interface>

  <composite:implementation>
    <h:outputText value="Hola "/>
    <h:outputText value="#{cc.attrs.nombre}"/>
  </composite:implementation>
</h:body>
   
</html>
```

En la interface del componente de ejemplo se ha definido un atributo opcional llamado ```nombre```, y en la implementación se muestra el texto ```Hola``` junto con el valor del atributo. Como se observa, para acceder a los atributos se utiliza la variable ```cc``` (_composite component_). Esta variable es definida por JSF y está disponible para su uso dentro de las expresiones de EL.

Para continuar con el ejemplo, supondremos que el componente se guarda con el nombre ```saludo.xhtml``` dentro del directorio ```resources/libreria``` del _war_ de la aplicación. Donde el nombre ```resources``` es obligatorio, viene impuesto por JSF. El nombre ```libreria``` es el nombre de la libreria de componentes que se quiere utilizar, que puede puede ser cualquiera. Y el nombre ```saludo``` es el nombre que identifica al componente, que también puede ser cualquiera.

Para utilizar el componente dentro de una página tiene que incluirse el _namespace_ ```http://java.sun.com/jsf/composite/libreria```, hacer referencia al componente utilizando un _tag_ con el nombre del componente, y dar valor a los atributos definidos en su interface:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:libreria="http://java.sun.com/jsf/composite/libreria">

  <body>
    <libreria:saludo nombre="mundo"/>
  </body>
 
</html>
```

## 27.2. Conversores, Validadores y Listeners

Para que un componente sea realmente reutilizable se tiene que poder cambiar el comportamiento de sus elementos internos desde fuera del mismo. Para poder aplicar conversores, validadores y _listeners_ a dichos elementos se tienen que utilizar las acciones ```composite:valueHolder```, ```composite:editableValueHolder```, y ```composite:actionSource```. Cada una de ellas expone una interface que puede ser utilizada por las páginas para ajustar determinados aspectos del comportamiento de un componente a sus necesidades.

Por ejemplo, en el siguiente componente se declara un formulario con un par de valores, uno inmutable (```ValueHolder```), otro editable (```EditableValueHolder```), y un botón (```ActionSource```):

```xml
<composite:interface>
  <composite:attribute name="valor" required="false"/>
  <composite:attribute name="texto" required="false"/>

  <composite:valueHolder name="etiquetaValor" targets="form:etiqueta"/>           
  <composite:editableValueHolder name="editorTexto" targets="form:editor"/>
  <composite:actionSource name="botonEnviar" targets="form:boton"/>
</composite:interface>

<composite:implementation>
  <h:form id="form">
    <h:outputText id="etiqueta" value="#{cc.attrs.valor}">
    <h:inputText id="editor" value="#{cc.attrs.texto}"/>
    <h:commandButton id="boton" value="Enviar"/>
  </h:form>
</composite:implementation>
```

La interface declarada de esa forma permite asociar los conversores, validadores y _listeners_ que se quieran a los elementos internos desde una página que utilice el componente:

```xml
<libreria:componente>
  <f:convertNumber for="etiquetaValor" pattern="#,##0.00"/>
  <f:validateLength for="editorTexto" maximum="10"/>
  <f:actionListener for="botonEnviar" binding="#{bean.listener}"/>
</libreria:componente>
```

## 27.3. Acciones

En la interface de un componente, además de atributos que actúan como variables, se pueden definir atributos que actúan como métodos.

Sirva de ejemplo el cuerpo del siguiente componente:

```xml
<composite:interface>
  <composite:attribute name="metodo" method-signature="java.lang.String action()"/>
</composite:interface>

<composite:implementation>
  <h:commandButton value="Método" action="#{cc.attrs.metodo}"/>
</composite:implementation>
```

El atributo ```metodo``` se ha declarado utilizando ```method-signature```, que permite especificar la definición de un método.

Una página que incluya el componente debe suministrar la implementación del método:

```xml
<libreria:saludo metodo="#{hello.saludo}"/>
```

El método puede estar definido en un _bean_, y seguir la nomenclatura definida en el componente:

```java
@ManagedBean
public class Hello {

  public String metodo() {
    return "Hola mundo";
  }
...
```
