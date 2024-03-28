# 25. JSF (7) - Listeners

_02-06-2012_ _Juan Mellado_

JSF notifica a las aplicaciones la ocurrencia de determinados eventos, y los _listeners_ son los objetos receptores de dichos eventos.

Se distinguen dos tipos de eventos. Por una parte los eventos de aplicación, asociados a componentes, como la pulsación de un botón por ejemplo. Y por otra parte los eventos de sistema, que ocurren en determinados momentos del tiempo a lo largo del ciclo de vida de un componente o una aplicación JSF.

## 25.1. Eventos

Todos los eventos heredan de la clase ```FacesEvent```. Las clases principales son ```SystemEvent``` y ```PhaseEvent```, para los eventos del ciclo de vida de la aplicación, ```ComponentSystemEvent``` para los eventos del ciclo de vida de los componentes, ```ActionEvent``` para las acciones ejecutadas sobre los componentes de la interface de usuario, y ```ValueChangeEvent``` para los cambios de valor de los componentes.

A partir de ```SystemEvent``` se definen los siguientes tipos de eventos:

- ```PostConstructApplicationEvent```: Emitido justo después de arrancar la aplicación.

- ```PreDestroyApplicationEvent```: Emitido justo antes de parar la aplicación.

- ```ExceptionQueuedEvent```: Emitido al producirse una excepción inesperada.

A partir de ```ComponentSystemEvent``` se definen los siguientes tipos de eventos:

- ```PostAddToViewEvent```: Emitido justo después de que un componente se añade a una vista.

- ```PostConstructViewMapEvent```: Emitido justo después de que se cree una vista.

- ```PostRestoreStateEvent```: Emitido justo después de que un componente se restaure.

- ```PostValidateEvent```: Emitido justo después de que un componente se valide.

- ```PreDestroyViewMapEvent```: Emitido justo antes de que una vista se destruya.

- ```PreRenderComponentEvent```: Emitido justo antes de que un componente se renderice.

- ```PreRenderViewEvent```: Emitido justo antes de que una vista se renderice.

- ```PreValidateEvent```: Emitido justo antes que un componente se valide.

## 25.2. SystemEventListener

Los _listeners_ de eventos del sistema son clases que implementan la interface ```SystemEventListener```. Esta interface obliga a implementar el método ```processEvent``` que recibe el evento, y el método ```isListenerForSource``` que permite decidir si se quieren recibir eventos en función de su origen:

```java
public class Sistema implements SystemEventListener {

  @Override
  public void processEvent(SystemEvent event){
  ...
  }

  @Override
  public boolean isListenerForSource(Object source) {
  ...
  }
...
```

Los _listeners_ que capturan el ciclo de vida de la aplicación se declaran en el fichero ```faces-config.xml``` indicando el nombre de la clase que los implementan y las clases de eventos que quieren recibir:

```xml
<application>
  <system-event-listener>
    <system-event-listener-class>
      com.company.listeners.Sistema
    </system-event-listener-class>
    <system-event-class>
      javax.faces.event.PostConstructApplicationEvent
    </system-event-class>
  </system-event-listener>
</application>
```

## 25.3. ComponentSystemEventListener

Los eventos del ciclo de vida de los componentes se capturan con _listeners_ que implementan la interface ```ComponentSystemEventListener```. Esta interface es igual que ```SystemEventListener```, pero carece del método ```isListenerForSource```, ya que estos _listeners_ se registran sobre un componente determinado:

```java
public class Componente implements ComponentSystemEventListener {

  @Override
  public void processEvent(SystemEvent event){
    ...
  }
...
```

Se puede utilizar las anotaciones ```@ForListener``` y ```@ForListeners``` para indicar las clases de los eventos concretos que se quieren capturar.

En una vista se asocian a un componente utilizando la acción ```f:event```, indicando en el atributo ```type``` el tipo de evento concreto que se quiere capturar.

```xml
  <f:event type="preRenderView" listener="#{componente.escuchador}"/>
```

## 25.4. PhaseListener

Los _listeners_ de eventos de fase son similares a los de sistema, pero tienen una mayor granularidad. Son clases que implementan la interface ```PhaseListener``` y reciben los eventos antes y después de cada fase, al tiempo que permiten definir que fase quieren procesar:

