# 22. Spring: AOP (3) - Schema

_24-04-2012_ _Juan Mellado_

Además de usando anotaciones, Spring permite utilizar la programación orientada a aspectos definiendo los elementos que intervienen en la misma a través de ficheros de configuración XML.

Cuando se trata de introducir una tecnología como AOP es más sencillo empezar viendo ejemplos sencillos sobre el código fuente, y es por ello que se ha empezado viendo primero las anotaciones, y ahora se expone la configuración XML.

## 22.1. Schema

Para poder utilizar las etiquetas XML hay que incluir el namespace AOP de Spring:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">
...
</beans>
```

## 22.2. Configuración

La configuración de AOP en un fichero de configuración se declara dentro de bloques etiquetados con ```aop:config```, pudiendo existir tantos como se necesiten:

```xml
<aop:config>
...
</aop:config>
```

## 22.2. Aspect

Un _aspect_ se declara utilizando la etiqueta ```aop:aspect```, y haciendo referencia a un _bean_:

```xml
<aop:config>
  <aop:aspect id="espia" ref="espia"/>
</aop:config>

<bean id="espia" class="com.empresa.espia"/>
```

## 22.3. Pointcut

Un _pointcut_ se declara utilizando la etiqueta ```aop:pointcut``` acompañada de una expresión regular:

```xml
<aop:aspect ref="espia">
  <aop:pointcut id="anyConversacion"
                expression="execution(public * conversacion(..))"/>
</aop:aspect>
```

Los _pointcuts_ se pueden combinar utilizando ```or```, ```and```, y ```not```.

## 22.4. Advice

Un _advice_ se declara utilizando la etiqueta correspondiente al tipo de ejecución deseada, una expresión regular o referencia a un _pointcut_, y el nombre del método con la implementación.

Con ```aop:before``` se captura la llamada al método antes de que se produzca:

```xml
<aop:before
  pointcut-ref="anyConversacion"
  method="interceptar"/>
```

Con ```aop:after-returning``` se captura la llamada al método cuando termina correctamente:

```xml
<aop:after-returning
  pointcut-ref="anyConversacion"
  returning="cotilleo"
  method="grabarConversacion"/>
```

Con ```aop:after-throwing``` se captura la llamada al método cuando termina elevando una excepción:

```xml
<aop:after-throwing
  pointcut="execution(public * *(..))"
  throwing="gritito"
  method="grabarGrititos"/>
```

Con ```aop:after``` se captura la llamada al método cuando termina normalmente o elevando una excepción:

```xml
<aop:after
  pointcut-ref="anyConversacion"
  method="interceptar"/>
```

Con ```aop:around``` se puede reemplazar un método completo:

```xml
<aop:around
  pointcut-ref="anyConversacion"
  method="interceptar"/>
```

En todos los casos, las reglas que tienen que seguir los métodos definidos en las etiquetas son las mismas que las indicadas con el uso de anotaciones.
