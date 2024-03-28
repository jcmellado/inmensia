# 4. Spring: Beans (2) - Declaración

_19-04-2012_ _Juan Mellado_

En este artículo se siguen viendo más detalles de la declaración y uso de los _beans_. Recordando siempre que todos ellos son instanciados y gestionados por el contexto de Spring, produciéndose así la consabida inversión de control, al encargarse él de esas labores en vez de la aplicación.

## 4.1. Herencia

Si se tienen varios _beans_ que comparten definiciones comunes, Spring permite utilizar un _bean_ abstracto a modo de plantilla para evitar tener que repetir mucho código.

El _bean_ utilizado como plantilla tiene que tener activo el atributo ```abstract```. Y los _beans_ que quieran utilizarlo como base para su propia definición deben referenciarlo a través del atributo ```parent```.

```xml
<bean id="motocicleta" abstract="true">
  <property name="manillar" value="1"/>
  <property name="ruedas" value="2"/>
</bean>

<bean id="sidecar" class="..." parent="motocicleta">
  <property name="ruedas" value="3"/>
</bean>
```

Se siguen los principios de herencia habituales, por lo que las propiedades del padre son heredadas por los hijos y sobreescritas si se redefinen.

## 4.2. Referencias

En el artículo anterior se vieron algunas formas en las que un _bean_ puede referenciar a otro. Como por ejemplo utilizando el "atributo" ```ref```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService" ref="mexicanFood"/>
</bean>
```

Un método alternativo es utilizar la "etiqueta" ```ref```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService">
    <ref bean="mexicanFood"/>
  </property>
</bean>
```

Esto permite referenciar cualquier _bean_ instanciado desde cualquier fichero XML de configuración. No obstante, se puede acotar la búsqueda a sólo los _beans_ declarados dentro del fichero utilizando el atributo ```local```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService">
    <ref local="mexicanFood"/>
  </property>
</bean>
```

Una opción algo distinta es utilizar ```idref```, que sirve para referenciar el "nombre" de un _bean_. Es decir, para cargar la propiedad con el nombre del _bean_ en vez de con el objeto al que referencia. Su uso es un tanto limitado, pero presenta la peculiaridad de que Spring es capaz de validar el nombre en el momento de hacer el despliegue, en vez de provocar un error en tiempo de ejecución.

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodServiceName">
    <idref bean="mexicanFood"/>
  </property>
</bean>
```

Comentar que Spring es capaz naturalmente de detectar referencias circulares y elevar una excepción en dicho caso.

## 4.3. Dependencias

Cuando se tiene un _bean_ que depende de otro, sin necesidad de tener una propiedad que haga referencia a él, se puede utilizar el atributo ```depends-on``` para que Spring lo tenga en cuenta a la hora de inicializar e inyectar las dependencias.

Aunque parezca algo rebuscado, esto es algún común en algunas librerías, que requieren que se ejecute su constructor estático antes de poder utilizarlas.

```xml
<bean id="aplicacionBean" depends-on="libreriaBean"/>
```

## 4.4. Beans Internos

Si un _bean_ va a utilizarse de forma puntual dentro de otro, no hace falta declararlo de forma global. Se puede declarar como un _bean_ interno de manera anónima, sin que ni siquiera necesite que se le asigne un identificador:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService">
    <bean class="com.empresa.zoo.MexicanFoodServiceImpl"/>
  </property>
</bean>
```

## 4.5. Propiedades Internas

Se pueden inicializar propiedades internas indicando la ruta completa en el nombre. Por ejemplo, si un _bean_ tiene una propiedad ```auditorio```, y esta propiedad a su vez tiene una propiedad ```capacidadMaxima```, se le puede asignar un valor directamente utilizando ```auditorio.capacidadMaxima``` como nombre:

```xml
<property name="auditorio.capacidadMaxima" value="250">
```

Se puede descender por la cadena utilizando la notación punto tantas veces como sea necesario.

## 4.6. Colecciones

Spring soporta también la instanciación e inicialización de colecciones. Es decir, es capaz de crear un objeto ```List```, ```Map```, ```Set```, e incluso ```Properties```, y rellenarlo con una serie de valores.

Imaginemos que nuestro zoológico tiene una lista de direcciones de correo de contacto:

```java
public class ZooServiceImpl implements ZooService{

  private Map<String, String> emails;

  public void setEmails(Map<String, String> emails) {
    this.emails = emails;
  }
...
```

En vez de poner las direcciones de correo en el código, se pueden poner en la configuración y dejar que Spring las inicialice.

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl">
  <property name="emails">
     <map>
       <entry key="tickets" value="tickets@zoo.com"/>
       <entry key="tours" value="tours@zoo.com"/>
       <entry key="manager" value="manager@zoo.com"/>
      </map>
  </property>
...
```

Spring soporta ```list```, ```set```, ```map``` y ```props```. Y también tiene algunas características más avanzadas como la mezcla (```merging```) de colecciones para algunas configuraciones más complejas donde se pudiera llegar a necesitar.

## 4.7. Nulos

Se debe utilizar la etiqueta ```null``` para que una propiedad tome valor nulo:

```xml
<property name="hazmeNulo"><null/></property>
```

## 4.8. p-name y c-name

Una de las principales quejas que se hacen cuando se usa Spring es que los ficheros XML tienden a crecer bastante. Para intentar paliar este problema Spring permite utilizar una notación abreviada definida en dos _namespaces_ llamados ```p-name```, para las propiedades, y ```c-name```, para los constructores.

Veamos un ejemplo usando ```p-name``` para inicializar dos propiedades, por una parte una cadena de texto con el nombre del zoológico, y por otra parte una referencia a otro _bean_ con el servicio de comidas:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl"
      p:nombreZoo="Wild Park"
      p:foodService-ref="mexicanFood"/>
```

Esta nomenclatura más compacta evita incluir las etiquetas ```property```, permitiendo definir las propiedades dentro de la propia etiqueta ```bean```. Notar que para referenciar a otro _bean_ se utiliza la terminación ```-ref```.

De igual forma, el _namespace_ ```c-name``` permite abreviar la declaración de los constructores al definir los parámetros dentro de la propia etiqueta ```bean```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl"
      c:nombreZoo="Wild Park"
      c:foodService-ref="mexicanFood"/>
```

Si los nombres de los parámetros del constructor no están disponibles, porque no se compiló en modo _debug_, se pueden referenciar por su posición, con un guión bajo antepuesto para que el XML resultante sea válido:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl"
      c:_0="Wild Park"
      c:_1-ref="mexicanFood"/>
```

Los _beans_ son una de las piezas claves dentro de Spring, motivo por el que se pueden declarar de formas tan distintas. Más información en el siguiente artículo de esta serie.
