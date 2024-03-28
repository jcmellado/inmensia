# 12. Spring: ApplicationContext (1) - Internacionalización

_22-04-2012_ _Juan Mellado_

Este artículo muestra las capacidades básicas de internacionalización (_i18n_) que ofrece Spring a través del contexto.

## 12.1. Internacionalización

El contexto de Spring ofrece soporte para la internacionalización de aplicaciones a través de la interface ```MessageSource```. La forma de acceder a ella es declarar un _bean_ que se llame ```messageSource``` en el fichero (o clase) de configuración y que tenga asignada una clase que implemente dicha interface:

```xml
<bean id="messageSource"
      class="org.springframework.context.support.ResourceBundleMessageSource">
  <property name="basename" value="mensajes"/>
</bean>
```

La propiedad ```basename``` permite indicar el nombre del típico fichero ```.properties``` que contiene los mensajes de la aplicación.

Como ejemplo se usará el siguiente fichero ```mensajes.properties``` ubicado en el directorio ```src/main/resources``` del proyecto:

```properties
saludo=Hola cocodrilo
bienvenida=Bienvenida al {0}
```

Para probar que la configuración es correcta se tiene que instanciar un contexto y obtener una referencia a la interface ```MessageSource```:

```java
MessageSource messages =
    new ClassPathXmlApplicationContext("applicationContext.xml");
```

Y con dicha interface tratar de obtener un mensaje almacenado en el fichero ```.properties``` a través del método ```getMessage```, que espera el código de un mensaje, sus argumentos si los tuviera, y el idioma o nulo para especificar un valor por defecto:

```java
String message = messages.getMessage("saludo", null, null);

System.out.println(message);
```

Si todo es correcto, se debe mostrar el mensaje correspondiente:

```text
Hola cocodrilo
```

Para poder recuperar el mismo mensaje en otro idioma habría que preparar un fichero ```.properties``` con el mismo nombre que el original, pero seguido por un guión bajo y el código estándar del idioma deseado. Así, el fichero de mensajes en chino tendría por nombre ```messages_zh.properties```:

```properties
saludo=Welcome Mandarín
bienvenida=My {0}
```

Para probar que la configuración es correcta se debe volver a pedir el mensaje, pero indicando el código del idioma deseado:

```java
String message = messages.getMessage("saludo", null, Locale.CHINESE);

System.out.println(message);
```

Si todo es correcto, se debe mostrar por consola el mensaje correspondiente al código pasado:

```text
Welcome Mandarín
```

Spring soporta también los mensajes dinámicos dando la posibilidad de añadir argumentos a los textos en forma de _array_ de objetos:

```java
String message =
    messages.getMessage("bienvenida", new Object[]{"convento"}, null);

System.out.println(message);
```

Si todo es correcto, se debe mostrar el mensaje correspondiente:

```text
Bienvenida al convento
```

Por último, comentar que Spring ofrece algunas posibilidades más avanzadas de configuración, que permiten por ejemplo cargar los ficheros de mensajes de forma dinámica en tiempo de ejecución.
