# 32. Allods Online Addons (8/10): AutoSellGreyAddon

_01-10-2010_ _Juan Mellado_

En este artículo se analiza un _addon_ llamado ```AutoSellGreyAddon```. Su función es la de vender de forma automática todos los objetos grises que lleva en la bolsa nuestro personaje en el momento que se abra una ventana de diálogo con un vendedor.

El _addon_ se compone de sólo dos ficheros: el obligatorio ```AddonDesc.(UIAddon).xdb```, y un _script_ llamado ```ScriptAutoSell.lua```. No hay más. Ni formularios, ni imágenes, ni nada más. Aunque la verdad es que no lo necesita.

El código del _script_ empieza con la habitual llamada a la función ```Init```, que en ese caso se encarga de registrar dos eventos. El evento ```EVENT_TALK_STARTED```, que se lanza cuando se inicia una conversación con cualquier NPC del juego, no necesariamente vendedor. Y el evento ```EVENT_TALK_STOPPED```, que se lanza cuando se termina una conversación.

```lua
function Init()
  common.RegisterEventHandler(OnTalkStarted, "EVENT_TALK_STARTED")
  common.RegisterEventHandler(OnTalkStopped, "EVENT_TALK_STOPPED")
end
```

Al iniciarse una conversación con un NPC el juego invoca a la función ```OnTalkStarted```, y registra una nueva función para el evento ```EVENT_VENDOR_LIST_UPDATED```, que se lanza cuando un vendedor solicita a nuestro personaje la lista de objetos que tiene en la bolsa disponibles para la venta.

```lua
function OnTalkStarted()
  common.RegisterEventHandler(OnVendorListUpdated, "EVENT_VENDOR_LIST_UPDATED")
end
```

Al terminar la conversación se invoca a la función ```OnTalkStopped``` que elimina el registro del evento ```EVENT_VENDOR_LIST_UPDATED```.

```lua
function OnTalkStopped()
  common.UnRegisterEventHandler(OnVendorListUpdated, "EVENT_VENDOR_LIST_UPDATED")
end
```

Finalmente, la función ```OnVendorListUpdated``` es la que realmente hace todo el trabajo en una serie de pasos muy claros:

- Obtiene el número de objetos que tiene nuestro personaje en la bolsa
- Recorre cada objeto que hay en la bolsa
- Si el objeto tiene una calidad inferior a 1 (gris) lo vende
- Y elimina el registro del evento

```lua
function OnVendorListUpdated()
  local currentBagSize = avatar.InventoryGetBaseBagSlotCount()
  for slotIndex = 0, currentBagSize - 1 do
    local itemId = avatar.GetInventoryItemId(slotIndex)
    if itemId then
      local itemInfo = avatar.GetItemInfo(itemId)
      if itemInfo.quality <= 1 then
        avatar.Sell(slotIndex, itemInfo.stackCount)
      end
    end
  end
  common.UnRegisterEventHandler(OnVendorListUpdated, "EVENT_VENDOR_LIST_UPDATED")
end
```

La función ```avatar.InventoryGetBaseBagSlotCount``` es la que obtiene el número de objetos de la bolsa nuestro personaje. La función ```avatar.GetInventoryItemId(slotIndex)``` es la que obtiene el ID de un objeto concreto de la bolsa dada su posición dentro de la misma, que recordemos que puede ser de una casilla que está vacía, por lo que es necesario comprobar que ha devuelto un ID de un objeto válido. La función ```avatar.GetItemInfo(itemId)``` es la que obtiene el detalle de los atributos del objeto dado, siendo el atributo ```quality``` el que indica el nivel del objeto, aunque este es sólo uno de las varias decenas de atributos que devuelve la función, merece la pena echar un vistazo en la documentación. Finalmente, la función ```avatar.Sell(slotIndex, itemInfo.stackCount)``` es la que vende el objeto de la bolsa dada su posición.

Sin lugar a dudas _este_ addon es muy claro ejemplo de sencillez y utilidad.
