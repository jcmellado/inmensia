# 22. JSF (4) - Navegación

_02-06-2012_ _Juan Mellado_

Las reglas de navegación determinan qué página se tiene que visualizar en cada momento, normalmente en respuesta a la pulsación de un botón o enlace.

JSF implementa una regla de navegación de manera implícita, pero permite definir otro tipo de reglas de forma explícita por parte de las aplicaciones.

## 22.1. Navegación Implícita

La regla implícita consiste en buscar una página con el mismo nombre que el atributo ```action``` del componente que inició la navegación.

Sirva de ejemplo un botón con la siguiente definición:

```xml
<h:form>
  <h:commandButton value="Enviar" action="envio"/>
</h:form>
```

Al pulsarse el botón, JSF intentará navegar automáticamente hacia una página llamada ```envio.xhtml```. Si no se encuentra se navegará a la página actual.

Un detalle importante a tener en cuenta es que el botón tiene que estar contenido dentro de un formulario para que se ejecute la acción correctamente.

## 22.2. Navegación Explícita

Lo normal dentro de una aplicación es que la navegación se dirija hacia una página u otra en función del resultado de aplicar algún tipo de lógica de negocio.

JSF permite que los métodos de los _beans_ llamados en respuesta a acciones por parte de los clientes retornen el nombre de la página a la que tienen que dirigirse. Si un método retorna una cadena de texto, dicho valor retornado se interpreta como el nombre de la página destino de la acción. Si el método retorna ```null```, o es un método que no retorna valor (```void```), se interpreta que la página destino es la actual.

Sirva de ejemplo un botón con la siguiente definición:

```xml
<h:form>
  <h:commandButton value="Comprar" action="#{tienda.comprar}"/>
</h:form>
```

Dentro del método comprar del _bean_ ```tienda``` se puede evaluar si la acción de compra puede realizarse, y navegar hacia una página de envío o una de error:

```java
@ManagedBean
public class Tienda {

  public String comprar() {
    return correcto ? "envio" : "error";
  }
...
```

Si se retorna ```"envio"``` se navegará hacia la página ```envio.xhtml```, y en caso contrario hacia ```error.xhtml```.

## 22.3. Configuración

Para evitar tener que especificar el nombre concreto de las páginas dentro de los métodos, se puede hacer que estos retornen un valor indicativo del resultado de la acción, en vez del nombre de la página. El valor retornado puede ser cualquiera, pero normalmente se utilizan las cadenas ```success```, ```failure```, ```login``` y ```no results```.

Continuando con el ejemplo del apartado anterior, habría que modificar el método del _bean_ de la siguiente forma:

```java
public String comprar() {
  return correcto ? "success" : "failure";
}
```

Y configurar dentro del fichero ```faces-config.xml``` las reglas de navegación, partiendo de la página donde se lanza la acción, e indicando la página a la que se debe navegar en función del resultado de la acción:

```xml
<navigation-rule>
  <from-view-id>/tienda.xhtml</from-view-id>
  <navigation-case>
    <from-action>#{tienda.comprar}</from-action>
    <from-outcome>success</from-outcome>
    <to-view-id>/exito.xhtml</to-view-id>
  </navigation-case>
  <navigation-case>
    <from-action>#{tienda.comprar}</from-action>
    <from-outcome>failure</from-outcome>
    <to-view-id>/fracaso.xhtml</to-view-id>
  </navigation-case>
</navigation-rule>
```