```java
public class CustomPhaseListener implements PhaseListener{
 
  @Override
  public void afterPhase(PhaseEvent event) {
    ...
  }

  @Override
  public void beforePhase(PhaseEvent event) {
    ...
  }

  @Override
  public PhaseId getPhaseId() {
    ...
  }
...
```

En una vista se pueden asociar a un componente utilizando la acción ```f:phaseEvent```.

## 25.5. ActionListener

Un _listener_ de acciones sobre un componente es una clase que implementa la interface ```ActionListener```. Esta interface obliga a implementar el método ```processAction``` que recibe un objeto con el detalle del evento, particularmente con una referencia al origen del mismo:

``` java
public class Accion implements ActionListener {

  @Override
  public void processAction(ActionEvent event) throws AbortProcessingException {
  ...
  }
...
```

Para utilizar el _listener_ dentro de un componente hay que hacer referencia al mismo indicando el nombre de la clase:

```xml
<h:form id="formulario">
  <h:commandLink id="enlace" action="enlazar">
    <h:outputText value="#{usuario.nombre}"/>
    <f:actionListener type="com.company.listeners.Accion"/>
  </h:commandLink>
  ...
```

Una alternativa que evita tener que declarar una clase es definir el método en un _bean_:

```java
@ManagedBean
public class Usuario {

  public void accion(ActionEvent event) throws AbortProcessingException {
  ...
  }
...
```

Y referenciar a dicho método directamente con el atributo ```actionListener```:

```xml
<h:inputText ... value="#{usuario.nombre}" actionListener="#{usuario.accion}"/>
```

De forma alternativa, se puede utilizar la acción ```f:setPropertyActionListener```, que es una facilidad que ofrece JSF para establecer el valor de un _bean_ cuando ocurre una acción sin necesidad de crear un método:

```xml
<f:setPropertyActionListener target="#{requestScope.seleccionado}"
                             value="#{articulo}"/>
```

## 25.6. ValueChangeListener

Dentro de los eventos de aplicación se distingue un caso especial correspondiente al cambio de un valor de un componente, como cuando se cambia el valor de un ```checkbox```.

Un _listener_ de eventos por cambio de valor es una clase que implementa la interface ```ValueChangeListener```. Esta interface obliga a implementar el método ```processValueChange``` que recibe un objeto con el evento que contiene el valor anterior y el nuevo:

```java
public class Cambio implements ValueChangeListener{

  @Override
  public void processValueChange(ValueChangeEvent event) throws AbortProcessingException {
  ...
  }
...
```

Para utilizar el _listener_ con un componente hay que añadir la acción ```f:valueChangeListener``` con el nombre de la clase que lo implementa:

```xml
<h:inputText id="nombre" value="#{usuario.nombre}" onchange="submit()">
  <f:valueChangeListener type="com.company.listeners.Cambio"/>
</h:inputText>
```

Una alternativa que evita tener que declarar una clase es definir el método en un _bean_:

```java
@ManagedBean
public class Usuario {

  public void cambio(ValueChangeEvent event) throws AbortProcessingException {
  ...
  }
...
```

Y referenciar a dicho método directamente con el atributo ```valueChangeListener```:

```xml
<h:inputText ... value="#{usuario.nombre}" valueChangeListener="#{usuario.cambio}"/>
```

## 25.7. Binding

De igual forma que lo explicado para validadores y conversores, JSF permite aplicar la capacidades de _binding_ a los _listeners_. Para ello hay que declarar una propiedad dentro de un _bean_ con el mismo tipo del _listener_:

```java
@ManagedBean
public class Usuario {

  ValueChangeListener escuchador = new ValueChangeListener() {

    public void processValueChange(ValueChangeEvent arg0) throws AbortProcessingException {
    ...
    }
  };

  public ValueChangeListener getEscuchador() {
    return escuchador;
  }
...
```

Y hacer referencia a la propiedad del _bean_ dentro del componente:

```xml
<h:inputText id="nombre" value="#{usuario.nombre}">
  <f:valueChangeListener binding="#{usuario.escuchador}"/>
</h:inputText>
```
