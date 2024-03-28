# 14. Spring: ApplicationContext (3) - Recursos

_23-04-2012_ _Juan Mellado_

El contexto de Spring ofrece características de acceso a bajo de nivel de recursos más allá de las ofrecidas por la clase ```java.net.URL``` estándar de Java.

## 14.1. Interface

Toda la gestión de recursos recae en la interface ```Resource```, que extiende ```InputStreamSource```, y actúa como un _wrapper_ de ```java.net.URL```:

```java
public interface InputStreamSource {

  InputStream getInputStream() throws IOException;
}

public interface Resource extends InputStreamSource {

  boolean exists();

  boolean isOpen();

  URL getURL() throws IOException;

  File getFile() throws IOException;

  Resource createRelative(String relativePath) throws IOException;

  String getFilename();

  String getDescription();
}
```

El método ```getInputStream``` es el que se encarga de localizar y dar acceso físico a los recursos, aunque es responsabilidad de la aplicación cerrar el _stream_ una vez haya terminado de operar con él.

La disponibilidad de los métodos ```getURL``` y ```getFile``` dependen del tipo y ubicación del recurso concreto solicitado. Por ejemplo, una petición HTTP a un servidor remoto normalmente no se puede resolver como una instancia de ```java.io.File```.

Spring recomienda utilizar la interface ```Resource``` en las aplicaciones, como cualquier otra librería de utilidades, aunque ello suponga añadir una dependencia por código con Spring.

## 14.2. Implementaciones

Spring proporciona varias implementaciones para la interface ```Resource``` que pueden instanciarse directamente para acceder a un recurso:

- ```UrlResource```: La implementación genérica que funciona como ```java.net.URL```. Admite una URL válida como parámetro con cualquier tipo de prefijo habitual: ```file:```, ```http:```, ```ftp:```, etc.

- ```ClassPathResource```: Una implementación específica para acceder a recursos que se encuentran dentro del _classpath_.

- ```FileSystemResource```: Una implementación específica para acceder a recursos dentro de un sistema de ficheros.

- ```ServletContextResource```: Una implementación específica para acceder a recursos de una aplicación web relativos a su directorio de despliegue.

- ```InputStreamResource```: Una implementación específica para tratar con recursos que se encuentren ya abiertos y accesibles a través de ```InputStream```.

- ```ByteArrayResource```: Una implementación específica para acceder a un _array_ de _bytes_ que se encuentre ya disponible.

## 14.3. Contexto

El contexto implementa el método ```getResource``` que permite acceder a los recursos. El tipo de objeto concreto devuelto por este método será una de las implementaciones indicadas en el apartado anterior en función de la URL dada:

```java
Resource resource = context.getResource("http://www.inmensia.com");
Resource resource = context.getResource("file://c:/fichero.txt");
Resource resource = context.getResource("classpath://jdbc.properties");
```

De la forma acostumbrada, Spring proporciona una interface ```Aware``` que cualquier _bean_ que quiera acceder a ese aspecto del contexto puede implementar:

```java
public class Cargador implements ResourceLoaderAware  {

  private ResourceLoader resourceLoader;

  @Override
  public void setResourceLoader(ResourceLoader resourceLoader) {
    this.resourceLoader = resourceLoader;
  }

  public Resource cargar(String recurso) {
    return this.resourceLoader.getResource(recurso);
  }
)
```

Aunque por supuesto se puede utilizar cualquier otro medio de inyección de dependencias de Spring para obtener acceso al cargador de recursos, como la anotación ```@Autowired``` por ejemplo.

## 14.4. Carga por Dependencia

Si un recurso sólo se va cargar de forma puntual en un _bean_, no hace falta tener acceso a la interface, basta con inyectar el recurso directamente en una propiedad del _bean_.

```java
public class Cargador {

  private Resource fichero;

  public void setFichero(Resource fichero) {
    this.fichero = fichero;
  }
...
```

```xml
<bean id="cargador" class="com.empresa.Cargador">
  <property name="fichero" value="classpath:fichero.txt"/>
</bean>
```

## 14.5. Wildcards

Spring deja también especificar _wildcards_ (caracteres comodines), pero sólo tiene sentido en algunos casos concretos.

Por ejemplo, usando ```classpath*:``` cuando se quiere instanciar un ```ApplicationContext``` a partir de varios ficheros de configuración ubicados en el _classpath_, y evitar así tener que especificarlos uno a uno:

```text
classpath*:applicationContext.xml
```

O usando ```**``` cuando se quiere buscar en subdirectorios a partir de una raíz dada:

```text
classpath:com/empresa/proyecto/**/fichero.txt
```

No todas las combinaciones son válidas, ni dan el resultado esperado, además de ciertos problemas de compatibilidad. Lo mejor es revisar la documentación oficial en cada caso.
