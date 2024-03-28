# 1. HTML5: Web Workers (1)

_11-12-2010_ _Juan Mellado_

Los _Web Workers_ son tareas escritas en JavaScript que pueden lanzarse en segundo plano, a modo de _threads_. Su objetivo es permitir que las aplicaciones _web_ tengan la posibilidad de lanzar hilos de ejecución concurrentes con una gran carga de trabajo y duración indeterminada.

Su propósito principal es que se puedan crear tareas que funcionen al margen del proceso normal de gestión de eventos de los controles de la interface de usuario, evitando bloquear la página durante su ejecución, y que no aparezca la típica ventana de aviso del navegador diciendo que se ha superado el tiempo máximo de ejecución para un _script_.

El API resulta bastante sencillo y la documentación oficial está llena de ejemplos. De hecho, lanzar un _worker "normalito" es algo tan simple como escribir la siguiente línea de código:

```javascript
var worker = new Worker("worker.js");
```

Sólo se pueden lanzar _scripts_ de esa forma. Es decir, indicando la URL donde se encuentra el código JavaScript a ejecutar. No se puede utilizar ni ```javascript:``` ni ```data:```. La URL resultante del nombre del fichero externo debe tener el mismo esquema que la página original que motivó la ejecución del script. Por ejemplo, es imposible desde una página cargada desde una dirección ```https:``` lanzar un _script_ de la misma dirección pero con ```http:```. Y es más, el único lenguaje soportado es JavaScript, el atributo de tipo MIME es ignorado, el navegador asume siempre que el código está escrito en JavaScript.

Matar de manera inmediata un _worker_ que se encuentra en ejecución es tan sencillo como llamar a su función ```terminate```. Algo que se puede hacer por ejemplo desde un botón a petición del usuario:

```html
<button type="button" onclick="worker.terminate();">Kill</button>
```

Por su parte, la forma preferida de terminar un _worker_ desde dentro debería ser llamando a su propia función ```close```. Pero teniendo muy en cuenta que esta función no fuerza a la terminación de forma brusca del código del _worker_ en el punto que se produce la llamada a dicha función de cierre, como lo hace la función ```terminate```. El navegador ejecutará las líneas de código siguientes, si las hubiera.

```javascript
while(true){
  self.close(); // El worker seguirá ejecutándose
}
```

La función ```close``` detiene el lanzamiento de eventos, previene el disparo de _timers_, descarta las notificaciones pendientes de operaciones asíncronas, etc.

Dentro de un _worker_ se puede hacer prácticamente cualquier cosa. Pueden procesar eventos, _callbacks_, e incluso es posible crear otros _workers_. La única limitación lógica es que no tienen un contexto de navegación asociado. No pueden acceder al DOM, ```window```, ```document```, o ```parent```. Pero sí a ```navigator```, ```location```, ```XMLHttpRequest```, _timers_, ```applicationCache```, o _Web SQL database_. Y una opción interesante es que tienen la posibilidad de ejecutar el código de otros _scripts_ mediante la función ```importScripts```.

```javascript
importScripts("script1.js"); // De uno en uno...
importScripts("script2.js");
importScripts("script3.js", "script4.js"); // ... o varios a la vez
```

Para capturar posibles errores no controlados que se produzcan dentro del _thread_ hijo, y que se quieran capturar desde el padre, se puede indicar un manejador de eventos en su atributo ```onerror```. Esta función recibe un objeto con el detalle concreto del error: nombre del fichero, número de línea y mensaje de texto.

```javascript
worker.onerror = function(event){
  alert(event.filename + " [" + event.lineno + "]: " + event.message);
};
```

Lógicamente, la parte más interesante de este API es la que permite comunicar el hilo padre principal con el _worker_ hijo. Para que el padre pueda darle trabajo al hijo, y para que el hijo pueda comunicarle al padre los resultados de su trabajo. La comunicación se realiza a través de mensajes y manejadores. Más concretamente, para enviar mensajes se utiliza la función ```postMessage```, y para recibir mensajes se utiliza el manejador ```onmessage```.

Una comunicación básica entre padre e hijo se realiza en cuatro pasos:

1. El hilo padre manda un mensaje (evento) al _worker_:

    ```javascript
    worker.postMessage("Información para el worker");
    ```

2. El _worker_ recibe el mensaje del padre en el atributo ```data``` del evento:

    ```javascript
    self.onmessage = function(event){
      // Recibe "Información para el worker" en event.data
    };
    ```

3. Cuando el _worker_ termina su proceso envía un mensaje (evento) al padre:

    ```javascript
    self.postMessage("Información para el padre");
    ```

4. El padre recibe el mensaje del hijo en el atributo ```data``` del evento:

    ```javascript
    worker.onmessage = function(event){
      // Recibe "Información para el padre" en event.data
    };
    ```

Y esto es básicamente lo más importante. Recomiendo mirar los ejemplos que vienen dentro de la propia especificación para ver situaciones un poco más complejas.
