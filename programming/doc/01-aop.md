# 1. Programación Orientada a Aspectos

_20-02-2009_ _Juan Mellado_

Recientemente, en la documentación técnica aportada por un cliente para un proyecto, se insitía en la imperiosa necesidad de utilizar el paradigma de la programación orientada a aspectos para los nuevos desarrollos. A estas alturas, cuando uno lee estas cosas, lo primero que piensa es "_Golden Hammer habemus_". Pero ya se sabe que donde manda patrón (cliente), no manda marinero (desarrollador). En San Google y Santa Wikipedia puede encontrarse bastante información sobre el tema, aunque no está nunca de más intentar sacar conclusiones por uno mismo.

La definición de "Programación Orientada a Aspectos" (AOP) es similar a cualquier otra que se pueda leer acerca de una nueva tecnología que pretende solventar los problemas recurrentes de la construcción de _software_. Separar, encapsular, modularizar, facilitar, etc. ¿Pero qué tiene entonces de especial este paradigma? ¿No tenemos ya la ubicua programación orientada a objetos (OOP) para eso? Pues la verdad es que a lo que estamos acostumbrados está bien, pero la AOP aporta un puntito interesante al asunto. La gracia del tema está en que parece haber sido ideada más por programadores que por arquitectos. Su índole es de carácter práctico, y va más allá del trazado de subdivisones, diagramas de abstracciones, o esquemas de muy alto nivel de más que cuestionable utilidad en el día a día.

Para entender que es lo que aporta la AOP frente a otras tecnologías, lo mejor es utilizar un ejemplo. Esto es lo que hacen prácticamente todos los artículos que pueden leerse por Internet, y resulta comprensible, los objetivos que persigue este paradigma son tan génericos que resulta complicado entender la filosofía de su enfoque si no se ve su aplicación sobre casos concretos.

El ejemplo "paradigmático" de esta tecnología consiste en imaginar un paquete de clases cualquiera, escrito en cualquier lenguaje de programación disponible, al que se quiere añadir una nueva responsabilidad. Por ejemplo, imaginemos que se trata de un paquete de clases que se utiliza para gestionar las cuentas de usuarios de un sistema _online_. Por política de la empresa propietaria del sistema, se decide que cada vez que se ejecute un método de estas clases, se debe escribir en un fichero de texto de trazas (_log_) un mensaje con el nombre del método ejecutado para hacer un seguimiento exhaustivo del uso de este paquete.

Siguiendo un procedimiento bastante habitual, se abriría el IDE, y se empezaría a cambiar uno a uno los métodos de las clases, añadiendo las líneas de código oportunas para escribir al _log_. Aunque es posible también que se creara un paquete nuevo de clases por encima, a modo de _proxy_, que se dedicase solamente a escribir al _log_ y pasar la llamada al paquete original. Con este último enfoque incluso se podría utilizar un patrón _Factory_, e instanciar clases de un paquete u otro a conveniencia, por ejemplo para los entornos de desarrollo donde el seguimiento exhaustivo no es preciso. No obstante, aún quedarían bastantes detalles por afinar, como qué sistema de _logger_ utilizar, y sobre todo, qué ocurrirá si en un futuro se decide cambiar dicho sistema de, supongamos inicialmente uno _in-house_, a un paquete comercial, o tal vez a uno libre y de código abierto. Aunque sobre este último punto se podría argumentar que lo correcto sería ceñirse a una interface bien definida, y posiblemente estándar, para que el cambio fuese lo menos dramático posible. Y aún asi, habría que seguir decidiendo por ejemplo que ocurre si se produce una excepción al escribir al _log_, y una de serie de cosas por el estilo.

La clave del asunto está en observar dos puntos importantes. Primero, que las distintas soluciones implican modificar el código ya existente, y segundo, que el paquete afectado asume una nueva responsabilidad más allá de para la que fue creada originalmente, esto es, gestionar cuentas de usuario. La programación orientada a aspectos promete que es capaz de conseguir lo que se pide sin tener que modificar ni una sola línea de código ya existente, y manteniendo claramente separadas ambas funcionalidades, a pesar de que una utilice la otra.

¿Y cómo obra esta magia? Pues mirando los programas como lo que son en la práctica, cuando se encuentran en ejecución: flujos de instrucciones y datos. Considera los acontecimientos que suceden durante la ejecución de un programa (creación de una instancia de la clase _C_, ejecución del método _M_, lanzamiento de la excepción _E_, etc), y se los ofrece al programador, para que pueda asociarles el código que quiera que se ejecute cuando ocurra alguno de esos acontecimientos. Es similar a la programación basada en eventos, donde se indica a que método se tiene que llamar cuando ocurre algo de particular interés, como la pulsación de un botón por ejemplo, pero llevada a otro ámbito, el de la ejecución del código. Similar a lo que se hace desde un depurador cuando se pone un punto de parada condicional. La AOP permite al programador escribir una regla del estilo "cuando se ejecute cualquier método del paquete [de cuentas], llama a mi función [que escribe un mensaje en el _log_]". Y lo más grande es que en la práctica (en la vida real, con un compilador real que soporte AOP) basta crear un fichero con apenas 10 líneas de código para escribir esa regla y conseguir el resultado deseado.

Esta tecnología no espera que el programador se ciña a una interface, a un diseño por contrato, que utilice anotaciones, o cualquier otro tipo de meta-información embebida dentro de "su" código. El enfoque es radicalmente opuesto. La tecnología ofrece cosas al programador en vez de exigirle. Permite inyectar "por fuera" la funcionalidad requerida en el momento deseado.

