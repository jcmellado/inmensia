# 26. Allods Online Addons (2/10): Estructura

_21-08-2010_ _Juan Mellado_

Los _addons_ de _Allods Online_ son pequeños programas que se componen de las distintas partes de las que habitualmente consta una aplicación: interface de usuario, código de proceso, ficheros de recursos, y librerías adicionales. El conjunto de todos estos componentes es lo que se conoce como _addon_.

Para crear un _addon_ en _Allods Online_ hay que crear un directorio donde se encuentra instalado el juego, dentro de ```data/Mods/Addons```, y poner en él todos los ficheros que forman el _addon_. Por ejemplo, si queremos crear un _addon_ llamado ```Prueba```, debemos crear un directorio ```data/Mods/Addons/Prueba``` y poner dentro de él todos los ficheros. Aunque también es posible crearlos con varios niveles de sub-directorios, como ```data/Mods/Addons/Pruebas/Prueba01```.

¿Pero qué ficheros forman un _addon_? Pues lo mejor es verlo con un ejemplo real. Con uno que viene con el juego y se llama ```SampleInit```. Se encuentra en el directorio ```Mods/SampleAddons/SampleInit```, y consta de tres ficheros:

```text
AddonDesc.(UIAddon).xdb
MainForm.(WidgetForm).xdb
ScriptSampleInit.lua
```

El primer fichero es obligatorio para todos los _addons_ y tiene que tener siempre ese nombre fijo de ```AddonDesc.(UIAddon).xdb```. Es un XML en formato UTF-8 con una serie de etiquetas obligatorias que describe al _addon_.

La etiqueta ```<Name>``` contiene el nombre del _addon_.

```xml
<Name>SampleInit</>
```

La etiqueta ```<AutoStart>``` indica si el _addon_ tiene que ejecutarse automáticamente al arrancar el juego. Es un simple valor _booleano_.

```xml
<AutoStart>True</>
```

La etiqueta ```<ScriptFileRefs>``` contiene la lista de ficheros con los _scripts_ de Lua que utiliza el _addon_. Cada entrada de la lista contiene el nombre de un fichero, que puede ser absoluto con respecto al directorio ```data``` del juego, o relativo a la ruta donde se encuentra el ```addon```:

```xml
<ScriptFileRefs>
  <Item href="/Mods/SampleCommon/SampleAddonBase.lua" />
  <Item href="ScriptSampleInit.lua" />
</ScriptFileRefs>
```

La etiqueta ```<Forms>``` contiene la lista de formularios que utiliza el _addon_. Cada elemento de la lista se compone de un identificador único y una referencia a un fichero ```.xdb``` donde se encuentra descrito el formulario correspondiente. Se utiliza XPointer, que es una notación estándar que permite desde un fichero XML hacer referencia a una sección de otro fichero XML.

```xml
<Forms>
  <Item>
    <Id>Main</Id>
    <Form href="MainForm.(WidgetForm).xdb#xpointer(/WidgetForm)"/>
  </Item>
</Forms>
```

En este último ejemplo se define un formulario identificado como ```Main```, y que se encuentra definido dentro de la sección ```WidgetForm``` de un fichero llamado ```MainForm.(WidgetForm).xdb```.

La etiqueta ```<MainFormId>``` contiene el nombre del formulario principal del _addon_, que debe ser un identificador definido dentro de la lista contenida en la etiqueta ```<Forms>```:

```xml
<MainFormId>Main</MainFormId>
```

De igual forma, es posible añadir otras etiquetas predefinidas, o incluso propias, como el nombre del autor, la versión de _addon_, o la fecha de última actualización, aunque en la documentación no se indica más detalle acerca de ellas.

El fichero ```AddonDesc.(UIAddon).xdb``` de este _addon_, al completo, contiene lo siguiente:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<UIAddon>
  <Name>SampleInit</Name>
  <AutoStart>true</AutoStart>
  <ScriptFileRefs>
    <Item href="/Mods/SampleCommon/SampleAddonBase.lua" />
    <Item href="ScriptSampleInit.lua" />
  </ScriptFileRefs>
  <MainFormId>Main</MainFormId>
  <Forms>
    <Item>
      <Id&gtMain&lt/Id>
      <Form href="MainForm.(WidgetForm).xdb#xpointer(/WidgetForm)" />
    </Item>
  </Forms>
  <visObjects href="" />
  <texts href="" />
  <textures href="" />
</UIAddon>
```

A estas alturas ya debería adivinarse el patrón que sigue el nombre de los ficheros ```.xdb```, con un título (```AddonDesc```, ```MainForm```, etc) seguido de un tipo (```UIAddon```, ```WidgetForm```, etc). Estos tipos están predefinidos y sirven para indicar la clase de información que contiene el fichero, que además de información en formato de texto también admite ficheros de recursos binarios como pueden ser texturas.

A continuación el fichero ```MainForm.(WidgetForm).xdb```, y que resulta bastante fácil de entender leyendo entre líneas:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<WidgetForm>
  <Name>SampleInit</Name>
  <Priority>0</Priority>
  <Children>
  </Children>
  <BackLayer href="" />
  <FrontLayer href="" />
  <Placement>
    <QuantumScale>false</QuantumScale>
    <X>
      <Align>WIDGET_ALIGN_LOW</Align>
      <Sizing>WIDGET_SIZING_DEFAULT</Sizing>
      <Pos>0</Pos>
      <HighPos>0</HighPos>
      <Size>0</Size>
    </X>
    <Y>
      <Align>WIDGET_ALIGN_LOW</Align>
      <Sizing>WIDGET_SIZING_DEFAULT</Sizing>
      <Pos>0</Pos>
      <HighPos>0</HighPos>
      <Size>0</Size>
    </Y>
  </Placement>
  <Visible>true</Visible>
  <Enabled>true</Enabled>
  <TransparentInput>true</TransparentInput
  <PickChildrenOnly>false</PickChildrenOnly>
</WidgetForm>
```

Y por último, el contenido del fichero ```ScriptSampleInit.lua```, que se limita a dejar constancia de su arranque en el fichero de _log_ por defecto ```mods.txt``` en el directorio ```Personal/Logs```:

```lua
-----------------
-- INITIALIZATION
-----------------
function Init()
  LogInfo("Initialization sample")
end
-----------------
Init()
-----------------
```

Para probar este simple _addon_ de ejemplo basta con copiar el directorio ```SampleInit``` de ```data/Mods/SampleAddons``` a ```data/Mods/Addons```. Al arrancar el juego aparecerá un nuevo botón, en la ventana de _login_, que nos permite activar o desactivar los _addons_, que por defecto se encuentran desactivados. Al entrar en el juego propiamente dicho, después de la ventana de selección de personaje, veremos como se graba el mensaje en el fichero de _log_, señal de que el _addon_ se encuentra correctamente instalado y funcionando.
