# 23. JSF (5) - Validadores

_02-06-2012_ _Juan Mellado_

JSF proporciona una serie de validadores estándar a través de sus librerías de acciones, a la par que permite definir validadores propios por parte de las aplicaciones. Su propósito es validar la información introducida en el cliente, habitualmente a través de un formulario.

## 23.1. Estándar

Los validadores estándar se encuentran dentro de la librería ```core``` de JSF, y se usan en conjunción con los componentes HTML de la librería ```html```.

- ```f:validateLength```: Valida que la longitud de una cadena de texto esté dentro de un rango.

- ```f:validateDoubleRange```: Valida que un valor numérico esté dentro de un rango.

- ```f:validateLongRange```: Valida que un valor numérico, o una cadena de texto convertible a número, esté dentro de un rango.

- ```f:validateRegEx```: Valida que un valor case con una expresión regular.

- ```f:validateRequired```: Valida que un elemento tenga valor. Es equivalente al atributo ```required``` de HTML.

- ```f:validateBean```: Registra un validador personalizado para el _bean_.

En el siguiente ejemplo se muestra un formulario para la introducción de un nombre que debe tener al menos cinco caracteres de longitud:

```xml
<h:form id="formulario">
  <h:outputLabel for="nombre" value="Nombre:"/>
  <h:inputText id="nombre" value="#{usuario.nombre}">
    <f:validateLength minimum="5"/>
  </h:inputText>
  <h:commandButton type="submit" value="Enviar" action="enviar"/>
  <h:message for="nombre" style="color: red;"/>
</h:form>
```

Si el valor no pasa la validación se muestra un mensaje en el elemento ```h:message``` con el siguiente detalle:

```text
formulario:nombre: Error de validación: el largo es inferior que el mínimo permitido de '5'
```

Un aspecto que resulta un tanto confuso a veces es que JSF inicializa las cadenas de texto con una cadena vacía por defecto, y no a nulo. Si se quiere cambiar este comportamiento se puede añadir un parámetro al contexto a través del fichero ```web.xml``` de la siguiente forma:

```xml
<context-param>
  <param-name>
    javax.faces.INTERPRET_EMPTY_STRING_SUBMITTED_VALUES_AS_NULL
  </param-name>
  <param-value>true</param-value>
</context-param>
```

## 23.2. Anotaciones

JSF permite utilizar el conjunto de anotaciones estándar definido por la especificación JavaBean Validation. Estas anotaciones se aplican a las propiedades de los _bean_ y permiten indicar las condiciones que deben de cumplir para considerarse válidas.

Algunas de las anotaciones disponibles:

- ```@AssertFalse```: Valida que el valor es ```false```.

- ```@AssertTrue```: Valida que el valor es ```true```.

- ```@DecimalMax```: Valida que el valor es menor o igual que el indicado en la anotación.

- ```@DecimalMin```: Valida que el valor es mayor o igual que el indicado en la anotación.

- ```@Digits```: Valida que el valor tiene como mucho la cantidad de enteros y decimales indicados en la anotación.

- ```@Future```: Valida que el valor es una fecha a futuro.

- ```@Max```: Valida que el valor es un entero menor o igual que el indicado en la anotación.

- ```@Min```: Valida que el valor es un entero mayor o igual que el indicado en la anotación.

- ```@NotNull```: Valida que el valor no es nulo.

- ```@Null```: Valida que el valor es nulo.

- ```@Past```: Valida que el valor es una fecha del pasado.

- ```@Pattern```: Valida que el valor casa la expresión regular indicada en la anotación.

- ```@Size```: Valida que la longitud del valor está dentro de un rango indicado en la anotación.

En el siguiente ejemplo se muestra un _bean_ con anotaciones de validación sobre sus propiedades:

```java
@ManagedBean
public class Usuario {

  @NotNull
  @Size(min=5, max=25)
  private String nombre;

  @NotNull
  @Past
  private Date nacimiento;

  @AssertTrue
  private Boolean activo;
...
```

No obstante, para utilizar estas anotaciones es necesario disponer de una librería (_jar_) que las implemente, como por ejemplo la desarrollada por Hibernate, y que puede obtenerse añadiendo la siguiente dependencia en el fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>4.3.0.Final</version>
</dependency>
```

## 23.3. Binding

Otra forma de definir validadores por parte de una aplicación es utilizar las facilidades de _binding_ que ofrece JSF.

De forma general, el _binding_ es el proceso de asociar el valor de un componente a una propiedad de un _bean_, de manera que cuando cambia el valor del componente se refleja en el _bean_, y cuando cambia en el _bean_ se refleja en el componente.

JSF extiende el concepto de _binding_ al permitir asociar un validador a una propiedad de un _bean_:

```xml
<h:inputText id="nombre" value="#{usuario.nombre}">
  <f:validateLength binding="#{usuario.validador}"/>
</h:inputText>
```

En buena lógica, la propiedad del _bean_ debe ser del mismo tipo que el validador:

```java
@ManagedBean
public class Usuario {

  private LengthValidator validador = new LengthValidator();

  public LengthValidator getValidador() {
    validador.setMinimum(10);
    return validador;
  }
...
```

Lo interesante del código es que las propiedades del validador pueden hacerse variar de forma dinámica dentro del _bean_. Por ejemplo, el tamaño mínimo puede hacerse que cambie en función de alguna lógica determinada, y dicho cambio afectaría inmediatamente al componente.

## 23.4. Validator

Otra posibilidad que ofrece JSF a la hora de definir validadores es crear una clase que implemente la interface ```Validator```. Los validadores implementados de esta forma se anotan con ```@FacesValidator```, o se definen dentro del fichero ```faces-config.xml```. Normalmente tienen que implementar otras interfaces como ```StateHolder``` e incluirse dentro de una TLD para poder ser utilizados como una acción más dentro de una vista.

Una alternativa más sencilla que ofrece JSF es implementar un método propio dentro de un _bean_, sin que la clase del _bean_ tenga que heredar de ninguna otra ni implementar ninguna interface. Basta con que el método que se quiera utilizar como validador tenga los mismos parámetros que admite la interface definida por ```Validator```:

```java
@ManagedBean
public class Usuario {

  public void validateNombre(
    FacesContext context,
    UIComponent toValidate,
    Object value
  ) {
    String nombre = (String)value;

    if (nombre.equals("admin")) {
      ((UIInput)toValidate).setValid(false);

      FacesMessage message = new FacesMessage("¡Tramposo!");
      context.addMessage(toValidate.getClientId(context), message);
...
```

El valor a validar se recibe en el parámetro ```value```, el componente siendo validado en ```toValidate```, y el contexto en ```context```. Si el valor no pasa la validación se debe cambiar a ```false``` la propiedad ```valid``` del componente y opcionalmente añadir un mensaje al contexto.

Para utilizar un método de validación propio se debe utilizar el atributo ```validator``` del componente:

```xml
<h:inputText ... value="#{usuario.nombre}" validator="#{usuario.validateNombre}">
```
