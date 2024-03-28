# 5. Spring: Beans (3) - Inicialización Perezosa y Autowiring

_20-04-2012_ _Juan Mellado_

Se examinan más posibilidades que ofrece Spring a los desarrolladores para la gestión de _beans_, como la inicialización perezosa, o el descubrimiento automático de dependencias, entre otras.

## 5.1. Tipos de Beans

Por lo general se da por supuesto que todos los _beans_ que instancia Spring implementan automáticamente el patrón _Singleton_. Sin embargo esto no es del todo cierto. Lo correcto es decir que ese es el funcionamiento por defecto, pero se pueden especificar otras formas de gestionar el ciclo de vida de los _beans_. De hecho, Spring permite incluso definir ciclos de vida personalizados por parte de las aplicaciones.

Spring soporta cinco tipos distintos de _beans_ a través del atributo ```scope```:

- ```singleton```: Valor por defecto. Una única instancia del _bean_.

- ```prototype```: Instancia tantos _beans_ como se quiera con las características descritas. Es decir, no se crea un objeto al inicializar el contexto, sino cada vez que se requiere.

- ```request```: Instancia _beans_ que tienen vigencia sólo durante el tiempo de vida de una petición HTTP. Válido sólo para aplicaciones _web_.

- ```session```: Instancia _beans_ que tienen vigencia sólo durante el tiempo de vida de una sesión HTTP. Válido sólo para aplicaciones _web_.

- ```globalSession```: Instancia _beans_ que tienen vigencia sólo durante el tiempo de una sesión global de un _portlet_. Válido sólo para aplicaciones _web_.

La opción por defecto no requiere mayor explicación. No obstante, la opción ```prototype``` se puede entender mejor con el siguiente ejemplo, donde se definen dos _beans_ que hacen referencia a otro que usa el valor ```prototype```:

```xml
<bean id="zoo1" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService" ref="mexicanFood"/>
</bean>

<bean id="zoo2" class="com.empresa.zoo.ZooServiceImpl">
  <property name="foodService" ref="mexicanFood"/>
</bean>
  
<bean id="mexicanFood" class="com.empresa.zoo.MexicanFoodServiceImpl"
      scope="prototype"/>
```

A diferencia de los ejemplos vistos hasta ahora, el _bean_ ```mexicanFood``` no será un _Singleton_, y por lo tanto cada uno de los _beans_, ```zoo1``` y ```zoo2```, recibirá una instancia distinta de ```mexicanFood```.

El resto de opciones para el atributo ```scope``` se verán cuando se traten los temas específicos relacionados con el desarrollo para la _web_.

## 5.2. Inicialización Perezosa

Por defecto, cuando Spring lee un fichero de configuración instancia automáticamente todos los _beans_ definidos en el mismo. Si se quiere evitar que algún _bean_ sea instanciado automáticamente se puede utilizar el atributo ```lazy-init```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl" lazy-init="true"/>
```

De esta forma el _bean_ se instanciará la primera vez que sea usado, y no cuando arranque la aplicación. No obstante, si el _bean_ "perezoso" forma parte de las dependencias de otro _bean_, Spring lo instanciará ignorando el atributo para asegurar que todo el sistema se inicializa correctamente.

Spring permite también cambiar la política de instanciación por defecto, permitiendo que ningún _bean_ se instancie al arrancar si no se desea:

```xml
<beans default-lazy-init="true">
...
</beans>
```

## 5.3. Autowiring

Una forma de reducir el tamaño de los ficheros XML de configuración es utilizar la capacidad de _autowiring_ de Spring. Esta característica permite a Spring detectar automáticamente las dependencias de un _bean_ con otros, instanciando e inyectando por sí solo todos los objetos necesarios.

Spring soporta cuatro formas de _autowiring_ a través del atributo ```autowire```:

- ```no```: Opción por defecto, obliga a especificar las dependencias de forma manual utilizando ```ref```.

- ```byName```: Fuerza a Spring a resolver dependencias por nombre. Es decir, si el _bean_ tiene un _setter_ llamado ```setAuditorio``` buscará un _bean_ con nombre ```auditorio```.

- ```byType```: Fuerza a Spring a resolver dependencias en base a los tipos de las propiedades del _bean_. Es decir, Spring examinará el tipo de las propiedades del _bean_ y buscará algún _bean_ que tenga el mismo tipo.

- ```constructor```: Fuerza a Spring a resolver dependencias en base a los tipos de los parámetros del constructor del _bean_.

Recuperemos por un momento el código del ejemplo original del zoológico con el servicio de comidas:

```java
public class ZooServiceImpl implements ZooService{

  private FoodService foodService;

  public ZooServiceImpl(){
  }

  public void setFoodService(FoodService foodService){
    this.foodService = foodService;
  }
  ...
}

public class MexicanFoodServiceImpl implements FoodService{

  public MexicanFoodServiceImpl(){
  }
  ...
}
```

Y ahora consideremos la siguiente configuración, donde se definen dos _beans_ sin aparente relación entre ellos, pero en la que se activa el _autowiring_ por tipo en el _bean_ ```zoo```:

```xml
<bean id="zoo" class="com.empresa.zoo.ZooServiceImpl" autowire="byType"/>

<bean id="mexicanFood" class="com.empresa.zoo.MexicanFoodServiceImpl"/>
```

Cuando se instancie el _bean_ ```zoo```, Spring extraerá automáticamente sus propiedades, buscará otros _beans_ que tengan el mismo tipo que cada una de ellas, y si los encuentra los inyectará como dependencias. En la práctica quiere decir que la propiedad ```foodService``` de ```zoo``` hará referencia al _bean_ ```mexicanFood```, ¡como por arte de magía!, ya que implementa la interface ```FoodService```.

Si se quiere excluir un _bean_ de toda la infraestructura de _autowiring_ se puede utilizar el atributo ```autowire-candidate```:

```xml
<bean id="mexicanFood" class="..." autowire-candidate="false"/>
```

Notar que esto no provocaría una excepción, sino que simplemente la propiedad ```foodService``` del _bean_ ```zoo``` quedaría a nulo al no encontrar ningún candidato válido.

Más adelante se retomará el tema del _autowiring_, ya que esta no es la única forma de activarlo, ni probablemente la más recomendable.
