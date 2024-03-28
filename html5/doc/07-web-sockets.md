# 7. HTML5: Web Sockets

_25-01-2011_ _Juan Mellado_

El objetivo de los _Web Sockets_ es permitir que las aplicaciones _web_ cliente puedan abrir desde JavaScript conexiones de red bidireccionales con cualquier servidor de forma arbitraria mediante un determinado protocolo de red. El W3C se encarga de definir el API para JavaScript y la IETF de elaborar el protocolo de red.

Para abrir una conexión desde JavaScript basta con instanciar un objeto de tipo ```WebSocket``` indicándole la URL del servidor con el que se quiere establecer la conexión.

```javascript
var ws = new WebSocket("ws://echo.websocket.org");
```

De igual forma que utilizamos ```http://``` para establecer conexiones con el protocolo HTTP, debemos utilizar ```ws://``` para establecer conexiones con el protocolo _websocket_. El constructor admite además un parámetro adicional para que se indique un conjunto de subprotocolos, pero esto es algo que aún se está definiendo a día de hoy en el último borrador de la especificación.

Por cierto, la URL que he puesto en el código de ejemplo es válida, en ella se encuentra escuchando un sencillo servidor _websocket_ que actúa como eco. Es decir, que reenvía al cliente lo mismo que recibe. Muy útil para hacer pruebas.

La conexión con el servidor se establece de forma asíncrona, en segundo plano, y la gestión del todo su ciclo de vida se realiza a través de _callbacks_ que reciben eventos de la forma acostumbrada. La última especificación actual publicada define tres _callbacks_: ```onopen```, ```onmessage``` y ```onclose```. La especificación en borrador añade uno más: ```onerror```.

```javascript
ws.onopen = function(event){
  // Conexión abierta
}

ws.onmessage = function(event){
  // Mensaje recibido en event.data;
}

ws.onclose = function(event){
  // Conexión cerrada
}

ws.onerror = function(event){
  // Error en la conexión
}
```

Para mandar un mensaje al servidor desde el cliente se tiene que utilizar la función ```send```, que admite como parámetro una cadena de texto con el mensaje a enviar.

```javascript
ws.send("ping");
```

Para cerrar la conexión hay que llamar a la función ```close```.

```javascript
ws.close();
```

Si se intenta enviar un mensaje cuando la conexión aún no está abierta se eleva un error. Y de igual forma ocurre si se intenta cerrar la conexión cuando no encuentra en el estado adecuado. Los mensajes enviados después de haber llamado al método ```close``` se descartan automáticamente.

La clase ```WebSocket``` tiene además otros cuatro atributos: ```url``` que es un ```String``` con la dirección a la que se encuentra conectado, ```protocol``` que es un ```String``` con el protocolo utilizado para la conexión, ```readyState``` que es un ```int``` que representa el estado de la conexión (```CONNECTING=0```, ```OPEN=1```, ```CLOSING=2```, ```CLOSED=3```), y ```bufferedAmount``` que es un ```long``` con el número de _bytes_ pendientes de enviar al servidor.

Y poco más, la verdad es que, como de costumbre, el API JavaScript para el cliente destaca por su simpleza y elegancia. Por su parte, para el servidor, ya existen algunas implementaciones del protocolo _websocket_. Por poner una sola, por ejemplo, la del popular Jetty para Java.
