# 28. Allods Online Addons (4/10): Textos

_04-09-2010_ _Juan Mellado_

Aunque es posible escribir _addons_ que sólo contengan código, como los ejemplos vistos hasta ahora, lo normal es incluir algún tipo de texto o gráfico. La documentación oficial dice al respecto que, además de ficheros con extensión ```.xdb``` y ```.lua```, se pueden incluir también ficheros con extensión ```.txt``` y ```.bin``` como parte de un _addon_.

Los ficheros ```.txt``` deben contener los textos que se quieran mostrar a los jugadores, y deben estar escritos en formato UTF-16LE. Usar este formato permite utilizar todo tipo de juego de caracteres especiales para cada idioma, como el cirílico por ejemplo. Colocar los textos en un fichero independiente facilita su rápida localización por parte de cualquier persona, sin tener que andar revisando y modificando el código fuente de los _scripts_ Lua.

La documentación es un poco parca con respecto al formato de los ficheros de texto, así que lo mejor es echar un vistazo a los _addons_ que vienen con el juego. Por ejemplo, en el directorio del addon ```SampleReactionHandler``` pueden verse tres ficheros con extensión ```.txt```: ```ButtonExecute.txt```, ```Header.txt``` y ```Text.txt```. El primero se corresponde al texto que se muestra en un botón, y sólo contiene el texto ```Ejecutar``` (los textos originales están en ruso). Los otros dos contienen los textos que se muestran en el título y cuerpo de una ventana respectivamente. No obstante, la diferencia de estos dos últimos con respecto al primero es que contienen una etiqueta XML con un atributo para indicar la alineación horizontal deseada. Por ejemplo, el fichero ```Header.txt``` contiene lo siguiente:

```xml
<header alignx="center">Título</header>
```

Como ya he comentado, no hay mucha información acerca de las etiquetas y atributos permitidos, aunque buscando con un poco de imaginación en la documentación del API se pueden encontrar algunos. Por ejemplo, en la función ```SetPlacementPlain```, que sirve para cambiar la posición de un control, se pueden ver atributos de posicionamiento y alineamiento.

Además de los textos prefijados que aparecen en los controles de la interface gráfica, como los botones, también es posible definir una serie de textos libres utilizando un fichero ```.xdb``` de tipo ```UIRelatedTexts```. Esta clase de ficheros contienen una simple lista en el que cada elemento tiene un nombre y una referencia a un fichero de texto. Sirva de ejemplo un fichero, al que llamaré ```Texts.(UIRelatedTexts).xdb```, con el siguiente contenido:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
  <UIRelatedTexts>
    <items>
      <Item>
        <name>Texto01</name>
        <resource href="Texto01.txt" />
      </Item>
      <Item>
        <name>Texto02</name>
        <resource href="Texto02.txt" />
      </Item>
   </items>
</UIRelatedTexts>
```

El fichero de ejemplo contiene una lista de dos elementos. No obstante, para que esos dos textos sean visibles dentro del _addon_, es necesario añadir una referencia a ellos en ```AddonDesc.(UIAddon).xdb```:

```xml
...
<texts href="Texts.(UIRelatedTexts).xdb" />
...
```

Y de esta forma desde el código de un script Lua se podrá obtener el texto de cualquiera de ellos por su nombre utilizando la función ```common.GetAddonRelatedText(nombre)```:

```lua
...
texto = common.GetAddonRelatedText("Texto01")
...
texto = common.GetAddonRelatedText("Texto02")
...
```

En otro de los _addons_ que vienen de ejemplo con el juego, el llamado ```SampleZoneAnnounce```, se puede ver un fichero ```Announce.txt``` que contiene lo siguiente:

```xml
<header color="0xFFCC1111" alignx="center" fontsize="36" shadow="1">
  <rs class="class">
    <r name="value"/>
  </rs>
</header>
```

En este ejemplo se establece el color, el alineamiento, tamaño de fuente, y efecto de sombreado. Establecer este tipo de atributos en ficheros separados de los _scripts_ también es una estrategia acertada, ya que permite modificar la presentación sin tener que tocar la lógica de proceso. Otra curiosidad de este ejemplo es que no utiliza una cadena de texto fijo, sino que se deja abierto para que el valor a mostrar en el control se establezca desde el código de un _script_ en tiempo de ejecución.

Naturalmente, cuando uno empieza a trastear con este tipo de cosas, lo que a uno le interesa es mostrar un texto por pantalla y cambiarlo de forma dinámica. Para ver como se hace esto es mejor recurrir de nuevo al _addon_ de ejemplo ```SampleZoneAnnounce```. Lo primero que hace es añadir en el fichero ```MainForm.(WidgetForm).xdb```, que describe el formulario principal del addon, una etiqueta de texto hija llamada ```Announce```:

```xml
...
<Children>
  <Item href="Announce.(WidgetTextView).xdb#xpointer(/WidgetTextView)" />
</Children>
...
```

Todos los controles, ya sea el formulario principal o alguno de sus hijos, tienen que tener su propio fichero ```.xdb``` que lo describa. Y así, el control con la etiqueta de texto hija se describe en el fichero ```Announce.(WidgetTextView).xdb```. Como se observa, se utiliza el tipo ```WidgetTextView``` para indicar que es un control de texto, a diferencia del formulario principal que utiliza el tipo ```WidgetForm```. Existe un tipo distinto para cada control, y mediante la referencia de unos dentro de otros de forma jerárquica se pueden construir interfaces bastantes elaboradas.

El siguiente paso es añadir en el fichero ```Announce.(WidgetTextView).xdb``` una referencia al fichero que contiene el texto que debe mostrarse en la etiqueta:

```xml
...
<FormatFileRef href="Announce.txt" />
...
```

Como se observa, se utiliza una etiqueta llamada ```FormatFileRef```. Es decir, que a lo que se hace referencia es a un fichero de formato, no a un simple fichero de texto, aunque su extensión haga pensar lo contrario. Algo totalmente coherente con el contenido del fichero, que como ya se ha visto, contiene tanto la cadena de texto como el formato que tiene que aplicársele.

El último paso es obtener una referencia al control hijo ```Announce```, de tipo etiqueta, declarado en el formulario principal, y cambiar su valor de forma dinámica, en el _script_ ```ScriptSampleZoneAnnounce.lua```:

```lua
...
wtMessage = mainForm:GetChildChecked("Announce", false)
...
wtMessage:SetVal("value", "Texto que se quiere mostrar")
...
```

La variable ```mainForm``` es global, y la instancia automáticamente el juego, no es necesario que lo hagamos nosotros. La función ```GetChildChecked``` obtiene una referencia al control cuyo nombre se pasa como primer argumento. El segundo argumento indica si se tiene que buscar recursivamente por toda la jerarquía de controles (```true```), o sólo los hijos directos al padre dado (```false```). La función ```SetVal``` es la que cambia el valor del texto que se muestra en el control.

Como último comentario, para que los que como yo sean nuevos en esto de Lua, decir que usar los dos puntos (```:```) en vez del punto (```.```) en una expresión de la forma ```objeto:funcion(parametros)``` es equivalente a ```objeto.funcion(self, parametros)```. Es decir, es la manera de invocar "métodos" sobre objetos concretos, en vez de invocar "funciones" de librerías. El primer parámetro de un "método" es un argumento explícito llamado ```self```, al que Lua le asigna el valor del objeto sobre el que se realiza la llamada automáticamente cuando se utiliza la notación de los dos puntos.
