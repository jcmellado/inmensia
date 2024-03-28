# 30. Allods Online Addons (6/10): Formularios

_18-09-2010_ _Juan Mellado_

En el _zip_ que contiene la documentación oficial, que se distribuye con el propio juego, hay un directorio llamado ```ResourceSystem```. Dentro de dicho directorio hay un ejemplo de cada tipo de fichero ```.xdb``` que puede usarse para crear _addons_, incluyendo todos los tipos de controles que pueden utilizarse dentro de los formularios con los que se implementan las interfaces gráficas.

Desgraciadamente no hay mucha más documentación acerca de los controles y parámetros de configuración que admiten los formularios. Los ficheros que empiezan por ```SampleDefault``` contienen un ejemplo completo con todos los parámetros válidos para cada tipo de ```.xdb```. Los ficheros que empiezan por ```SampleDefaultExt``` contienen además un ejemplo completo con todos los parámetros válidos para las opciones que admiten listas de valores. Son bastantes, y se echa de menos un IDE para construir formularios de forma gráfica y posteriormente exportarlos a ```.xdb```. ¿Alguien se anima?

En cualquier caso, en base a esos ejemplos, y aparte de los ya vistos para los textos y las texturas, los controles disponibles son los siguientes:

- ```WidgetForm```: Formulario. Este es el tipo que tienen las ventanas principales. Todos los _addons_ tienen un fichero de este tipo para su formulario principal, independientemente de que luego implementen o no una interface gráfica.

- ```WidgetPanel```: Panel. Los formularios normalmente se componen de uno o más de tipo de controles, los paneles sirven para agruparlos e ir formando una jerarquía.

- ```WidgetButton```: Botón. Un clásico, poco más que añadir.

- ```WidgetTextView```: Texto. Otro que no merece más comentarios.

- ```WidgetEditLine```: Línea de edición. Este control presenta una línea de texto en la que se puede introducir valores manualmente por teclado. Permite configurar algunos aspectos básicos como el aspecto del cursor, la velocidad de parpadeo, o indicar si se va a utilizar para introducir _passwords_. En los ejemplos que vienen con el juego no se utiliza, y de hecho, ninguno de los tipos que siguen a continuación, por lo que prácticamente no existe ningún _addon_ que implemente ninguno de estos tipos. Buscando por Internet apenas he encontrado un _addon_ que utiliza este tipo, pero del resto que siguen ninguno.

- ```WidgetTextContainer```: Contenedor de texto. Parece que su objetivo obviamente es contener texto, pero resulta difícil precisar su funcionamiento. Debería ser el típico control en el que se puede ir añadiendo o quitando líneas de texto libre con formato, como en la salida de una ventana de _chat_ por ejemplo.

- ```WidgetScrollableContainer```: Contenedor _scrollable_. Aparenta el típico control que se usa para contener a otros y que muestra una barra de _scroll_, horizontal o vertical, para mostrar los que quedan fuera. No queda claro si es necesario definir una barra de _scroll_ propia o usa automáticamente una por defecto.

- ```WidgetSimpleTable```: Tabla. Este debería ser el tipo de control que permita construir tablas con varias filas y columnas para listar registros. Sólo tiene un atributo llamado ```tableStep``` que parece hacer referencia al número de columnas deseadas. Aunque también podría ser un control para definir un _layout_ en forma de tabla.

- ```WidgetDiscreteScrollBar```: Barra de _scroll_ de valores discretos. Debe ser la típica barra de _scroll_ que permite la elección de unos determinados valores concretos prefijados de antemano.

- ```WidgetGlideScrollBar```: Barra de _scroll_. He de suponer que de valores continuos, sin la restricción de valores prefijados de la anterior. Aunque de hecho tiene los mismos atributos que la anterior. En el API curiosamente no se encuentran funciones para este tipo de control, pero si para el anterior.

- ```WidgetDiscreteSlider```: Selector de valores discretos. Debería ser igual que las barras de _scroll_, pero sin los botones de incremento y decremento, sólo el botón central.

- ```WidgetGlideSlider```: Selector igual que el anterior, pero para valores continuos. De igual forma que con las barras de _scroll_, en el API sólo hay funciones para el tipo de valores discretos.

- ```WidgetConsole```: Consola. Debería ser la típica ventana de introducción de comandos. Entre sus atributos hay una referencia a un ```EditLine```, por lo que no queda claro que valor añadido tiene frente a una línea de introducción de textos ordinario, a menos claro que sea capaz de procesar los comandos introducidos.

- ```WidgetConsoleOutput```: Consola de salida. Aparenta ser la salida con los resultados de la ejecución de comandos introducidos por consola. Al igual que en el anterior tipo, entre sus atributos se hace referencia a un ```textContainer```, por lo que arroja algunas dudas acerca de su diferencia con una salida de texto ordinaria.

- ```WidgetControl3D```: Control 3D. Este tiene una pinta muy interesante, una pena que no exista documentación. Aparenta ser un control que permite visualizar una entidad del juego en tres dimensiones, como la ventana de información del personaje que muestra a nuestro avatar en miniatura, e incluso nos permite rotarlo.

El único _addon_ que viene de ejemplo con el juego y que define un formulario algo elaborado es el llamado ```SampleReactionHandler```, aunque en la práctica apenas tiene un panel con una textura de fondo, dos textos y un botón. No obstante, aparte del propio formulario en si mismo, este _addon_ tiene cosas interesantes.

La primera cosa interesante es ver como se construye el botón, que hace uso del prototipo que se encuentra en el directorio ```data/Mods/SampleCommon/Button```. Dentro de ese directorio hay ni más ni menos que 23 ficheros (aunque los cuatro ```.tga``` originales no son realmente necesarios). Todos esos ficheros definen el aspecto de un botón en cada uno de los estados en los que puede estar: normal, seleccionado, presionado, o deshabilitado. Y dan una idea de la enorme cantidad de trabajo que hace falta para definir completamente cualquier control. Realmente se echa en falta un IDE que simplifique el trabajo, o al menos una librería completa de este tipo de controles prototipo para usarlos como plantillas.

La segunda cosa interesante de ese _addon_ de ejemplo es ver el _script_ Lua. Los controles registran reacciones en vez de eventos. ¿Y qué quiere decir esto en la práctica? Pues que en vez de subscribir el _addon_ a eventos del mundo virtual utilizando la función ```RegisterEventHandler``` hay que utilizar la función ```RegisterReactionHandler``` para subscribirlo a reacciones de la interface gráfica. Poca cosa en realidad, simplemente que los desarrolladores decidieron diferenciar entre ambos tipos de sucesos. No existe un evento "botón presionado", sino que en cada ```WidgetButton``` se define el nombre de una ```Reaction```, y es a ese nombre al que se subscribe el _addon_. Por lo demás es igual, cuando se produce el suceso se llama a la función indicada en el _script_ con los parámetros adecuados en función del tipo de evento.

Por último, comentar que toda la interface gráfica propia del juego utiliza estos mismos controles que podemos utilizar nosotros para crear nuestros propios _addons_. Desgraciadamente todos los ficheros ```.xdb``` de los formularios (personaje, correo, comercio, subasta, etc) no están incluidos dentro del juego en formato de texto, sino en formato binario, dentro del fichero ```pack.bin``` en ```data/Packs/Bin.pack```, y no se pueden examinar.
