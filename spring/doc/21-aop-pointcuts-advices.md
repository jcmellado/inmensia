# 21. Spring: AOP (2) - Pointcuts y Advices

_24-04-2012_ _Juan Mellado_

Una vez realizada y comprobada la configuración indicada en el artículo anterior, es hora de empezar a definir más elementos de la programación orientada a aspectos.

## 21.1. Pointcut

Spring sólo soporta _pointcuts_ sobre la ejecución de métodos. Para definirlos hay que añadir la anotación ```@Pointcut``` con la expresión regular asociada en la declaración de un método:

```java
@Aspect
public class Espia {

  @Pointcut("execution(public * conversacion(..))")
  public void anyConversacion() {
  }
}
```

La expresión regular define un área dentro del código de una aplicación que resulta de interés. Y este ejemplo en concreto define la ejecución de cualquier método público llamado ```conversacion```, independientemente del tipo retornado o de los parámetros esperados.

El nombre del método en la clase que tiene aplicado la anotación no tiene que seguir ningún patrón concreto, pero es conveniente que sea congruente con la expresión regular.

Dentro de las expresiones regulares pueden utilizarse las siguientes designaciones para limitar los _join points_:

- ```execution```: Ejecución de métodos.

- ```@annotacion```: Una anotación concreta.

- ```within```: Un tipo concreto, como un paquete de clases por ejemplo.

- ```@within```: Tipos que tienen una anotación concreta.

- ```target```: Instancia de un objeto de un tipo concreto.

- ```@target```: Instancia de un objeto con una anotación concreta.

- ```args```: Argumentos de un tipo concreto.

- ```@args```: Argumentos con una anotación concreta.

- ```this```: Instancia de un objeto de un tipo concreto. Pero referido al objeto _proxy_ de Spring, no al original, al que se hace referencia con ```target```.

- ```bean```: Instancia de un _bean_ concreto.

Los _pointcuts_ se puede combinar utilizando los operadores lógicos habituales ```||```, ```&&``` y ```!```:

```java
@Aspect
public class Espia {

  @Pointcut("execution(public * conversacion(..))")
  public void anyConversacion() {
  }

  @Pointcut("within(com.empresa.despacho..*)")
  public void inDespacho() {
  }

  @Pointcut("anyConversacion() && inDespacho()")
  ...
```

## 21.2. Advice

Los _advices_ se declaran junto con su expresión de configuración, que indica el momento en que tienen que aplicarse.

Una expresión de configuración puede hacer referencia a un _pointcut_ previamente declarado:

```java
@Aspect
public class Grabadora {

  @After("com.empresa.Espia.anyConversacion()")
  public void grabarConversacion() {
  }
}
```

O usar directamente una expresión regular:

```java
@Aspect
public class Grabadora {

  @After("execution(public * conversacion(..))")
  public void grabarConversacion() {
  }
}
```

Los _advices_ se pueden configurar para que se ejecuten en los siguiente momentos:

- ```@Before```: Antes de la ejecución de un método.

- ```@AfterReturning```: Después de la ejecución de un método que termina normalmente.

- ```@AfterThrowing```: Después de la ejecución de un método que termina elevando una excepción.

- ```@After```: Después de la ejecución de un método que termina normalmente o elevando una excepción.

- ```@Around```: Sustituyendo por completo el código original.

Los _advices_ ejecutados al terminar la ejecución normal de un método pueden tener acceso al objeto retornado utilizando el atributo ```returning``` en la anotación:

```java
@AfterReturning(pointcut="com.empresa.Espia.anyConversacion()",
                returning="cotilleo")
public void grabarConversacion(Object cotilleo) {
}
```

Los _advices_ ejecutados al terminar un método elevando una excepción pueden tener acceso a la excepción elevada utilizando el atributo ```throwing``` en la anotación:

```java
@AfterThrowing(pointcut="execution(public * *(..))",
               throwing="gritito")
public void grabarGrititos(Exception gritito) {
}
```

Los _advices_ que sustituyen un método pueden tener acceso a los detalles del método utilizando un parámetro de tipo ```ProceedingJoinPoint```, que permite conocer el detalle del objeto, métodos, argumentos, e invocar al método original interceptado:

```java
@Around("com.empresa.Espia.anyConversacion()")
public Object interceptar(ProceedingJoinPoint pjp) throws Throwable {
  return pjp.proceed();
}
```

Las posibilidades de la programación orientada a aspectos son enormes, sobre todo la flexibilidad que ofrecen las expresiones regulares. La documentación oficial de referencia contiene muchos más detalles y ejemplos que deben consultarse para obtener un mejor aprovechamiento de su filosofía de funcionamiento.
