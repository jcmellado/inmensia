# 28. JSF (10) - HTML Tag Library

_05-06-2012_ _Juan Mellado_

La librería de acciones ```html``` contiene etiquetas equivalentes a cada uno de los componentes HTML estándar.

- ```h:body```: Equivale a un elemento ```<body>``` en HTML.

    ```xml
    <h:body>
    ...
    </h:body>
    ```

- ```h:head```: Equivale a un elemento ```<head>``` en HTML.

    ```xml
    <h:head>
      <title>Título<title>
    <h:head>
    ```

- ```h:outputStyleSheet```: Enlaza una hoja de estilo. Equivale a un elemento ```<link>``` en HTML.

    ```xml
    <h:outputStylesheet library="css" name="estilos.css"/>
    ```

    El atributo ```library``` indica que ```estilos.css``` debe encontrarse en el directorio ```resources/css```.

- ```h:outputScript```: Equivale a un elemento ```<script>``` en HTML.

    ```xml
    <h:outputScript library="js" name="script.js"/>
    ```

    El atributo ```library``` indica que ```script.js``` debe encontrarse en el directorio ```resources/js```.

- ```h:panelGroup```: Grupo de componentes. Utiliza un elemento ```<div>``` o ```<span>``` de HTML para encapsular los componentes.

    ```xml
    <h:panelGroup id="contenido">
    ...
    </h:panelGroup>
    ```

- ```h:form```: Formulario de entrada de datos. Equivale a un elemento ```<form>``` en HTML.

    ```xml
    <h:form>
    ...
    </h:form>
    ```

- ```h:outputLabel```: Utilizado para asociar una etiqueta a un campo de entrada de datos de un formulario. Equivale a un elemento ```<label>``` en HTML.

    ```xml
    <h:outputLabel for="nombre" value="Nombre:"/>
    ```

- ```h:inputText```: Utilizado para introducir una línea de texto. Equivale a un elemento ```<input type="text">``` en HTML.

    ```xml
    <h:inputText id="nombre" value="#{libro.titulo}" required="true"/>
    ```

- ```h:inputTextarea```: Utilizado para introducir un texto con varias líneas. Equivale a un elemento ```<textarea>``` en HTML.

    ```xml
    <h:inputTextarea id="sinopsis" value="#{libro.sinopsis}"/>
    ```

- ```h:inputSecret```: Utilizado para la introducción de información sensible, como contraseñas, para que no se muestre lo que se teclea. Equivale a un elemento ```<input type="password">``` en HTML.

    ```xml
    <h:inputSecret id="clave" value="#{usuario.clave}"/>
    ```

- ```h:inputHidden```: Utilizado para introducir variables escondidas dentro de una página. Equivale a un elemento ```<input type="hidden">``` en HTML.

    ```xml
    <h:inputHidden id="secreto" value="#{oculto}"/>
    ```

- ```h:selectOneListbox```: Permite seleccionar un elemento de una lista. Equivale a un elemento ```<select>``` en HTML.

    ```xml
    <h:selectOneListbox value="#{cosecha.fecha}">
      <f:selectItem itemValue="1" itemLabel="2000"/>
      <f:selectItem itemValue="2" itemLabel="2001"/>
      <f:selectItem itemValue="3" itemLabel="2002"/>
    </h:selectOneListbox>
    ```

    Las listas se apoyan en la acción ```selectItem``` de la librería ```core``` que debe incluirse a través del _namespace_ ```xmlns:f="http://java.sun.com/jsf/core"```.

- ```h:selectManyListbox```: Permite seleccionar múltiples elemento de una lista. Equivale a un elemento ```<select>``` en HTML con el atributo ```multiple``` con valor ```multiple```.

    ```xml
    <h:selectManyListbox value="#{cosecha.fecha}">
      <f:selectItem itemValue="1" itemLabel="2000"/>
      ...
    </h:selectManyListbox>
    ```

- ```h:selectOneMenu```: Permite seleccionar un elemento de un desplegable. Equivale a un elemento ```<select>``` en HTML con el atributo ```size``` igual a ```1```.

    ```xml
    <h:selectOneMenu value="#{cosecha.fecha}">
      <f:selectItem itemValue="1" itemLabel="2000"/>
      ...
    </h:selectOneMenu>
    ```

- ```h:selectManyMenu```: Permite seleccionar múltiples elementos de una lista. Equivale a un elemento ```<select>``` en HTML con el atributo ```multiple``` con valor ```multiple``` y ```size``` igual a ```1```.

    ```xml
    <h:selectManyMenu value="#{cosecha.fecha}">
      <f:selectItem itemValue="1" itemLabel="2000"/>
      ...
    </h:selectManyMenu>
    ```

- ```h:selectBooleanCheckbox```: Checkbox. Equivale a un elemento ```<input type="checkbox">``` en HTML.

    ```xml
    <h:selectBooleanCheckbox value="#{chat.ausente}"/>
    ```

- ```h:selectManyCheckbox```: Lista de checkboxes. Equivale a una lista de elementos ```<input type="checkbox">``` en HTML.

    ```xml
    <h:selectManyCheckbox value="#{habitacion.extras}">
      <f:selectItem itemValue="1" itemLabel="Caja fuerte"/>
      <f:selectItem itemValue="2" itemLabel="Secador"/>
    </h:selectManyCheckbox >
    ```

- ```h:selectOneRadio```: Permite seleccionar un elemento de un conjunto de opciones. Equivale a un elemento ```<input type="radio">``` en HTML.

    ```xml
    <h:selectOneRadio value="#{usuario.genero}">
      <f:selectItem itemValue="1" itemLabel="Hombre"/>
      <f:selectItem itemValue="2" itemLabel="Mujer"/>
    </h:selectOneRadio>
    ```
