# 31. Allods Online Addons (7/10): Addon Manager

_24-09-2010_ _Juan Mellado_

Uno de los objetivos principales de los _addons_ es sustituir la interface gráfica por defecto del juego por otra personalizada que se adapte mejor a las necesidades de cada jugador, o que añada algún tipo de información u opción que facilite el desempeño de su rol. _Allods Online_ trata todos los componentes de su interface de usuario como _addons_, incluidos los elementos propios internos que son parte del núcleo del juego, como el _chat_ o el mapa por ejemplo. Y este tratamiento homogéneo es el que permite "descargar" (desactivar) un _addon_ original del juego y sustituirlo por uno propio.

Para obtener todos los addons cargados en el juego se debe utilizar la función ```GetStateManagedAddons```. Esta función devuelve una tabla Lua en la que cada elemento representa un _addon_. Por ejemplo, con el siguiente código se escribe en el _log_ un listado con todos los _addons_ cargados en el juego:

```lua
...
local addons = common.GetStateManagedAddons()
for i = 0, GetTableSize(addons) - 1 do
  LogInfo(addons[i].name)
end
...
```

En la versión europea actual del juego, la ```1.1.00.62```, aparecen listados los siguientes 79 _addons_:

```text
Alchemy                                  ContextShipDevice
ArmorCraft                               ContextShipDeviceNavigator
BuffInfo                                 ContextShipDeviceOvertip
BuffInfoParty                            ContextShipHangar
BuffInfoPet                              ContextShipPlate
BuffInfoTarget                           ContextSplitstack
Castbar                                  ContextTalents
Chat                                     ContextTooltip
ContextActionbar                         ContextTooltipCompare
ContextActions                           ContextTutorial
ContextAEMarker                          ContextUniMessageBox
ContextAnnounceCustom                    CraftBag
ContextAstralHubMap                      Death
ContextAuction                           EnemyShipDamage
ContextCartographer2                     EscMenu
ContextChatLine                          Guilds
ContextCharacter                         InternalContextActions
ContextCompass                           IslandTimer
ContextDamageVisualization               LagMeter
ContextDepositeBox                       MageEnergyInstability2
ContextDragNDrop                         NecromancerBloodPool
ContextFXPlayer                          NecromancerPet2
ContextInspect                           Options
ContextItemMall                          PaladinShields
ContextLootBag                           PetCommandPoints
ContextMail                              Plates
ContextMounts                            PsionicContact2
ContextMultibag                          RollGreedNeed
ContextNpcTalk                           Sounds
ContextNpcTeleport                       Spellbook
ContextOvertip                           StalkerCartridgeBelt
ContextPartyPlate                        SubtitleShipInfo
ContextPinMenu2                          TabSelector
ContextPlayerTrade                       TalentInformer
ContextPOIMarker                         TargetSelection
ContextPopup                             VendorTrade
ContextQuestLog                          Warnings
ContextQuestTracker                      WarriorCombatAdvantage
ContextRaidPlate                         ZoneAnnounce
ContextRuneCombiner
```

Los nombres son bastante significativos, y como se observa, aparecen prácticamente todos los componentes internos del juego con algún tipo de interface gráfica.

Cuando se carga un _addon_ propio, este también aparece en la lista, precedido por ```UserAddon```. Así, si se carga el _addon_ ```SampleInit```, que viene de ejemplo con el juego, aparece en el listado de la siguiente guisa:

```text
UserAddon/SampleInit
```

Conociendo los nombres de los _addons_, es posible cargarlos o descargarlos utilizando las funciones ```StateLoadManagedAddon``` y ```StateUnloadManagedAddon``` respectivamente. Aunque naturalmente no todos los _addons_ pueden ser descargados, ya que algunos son demasiado básicos, y su ausencia dejaría el juego prácticamente inutilizable.

Para descargar o cargar un _addon_, como el de la brújula por ejemplo (```ContextCompass```), basta una simple línea de código:

```lua
...
-- Descargar
common.StateUnloadManagedAddon("ContextCompass")
...
-- Cargar
common.StateLoadManagedAddon("ContextCompass")
...
```

La idea de todo esto es que si queremos hacer un _addon_ propio, que sustituya la brújula original, lo primero que tendremos que hacer será descargar el _addon_ original, para que no se solape con el nuestro. Sencillo, ¿no?.

Dentro de los _addons_ de ejemplo que vienen con el juego hay uno llamado ```SampleZoneAnnounce``` que sustituye al _addon_ ```ZoneAnnounce``` original, que es el encargado de mostrar por pantalla los nombres de las zonas por las que van pasando los personajes.

Y como curiosidad, el código Lua de los _addons_ internos del juego se encuentran en los ficheros ```LuaCompiledSystem.pak``` y ```LuaCompiledIngame.pak``` del directorio ```data/Packs``` donde se encuentra instalado el juego. Desgraciadamente están compilados y distribuidos en formato binario, por lo que no se pueden examinar. En versiones más antiguas del juego se distribuían en formato de texto y se podían examinar.
