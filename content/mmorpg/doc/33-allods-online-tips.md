# 33. Allods Online Addons (9/10): Code Tips

_08-10-2010_ _Juan Mellado_

Para ir acabando con esta serie voy a poner unos bloques de código que he encontrado útiles. Cosas sencillas que suelen ser necesarias hacerlas de vez en cuando.

## 9.1. Ocultar la Interface

_Allods Online_ tiene la típica opción para grabar a disco un _screenshot_ ("pantallazo"), a modo de instantánea del juego. Y es bastante habitual que antes de hacerlo los jugadores prefieran quitar la interface gráfica del juego, para que se vea la mayor parte del mundo virtual posible, en vez de los marcos, ventanas o barras de acciones. Por ello, si implementamos un _addon_ que muestre algún tipo de ventana o control gráfico, tenemos que hacer que se oculte automáticamente como hace el resto de _addons_ del juego.

Cuando el usuario solicita que se oculte o restaure la interface del juego (combinación _ALT+Z_ por defecto), se lanza el evento ```SCRIPT_TOGGLE_UI``` con un parámetro de tipo ```boolean``` que indica si la interface se está ocultando (```false```) o restaurando (```true```). En consecuencia, resulta muy sencillo ocultar o mostrar las ventanas de nuestros _addons_, consiguiendo un acabado más profesional.

Como siempre, lo primero es registrar un manejador para el evento:

```lua
common.RegisterEventHandler(OnToggleUI, "SCRIPT_TOGGLE_UI")
```

Y en la función del manejador ocultar o mostrar nuestra interface:

```lua
function OnToggleUI(params)
  formulario:Show(params.visible)
end
```

Evidentemente, en el código del ejemplo se supone que hay declarada una variable global ```formulario``` que contiene una referencia a la ventana principal del _addon_. Si hubiera varias ventanas independientes habría que realizar la misma operación con todas ellas de forma individual.

## 9.2. Guardar Información

Una necesidad bastante habitual de algunos _addons_ es guardar información, como por ejemplo los parámetros de configuración seleccionados por el jugador. Algunos juegos de este tipo guardan las opciones del jugador en el servidor, e incluso permiten que los _addons_ guarden las suyas propias, de forma que da igual la máquina desde la que un jugador ejecuta el juego, siempre lo ve con su configuración personalizada. Pero en _Allods Online_ la configuración se guarda en local, en la máquina donde se juega.

Lógicamente, por temas de seguridad, no permite que el código de un _addon_ acceda libremente al disco duro de la máquina donde se ejecuta el juego. La máquina virtual de Lua que se utiliza está "capada" en ese sentido. Pero si deja la posibilidad de que se escriba y lea del fichero de configuración de opciones globales del juego. Dicho fichero tiene por nombre ```user.cfg```, y se encuentra en el sub-directorio ```Personal``` del directorio donde se encuentra instalado el juego. Es un fichero de texto ordinario, por lo que se puede abrir y examinar con cualquier editor.

Para escribir y leer del fichero hay que utilizar las funciones ```SetGlobalConfigSection(section, table)``` y ```GetGlobalConfigSection(section)``` respectivamente. Siendo ```section``` el nombre de la etiqueta bajo la cual queremos grabar los valores contenidos en ```table```, que ha de ser una variable de tipo table de Lua.

El siguiente código de ejemplo crea una tabla con un valor y la graba:

```lua
local tabla = {}
tabla["variable1"] = "prueba"
common.SetGlobalConfigSection("TABLA", tabla)
```

El resultado de la ejecución del código anterior es la actualización del fichero ```user.cfg```, al que se le añaden las siguientes líneas dentro de la sección ```global```:

```lua
...
table_begin global

table_begin ScriptLocal_TABLA

table_begin data
variable1 = l"prueba"
table_end data

remote_version = -1
table_end ScriptLocal_TABLA
...
```

El proceso contrario, para leer del fichero, es bien sencillo:

```lua
tabla = common.GetGlobalConfigSection("TABLA") or {}
```

El viejo truco de añadir ```or {}``` al final de la sentencia sirve para que si en el fichero no se encuentra la tabla pedida entonces se devuelva una tabla vacía, en vez de un valor nulo.

Aunque útil, esta forma de proceder tiene el incoveniente de que el fichero puede llegar a tener un tamaño bastante grande si todos los _addons_ se dedican a escribir en él de forma indiscriminada. Y de igual forma, con el paso del tiempo, puede llegar a contener mucha información inútil si no se depura cuando se desinstalan los _addons_.
