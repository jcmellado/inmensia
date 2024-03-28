# 16. Spring: Binding, Converter y Formatting

_23-04-2012_ _Juan Mellado_

El proceso de _Binding_ normalmente se relaciona con la asociación entre la información introducida por el usuario y el correspondiente modelo de datos de negocio de la aplicación. Aunque la acepción de este término es un poco muy amplio y se extiende a la asociación entre dos modelos distintos cualesquiera.

Spring utiliza esta característica de forma interna en muchos de sus componentes, para así poder "obrar su magia", y siguiendo su filosofía de diseño habitual, la expone para que las aplicaciones pueden hacer también uso de ella. Es un tema un tanto avanzado que no se presupone que vaya a necesitarse normalmente en el desarrollo de una aplicación corriente.

Relacionados con el _Binding_ se encuentran los servicios de conversión de tipos y formateo de valores.

## 16.1. Binding

Las clases con las que se suele trabajar para representar entidades de negocio normalmente siguen el estándar JavaBean, que determina que tengan un constructor sin parámetros, y métodos _getters_ y _setters_ para cada una de sus propiedades.

La interface ```BeanWrapper```, y su implementación ```BeanWrapperImpl```, permiten acceder a las propiedades de cualquier tipo de clase que siga el estándar JavaBean. Y lo que es más importante, permite definir _listeners_ que reciban notificaciones cuando se produzcan cambios en los valores de las propiedades, sin cambiar la clase original, que es precisamente la esencia del _binding_.

Supongamos el siguiente esquema de clases:

```java
public class Autor {

  private String nombre;

  public String getNombre() {
    return nombre;
  }
  
  public void setNombre(String nombre) {
    this.nombre = nombre;
  }
}

public class Libro {

  private String titulo;
  
  private Autor autor;

  public String getTitulo() {
    return titulo;
  }
  
  public void setTitulo(String titulo) {
    this.titulo = titulo;
  }
    
  public Autor getAutor() {
    return autor;
  }

  public void setAutor(Autor autor) {
    this.autor = autor;
  }
}
```

La interface ```BeanWrapper``` es en realidad una cadena de interfaces extendida que incluyen el uso de ```PropertyEditors```, para la conversión de objetos en cadenas de texto, y viceversa, y que además permite definir conversores propios para cada aplicación. Utilizándola es posible manipular una instancia de cualquiera de este tipo de clases, para obtener o establecer valores a sus propiedades, sin llamar explícitamente a sus métodos _getters_ y _setters_:

```java
Libro libro = new Libro();

BeanWrapper libroWrapper = new BeanWrapperImpl(libro);
   
libroWrapper.setPropertyValue("titulo", "Don Quijote");

Autor autor = new Autor();
   
BeanWrapper autorWrapper = new BeanWrapperImpl(autor);
   
PropertyValue value = new PropertyValue("nombre", "Cervantes");
   
autorWrapper.setPropertyValue(value);

libroWrapper.setPropertyValue("autor", autorWrapper.getWrappedInstance());

System.out.println(libroWrapper.getPropertyValue("autor.nombre"));
```

## 16.2. Converter

Relacionado con el _binding_, Spring ofrece un servicio de conversión de tipos a través de la interface ```Converter```:

```java
public interface Converter<S, T> {

  T convert(S source);
}
```

Es un sistema de conversión genérico, como el sistema de validación, y el propósito es que cualquier clase que implemente esta interface sirva para convertir un tipo en otro:

```java
public class CigarroToHumo implements Converter<Cigarro, Humo> {

  @Override
  public Humo convert(Cigarro cigarro) {
    return cigarro.fumar();
  }
}
```

La implementación debe ser segura para su ejecución en entornos multihilos, y elevar una excepción ```IllegalArgumentException``` en caso de que la conversión no pueda llevarse a cabo.

Para implementaciones más avanzadas se puede usar la interface ```ConverterFactory```, que retorna conversores, ```GenericConverter```, que permite especificar varios pares de tipos entre los que realizar la conversión, o ```ConditionalGenericConverter```, que permite realizar la conversión de manera condicional en función de la existencia de una anotación o método determinado, por ejemplo.

Spring ofrece también un API unificado para conversiones que puede inyectarse y utilizarse de forma segura en cualquier _bean_:

```xml
<bean id="conversionService"
      class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```

## 16.3. Formatting

Otro servicio ofrecido por Spring y relacionado con los anteriores es el de formateo de valores, utilizado tradicionalmente para las conversiones de todo tipo de objetos, desde o hacia una cadena de texto, para su representación de cara a un usuario.

Cualquier clase que implemente la interface ```Formatter``` puede actuar como un formateador:

```java
public interface Printer<T> {

    String print(T fieldValue, Locale locale);
}

public interface Parser<T> {

    T parse(String clientValue, Locale locale) throws ParseException;
}

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

El método ```print``` tiene que devolver el objeto de tipo ```T``` formateado para el cliente. Y el método parse tomar un valor del cliente y convertirlo a un objeto de tipo ```T```.

La implementación debe ser segura para su ejecución en entornos multihilos, y elevar una excepción ```IllegalArgumentException```, o ```ParseException```, en caso de que la conversión no pueda llevarse a cabo.

Spring ofrece también la posibilidad de usar anotaciones sobre cada propiedad de un _bean_, de forma que al formatear un objeto esté ya especificado el formato a aplicar.

```java
public class Libro {

    @DateTimeFormat(iso=ISO.DATE)
    private Date fechaPublicacion;
...
```
