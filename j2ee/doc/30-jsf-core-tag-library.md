# 30. JSF (12) - Core Tag Library

_05-06-2012_ _Juan Mellado_

Esta librería de acciones proporciona funciones de uso general.

- ```f:verbatim```: Renderiza el contenido tal cual se suministra. Útil para añadir bloques HTML propios que no deben procesarse por JSP.

    ```xml
    <f:verbatim><div class="especial"></f:verbatim>
    ...
    <f:verbatim></div></f:verbatim>
    ```

- ```f:view```: Contenedor principal de todos los componentes de una vista. Útil para añadir el tipo de contenido, el juego de caracteres, o la configuración regional.

    ```xml
    <f:view contentType="text/html" encoding="UTF-8">
    ...
    </f:view>
    ```

- ```f:subview```: Contenedor de todos los componentes incluidos a través de una acción ```jsp:include``` o ```c:import```. Es obligatorio utilizarlo cuando se quiere incluir una página JSP o JSF utilizando las acciones ```jsp:include``` o ```c:import```.

    ```xml
    <f:subview>
      <jsp:include page="un.jsp"/>
    </f:subview>
    ```

- ```f:viewParam```: Captura el valor de los parámetros HTTP de una petición. Utilizado conjuntamente con ```f:metadata```.

    ```xml
    <f:metadata>
      <f:viewParam name="idUsuario" value="#{bean.idUsuario}"/>
    </f:metadata>
    ```

- ```f:metadata```: Define un bloque de metainformación para una vista. Utilizado en conjunción con ```f:viewParam```.

- ```f:localBundle```: Carga un recurso de mensajes de texto de forma local a una vista y lo expone como un ```Map``` de (clave, texto). Los ficheros de mensajes tienen que estar ubicados dentro de una carpeta del directorio ```resources``` dentro del _war_ de la aplicación.

    ```xml
    <f:loadBundle basename="i18n" var="mensajes"/>
    ```

    Si se quieren cargar los mensajes de forma global para toda una aplicación se pueden declarar en el fichero ```faces-config.xml```.

    ```xml
    <application>
      <resource-bundle>
        <base-name>i18n</base-name>
        <var>mensajes</var>
      </resource-bundle>
    </application>
    ```

    Para utilizar los mensajes hay que utilizar el nombre de la variable indicada y la clave del mensaje correspondiente.

    ```xml
    <h:outputText value="#{mensajes.clave}"/>
    ```

- ```f:facet```: Añade una característica a un componente. Dicha característica actúa como "rol" de dicho componente para distinguirlo del resto.

    ```xml
    <h:dataTable value="#{usuarios}" var="usuario">
      <h:column>
        <f:facet name="header">
          <h:outputText value="Nombre"></h:outputText>
        </f:facet>
        <h:outputText value="#{usuario.nombre}"></h:outputText>
      </h:column>
    </h:dataTable>
    ```

- ```f:validateRequired```: Fuerza a que un componente se le tenga que introducir un valor.

    ```xml
    <h:inputText ... value="#{usuario.password}">
      <f:validateRequired/>
    </h:inputText>
    ```

- ```f:validateLength```: Añade un validador de tipo ```LengthValidator``` a un componente. Comprueba que la longitud de la cadena de texto que almacena el valor esté dentro de un rango.

    ```xml
    <h:inputText ... value="#{cliente.nombre}">
      <f:validateLength minimum="3" maximum="25"/>
    </h:inputText>
    ```

- ```f:validateDoubleRange```: Añade un validador de tipo ```DoubleRangeValidator``` a un componente. Comprueba que la variable numérica que almacena el valor esté dentro de un rango.

    ```xml
    <h:inputText ... value="#{cliente.saldo}">
      <f:validateDoubleRange minimum="0.01"/>
    </h:inputText>
    ```

- ```f:validateLongRange```: Añade un validador de tipo ```LongRangeValidator``` a un componente. Comprueba que la variable numérica que almacena el valor, o la cadena de texto que representa un número, esté dentro de un rango.

    ```xml
    <h:inputText ... value="#{cliente.cuentas}">
      <f:validateLongRange minimum="1"/>
    </h:inputText>
    ```

- ```f:validateRegex```: Añade un validador de tipo ```RegExValidator``` a un componente. Comprueba que la cadena de texto que almacena el valor case con un patrón dado.

    ```xml
    <h:inputText ... value="#{cliente.alias}">
      <f:validateRegex pattern="[a-zA-Z0-9]+"/>
    </h:inputText>
    ```

- ```f:validator```: Añade un validador a un componente que llama a un método concreto dado.

    ```xml
    <h:inputText ... value="#{usuario.nombre}" validator="#{usuario.validateNombre}">
    ```

