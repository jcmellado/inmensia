# 31 - JSF (13) - Facelets Tag Library

_05-06-2012_ _Juan Mellado_

Esta librería de acciones proporciona facilidades para la construcción de componentes y vistas basadas en plantillas.

- ```ui:composition```: Define una disposición de una página basada en una plantilla. El contenido definido fuera de esta acción es ignorado.

    ```xml
    <ui:composition template="plantilla.xhtml">
    ...
    </ui:composition>
    ```

- ```ui:decorate```: Actúa de forma similar a la acción ```ui:composition```, pero sin ignorar el contenido que se encuentra fuera de la acción.

- ```ui:insert```: Especifica un nombre que las páginas cliente deben utilizar para insertar contenido dentro de una plantilla.

    ```xml
    <ui:insert name="contenido"/>
    ```

- ```ui:define```: Define contenido a ser insertado dentro de una plantilla utilizando el nombre indicado dentro de la misma.

    ```xml
    <ui:define name="titulo">Título</ui:define>
    ```

- ```ui:include```: Incluye un fichero dentro de otro. La ruta del fichero indicado es relativa a la del fichero que lo incluye.

    ```xml
    <ui:include src="contenido.xhtml"/>
    ```

- ```ui:param```: Utilizado para pasar valores a plantillas.

    ```xml
    <ui:composition template="plantilla.xhtml">
      <ui:param name="titulo" value="Título"/>
    </ui:composition>
    ```

- ```ui:component```: Define un componente. Actúa de forma similar a la acción ```ui:composition``` pero sin definir una plantilla. El contenido de la acción es añadido directamente al árbol de componentes de la vista. El contenido definido fuera de esta acción es ignorado.

    ```xml
    <ui:component>
      <h:outputText value="${1 + 1}"/>
    </ui:component>
    ```

- ```ui:fragment```: Actúa de forma similar a la acción ```ui:component```, pero sin ignorar el contenido que se encuentra fuera de la acción.

- ```ui:debug```: Añade un componente de depuración a un página. Al pulsar ```shift+control+d``` en el navegador del cliente abre una ventana con información detallada. Admite un atributo que permite cambiar la tecla por defecto.

    ```xml
    <ui:debug hotkey="e"/>
    ```

- ```ui:repeat```: Sustituye el uso de ```h:dataTable``` y ```c:forEach``` cuando se utiliza el atributo ```jsfc``` para definir las vistas.

    ```xml
    <div jsfc="ui:repeat" value="#{libros}" var="libro" ...
    ```

- ```ui:remove```: Usado cuando se utiliza el atributo ```jsfc``` para definir las vistas. Los elementos HTML con esta acción no se tienen en cuenta.

    ```xml
    <div jsfc="ui:remove" ...
    ```
