# 9. js-aruco: De 2 a 3 dimensiones

_19-03-2012_ _Juan Mellado_

He actualizado los fuentes de js-aruco con los últimos cambios en los que he estado trabajando últimamente. La principal novedad es la posibilidad de calcular la pose en tres dimensiones de un marcador a partir de su proyección en dos dimensiones. Aunque también hay otros cambios importantes, así como varias mejoras realizadas para optimizar el rendimiento.

## 9.1. Estimación de la Pose

Para calcular las poses en tres dimensiones he utilizado el método descrito en "_Iterative Pose Estimation using Coplanar Feature Points_". Este no es el método que utiliza la librería original que tomé como punto de partida, pero lo he preferido porque no requiere los parámetros de calibración de la cámara. Sus únicos parámetros de entrada son el tamaño real del marcador y la longitud focal de la cámara. El primero es sencillo, es lo que mide realmente de lado a lado el marcador (en milímetros). Y aunque el segundo suena un poco más difícil de calcular, en realidad se puede aproximar asumiendo simplemente que es igual al ancho de la imagen con la que se esté trabajando (en _pixels_).

El algoritmo devuelve dos poses estimadas caracterizadas por una matriz de rotación y un vector de traslación. Esto es así porque la proyección de un cuadrilátero en dos dimensiones puede corresponderse con dos posiciones distintas del mismo cuadrilátero en tres dimensiones. Para averiguar cual es la proyección más correcta, si es que existe alguna, se calcula el error que existe entre el modelo del marcador de partida, y el resultado de aplicar cada rotación y traslación calculada a la proyección dada. La pose estimada que produce el menor error se considera la más correcta de las dos.

He acabado realizando dos implementaciones, la primera basada en el código original utilizado por Daniel DeMenthon escrito en C, y la segunda basada en el código en C# descrito en un artículo por Andrew Kirillow. Dan resultados bastantes similares, aunque hay ciertas diferencias en el cálculo del método del error. Las he exportado con el mismo nombre, ya que la idea es utilizar un método u otro, y para ello basta con incluir ```posit1.js``` o ```posit2.js``` según cual método se prefiera. Está explicado en la _web_ del proyecto.

Como añadido, he tenido que implementar en JavaScript el algoritmo de descomposición en valores singulares de una matriz, más conocido por su siglas en inglés SVD. Para ello he seguido la implementación original descrita en "_Numerical Recipes in C - Second Edition_".

## 9.1. Stack Box Blur

Para aumentar el rendimiento general de la librería he sustituido el cálculo del _Gaussian Blur_, que se utilizaba para realizar el _Threshold Adaptativo_, por un _Stack Box Blur_.

El cálculo del _Gaussian Blur_ era la parte que más tiempo de proceso llevaba de toda la librería. Utilizaba un tamaño de kernel de 7, lo que implicaba que para cada _pixel_ de la imagen se tenían que realizar del orden de un centenar de accesos a memoria y otras tantas operaciones matemáticas. El _Stack Box Blur_ utiliza un tamaño de kernel de 2, reduce considerablemente el número de accesos a memoria utilizando una pequeña pila (_stack_), evita tener que utilizar un _buffer_ intermedio del mismo tamaño que la imagen, y gracias a una tabla precalculada apenas realiza unas pocas operaciones aritméticas por _pixel_. Y lo mejor de todo es que el resultado final es apenas indistinguible del original _Gaussian Blur_ cuando se utiliza para calcular el _Threshold Adaptativo_.

He implementado una versión adaptada a las necesidades de la librería, que en este punto concreto trabaja con imágenes en escala de grises, utilizando un sólo canal, frente a las implementaciones originales, que sólo permiten trabajar con imágenes de tres o cuatro canales.

## 9.2. Supersampling

Una de las cosas que tenía pendiente prácticamente desde que implementé la librería era mejorar el _warp_. Este proceso es el que extrae de la imagen las áreas donde se encuentran los marcadores detectados y las proyecta en cuadriláteros con la idea de reconstruir el aspecto original real de cada uno de ellos. Su principal carencia era la falta de algún tipo de interpolación entre los _pixeles_ adyacentes, por lo que las imágenes resultantes no eran todo lo buenas que podían llegar a ser cuando los marcadores estaban en ángulos "complicados" en vez de directamente enfrentados a la cámara.

Los cambios realizados han sido de dos tipos. Por una parte optimización del código existente, y por otra parte adición de _supersampling_. La optimización no ha sido demasiado difícil, ya que originalmente la función no estaba nada optimizada y ha sido fácil obtener una ganancia de rendimiento rápidamente. Desgraciadamente la implementación del _supersampling_ se ha comido la ganancia y algo más. No estoy nada contento con la implementación, se basa en el uso de un par de decenas de variables locales, y eso resulta difícil que la máquina virtual pueda optimizarlo tratando de _cachear_ valores en registros del microprocesador. La parte positiva es que las imágenes que se obtienen ahora son de muchísima más calidad, con bordes rectos en vez de dentados como ocurría antes.

## 9.3. Rendimiento

Utilizando como referencia la última demo, el rendimiento global del sistema en mi equipo es de 60 FPS estables con un consumo de entre el 7 y el 9 por ciento de CPU. Los FPS están perfectos, ya que es el máximo que permite el navegador, y la CPU no está demasiado mal, aunque me haría un poco más feliz ver una cantidad menor ahí. Afortunadamente creo que aún hay bastante margen de mejora.
