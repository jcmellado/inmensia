# 20. Spring: AOP (1) - Introducción

_24-04-2012_ _Juan Mellado_

Spring permite utilizar las características de la programación orientada a aspectos (AOP) en las aplicaciones. No obstante, antes de empezar a aplicar este concepto es recomendable estar familiarizado con su filosofía de funcionamiento y los nuevos términos que introduce.

Spring tiene ampliamente documentado como integrar AspectJ, una librería que implementa AOP para su uso dentro de una aplicación. Se considera un tema avanzado, y la documentación oficial de referencia entra en bastantes detalles, mucho más de los que normalmente se necesitan para implementar una aplicación que no tenga requerimientos especiales al respecto, y no se comentará en esta serie de artículos.

## 20.1. AOP

Esta serie está dedicada a Spring, no a la programación orientada a aspectos. No obstante, se realizará una rápida definición de los términos que usa Spring respetando el nombre original en inglés para facilitar las consultas que se quieran hacer posteriormente en la documentación oficial:

- **Aspect**: Característica que se distribuye transversalmente en una aplicación, como un sistema de gestión de transacciones por ejemplo.

- **Join Point**: Punto concreto de la ejecución de un programa, como la ejecución de un método por ejemplo.

- **Advice**: Código a inyectar en un punto concreto de la ejecución de un programa, es decir, en un _join point_. Los _advices_ se pueden configurar para que se ejecuten antes (_before_), después (_after_), sólo si se produce una excepción (_after throwing_), al terminar siempre (_after finally_), o sustituyendo por completo el código original (_around_).

- **Pointcut**: Expresión regular que acota áreas de interés dentro del código de un programa donde aplicar _advices_.

- **Introduction**: Técnica que permite añadir propiedades o métodos a una clase existente sin modificar su código. También recibe el nombre de _Inter-type Declaration_ (ITD).

- **Weaving**: Técnica que permite modificar el _bytecode_ de una clase. Utilizada para generar nuevas clases a partir de las originales, pero en las que se encuentran físicamente inyectados los _advices_ en sus correspondientes _join points_.

## 20.2. @AspectJ

```@AspectJ``` es el nombre con el que se conoce el estilo de declaración basado en anotaciones que da acceso a las características de programación orientada a aspectos con AspectJ. Para utilizarlo con Spring es necesario realizar la configuración pertinente.

En el fichero ```pom.xml``` hay que incluir la dependencia con el _jar_ de AspectJ que contiene las anotaciones, y con el _jar_ de cglib que permite modificar las clases dinámicamente:

```xml
<properties>
...
  <aspectj-version>1.5.4</aspectj-version>
  <cglib-version>2.2.2</cglib-version>
</properties>

<dependencies>
...
  <dependency>
    <groupId>aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>${aspectj-version}</version>
  </dependency>
  <dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>${cglib-version}</version>
  </dependency>
<dependencies>
```

Y en el fichero de configuración XML hay que añadir el _namespace_ de AOP, y la etiqueta ```aop:aspectj-autoproxy``` que activa el uso de la programación orientada a aspectos con anotaciones:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">

  <aop:aspectj-autoproxy/>

...

</beans>
```

## 20.3. Aspect

Para probar la correcta configuración del sistema, se puede declarar un _aspect_ utilizando la etiqueta ```@Aspect``` sobre una clase ordinaria:

```java
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class Espia {
}
```

Estas clases no son detectadas por Spring con la opción de escaneado automático, ya que no es un componente, por lo que es necesario declararlas en el fichero (o clase) de configuración como cualquier otro tipo de _bean_:

```xml
<bean id="espia" class="com.empresa.Espia"/>
```

Si se quiere que se detecten automáticamente estas clases, se puede aplicar la anotación ```@Controller``` sobre ellas, o cualquier otra anotación de tipo componente, aunque fuerza un poco la semántica.
