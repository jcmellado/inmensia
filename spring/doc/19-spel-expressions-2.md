# 19. Spring: SpEL (3) - Expresiones Avanzadas

_24-04-2012_ _Juan Mellado_

En este artículo se revisan algunas características más avanzadas que permitir escribir expresiones más complejas con SpEL.

## 19.1. Variables

Cuando se utiliza una instancia del _parser_ de expresiones se pueden definir variables, y luego hacer referencia a ellas usando la nomenclatura de almohadilla:

```java
context.setVariable("alias", "Clark Kent");

parser.parseExpression("nombre=#alias").getValue(context);
```

Existen dos variables especiales llamadas ```#root``` y ```#this```. La primera hace referencia siempre al objeto contextual sobre el que se aplica la expresión siendo evaluada. La segunda toma el valor del objeto siendo evaluado.

## 19.2. Funciones

Cuando se utiliza directamente una instancia del _parser_ de expresiones se pueden definir funciones, y luego hacer referencia a ellas usando la nomenclatura de almohadilla:

```java
context.registerFunction("reverse",
    StringUtils.class.getDeclaredMethod("reverse", new Class[]{String.class}));

parser.parseExpression("#reverse('monja')").getValue(context);
```

## 19.3. Beans

Cuando se utiliza directamente una instancia del _parser_ de expresiones se pueden hacer referencia a _beans_ utilizando una instancia de la interface ```BeanResolver```, y utilizando la nomenclatura de arroba:

```java
BeanResolver resolver = new BeanFactoryResolver(applicationContext);
   
context.setBeanResolver(resolver);

parser.parseExpression("@fooBean").getValue(context);
```

## 19.4. if-then-else

Se puede utilizar el operador ternario ```if-then-else``` de la forma habitual:

```text
#{1 == 1 ? 'churra' : 'merina'}
#{'abc' instanceof T(int) ? 'entero' : 'desconocido'}
```

## 19.5. Elvis

El operador Elvis (```?:```) es una forma abreviada del operador ternario anterior. Permite omitir una expresión resultado si esta es igual a la condición. Se entiende mejor con un ejemplo.

Esta expresión:

```text
valor != null ? valor : 'error'
```

Es igual a la siguiente utilizando el operador Elvis:

```text
valor?:'error'
```

La documentación oficial comenta que el operador proviene del lenguaje Groovy, y que su nombre deriva del hecho de que ```?:``` asemeja a un _emoticon_ con el típico tupé de Elvis Presley.

## 19.6. Safe Navigation

El operador Safe Navigation (```?.```) evita que se eleve una excepción de tipo ```NullPointerException``` cuando se intenta acceder a una propiedad sobre un objeto nulo. Si el objeto es nulo el operador retorna directamente nulo en vez de intentar acceder a la propiedad:

```text
#{biblioteca?.libros}
```

Este operador también proviene también de Groovy.

## 19.7. Collection Selection

El operador Collection Selection (```?[<expression>]```) permite filtrar elementos de una colección retornando sólo las que cumplan una determinada condición:

```text
#{ovejas.?[tipo == 'merina']}
```

## 19.8. Collection Projection

El operador Collection Projection(```![<expression>]```) permite construir una colección de objetos de un tipo distinto a los originales de una colección dada.

Por ejemplo, se puede partir de una lista de libros y obtener una lista de autores:

```text
#{libros.![autor]}
```

## 19.9. Templates

Dentro de las expresiones se pueden utilizar otras expresiones, que tienen que ser evaluadas para obtener su valor usando la clase ```TemplateParserContext```:

```java
parser.parseExpression("Número ganador: #{ T(java.lang.Math).random() }",
    new TemplateParserContext() ).getValue(String.class);
```
