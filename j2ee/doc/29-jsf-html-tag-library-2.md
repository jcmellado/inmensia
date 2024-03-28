# 29. JSF (11) - HTML Tag Library (Continuación)

_05-06-2012_ _Juan Mellado_

- ```h:commandButton```: Envía los datos de un formulario al servidor. Equivale a un elemento ```<input>``` en HTML, donde el atributo ```type``` puede tener el valor ```submit```, ```reset``` o ```image```.

    ```xml
    <h:commandButton type="submit" value="Enviar" action="envio"/>
    ```

- ```h:button```: Equivale a un elemento ```<input type="button">``` en HTML.

    ```xml
    <h:button value="Comprobar"/>
    ```

- ```h:message```: Muestra un mensaje de error asociado a un componente sobre el que se ha aplicado la configuración regional. Su uso principal es definir el estilo con el que debe mostrarse. Utiliza un elemento ```<span>``` de HTML para encapsular el mensaje.

    ```xml
    <h:message for="nombre" style="color: red;"/>
    ```

- ```h:messages```: Muestra varios mensajes de error a los que se le ha aplicado la configuración regional. Su uso principal es definir el estilo con el que deben mostrarse. Utiliza un elemento ```<span>``` para encapsular cada mensaje de forma individual.

    ```xml
    <h:messages style="color: red;"/>
    ```

- ```h:outputText```: Muestra una línea de texto plano.

    ```xml
    <h:outputText value="#{usuario.nombre}"/>
    ```

- ```h:outputFormat```: Muestra un mensaje de texto al que se le pueden pasar parámetros. No utiliza ningún elemento HTML para encapsular el mensaje.

    ```xml
    <h:outputFormat value="Bienvenida al {0}">
      <f:param value="convento"/>
    </h:outputFormat>
    ```

    Los parámetros se apoyan en la acción ```param``` de la librería ```core``` que debe incluirse a través del _namespace_ ```xmlns:f="http://java.sun.com/jsf/core"```.

- ```h:graphicImage```: Imagen. Equivale a un elemento ```<img>``` en HTML.

    ```xml
    <h:graphicImage value="resources/images/logo.png"/>
    ```

- ```h:commandLink```: Enlace a otra página o elemento de la página actual que genera una acción. Equivale a un elemento ```<a href>``` en HTML.

    ```xml
    <h:commandLink value="Liga Mágica"/>
    ```

- ```h:outputLink```: Enlace a otra página o elemento de la página actual que no genera ninguna acción. Equivale a un elemento ```<a>``` en HTML.

    ```xml
    <h:outputLink value="Liga Ordinaria"/>
    ```

- ```h:link```: Equivale a un elemento ```<a>``` en HTML.

    ```xml
    <h:link value="Login Anónimo" outcome="login" >
      <f:param name="usuario" value="anonimo"/>
    </h:link>
    ```

- ```h:panelGrid```: Organiza los componentes de una forma tabular. Equivale a maquetar utilizando los elementos ```<table>```, ```<tr>``` y ```<td>``` en HTML.

    ```xml
    <h:panelGrid columns="2">
    ...
    </h:panelGrid>
    ```

- ```h:dataTable```: Muestra los objetos de una colección organizados de forma tabular. Equivale a un elemento ```<table>``` en HTML.

    ```xml
    <h:dataTable value="#{pedido.lineas}"
                var="linea"
                styleClass="tabla"
                headerClass="cabecera"
                rowClasses="impar,par">
      <h:column>#linea.articulo</h:column>
      <h:column>#linea.precio</h:column>
    </h:dataTable>
    ```

- ```h:column```: Columna de datos en una tabla HTML. Se usa en conjunción con ```h:dataTable```.

## 29.1. Atributos

Asociados a cada acción hay una serie de atributos. Por una parte los estándares correspondientes a cada elemento HTML, y por otra parte los definidos por JSF:

- ```id```: Identificador de componente. Si no se especifica JSF genera uno automáticamente.

    ```xml
    <h:inputText id="nombre" ...
    ```

- ```value```: Valor asociado a un componente.

    ```xml
    <h:outputText value="#{pedido.total}" ...
    ```

- ```binding```: Propiedad de un _bean_ al que se asocia el valor de un componente.

    ```xml
    <h:selectBooleanCheckbox binding="#{usuario.sexo}" ...
    ```

- ```immediate```: Especifica cuando se tiene que ejecutar los eventos, y aplicar validadores y conversores.

    ```xml
    <h:commandLink id="continuar" immediate="true" ...
    ```

- ```rendered```: Especifica una condición que debe cumplirse para que un componente se renderice.

    ```xml
    <h:commandLink id="pagar" rendered="#{pedido.total > 0}" ...
    ```

- ```style```: Estilos CSS indicados directamente sobre un componente.

    ```xml
    <h:message for="nombre" style="color: red;" ...
    ```

- ```styleClass```: Referencia a una clase CSS especificada en una hoja de estilos.

    ```xml
    <h:dataTable id="tablero" styleClass="estilizado" ...
    ```
