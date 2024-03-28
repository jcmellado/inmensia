# 11. Java: jshell

_09-05-2018_ _Juan Mellado_

jshell es una herramienta de línea de comandos introducida en Java 9. Permite ejecutar código de una manera sencilla sin necesidad de un IDE, aunque no es un sustituto de estos. Se arranca, se escribe la sentencia en Java que se quiere ejecutar, y automáticamente se muestra el resultado de la ejecución. Es decir, jshell implementa el clásico patrón REPL (_Read-Evaluate-Print Loop_).

Es una herramienta bastante útil cuando se está escribiendo determinado tipo de código. Lo habitual es tenerla abierta en una consola aparte e ir probando en ella pequeños _snippets_ de código a medida que se desarrolla. Esto permite tener un _feedback_ inmediato en vez de esperar a compilar y ejecutar toda la aplicación en el IDE. Es útil sobre todo para probar variantes de un mismo código, experimentar con una librería con la que no se tenga demasiada experiencia, evaluar una secuencia larga de métodos concatenados, o ejecutar código copiado directamente de un sitio _web_.

jshell se encuentra en el subdirectorio ```bin``` del directorio donde se encuentre instalada la máquina virtual de Java. Se ejecuta arrancando directamente el programa desde línea de comandos:

```text
$jshell
| Welcome to JShell -- Version 10
| For an introduction type: /help intro
```

jshell admite el acostumbrado parámetro ```-help```, que muestra todas las opciones disponibles, y el no menos clásico parámetro ```-v``` (_verbose_), que muestra información más detallada durante la ejecución. Parámetros ambos que merece la pena utilizar, al menos las primeras veces que se utilice la herramienta.

Una vez arrancado jshell se puede ejecutar código Java directamente en la propia línea de comandos. Por ejemplo, se puede definir una variable y asignarle un valor:

```text
jshell> var x = 42
x ==> 42
| created variable x : int
```

Como se observa, a pesar de ser Java, no es necesario poner punto y coma al final de la línea. La variable es automáticamente instanciada e inicializada. Se puede comprobar su valor escribiendo una expresión que haga referencia a ella:

```text
jshell> x
x ==> 42
| value of x : int
```

La expresión es evaluada y el valor resultante es automáticamente impreso. Lo interesante es que la expresión puede ser todo lo compleja que se quiera:

```text
jshell> x * x
$3 ==> 1764
| created scratch variable $3 : int
```

Cuando jshell evalúa una expresión le asigna un identificador al resultado. En el ejemplo anterior, el resultado se asigna a una variable de nombre ```$3```. Esta variable se puede referenciar y utilizar de manera independiente, o dentro de una expresión, lo que a su vez creará una nueva variable:

```text
jshell> $3
$3 ==> 1764
| value of $3 : int

jshell> $3 * 2
$5 ==> 3528
| created scratch variable $5 : int
```

Además de variables, también se pueden definir clases y métodos:

```text
jshell> int doble(int value) {
...> return value * value;
...> }
| created method doble(int)
```

jshell evalúa las expresiones cuando se pulsa retorno de carro. Si la expresión es válida, pero no está completa, entonces las siguientes líneas formarán también parte de la expresión en curso, tantas como sean necesarias para completarla.

Las clases y métodos definidos se pueden referenciar dentro de expresiones:

```text
jshell> doble(2)
$7 ==> 4
| created scratch variable $7 : int
```

Si se quiere cambiar una definición sólo hay que volver a declararla, la nueva declaración sobrescribirá la anterior:

```text
jshell> int doble(int value) {
...> return value * 2;
...> }
| modified method doble(int)
| update overwrote method doble(int)
```

jshell es un editor de líneas que permite moverse por ellas con las teclas de cursor, tiene teclas de acceso rápido para algunas acciones de edición comunes en la mayoría de editores, algunas características básicas como un histórico de las líneas introducidas, y una potente opción de autocompletar.

El autocompletar más básico se obtiene pulsando la tecla de tabulación a medida que se escribe. Esto hace que se muestren todas las opciones disponibles que casen con la expresión en curso, o directamente se complete la expresión si sólo hay una opción disponible. Dependiendo del contexto, volviendo a pulsar tabulador, se muestra información adicional, como por ejemplo la documentación si está disponible:

```text
jshell> String.
CASE_INSENSITIVE_ORDER   class   copyValueOf(   format(   join(   valueOf(

jshell> Long.c
class              compare(           compareUnsigned(

jshell> Long.compare
compare(           compareUnsigned(

jshell> Long.compare(
$3       $5       $7       $9       doble(   x

Signatures:
int Long.compare(long x, long y)

<press tab again to see documentation>

jshell> Long.compare(
int Long.compare(long x, long y)
<no documentation found>

<press tab again to see all possible completions; total possible completions: 547>
```

Otras opciones de autocompletar se ejecutan pulsando _shift_ junto con el tabulador. En ese punto el editor se queda esperando que se pulse una tecla. La ```v``` sirve para crear una variable con la expresión en curso, la ```m``` para crear un método, y la ```i``` para importar una clase.

Por defecto están disponibles paquetes del módulo base de Java, pero se pueden importar paquetes de librerías de terceros que se encuentren en el _classpath_:

```text
$jshell -v -c commons-lang3-3.7.jar
| Welcome to JShell -- Version 10
| For an introduction type: /help intro

jshell> import org.apache.commons.lang3.StringUtils

jshell> StringUtils.join('a', 'b', 'c')
$2 ==> "abc"
| created scratch variable $2 : String
```

jshell además de declaraciones y expresiones admite comandos. Los comandos se utilizan para mostrar información y controlar el entorno de ejecución. Para diferenciarlos de las expresiones, todos los comandos se escriben empezando por el carácter barra (```/```). Sirven para listar el historial (```/history```), las clases (```/types```), métodos (```/methods```), variables (```/vars```) y paquetes disponibles (```/imports```), volver a ejecutar un snippet (```/!```), utilizar un editor externo (```/edit```), y realizar muchas otras tareas. Es recomendable echar un vistazo a la ayuda para ver todos los comandos disponibles.

```text
jshell> /vars
| int x = 42
| int $3 = 1764
| int $5 = 3528
| int $7 = 4
| int $9 = 4

jshell> /methods
| int doble(int)
```

Un último comando que hay que conocer es ```/exit```, que sirve para cerrar jshell.

En definitiva, una herramienta muy útil de la que siempre han presumido muchos otros lenguajes, sobre todo los de _scripts_, y que ahora también está disponible de forma nativa para Java.
