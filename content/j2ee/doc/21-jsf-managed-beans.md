# 21. JSF (3) - Managed Beans

_02-06-2012_ _Juan Mellado_

Una de las características principales de JSF es que permite a las aplicaciones _web_ trabajar con clases POJO (Plain Old Java Object) en vez de _servlets_. Estas clases, al ser gestionadas por JSF, reciben el nombre de _managed beans_.

## 21.1. @ManagedBean

La anotación ```@ManagedBean``` permite denotar una clase como un _bean_:

```java
@ManagedBean
public class Hello {

  private String saludo = "Hola Bean";

  public void setSaludo(String saludo) {
    this.saludo = saludo;
  }

  public String getSaludo() {
    return saludo;
  }
}
```

Esta anotación admite un parámetro llamado ```name``` que permite indicar el nombre por el que se puede referenciar al _bean_ en Facelets dentro de una vista. Si no se indica, por defecto es igual el nombre de la clase con la primera letra en minúscula:

```jsp
#{hello.saludo}
```

JSF se encarga de instanciar los _beans_ cuando es necesario, aunque por defecto la inicialización es perezosa, se crean sólo cuando se necesitan por primera vez. No obstante, la anotación ```@ManagedBean``` admite un segundo parámetro llamado ```eager``` que permite que se instancien al arrancar la aplicación:

```java
@ManagedBean(eager=true)
public class Hello {
...
```

No obstante, este tipo de inicialización sólo tiene sentido en _beans_ que tienen un ámbito global. Es decir, que se encuentran definidos a nivel de aplicación.

## 21.2. Scope

La forma de definir el ámbito de un _bean_ es utilizar una de las siguientes anotaciones:

- ```@ApplicationScoped```: Ámbito a nivel de aplicación, persistente durante toda la vida de la misma.

- ```@SessionScoped```: Ámbito a nivel de sesión, persistente dentro del conjunto de peticiones de una sesión HTTP.

- ```@ViewScoped```: Ámbito a nivel de vista, persistente dentro de las peticiones realizadas a una vista concreta.

- ```@RequestScoped```: Ámbito a nivel de petición, persistente durante una petición HTTP.

- ```@NoneScoped```: Sin ámbito definido, se crea un _bean_ nuevo cada vez que se referencia. Utilizado cuando un _bean_ usa a otro.

- ```@CustomScoped```: Ámbito no estándar, definido por la aplicación. Raramente utilizado.

## 21.3. @ManagedProperty

La anotación ```@ManagedProperty``` representa una forma adecuada de inyectar dependencias entre _beans_, de forma similar a la anotación ```@Inject```.

Sirva de ejemplo un típico _bean_ DAO definido a nivel de aplicación:

```java
@ManagedBean(name="libroDao")
@ApplicationScoped
public class LibroDao {
...
```

Para que otro _bean_ haga referencia a él basta con añadir la anotación ```@ManagedProperty``` sobre la propiedad de la clase que se quiere contenga la referencia:

```java
@ManagedBean(name="dependiente")
@SessionScoped
public class Dependiente {

  @ManagedProperty(value="#{libroDao}")
  private LibroDao libroDao;

  public void setLibroDao(LibroDao libroDao) {
    this.libroDao = libroDao;
  }
...
```

JSF instanciará la clase DAO una vez por aplicación, y la inyectará en cada instancia de la clase dependiente creada por sesión.

Notar que el parámetro ```value``` admite una expresión EL, no es sólo una simple cadena de texto. Esto permite por ejemplo hacer referencia a un _bean_ concreto evaluado en tiempo de ejecución en función de una condición, no es sólo una simple referencia estática por nombre.

## 21.4. Configuración

De forma alternativa, en vez de utilizar anotaciones, los _beans_ se pueden definir en el fichero de configuración ```faces-config.xml```:

```xml
<managed-bean>
  <managed-bean-name>hello</managed-bean-name>
  <managed-bean-class>com.company.beans.Hello</managed-bean-class>
  <managed-bean-scope>application</managed-bean-scope>
  <managed-property>
    <property-name>saludo</property-name>
    <property-class>java.lang.String</property-class>
    <value>Hola Config</value>
  </managed-property>
</managed-bean>
```

La sintaxis es muy intuitiva y la configuración no debería representar ninguna dificultad. En la documentación de referencia pueden encontrarse los detalles necesarios para realizar configuraciones que impliquen el uso de colecciones, enumerados, u otro tipo de construcciones más elaboradas.
