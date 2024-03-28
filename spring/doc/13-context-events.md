# 13. Spring: ApplicationContext (2) - Eventos

_22-04-2012_ _Juan Mellado_

Spring expone el patrón ```Observable``` para que las aplicaciones puedan responder a eventos generados por el contexto, e incluso implementar una gestión personalizada de eventos propios.

## 13.1. Eventos

Para que un _bean_ reciba eventos tiene que implementar la interface ```ApplicationListener```. Los eventos que se reciben son por defecto de la clase ```ApplicationEvent```, aunque se permiten definir eventos propios.

Spring detecta automáticamente cuando un _bean_ implementa la interface y le notifica por defecto los siguientes tipos de eventos referidos al contexto:

- ```ContextRefreshedEvent```: Enviado cuando se inicializa el contexto o se llama a su método ```refresh```, y todos los _beans_ han sido detectados y activados.

- ```ContextStartedEvent```: Enviado cuando se llama al método ```start``` del contexto, y todos los _beans_ que implementan una interface ```Lifecycle``` para el control de ciclo de vida han recibido una notificación de arranque.

- ```ContextStoppedEvent```: Enviado cuando se llama al método ```stop``` del contexto, y todos los _beans_ que implementan una interface ```Lifecycle``` para el control de ciclo de vida han recibido una notificación de parada.

- ```ContextClosedEvent```: Enviado cuando se llama al método ```close``` del contexto, todos los _beans_ de tipo _Singleton_ han sido destruidos, y el contexto ya no puede recuperarse.

- ```RequestHandledEvent```: Este evento aplica sólo a entornos _web_, para su gestión por parte de los _servlets_, y es enviado cuando un petición HTTP (_request_) ha sido completada.

## 13.2. Event

Los eventos propios se crean extendiendo de la clase base ```ApplicationEvent``` y añadiendo los atributos que se quieran:

```java
public class Quejido extends ApplicationEvent {

  private String motivo;

  public Quejido(Object source, String motivo) {
    super(source);

    this.motivo = motivo;
  }
}
```

## 13.3. Listener

Para que una clase adopte el rol de _listener_ basta con que implemente la interface ```ApplicationListener```, tantas veces como tipos distintos de mensajes quiera recibir:

```java
public class Cotilla implements ApplicationListener<Quejido> {

  @Override
  public void onApplicationEvent(Quejido quejido) {
  }
}
```

Y declarar el _bean_ correspondiente en el fichero de configuración:

```xml
<bean id="cotilla" class="com.empresa.Cotilla"/>
```

## 13.4. Publisher

Para que un _bean_ publique eventos se le tiene que inyectar una dependencia de tipo ```ApplicationEventPublisher```. Y la forma recomendada es utilizar una de las interfaces de tipo ```Aware``` que ofrece Spring, concretamente la interface ```ApplicationEventPublisherAware```:

```java
public class Quejica implements ApplicationEventPublisherAware {

  private ApplicationEventPublisher publisher;

  @Override
  public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
    this.publisher = publisher;
  }

  public void queja(String motivo) {
    Quejido quejido = new Quejido(this, motivo);

    this.publisher.publishEvent(quejido);
  }
}
```

Y declarar el _bean_ correspondiente en el fichero (o clase) de configuración:

```xml
<bean id="quejica" class="com.empresa.Quejica"/>
```

Spring detecta automáticamente cuando un _bean_ implementa la interface, e inyecta directamente la dependencia, de forma que pueda utilizarla para publicar eventos.

Finalmente, se puede probar el sistema con las siguientes líneas de código:

```java
Cotilla cotilla = context.getBean(Cotilla.class);

Quejica quejica = context.getBean(Quejica.class);
quejica.queja("¡Como está el país!");
```
