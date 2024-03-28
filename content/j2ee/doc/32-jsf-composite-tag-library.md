# 32. JSF (14) - Composite Tag Library

_05-06-2012_ _Juan Mellado_

Esta librería de acciones proporciona facilidades para la construcción de componentes.

- ```composite:interface```: Declara la interface de un componente.

    ```xml
    <composite:interface>
      <composite:attribute name="nombre"/>
    </composite:interface>
    ```

- ```composite:attribute```: Define un atributo en la interface de un componente. Los atributos pueden ser variables o métodos.

    ```xml
    <composite:interface>
      <composite:attribute name="texto" required="false"/>
      <composite:attribute name="valor" type="java.lang.String"/>
      <composite:attribute name="metodo" method-signature = "java.lang.String accion()"/>
    </composite:interface>
    ```

- ```composite:facet```: Define un _facet_ (rol) en la interface de un componente.

    ```xml
    <composite:interface>
      <composite:facet name="cabecera"/>
      <composite:facet name="contenido"/>
    </composite:interface>
    ```

- ```composite:valueHolder```: Expone una interface ```ValueHolder``` para una serie de elementos internos de un componente. Las páginas que utilicen el componente pueden utilizar la interface para variar el comportamiento de dichos elementos, inyectando conversores por ejemplo.

    ```xml
    <composite:interface>
      <composite:valueHolder name="estatico" targets="form:etiqueta"/>           
    </composite:interface>
    ```

- ```composite:editableValueHolder```: Expone una interface ```EditablesValueHolder``` para una serie de elementos internos de un componente. Las páginas que utilicen el componente pueden utilizar la interface para variar el comportamiento de dichos elementos inyectando validadores por ejemplo.

    ```xml
    <composite:interface>
     <composite:editableValueHolder name="editable" targets="form:editor"/>
    </composite:interface>
    ```

- ```composite:actionSource```: Expone una interface ```ActionSource``` para una serie de elementos internos de un componente. Las páginas que utilicen el componente pueden utilizar la interface para variar el comportamiento de dichos elementos, inyectando _listeners_ por ejemplo.

    ```xml
    <composite:interface>
      <composite:actionSource name="accionable" targets="form:boton"/>
    </composite:interface>
    ```

- ```composite:clientBehavior```: Expone una interface ```ClientBehaviorHolder``` para una serie de elementos internos de un componente. Las páginas que utilicen el componente pueden utilizar la interface para variar el comportamiento de dichos elementos, inyectando _listeners_ por ejemplo.

    ```xml
    <composite:interface>
      <composite:clientBehaviour name="accionable" targets="form:boton"/>
    </composite:interface>
    ```

- ```composite:extension```: Permite añadir cualquier tipo de contenido XML que no forma parte de JSF a un elemento declarado en la interface, como por ejemplo _microdata_.

    ```xml
    <html ...
        xmlns:md="...">
    ...
    <composite:interface>
      <composite:attribute name="atributo">
        <composite:extension>
          <md:propiedad>valor</md:propiedad>
        </composite:extension>
      </composite:attribute>
    <composite:interface>
    ```

- ```composite:implementation```: Define la implementación de un componente. Desde EL se puede hacer referencia al propio componente a través la variable ```cc``` (_composite component_), y a los atributos definidos en la interface con su propiedad ```attrs```.

    ```xml
    <composite:implementation>
      <h:outputText value="#{cc.attrs.nombre}"/>
    </composite:implementation>
    ```

- ```composite:renderFacet```: Fuerza que se renderice el _facet_ indicado:

    ```xml
    <composite:implementation>
      <composite:renderFacet name="cabecera"/>
      <composite:renderFacet name="contenido"/>
    </composite:implementation>
    ```

- ```composite:insertChildren```: Establece que el contenido hijo declarado dentro de una página que utiliza el componente debe añadirse a la página final resultante.

    ```xml
    <composite:implementation>
      <div>
        <composite:insertChildren/>
      </div>  
    </composite:implementation>
    ```

    Esta acción resulta de utilidad para añadir contenido a un componente por parte de una página que lo utilice:

    ```xml
    <libreria:componente>
      <outputText value="Apareceré donde esté la etiqueta insertChildren"/>
    </librería:componente>
    ```

- ```composite:insertFacet```: Inserta el contenido de un _facet_:

    ```xml
    <composite:implementation>
      <composite:insertFacet name="cabecera"/>
      <composite:insertFacet name="contenido"/>
    </composite:implementation>
    ```
