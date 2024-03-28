# 1. Ray Tracing: Introducción

_06-10-2005_ _Juan Mellado_

Ray Tracing es el nombre de una técnica muy popular utilizada para generar imágenes 3D por ordenador, destacando por la sencillez de su implementación y la calidad de los resultados.

Para entender el principio de funcionamiento y las ventajas que ofrece la técnica de Ray Tracing hay que tratar primero de comprender el proceso general por el que se forman las imágenes. Y me estoy refiriendo sólo a la impresión de las mismas en nuestras retinas, no a su interpretación por el cerebro.

Simplificando bastante la explicación, se puede decir que todo empieza en las fuentes de luz. Los focos emisores, como pueden el sol o una simple cerilla, emiten rayos que se propagan a través del medio en el que se encuentran. Cuando uno de estos rayos se tropieza en su camino con un objeto puede ocurrir que la energía transportada sea absorbida o reflejada. Dependiendo de las características del objeto, ocurrirá que el rayo continuará su camino a través suyo, caso de los cristales, o saldrá reflejado, caso de los espejos. Aunque normalmente ocurrirán ambos fenómenos en cierta medida, apareciendo nuevos rayos, algunos transmitidos a su través, y otros reflejados, que a su vez volverán a propagarse por el medio y rebotar contra los objetos formando nuevos rayos. Y así sucesivamente hasta que finalmente algunos de estos rayos acaben tropezándose con nuestros ojos, generando la combinación de todos ellos la imagen que representa lo que percibimos de nuestro alrededor.

Ray Tracing simula de una forma muy simplificada este proceso de interacción de la luz con los objetos. Y una de sus características fundamentales es que, en vez de tomar como origen del proceso las fuentes de luz, y tratar de emitir rayos en todas las direcciones posibles, lo que hace es tomar como punto de partida al observador, y trazar desde él rayos en busca de las fuentes de luz. La ventaja de este enfoque es que simplifica enormemente el sistema, al evitar tener que estudiar todos los rayos posibles, y limitarse sólo a los rayos que llegan al observador, que son los que realmente aportan algo a la imagen final generada.

## 1.1. Algoritmo

Ray Tracing, al fin y al cabo, no es más que un algoritmo. Admite unas entradas, realiza un proceso y genera unas salidas.

Como entradas del algoritmo estarían la posición y características físicas de cada uno de los objetos que forman una escena. Seguiría la ubicación, intensidad y tipo de las fuentes de luz presentes en la misma. Después el modelo de iluminación a utilizar, que describiría como calcular el color en un punto de la superficie de un objeto. A continuación la posición y dirección desde la que se quiere representar la escena, es decir, la posición del observador. Y en último lugar la ubicación y dimensiones del plano de proyección sobre el que ha de quedar reflejada la imagen, que normalmente equivale a la pantalla del ordenador donde se verá finalmente el resultado.

En cuanto al algoritmo en sí mismo, decir que el proceso básico se basa en trazar un rayo a través de cada uno de los píxeles del plano de proyección, tomando como origen de los rayos la posición del observador. Y para cada uno de estos rayos trazados estudiar si se produce una intersección con alguno de los objetos de la escena. Si no se encuentra ningún objeto se termina el proceso para ese píxel y se le asigna un color de fondo por defecto. Por el contrario, si se encuentra un objeto, se miran sus características físicas y se decide que hacer en función del modelo de iluminación.

En la práctica, todos los elementos que intervienen en el proceso se tratan como entidades matemáticas muy simples. Los objetos se definen mediante primitivas geométricas como esferas, planos, o polígonos, aunque se tiende a utilizar mallas de triángulos, ya que asi funcionan la mayoría de programas de modelado actuales. Los rayos se tratan como vectores, con un punto de origen y una dirección de propagación. Y por tanto, el cálculo de intersecciones entre rayos y objetos se reduce a la resolución de ecuaciones en las que intervienen vectores y primitivas geométricas.