- ```f:validateBean```: Delega la validación de un componente a un validador de tipo ```BeanValidator``` personalizado.

- ```f:convertNumber```: Añade un convertidor de tipo ```NumberConverter``` a un componente. Especifica como convertir un valor numérico desde y hacía su representación en formato de texto.

    ```xml
    <h:outputText value="#{pedido.total}">
      <f:convertNumber currencySymbol="€" type="currency"/>
    </h:outputText>
    ```

- ```f:convertDateTime```: Añade un convertidor de tipo ```DateTimeConverter``` a un componente. Especifica como convertir un valor de tipo fecha desde y hacía su representación en formato de texto.

    ```xml
    <h:outputText value="#{persona.nacimiento}">
      <f:convertDateTime pattern="dd/MM/yyyy"/>
    </h:outputText>
    ```

- ```f:converter```: Añade un convertidor específico a un componente.

    ```xml
    <h:selectOneMenu value="#{cliente.ciudad}">
      <f:converter converterId="ciudadConverter"></f:converter>
      <f:selectItems value="#{pais.ciudades}"></f:selectItems>
    </h:selectOneMenu>
    ```

- ```f:actionListener```: Añade un _listener_ de acción a un componente.

    ```xml
    <h:form id="formulario">
      <h:commandLink id="enlace" action="enlazar">
        <h:outputText value="#{usuario.nombre}"/>
        <f:actionListener type="com.company.listeners.Accion"/>
      </h:commandLink>
    ...
    ```

- ```f:valueChangeListener```: Añade un _listener_ por cambio de valor a un componente.

    ```xml
    <h:inputText id="nombre" value="#{usuario.nombre}">
      <f:valueChangeListener binding="#{usuario.escuchador}"/>
    </h:inputText>
    ```

- ```f:setPropertyActionListener```: Añade un _listener_ especial a un componente cuyo único propósito es establecer el valor de la propiedad de un _bean_ cuando un formulario es enviado al servidor.

    ```xml
    <f:setPropertyActionListener target="#{requestScope.seleccionado}"
                                value="#{articulo}"/>
    ```

- ```f:event```: Añade un _listener_ de eventos del ciclo de vida de un componente.

    ```xml
    <f:event type="preRenderView" listener="#{componente.escuchador}"/>
    ```

- ```f:phaseListener```: Añade un _listener_ de eventos de ciclo de vida de fase a una vista.

    ```xml
    <f:phaseListener binding="#{usuario.escuchador}"/>
    ```

- ```f:ajax```: Añade una acción Ajax a un componente. Esta acción permite que se ejecute código en el servidor sin tener que recargar la página. Los atributos principales son ```execute``` y ```render``` que especifican listas de componentes, separadas por espacio, que tienen que ejecutarse en el servidor al realizar la llamada y recibir la respuesta respectivamente.

    ```xml
    <h:commandButton value="Comprobar">
      <f:ajax render="resultado"/>
    </h:commandButton>
    <h:outputText id="resultado" value="#{bean.resultado}"/>
    ```

- ```f:attribute```: Añade un atributo a un componente.

    ```xml
    <h:commandButton">
      <f:attribute name="value" value="Botón"/>
    </h:commandButton>
    ```

    El código anterior es equivalente a la siguiente declaración utilizando la forma clásica de escribir los atributos:

    ```xml
    <h:commandButton" value="Botón"/>
    ```

    También puede utilizarse para añadir atributos de forma genérica y que puedan ser utilizados desde los métodos de los _beans_.

    ```xml
    <h:commandButton actionListener="#{bean.escuchador}">
      <f:attribute name="nombre" value=valor"/>
    </h:commandButton>
    ```

    Desde un _listener_, por ejemplo, puede accederse a los atributos con la siguiente cadena:

    ```java
    event.getComponent().getAttributes().get("nombre");
    ```

- ```f:param```: Añade un parámetro genérico a un componente. Se utiliza en conjunción con otras acciones.

    ```xml
    <h:outputFormat value="Bienvenida al {0}">
      <f:param value="convento"/>
    </h:outputFormat>
    ```

- ```f:selectItem```: Representa un elemento dentro de una lista. Se utiliza en conjunción con elementos ```select``` de HTML.

    ```xml
    <h:selectOneRadio value="#{usuario.sexo}">
      <f:selectItem itemValue="1" itemLabel="Hombre"/>
      <f:selectItem itemValue="2" itemLabel="Mujer"/>
    </h:selectOneRadio>
    ```

- ```f:selectItems```: Representa una lista de elementos. Se utiliza en conjunción con elementos ```select``` de HTML.

    ```xml
    <h:selectOneMenu ...>
      <f:selectItems value="#{paleta.colores}"/>
    </h:selectOneMenu>
    ```
