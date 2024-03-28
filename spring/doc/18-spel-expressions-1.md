# 18. Spring: SpEL (2) - Expresiones Básicas

_24-04-2012_ _Juan Mellado_

SpEL soporta una gran cantidad de tipos de expresiones, este artículo enumera los más básicos, y muestra algunos ejemplos sencillos para cada uno de ellos.

## 18.1. Literales

Los literales son las expresiones más sencillas que pueden utilizarse. Constantes que representan cadenas de texto, valores numéricos, valores lógicos, etc, utilizando las reglas habituales:

```text
#{'¡Hola mundo cruel!'}
#{'3.14159'}
#{'0xff'}
```

## 18.2. Propiedades

Dado un objeto, se sigue la nomenclatura de punto para acceder a sus propiedades:

```text
#{libro.titulo}
#{continente.pais.ciudad}
```

## 18.3. Colecciones

Los objetos dentro de una colección se pueden referenciar por su índice, o por su clave en el caso de los _arrays_ asociativos:

```text
#{biblioteca.libros[0]}
#{paises['es'].nombre}
```

## 18.4. Listas

Se pueden construir listas embebidas dentro de las propias expresiones utilizando la notación de llaves:

```text
#{{1, 1, 2, 3, 5, 8, 13}}
#{{1, 3, 5}, {2, 4, 6}}
#{{'Gaspar', 'Melchor', 'Baltasar'}}
```

## 18.5. Arrays

Se pueden construir matrices embebidas dentro de las propias expresiones invocando el constructor de Java:

```text
#{new int[100]}
#{new int[4]{11, 21, 31, 41}}
```

## 18.6. Métodos

Dado un objeto, se sigue la nomenclatura de punto para invocar sus métodos:

```text
#{'¡Viva '.concat(' Las Vegas!')}
#{{1, 2, 3}.size()}
```

## 18.7. Operadores

Se pueden utilizar los operadores relacionales habituales para realizar comparaciones:

```text
#{1 == 1}
#{'madre' > 'hijo'}
```

Se pueden utilizar los operadores lógicos habituales para evaluar condiciones:

```text
#{true and false}
#{!('infinito'.length() == 0)}
```

Y se pueden utilizar operadores aritméticos habituales:

```text
#{1 + 1}
#{7 % 5}
```

## 18.8. Asignaciones

Aunque no muy habitual, se pueden realizar asignaciones directamente. A este punto hay que recordar la existencia del objeto contextual ```root``` sobre el que aplica la expresión:

```text
#{nombre = 'Cervantes'}
#{alpha = 47}
```

## 18.9. Tipos

El operador especial ```T(type)``` se puede utilizar para obtener una referencia a una clase, e invocar sus métodos estáticos por ejemplo:

```text
#{T(java.lang.String).valueOf(47)}
#{T(java.lang.Math).random() * 100}
```

## 18.10 Constructores

Se pueden construir objetos utilizando ```new``` y el nombre completo de la clase que se quiere instanciar, excepto para los tipos básicos que son directamente reconocidos:

```text
#{new int[4]}
#{new com.empresa.Libro('Fortunata y Jacinta')}
```
