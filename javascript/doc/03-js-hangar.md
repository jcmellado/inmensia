# 4. Hangar

_13-02-2012_ _Juan Mellado_

Hoy he liberado los fuentes del último proyecto en JavaScript en el que he estado trabajando. Es un visor de modelos 3D almacenados en formato AC3D (```.ac```) que utiliza WebGL para renderizar.He estado utilizando modelos de aviones de FlightGear para probarlo, así que he decidido llamarlo Hangar.

- [https://github.com/jcmellado/hangar](https://github.com/jcmellado/hangar)

He creado una pequeña página _web_ con cuatro modelos escogidos de prueba. Tiene implementado un controlador a modo de _trackball_ que permite rotar y ampliar los modelos utilizando el botón izquierda y la rueda del ratón. Necesita un navegador moderno como las últimas versiones de Chrome o Firefox para funcionar (Y no, IE 9 no es un navegador moderno)

![Hangar](img/03-js-hangar.png "Hangar")

El formato AC3D lo utilizan algunos programas, como FlightGear, un simulador aéreo de código abierto, o Torcs, un simulador de conducción deportiva también de código abierto. Es un formato de texto plano que originalmente creó un programa de modelado 3D llamado, lógicamente, AC3D.

Mi idea era hacer algo sencillo, pero al final he acabado incorporando un montón de esas pequeñas características que pasan desapercibidas cuando funcionan bien.

## 4.1. AC3D

Los ficheros ```.ac``` tienen dos partes, la primera es una lista de materiales y la segunda una lista de objetos. Los materiales incluyen los típicos atributos para ambiente, emisión, difusa, especular, brillo y transparencia. Los objetos son agrupaciones de entidades que reciben el nombre de superficies, aunque en realidad, además de polígonos, también pueden ser líneas simples o cerradas (el último punto se une con el primero). A mi me interesaban sólo los polígonos, pero al final he añadido soporte para representar las líneas también.

Los objetos están definidos de una manera jerárquica, de forma que un objeto puede tener hijos. Además, cada objeto tiene una matriz de rotación y un vector de rotación que aplica a sí mismo y a todos su hijos, por lo que hay que ir calculando la transformación resultante a medida que se baja por el árbol de descendencias.

## 4.2. Teselación

Debido a que para renderizar con WebGL es mejor tener las superficies divididas en triángulos, he implementado una versión en JavaScript de un algoritmo de teselación que soporta tanto polígonos convexos como cóncavos. Funcionó bastante bien casi a la primera, aunque haciendo pruebas detecté un problema con el orden en que devolvía los índices con modelos que tenían definidas las caras con los vértices en el sentido de la agujas del reloj, y que afortunadamente puede solucionar fácilmente. Me gustaría escribir en el futuro un articulo individual sobre este tema en particular.

Me he encontrado con modelos de todo tipo y definidos de casi cualquier forma. Al final he decidido ignorar los polígonos degenerados. Es decir, polígonos con menos de tres vértices, polígonos con coordenadas colineales, polígonos con vértices no contenidos dentro de un único plano, polígonos con aristas que se cruzan, etcs

## 4.3. Normales

El formato del fichero no incluye normales, por lo que se calculan _online_ mediante JavaScript. Por una parte la normal a cada superficie, para los sombreados planos. Y por otra parte la normal a cada vértice, para los sombreados más realistas.

La normal a un vértice concreto se calcula como la suma de todas las normales de todas las caras que comparten dicho vértice. Aunque a este respecto el fichero incorpora un parámetro por objeto llamado ```crease```. Este parámetro es un ángulo, y sirve para generar lo que habitualmente se conocen como bordes duros (_hard edges_). Si el ángulo que forman la normal de una cara y su vecina supera dicho parámetro entonces esa normal vecina no se tiene en cuenta para calcular la normal en el vértice compartido.

## 4.4. SGI

Bastantes modelos en formato AC3D utilizan texturas en formato SGI. Un formato antiguo que no soportan directamente los navegadores actuales. Una solución era transformar las texturas a PNG, pero después de leer la especificación del formato me decidí a implementar también un lector de este tipo de imágenes en JavaScript, ya que el método de compresión que utilizan es un simple RLE. ¡Un proyecto dentro de otro proyecto!

Mi implementación soporta ```.sgi```, ```.rgba```, ```.rgb```, ```.ra``` y ```.bw```. Es decir, todas las combinaciones posibles, aunque he ignorado deliberadamente algunas características de la especificación, como la posibilidad de usar paletas de colores o escoger entre 8 ó 16 _bits_ por canal. En la práctica todas las imágenes acaban siendo de cuatro canales y con 8 _bits_ por canal en el clásico formato RGBA.

## 4.5. Loader

Para agilizar la carga y proceso de las texturas, que son binarias, al contrario que los modelos que son de texto, he utilizado una de las nuevas características del objeto ```XMLHttpRequest``` de HTML5 que permite descargar un fichero directamente a un ```ArrayBuffer```. Muy útil.

## 4.6. Renderer

Como modelo de iluminación he utilizado una versión propia basada en el clásico de Phong con algunas modificaciones. Como los colores se me saturaban bastante, debido a como están definidos por lo general los materiales, he utilizado el valor de las texturas para ponderar la suma de emisiva, ambiente y difusa. De igual forma, para evitar los brillos exagerados he tomado sólo un 30% de la reflexión especular. Es más un modelo de prueba y error buscando un resultado agradable a la vista que uno realista. Es uno de mis puntos flacos.

El formato AC3D permite incluir luces en los modelos, pero las he ignorado a la hora de renderizar, utilizando en cambio una luz fija estática direccional sin factores de atenuación. Mi idea era representar modelos de objetos independientes, no escenas completas, por lo que no he encontrado sentido en incluirlas. Aparte de que acabado no fiándome de los modelos, con materiales un tanto extraños a mi parecer.

Para minimizar los cambios de estado de la tarjeta gráfica he agrupado todos los polígonos por programa, material, tipo, y todo lo que se me ha ocurrido. Por ello, a diferencia de otros proyectos anteriores, he decidido utilizar ```drawArrays``` en vez de ```drawElements``` para mi comodidad, aunque fuera a costa de repetir información al no utilizar índices.

## 4.7. Transparencias

Las transparencias han sido un dolor como de costumbre. Al final me he ceñido al guión. Primera pasada para los objetos opacos, con el _blending_ desactivado. Segunda pasada para los objetos transparentes, con el _blending activado_ y la escritura al _depth buffer_ desactivada. Lo único que no he hecho es ordenar en profundidad, no me ha hecho falta en ninguna de las pruebas que he realizado.

Lo que me duele es que se supone, repito, "se supone", que los materiales incluyen un atributo para indicar si tienen algún tipo de transparencia. Desgraciadamente esto no ocurre así en la práctica. Al final cuando algún modelo no se visualizaba correctamente, abría el fichero, localizaba el material y lo cambiaba a mano.

Esto es algo que en general creo que tendría que revisar con el objetivo de entender la manera en que interpretan los materiales programas como FligthGear o Torcs aprovechándome del hecho de que son de código abierto.

## 4.8. Autofit

Uno de los inconvenientes de hacer un visor genérico es que cada modelo viene posicionado y orientado según decidió su autor. Con centrarlos en el eje de coordenadas y alejarse una distancia prudencial prefijada no es suficiente, ya que depende del tamaño de cada modelo en particular. Para solventar este problema he implementado un auto-ajuste a la pantalla de los modelos basados en el tamaño de su _bounding box_.

De esto también me gustaría escribir un artículo individual en el futuro, ya que he encontrado distintas soluciones en Internet y no me he quedado del todo satisfecho con la que yo mismo he implementado.

## 4.9. TrackBall

Una vez dibujados los modelos lo lógico es poder interactuar con ellos. Ya había implementado anteriormente un simple controlador, cuando hice la demo de js-openctm, pero era un _fake_, así que esta vez he implementado un algoritmo de _trackball_ de verdad. No quiero acordarme de las horas que habré perdido por culpa de esto. No es que no viera la solución, ¡es que no veía el problema!.

## 4.10. Resumiendo

¿Mucho tiempo invertido? Sí. ¿Suficiente? ¡Nunca!.

Una aproximación más práctica al problema hubiera sido utilizar un motor 3D ya existente, como el popular Three.js, y limitarme a hacer un conversor del formato ```.ac``` a algún formato que acepte el motor. Pero claro, ¡me hubiera perdido la parte más divertida!
