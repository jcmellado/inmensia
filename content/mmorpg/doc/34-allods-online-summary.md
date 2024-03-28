# 34. Allods Online Addons (10/10): Recapitulando

_16-10-2010_ _Juan Mellado_

El último capítulo de esta serie coincide con la actualización a la versión ```1.1.02.58.1``` del juego. Así, con todos esos numericos. Más conocida por "_Rise of Gorluxor_".

Llegado este punto no me queda mucho por decir. He cubierto todos los apartados que me había propuesto, completando los 10 artículos que calculé que podía llevarme la tarea. Aunque en este último artículo es bueno echar la vista atrás y repasar lo aprendido, que no ha sido poco.

Lua es sin lugar a dudas una de las piezas claves de todo este invento de los _addons_. Ya lo conocía de antes, pero como otra más de esas tecnologías que uno va encontrando por el camino y a la que nunca acaba de dedicar todo el tiempo que debería. En particular me llamó mucho la atención que el juego levantara una máquina virtual de Lua independiente para cada _addon_. Aprovechando que Lua es un proyecto de código abierto me animé a bajarme los fuentes y echarles un vistazo para comprobar realmente lo liviano que resulta todo el sistema.

Otro factor clave desde el punto de vista del código de los _addons_ es la programación orientada a eventos. A día de hoy sigue pareciendo una de las formas más naturales de informar de la ocurrencia de determinados sucesos externos a un programa, evitando tener que consumir recursos haciendo _polling_ sobre algún tipo de estructura global. La contrapartida por supuesto es el número de eventos, que puede llegar a ser realmente grande si se pretende abarcar el enorme número de sucesos que pueden llegar ocurrir en el juego.

Un API bien definido que permita acceder y manipular todos los componentes principales del juego es otro punto importante a tener en cuenta. Aunque al igual que ocurre con los eventos, el número de funciones públicas se dispara. Afortunadamente, gracias al tipo ```table``` de Lua, que no deja de ser un _hash_ en los que sus elementos se referencian por su nombre, no es necesario definir múltiples estructuras o clases de datos, ya que se puede añadir cualquier atributo sin tener que modificar el código ya existente.

El hecho de que el código fuente interno del juego, al menos la parte que está escrita en Lua, utilice las mismas funciones del API que se exponen de forma pública es bastante interesante. Eso permite reemplazar funcionalidades completas del juego por las nuestras propias de forma bastante elegante.

Por su parte, la interface de usuario está resuelta mediante ficheros XML donde se describen las ventanas, los controles que contienen, y su aspecto en general. Esto es bastante común en muchas arquitecturas, y aunque tienden a generar muchos ficheros. a veces difíciles de mantener manualmente, permite separar claramente la capa de presentación de la lógica de negocio. Aunque en este punto se echa en falta sin lugar a dudas algún tipo de editor visual que facilite la construcción de las interfaces.

Y hablando de echar en falta, donde más naufraga el juego en este asunto de los _addons_ es en la falta de documentación. De acuerdo, el traductor de Google es nuestro amigo, pero aún así resulta complicado navegar por la documentación del API que se encuentra única y exclusivamente publicada en ruso. Y por no mencionar la diferencia entre las distintas versiones del juego, la rusa y las demás, con cambios en el API que hacen que los _addons_ no funcionen correctamente en unas versiones y otras.

En cualquier caso, siempre hay que tener presente que _Allods Online_ es un MMORPG gratuito, aunque con micropagos, de forma que cualquier característica que los desarrolladores incorporen al juego, como la posibilidad de escribir _addons_, se tendría que valorar en su justa medida teniendo en cuenta su gratuidad.

_Happy coding!_
