# 6. HTML5: Offline Web applications (y 2)

_18-01-2011_ _Juan Mellado_

Como complemento al ```Cache Manifest```, el fichero que permite declarar los recursos que una aplicación _web_ quiere que el navegador retenga de forma local en _cache_, la especificación de HTML5 define además un API para poder controlar desde JavaScript la gestión que se realiza de dicha _cache_, y un sistema para detectar cuando el navegador pierde o recupera la conexión a red.

## 6.1. Application Cache API

HTML5 introduce un nuevo objeto JavaScript llamado ```applicationCache``` a nivel de ```window```, al que también puede accederse a través de ```self``` en los _Web Workers_, aunque al parecer sólo desde los de tipo _Shared Web Workers_.

El objeto tiene un atributo ```status``` de tipo entero que indica en todo momento el estado en que se encuentra la _cache_ de la aplicación según las siguientes constantes:

```javascript
const unsigned short UNCACHED = 0;
const unsigned short IDLE = 1;
const unsigned short CHECKING = 2;
const unsigned short DOWNLOADING = 3;
const unsigned short UPDATEREADY = 4;
const unsigned short OBSOLETE = 5;
```

En estado ```UNCACHED``` la aplicación no se encuentra asociada a ninguna _cache_. En estado ```IDLE``` la aplicación está en _cache_ y además contiene la última versión. En estado ```CHECKING``` el navegador está comprobando si hay cambios en la _cache_, o descargando el _manifest_ por primera vez. En estado ```DOWNLOADING``` el navegador está descargando cambios a la _cache_, o descargando los recursos detallados en el _manifest_ por primera vez. En estado ```UPDATEREADY``` la _cache_ está actualizada pero la aplicación mostrada en el navegador no tiene los últimos cambios. En estado ```OBSOLETE``` el navegador recibió un error ```404``` (fichero no encontrado) o ```410``` (fichero no disponible definitivamente) al intentar recuperar el _manifest_, por lo que la _cache_ está siendo borrada.

El objeto ```applicationCache``` dispone además de manejadores de eventos definidos para identificar los cambios de estado más relevantes de la _cache_ y permitir realizar un seguimiento más detallado de todo el proceso de descarga y actualización.

```javascript
attribute Function onchecking;
attribute Function onerror;
attribute Function onnoupdate;
attribute Function ondownloading;
attribute Function onprogress;
attribute Function onupdateready;
attribute Function oncached;
attribute Function onobsolete;
```

Los eventos ```onchecking```, ```ondownloading```, ```onupdateready``` y ```onobsolete``` se disparan cuando la _cache_ pasa a los estados ```CHECKING```, ```DOWNLOADING```, ```UPDATEREADY``` y ```OBSOLETE``` respectivamente. Por su parte, ```onnoupdate``` cuando el manifiesto no tiene cambios, ```onprogress``` cuando se están descargando recursos, ```oncached``` cuando se termina la descarga de los recursos, y ```onerror``` cuando se produce algún problema leyendo el manifiesto o los recursos, o el propio manifiesto cambia mientras aún se están recuperando los recursos.

Por último, el objeto ```applicationCache``` tiene dos funciones. La función ```update```, que sirve para forzar la comprobación mediante código de si existe una versión más reciente del _manifest_, y la función ```swapCache``` que sirve para actualizar la aplicación a la versión más reciente que se encuentra en la _cache_. Ambas funciones elevan una excepción de tipo ```INVALID_STATE_ERR``` si la _cache_ no se encuentra en el estado adecuado cuando se invocan. Por ejemplo, no se puede pedir comprobar la _cache_ si ya se está haciendo, ni actualizar a una nueva versión si no existe.

```javascript
var cache = window.applicationCache;
...
cache.update();
...
if (window.applicationCache.UPDATEREADY == cache.status){
  cache.swapCache();
}
```

La llamada a la función ```swapCache``` no hace que se refresque automáticamente la página con los nuevos recursos, no cambia automáticamente imágenes ni hojas de estilos, ni nada parecido a eso, sólo fuerza a que la próxima vez que la aplicación solicite un recurso de la cache obtenga la versión más reciente.

## 6.2. Browser State

HTML5 introduce un nuevo atributo ```onLine``` de tipo ```boolean``` en el objeto ```navigator``` para poder saber desde JavaScript si el navegador puede tener acceso a red (```true```) o se encuentra totalmente desconectado (```false```).

```javascript
function getEstado(){
  return navi
  ```gator.onLine? "conectado": "desconectado";
}
```

No obstante, y como se afirma en la propia especificación, este indicador es bastante poco fiable, ya que un equipo puede tener conexión a red pero no a Internet. En la práctica parece más indicado para saber si un navegador está operando en el modo "_Trabajar sin conexión_".

Como la conexión y desconexión de la red puede producirse en cualquier momento, la especificación define además dos nuevos eventos ```ononline``` y ```onoffline``` que se lanzan cuando se detecta un cambio en el modo de funcionamiento.

```html
<body ononline="conectado()" onoffline="desconectado()">
```

El propósito de todo esto sería permitir a una aplicación utilizar algún tipo de almacenamiento local mientras se esté operando desconectado, como _Web Storage_ o _Indexed Database API_, aunque esta última está en desarrollo y todavía no se encuentra implementada en ningún navegador. O incluso _Web SQL Database_, una especificación que finalmente se ha eliminado de HTML5 pero que muchos navegadores soportan a modo de _wrapper_ sobre _SQLite_.
