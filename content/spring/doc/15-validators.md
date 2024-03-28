# 15. Spring: Validadores

_23-04-2012_ _Juan Mellado_

Spring ofrece una librería de utilidades de validación, a la par que permite a una aplicación definir validadores propios, o usar el paquete de anotaciones estándar de Java.

## 15.1. Validator

Un validador es toda clase que implemente la interface ```Validator```:

```java
public interface Validator {

  boolean supports(Class<?> clazz);

  void validate(Object target, Errors errors);
}
```

El método ```supports``` sirve para indicar qué tipo de clases es capaz de validar, y el método ```validate``` para implementar la validación en si misma. ```Errors``` es una interface genérica que define Spring para que las aplicaciones puedan acumular errores.

Supongamos una clase para registrar los datos de un libro:

```java
public class Libro {

  private String titulo;

  private int paginas;
...
```

Y su correspondiente validador:

```java
public class LibroValidator implements Validator {

  @Override
  public boolean supports(Class<?> clazz) {
    return Libro.class.isAssignableFrom(clazz);
  }

  @Override
  public void validate(Object target, Errors errors) {
    ValidationUtils.rejectIfEmptyOrWhitespace(errors, "titulo", "titulo.vacio");

    Libro libro = (Libro)target;

    if (libro.getPaginas() == 0){
      errors.reject("paginas.cero");
    }
    if (libro.getPaginas() < 1){
      errors.reject("paginas.negativo");
    }
  }
}
```

En el validador se aprecia el uso de la clase ```ValidationUtils```, para realizar algunas tareas comunes, y de la anteriormente mencionada ```Errors```, para almacenar los códigos de error, cuyos mensajes de texto asociados se podrán recuperar posteriormente utilizando las capacidades de internacionalización ya vistas en un artículo anterior.

Para probar el validador se puede utilizar un código como el siguiente, que presupone la existencia de un contexto y una configuración adecuada:

```java
Libro libro = new Libro();
   
Validator validator = new LibroValidator();
   
BeanPropertyBindingResult errors = new BeanPropertyBindingResult(libro, "libro");
   
ValidationUtils.invokeValidator(validator, libro, errors);

for (ObjectError error : errors.getAllErrors() ){
    System.out.println(context.getMessage(error.getCode(), null, null));
}
```

La clase ```BeanPropertyBindingResult``` es un recurso necesario que se introduce ahora para poder ejecutar el proceso de validación, pero que se discutirá en otro capítulo relativo al _binding_. Su propósito es extraer las propiedades del objeto dado para que se les pueda asociar directamente códigos de error.

## 15.2. Anotaciones

Spring soporta la familia de anotaciones estándar de Java que permiten especificar las validaciones a realizar directamente dentro de las clases. No obstante, para que funcionen, es necesario añadir una librería que las implemente, como la que proporciona Hibernate, en el fichero ```pom.xml```:

```xml
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>4.2.0.Final</version>
</dependency>
```

Una vez añadida la dependencia se puede prescindir de la clase de validación anterior, y usar las anotaciones para describir las validaciones a realizar directamente sobre los campos de los _beans_:

```java
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.Size;
import javax.validation.constraints.NotNull;

public class Libro {

    @NotNull
    @Size(min=1, max=250)
    private String titulo;

    @Min(1)
    @Max(99999)
    private int paginas;
...
```

Para probar las anotaciones se puede utilizar un código como el siguiente:

```java
Libro libro = new Libro();
   
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
   
Set<ConstraintViolation<Libro>> errors = validator.validate(libro);
   
for (Iterator<ConstraintViolation<Libro>> i = errors.iterator(); i.hasNext(); ){
    ConstraintViolation<Libro> error = i.next();

    System.out.println(error.getMessage());
}
```

Por último, comentar que si lo que se quiere es inyectar un validador a un _bean_, evitando tener que instanciar una implementación concreta por código, se puede añadir a la configuración dentro del fichero ```applicationContext.xml```:

```xml
<bean id="validator"
class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```

Esto fuerza a que Spring busque la implementación por su cuenta, y la inyecte en los _beans_ que lo soliciten, independientemente de que usen el paquete propio de Spring o el estándar:

```java
@Service
public class BibliotecaServiceImpl implements BibliotecaService {

    @Autowired
    private Validator validator;
...
```
