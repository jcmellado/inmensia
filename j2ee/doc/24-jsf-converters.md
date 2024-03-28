# 24. JSF (6) - Conversores

_02-06-2012_ _Juan Mellado_

JSF proporciona una serie de conversores estándar a través de sus librerías de acciones, a la par que permite definir conversores propios por parte de las aplicaciones.

El propósito es convertir la información introducida en el cliente, en formato de texto a través de un formulario, a su correspondiente tipo dentro de un _bean_. Y convertir la información de los _beans_ en el servidor a su correspondiente representación en modo texto para el cliente.

## 24.1. Estándar

Los conversores estándar de JSF se encuentran definidos dentro del paquete ```javax.faces.convert```, y sus nombres son lo suficientemente intuitivos como para saber que tipo convierte cada uno de ellos:

- ```NumberConverter```
- ```DateTimeConverter```
- ```ShortConverter```
- ```LongConverter```
- ```IntegerConverter```
- ```BigIntegerConverter```
- ```FloatConverter```
- ```DoubleConverter```
- ```BigDecimalConverter```
- ```BooleanConverter```
- ```CharacterConverter```
- ```ByteConverter```
- ```EnumConverter```

Para especificar que conversor utilizar dentro de un componente de forma explícita se utiliza el atributo ```converter```:

```xml
<h:inputText converter="javax.faces.convert.IntegerConverter"/>
```

No obstante, esto no hace falta en la mayoría de los casos, pues los conversores se aplican automáticamente en función del tipo de valor asociado a cada componente.

## 24.2. NumberConverter y DateTimeConverter

Sólo dos de los conversores estándar están asociados a acciones y pueden utilizarse mediante un _tag_: ```NumberConverter``` y ```DateTimeConverter```.

La acción ```NumberConverter``` admite atributos que permiten especificar el detalle con el que tiene que representarse una cantidad:

```xml
<h:outputText value="#{pedido.total}">
  <f:convertNumber currencySymbol="€" type="currency"/>
</h:outputText>
```

La acción ```DateTimeConverter``` admite atributos que permiten especificar el detalle con el que tiene que representarse una fecha:

```xml
<h:outputText value="#{persona.nacimiento}">
  <f:convertDateTime pattern="dd/MM/yyyy"/>
</h:outputText>
```

Además de los mostrados en los ejemplos, las acciones admiten otros atributos más que pueden consultarse en la documentación oficial de referencia.

## 24.3. Binding

De igual forma que se explicó para los validadores, se puede definir una propiedad dentro de un _bean_ que actúe como conversor personalizado:

```java
@ManagedBean
public class Usuario {

  private DateTimeConverter conversor = new DateTimeConverter();

  public DateTimeConverter getConversor() {
    conversor.setPattern("dd/MMM");
    return conversor;
  }
...
```

Y utilizando las facilidades de _binding_ de JSF, usarlo en un componente:

```xml
<h:outputText value="#{usuario.nacimiento}" >
  <f:convertDateTime binding="#{usuario.conversor}"/>
</h:outputText>
```

## 24.4. Converter

JSF permite a las aplicaciones implementar sus propios conversores. Un conversor es una clase que implementa la interface ```Converter```.

Un conversor se declara realizando la configuración correspondiente dentro del fichero ```faces-config.xml```, o de forma mucho más cómoda utilizando la anotación ```@FacesConverter``` en la clase que lo implementa:

```java
@FacesConverter(value="conversor")
public class Conversor implements Converter {
...
```

La anotación ```@FacesConverter``` admite dos atributos. El atributo ```value``` que permite especificar un nombre identificador para el conversor. Y el atributo ```forClass``` que permite especificar el nombre de una clase. Si se indica el nombre de una clase entonces JSF utilizará automáticamente el conversor para los objetos de dicha clase.

La interface ```Converter``` obliga a implementar dos métodos. El método ```getAsString``` para convertir el objeto a texto. Y el método ```getAsObject``` para convertir texto en el objeto:

```java
public String getAsString(FacesContext context, UIComponent component, Object value) {
...
}

public Object getAsObject(FacesContext context, UIComponent component, String value) {
...
}
```

Para utilizar un conversor propio dentro de un componente se tiene que utilizar la acción ```f:converter``` utilizando su identificador:

```xml
  <f:converter converterId="conversor"/>
```

Un conversor puede decidir elevar una excepción si una determinada conversión no puede realizarse, y añadir los mensajes de error oportunos al contexto.
