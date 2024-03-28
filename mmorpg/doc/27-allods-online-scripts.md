# 27. Allods Online Addons (3/10): Scripts, Eventos, Funciones y Variables

_28-08-2010_ _Juan Mellado_

En la documentación oficial de los _addons_ para _Allods Online_ hay unos pocos apuntes técnicos acerca de como se ejecutan los _scripts_ de Lua dentro del juego. Como por ejemplo que se utiliza la versión ```5.0.x```, o que para cada _addon_ se levanta su propia máquina virtual independiente, aunque existen técnicas para intercambiar información entre ellos mediante el uso de unos eventos especiales.

Una característica específica de la implementación de Lua en _Allods Online_ es que sólo están disponibles las librerías ```string```, ```math```, ```table``` y ```coroutine```. Es decir, las que permiten manipular cadenas de texto, realizar operaciones matemáticas, gestionar tablas (```arrays```, ```lists```, etc) e implementar multitarea. Y no se encuentran disponibles otras como ```file```, ```io```, ```os```, etc, que dan libre acceso a los recursos del sistema.

Otra característica de la implementación de Lua que utiliza el juego es que no se permite el uso de las variables globales por defecto. Es decir, en Lua las variables automáticamente son globales, pero dentro de _Allods Online_ esto no se permite, es necesario declararlas explícitamente de la forma ```Global(nombre, valor)```, como en el siguiente ejemplo:

```lua
Global("counter", 0)
```

En lineas más generales, comentar que los _scripts_ de los _addons_ se inicializan en el mismo orden que se encuentran listados dentro de la etiqueta ```<ScriptFileRefs>``` de sus correspondientes ficheros ```AddonDesc.(UIAddon).xdb```. Y se cargan automáticamente al iniciar el juego, o desde otro _script_, en función del valor de la etiqueta ```<AutoStart>```.

Aunque lo más importante sin lugar a dudas es tener claro que el funcionamiento de los _addons_ se basa principalmente en la gestión de eventos. Es decir, los _scripts_ dicen a qué sucesos quieren reaccionar, e indican la función a la que se debe llamar cuando estos se produzcan.

La lista de eventos disponibles es enorme, prácticamente existe un evento para casi cualquier suceso que pueda ocurrir en el juego. Por ejemplo, para un personaje podemos programar un _addon_ que reaccione cuando cambie de posición, cuando cambie su nivel de experiencia, cuando cambie la cantidad de dinero que tiene, cuando cambie su lista de amigos, cuando cambie de objetivo, cuando cambien sus talentos, y así un largo etcétera. Y lo mismo para las "_parties_", "_raids_", hermandades, objetos, inventarios, misiones, mapas, correo, etc.

Para registrar un manejador de eventos hay que realizar una llamada a ```common.RegisterEventHandler(función, evento)```, de forma que cuando ocurra el evento dado, _Allods Online_ llamará automáticamente a la función indicada. En los _addons_ de ejemplo que vienen con el juego hay uno muy sencillo llamado ```SampleEventRegistration```, que registra un evento que se llama automáticamente cada segundo. El _script_ ```ScriptSampleEventRegistration.lua``` de dicho _addon_ contiene un código similar al siguiente:

```lua
Global("counter", 0)

function OnEventSecondTimer(params)
  counter = counter + 1
  LogInfo("counter: ", counter)
end

function Init()
  common.RegisterEventHandler(OnEventSecondTimer, "EVENT_SECOND_TIMER")
end

Init()
```

El _script_ declara un contador (```counter```) en una variable global, para que su valor persista entre llamada y llamada, inicializándolo a cero. Registra una función (```OnEventSecondTimer```) para que Allods la llame cada vez que salte el evento (```EVENT_SECOND_TIMER```). Y en cada invocación de la función incrementa el valor del contador y escribe su valor en el fichero de _log_ por defecto, que recordemos se encuentra en el directorio ```Personal\Logs``` dentro del directorio donde se encuentra instalado el juego.

Todas las funciones manejadoras de eventos reciben un único parámetro cuya interpretación depende enteramente del tipo de evento. Algo que resulta bastante natural en Lua, ya que es un lenguaje tipado dinámicamente. Así, para el evento del ejemplo, el parámetro no contiene ningún atributo. Pero para otros eventos, como por ejemplo el que salta cuando con nuestro personaje aceptamos una misión, lo que se recibe es el identificador de la misión concreta que se ha aceptado, al que se accede con ```params.questId```. En la documentación se listan los eventos disponibles así como los parámetros concretos que recibe cada uno.

De forma general, en los eventos asociados a misiones se reciben identificadores de misiones, en los referidos a personajes se reciben identificadores de avatares, y así sucesivamente. Estos identificadores son los que permiten realizar tareas concretas sobre los objetos del juego mediante la llamada a las funciones ofrecidas por el API de _Allods Online_.

De igual forma que con los eventos, existen funciones para realizar prácticamente cualquier operación, excepto por supuesto para controlar los personajes. El API está pensado para reemplazar o añadir opciones a la interface de usuario del juego, o reaccionar a determinados estados de la lógica del juego, principalmente esto último para automatizar tareas, como por ejemplo vender todos los objetos grises automáticamente cuando se abre un diálogo con un vendedor.

La orientación a objetos de Lua permite escribir código ordinario, en el que sobre un objeto se invoca un método con una serie de argumentos. Por ejemplo, para que nuestro personaje acepte la realización de una misión dada basta con escribir la siguiente línea:

```lua
avatar.AcceptQuest(questId)
```

La variable ```avatar``` es global, se encuentra definida por defecto, y es accesible desde cualquier punto de cualquier _script_. Con ella podemos obtener información sobre el estado de nuestro personaje y realizar acciones con él. De igual forma se encuentran definidas otra serie de variables como ```group```, ```raid```, ```mailBox```, ```guild```, etc.

Hay que tener en cuenta que las variables globales siempre se encuentran instanciadas, pero habrá muchas operaciones que no puedan hacerse con ellas si el personaje no se encuentra en el estado adecuado. Por ejemplo, no podrá abrir el correo si no se encuentra ante un buzón. Para ello hay funciones específicas que nos indican si los objetos encuentran en un estado adecuado para usarlas:

```lua
LogInfo("mailBox: ", mailBox.IsReady() )
LogInfo("group: ", group.GetLeaderIndex() )
LogInfo("raid: ", raid.IsExist() )
LogInfo("guild: ", guild.IsExist () )
```
