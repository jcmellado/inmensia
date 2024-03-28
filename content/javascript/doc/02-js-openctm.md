# 2. js-openctm: Una implementación de OpenCTM en JavaScript

_16-04-2011_ _Juan Mellado_

OpenCTM es un formato de fichero que se utiliza para empaquetar mallas de triángulos de modelos tridimensionales, de manera similar a como lo hacen otros formatos más populares como 3DS (```.obj```), COLLADA (```.dae```), etc. Y js-openctm es el nombre de una librería que he desarrollado en JavaScript y que es capaz de leer este tipo de ficheros.

- [https://github.com/jcmellado/js-openctm](https://github.com/jcmellado/js-openctm)

La idea de implementar esta librería nació a raíz de unas pruebas que estuve haciendo con WebGL. Me surgió la necesidad de leer un modelo 3D de un fichero, y buscando información acerca de los distintos formatos me topé con este OpenCTM. Me llamó la atención el SDK, así como las herramientas de visualización y conversión que tiene implementadas, y me pareció un reto interesante intentar portar partes del SDK, escrito originalmente en C, a JavaScript.

El formato OpenCTM está bastante documentado, a excepción de una parte referida a como se empaquetan las normales, pero en líneas generales se puede entender todo sin demasiados problemas leyendo el código fuente y la documentación al mismo tiempo. Su principal característica es que saca partido del conocimiento del dominio para obtener unos ratios de compresión bastantes grandes. Por ejemplo, sabiendo que las mallas se describen con vértices, índices, normales, etc, lo que hace es reordenar dichos componentes para favorecer la compresión de los mismos. Por ejemplo, si se utiliza un entero de 32 bits para los índices, será bastante normal que el _byte_ más significativo sea cero, por lo que es mejor almacenar todos esos _bytes_ de forma consecutiva para obtener una larga cadena de ceros que cualquier compresor puede detectar y almacenar de forma muy eficiente. Y lo mismo para el resto de _bytes_, ya que si bien inicialmente serán todos ceros, luego pasarán todos a valor uno, luego a dos, y así sucesivamente. De igual forma, si en vez de almacenar los valores tal cual se guardan las diferencias entre ellos, se favorece la compresión, ya que estas representan valores más pequeños y que tienden a ser iguales.

OpenCTM define un formato de almacenamiento en binario con tres niveles de compresión llamados RAW, MG1 y MG2. El formato RAW simplemente almacena los distintos componentes de la malla en binario. El formato MG1 reordena, almacena diferencias, y utiliza LZMA para comprimir. Y por último, el formato MG2, además de LZMA, implementa un proceso de empaquetamiento más elaborado para obtener un mayor nivel de compresión.

Naturalmente, la ganancia en compresión se paga en el proceso de descompresión, bastante costoso en algunos casos para JavaScript. La librería js-lzma que escribí para descomprimir es bastante lenta en su estado actual, y los cálculos que tiene que realizar js-openctm para restaurar la malla original en algunos casos implica realizar bucles que recorren las listas completas de vértices e indices.

![js-openctm](img/02-js-openctm.png  "js-openctm")

En la imagen pueden verse los distintos modelos que he utilizado para ir probando la librería, dibujados directamente en el navegador utilizando WebGL. La primera figura es el famoso conejo de Stanford. Sólo índices y vértices ahí, sin ningún tipo de iluminación, de ahí que parezca una imagen 2D cuando en realidad es 3D. La segunda figura es un neumático de Audi, los colores en los vértices aportan algo de profundidad. El símbolo de Audi con los círculos entrelazados en el centro no es una textura, sino que la malla es muy densa. La tercera figura es el dragón de Stanford, la utilicé para probar la carga de normales, por lo que apliqué un shader muy sencillo con una luz que llega desde arriba para comprobar que se cargaban correctamente. La última figura es de una ambulancia con muy pocos polígonos pero con una textura bastante bien elaborada y muy resultona. Me sirvió para probar que las coordenadas de textura se estaban cargando de forma correcta.

En cuanto al uso en sí de la librería, está pensada para ser utilizada a partir de ficheros recuperados de la web con ```XMLHttpRequest``` de la forma habitual, utilizando el objeto request obtenido para crear un stream del que se obtendrá la información contenida en el fichero:

```javascript
var stream = new CTM.Stream(request.responseText);

var file = new CTM.File(stream);
```

Los ficheros ```.ctm``` tienen cabecera y cuerpo, y esto es lo que se obtiene en la variable ```file``` del ejemplo, en forma de dos atributos públicos que exponen los objetos de tipo ```CTM.File```. Un primer atributo ```header``` con el método de compresión, el número de vértices de la malla, el número de triángulos, y otra serie de parámetros descriptivos. Y un segundo atributo ```body``` con una serie de ```Typed Arrays``` que contienen los índices, vértices, normales, y el resto de atributos que tiene la malla, listos para ser utilizados directamente con WebGL.

El siguiente gran paso sería optimizar el rendimiento de las librerías, lo que representa una buena oportunidad de profundizar en el manejo de las herramientas de _profiling_ de Chrome, que es el navegador con el que estoy realizando las pruebas de todas estas tecnologías que siguen siendo bastante experimentales hoy en día.
