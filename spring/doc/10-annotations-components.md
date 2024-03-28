# 10. Spring: Anotaciones (2) - Componentes

_21-04-2012_ _Juan Mellado_

Además de para la inyección de dependencias, las anotaciones pueden utilizarse para que Spring detecte automáticamente las clases que queramos que gestione como _beans_ dentro del propio código fuente de Java, evitando tener que declararlas de manera explícita en ficheros XML.

## 10.1. @Component

Esta anotación sirve para añadir un estereotipo de forma genérica a una clase. Un "estereotipo" es una manera de clasificar las clases a un alto nivel, normalmente de manera semántica.

```java
import org.springframework.stereotype.Component;

@Component
public class Motor {
...
```

La anotación ```@Component``` es genérica y no se espera que se utilice realmente, lo esperado es que usen algunas de las otras anotaciones derivadas de ellas como ```@Repository```, ```@Service``` y ```@Controller```, para las capas de persistencia, servicios, y presentación, respectivamente.

## 10.2. @Named

Esta anotación es similar a la anterior ```@Component```, pero pertenece a la especificación estándar de Java, en vez de a Spring, por lo que requiere añadir la misma dependencia de Maven indicada para la anotación ```@Inject```.

```java
import javax.inject.Named;

@Named
public class Motor {
...
```

## 10.3. Autodetección

Hasta ahora, todos los ejemplos vistos en esta serie han implicado siempre tener que declarar los _beans_ en el fichero ```applicationContext.xml```. Sin embargo, gracias a las anotaciones, las declaraciones se pueden omitir y dejar que Spring detecte automáticamente los _beans_.

Supongamos las siguientes clases de ejemplo de una biblioteca:

```java
public class Libro {
}

public interface LibroDao {

  Libro getLibro();
}

@Repository
public class LibroDaoImpl implements LibroDao {

  @Override
  public Libro getLibro() {
    return new Libro();
  }
}

@Service
public class Biblioteca {

  private LibroDao libroDao;

  @Autowired
  public void setLibroDao(LibroDao libroDao) {
    this.libroDao = libroDao;
  }

  public Libro getLibro() {
    return libroDao.getLibro();
  }
}
```

En este ejemplo hay dos componentes que nos interesa dejar que sea Spring quien los gestione automáticamente. Por una parte el repositorio de libros, y por otra parte el servicio de biblioteca. Para conseguir este propósito se define el fichero ```applicationContext.xml``` de la siguiente forma:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.1.xsd">

  <context:component-scan base-package="com.empresa"/>

</beans>
```

Es decir, se utiliza la etiqueta ```context:component-scan```, con el nombre del paquete donde se encuentran las clases, para dejar que Spring se encargue de escanearlo por si solo, y que extraiga la definición de los _beans_, evitando tener que hacerlo nosotros manualmente.

Para comprobar su correcto funcionamiento se pueden ejecutar las siguientes líneas de código donde se observa como un _bean_ llamado ```biblioteca``` es automáticamente creado por Spring, a la par que todas sus dependencias han sido correctamente inyectadas:

```java
Biblioteca biblioteca = context.getBean("biblioteca", Biblioteca.class);

biblioteca.getLibro();
```

Cuando se usa la etiqueta ```context:component-scan``` para escanear componentes no es necesario incluir también la etiqueta ```context:annotation-config``` para detectar anotaciones, esta última está incluida por defecto en el comportamiento de la primera.

Por último, comentar que si se quieren excluir algunas clases de ser automáticamente detectadas por Spring se pueden establecer filtros en la misma etiqueta usando nombres de clases, interfaces y anotaciones, e incluso expresiones regulares y filtros personalizados.
