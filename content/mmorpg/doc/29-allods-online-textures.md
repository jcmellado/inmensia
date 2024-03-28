# 29. Allods Online Addons (5/10): Texturas

_11-09-2010_ _Juan Mellado_

Los _addons_ permiten utilizar texturas propias para construir interfaces personalizadas de usuario. Las texturas tienen que estar en formato ```.tga```, pero en los _addons_ se tienen que incluir en formato ```.bin```, que es el formato propio interno que utiliza el juego. Para convertir de ```.tga``` a ```.bin``` existe una herramienta que viene incluida con el propio juego.
Las texturas forman parte de las interfaces gráficas de los _addons_, como componentes hijos, o de apoyo a los formularios. Y como todo elemento que interviene dentro de un formulario, deben estar descritas en su correspondiente fichero ```.xdb```. La herramienta de conversión genera tanto el fichero ```.bin``` como los ficheros ```.xdb``` necesarios para que se pueda utilizar desde un formulario.

Dentro del directorio ```bin``` del directorio donde se encuentra instalado el juego está el programa ```UITextureConvertEditor.exe```. Al ejecutarlo se muestra la siguiente ventana (textos originales en ruso, traducción libre por mi cuenta y riesgo):

![UITextureConvertEditor](img/05-textures.png "UITextureConvertEditor")

El desplegable de "Tipo de textura" permite seleccionar entre los siguientes valores:

- **WidgetLayerSimpleTexture**: Esta es la opción más común y que normalmente se utilizará. Exporta la textura a ```.bin``` y crea tres ficheros ```.xdb``` con los tipos: ```WidgetLayerSimpleTexture```, ```UITextureElement``` y ```TextureSingleElement```.

- **WidgetLayerTiledTexture**: Esta es la opción habitual que se utiliza para las imágenes de fondo de las ventanas de los formularios. Exporta la textura a ```.bin``` y crea tres ficheros ```.xdb``` con los tipos: ```WidgetLayerTiledTexture```, ```UISingleTexture``` y ```UITexture```.

- **UISingleTexture**: Esta opción es para generar texturas individuales, para que se puedan reutilizar. Aunque normalmente se generarán automáticamente como resultado de seleccionar la opción anterior.

- **UITextureElement**: Esta opción es para generar un control con textura que se va a utilizar dentro de un formulario específico. Se genera automáticamente cuando se selecciona el primer tipo del desplegable.

El cuadro de "Opciones" da un mayor grado de control sobre la conversión:

- **Usar formato DXT3 en vez de DXT5**: Permite seleccionar el tipo de compresión a aplicar a la textura. El formato DXT3 proporciona por lo general una mayor calidad en los contornos. pero sacrificando algo la calidad en las transparencias. El formato DXT5 ofrece una mayor calidad en las transiciones de las transparencias a costa de algo de calidad en los contornos.

- **Sobreescribir si ya existe**: Fuerza a que el programa genere siempre los ficheros, independientemente de que estos ya existan en el directorio destino.

El botón "Seleccionar" permite elegir que texturas ```.tga``` se quieren convertir. Todas las texturas seleccionadas se añaden a la lista que se muestra en la ventana. Una manera totalmente equivalente de hacer esto es seleccionar una textura desde el explorador de _Windows_ y arrastrarla directamente con el ratón sobre la ventana.

Finalmente, el botón "Convertir" es el que realiza el proceso de conversión de todas las texturas seleccionadas. Por cada textura del listado se muestra un "Estado", afortunadamente en inglés. Inicialmente tienen valor ```added``` (añadida), y dependiendo del resultado de la conversión cambian a ```converted``` (convertida) o ```failed``` (error).

Unas cuantas cosas importantes a tener en cuenta:

- Se recomienda que las dimensiones de las texturas, ancho y alto, sean múltiplos de 2. Esto es una cuestión de carácter puramente técnica que nada tiene que ver con el juego.

- Las texturas a convertir tienen que estar dentro del directorio ```data``` donde se encuentra instalado el juego. Si no se encuentran directamente en ese directorio, o en una que cuelgue de esa ruta, el programa fallará al intentar convertir las texturas.

- No se puede reintentar convertir una textura que se acabe de convertir, independientemente de que la primera vez fuera bien o mal. Hay que cerrar el programa y volver a abrirlo repitiendo el proceso completo.

- En los _addons_ sólo se debería incluir los ```.bin``` generados, no es necesario distribuir los ```.tga``` originales.

En el _addon_ ```SampleReactionHandler``` que viene de ejemplo con el juego puede verse como se utiliza una textura llamada ```MainFrame.tga``` como imagen de fondo de la ventana principal del _addon_. Es un ejemplo muy sencillo que a estas alturas no debería costar mucho de entender.

Por último, comentar que las texturas se pueden agrupar de forma similar a los textos utilizando un fichero ```.xdb``` de tipo ```UIRelatedTextures```, para luego hacer referencia a las mismas desde los _scripts_ mediante la función ```GetAddonRelatedTexture```.
