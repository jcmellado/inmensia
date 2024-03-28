# 4. Second Life: Creación de scripts

_21-09-2007_ _Juan Mellado_

Second Life ofrece la posibilidad de crear _scripts_ a través del "Linden Scripting Language" (LSL), un lenguaje con una sintaxis muy similar a la de C/C++ o Java, y a través del que se tiene acceso a unas 250 funciones del API público. De igual forma que con los objetos, los _scripts_ pueden crearse de forma _online_ desde el propio _viewer_ a través de un sencillo editor de textos que lleva incorporado, aunque también cabe la posibilidad de hacerlo de forma externa con herramientas independientes como LSL-Editor.

Los _scripts_ escritos en LSL se convierten a _bytecode_ cuando se almacenan en Second Life, momento en que se muestran los posibles errores de sintaxis cometidos. Una vez almacenados aparecen en el inventario y pueden asignarse a cualquier tipo de objeto, que actuará como "contenedor" del mismo reaccionando en función de lo indicado en el _script_.

Uno de los _scripts_ más simples que puede hacerse con LSL, y que de hecho es la plantilla que aparece por defecto cuando se crea un _script_ nuevo, es el famoso "Hello, World!", rebautizado como "Hello, Avatar!":

```lsl
default
{
  state_entry()
  {
    llSay(0, "Hello, Avatar!");
  }
  touch_start(integer total_number)
  {
    llSay(0, "Touched.");
  }
}
```

A poco que se eche un vistazo al código, puede identificarse claramente la orientación del modelo de programación a estados y eventos. El motor de ejecución de _scripts_ se basa en una máquina de estados que busca en cada transición el correspondiente estado destino dentro del _script_, siendo obligatorio que todo _script_ tenga un estado por defecto, de nombre ```default```, que se ejecutará automáticamente cada vez que se inicie el mismo. Los estados permiten encapsular funciones propias para uso interno, o _callbacks_ receptoras de eventos, siendo los eventos los que permiten a un _script_ reaccionar a las acciones del mundo externo, como por ejemplo el contacto (```touch```) de un avatar con el objeto contenedor del _script_. La capacidad de tener varios estados dentro de un mismo _script_ permite que este pueda reaccionar de forma distinta a un mismo evento en función del estado en que se encuentre.

LSL admite varios tipos básicos de datos, como ```integer```, ```float```, ```string```, ```list```, ```vector```, ```rotation``` (_quaterniones_), y ```key``` (UUIDs). El flujo de control se realiza a través de sentencias como ```if```, ```for```, ```while```, ```do-while```, ```jump```, ```return```, y ```state```. Y las funciones del API, que comienzan con ```ll``` (Linden Lab), ofrecen un amplio abanico de funcionalidades, incluyendo matemáticas (```llAbs```, ```llFloor```, ```llCeil```, ```llSqrt```, ...), física (```llGetGeometricCenter```, ```llGetCenterOfMass```, ```llLookAt```, ```llApplyImpulse```, ...), control de cámara (```llGetCameraPos```, ```llTakeCamera```, ```llReleaseCamera```, ```llSetCameraAtOffset```, ...), relación con el entorno (```llGetTimeOfDay```, ```llGetRegionName```, ```llGetSunDirection```, ```llGroundNormal```, ...), y asi muchas más. Puede encontrarse una listado de todas las funciones disponibles con una breve descripción de cada una de ellas en LSL Portal, la wiki oficial de LSL.