Lo bueno de este enfoque, vuelvo a repetir, retornando al ejemplo, es que no hay que tocar el código fuente ya existente, las reglas se definen en otro fichero, y las funcionalidades quedan separadas completamente, es sólo en el fichero de reglas donde se encontrarán las referencias a los dos paquetes. Por un lado el sistema de gestión de cuentas, que se dedicará única y exclusivamente a gestionar cuentas, y por otro el sistema de _logging_, que se limitará a escribir al _log_.

¿Pero cómo se consigue esto en la práctica? Pues gracias a una serie de proyectos que han incorporado la AOP a lenguajes de programación de uso habitual. Para probar esto de una forma rápida y cómoda recomiendo AspectJ, que es una implementación para Java, y AJDT, que es su correspondiente _plugin_ para Eclipse. En este ambiente de desarrollo la definición de la regla se escribe en un fichero con extensión ```.aj``` de forma similar a como se muestra en el ejemplo siguiente:

```text
public aspect AspectLogging {
  
  pointcut logging() : execution( * com.enterprise.product.accounts.*.*(..) );

  before() : logging(){
    System.out.println(thisJoinPoint);
  }
}
```

El código es bastante sencillo, es más, tiene el aspecto de código Java ordinario, pero utilizando algunas palabras reservadas propias. Por ejemplo, ```aspect``` sirve para encapsular las reglas, de igual forma que lo hace ```class``` con los métodos y atributos. De hecho, un ```aspect``` funciona por defecto como un _Singleton_, y admite métodos y atributos como si fuera una clase ordinaria. La diferencia es que permite además incluir puntos de corte (```pointcut```), que sirven para definir los momentos concretos durante la ejecución del programa en los que se tiene que activar. Con ```execution``` se indica que se active, como si fuera un evento, cuando se ejecute un método que cumpla con el patrón que se le pasa como parámetro. Otras posibilidades incluyen ```handler``` para que se active al saltar una interrupción de un tipo concreto, ```call``` para cuando se invoque a un método, ```get``` para cuando se acceda a un atributo, y así sucesivamente. Merece la pena echar un vistazo a la documentación para hacerse una idea mejor de la potencia disponible. El parámetro que admite un punto de corte es una expresión regular que permite hacer referencia a una o más partes del código del programa. En el ejemplo, la expresión tiene que leerse como "cualquier método, de cualquier tipo, de cualquier número de parámetros, de cualquier clase, del paquete ```com.enterprise.product.accounts```".

La escritura de expresiones regulares siempre suele resultar una actividad un tanto viciante. Las combinaciones son infinitas, e incluso se pueden concatenar con ```||``` y ```&&``` como si fueran expresiones _booleanas_. Ahí van unas cuantas:

```text
//Ejecución de cualquier método
  execution( * *(..) )

//Ejecución de cualquier método público
  execution( public * *(..) )

//Ejecución de cualquier método no estático
  execution( ! static * *(..) )

//Ejecución de cualquier método que admita un String
  execution( * *(String) )

//Ejecución de cualquier método "setNif" que admita un String y no devuelva nada
  execution( void setNif(String) )

//Ejecución de una excepción de la clase CustomException
  handler(CustomException)

//Cualquier lllamada a un método "set" realizada desde un objeto de la clase Account
  call( void set*(..) ) && target( Account )
```

Una vez definido un punto de corte se puede refinar un poco más el momento concreto en que se quiere activar, y asociarle el código que se quiere ejecutar. En el código de ejemplo se ha utilizado ```before``` para indicar que se quiere activar antes de que se produzca la ejecución de los métodos que cumplen con la expresión regular, pero también se podría haber utilizado ```after``` para indicar que se activara después de la ejecución, e incluso ```around``` para sustituir completamente el comportamiento del método original por otro. Esta última posibilidad es ciertamente deliciosa, por decirlo de una forma suave.

Por último, queda el código que queremos que se ejecute. Código Java simple y llano, como una vulgar salida por la consola estándar, que redirigida a fichero podría hacer las veces de tosco sistema de _logging_. De esta forma, lo que ocurrirá, es que cuando se ejecute cualquier método del paquete cuentas se ejecutará también el código con la salida por consola. ¡Objetivo cumplido! Y todo ello sin haber tenido que tocar ni una sola línea de código original. ¡Promesa cumplida! Y es más, si algún día la empresa decidiera cambiar el sistema de _log_ por otro más elaborado, el cambio sería transparente, ya que bastaría con cambiar el código de la regla. E incluso si, en una última vuelta de tuerca, se decidiera cambiar el paquete de cuentas por otro, sería muy sencillo hacer que las nuevas clases escribiesen al _log_ cambiando la expresión regular. El único problema extremadamente grave que se plantea es que ya no podremos facturar un número insano de horas a nuestros clientes por hacer estos cambios como veníamos haciendo hasta ahora. ¡Maldita tecnología!

Un último comentario acerca del ejemplo, sobre la palabra reservada ```thisJoinPoint```. Almacena el punto de concreto dentro de la ejecución del código original en el que ha activado la regla. La he puesto a modo de referencia, para destacar el hecho de que dentro del código se puede hacer referencia a información de contexto. De hecho, se puede incluso referenciar al propio objeto dentro del que se ha producido el evento, y los argumentos que recibe o retorna el método. Recomiendo nuevamente mirar la documentación para ver las enormes posibilidades que ofrece AspectJ a este respecto.
