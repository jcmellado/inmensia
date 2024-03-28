# 2. HTML5: Web Workers (y 2)

_18-12-2010_ _Juan Mellado_

Utilizar _Shared Workers_ es otra forma de crear _threads_ en JavaScript. La principal diferencia con respecto a los _workers_ "ordinarios" es que permiten crear hilos de ejecución con múltiples manejadores de mensajes, en vez de uno sólo. De esta forma un mismo _thread_ puede tener varios clientes, cada uno de ellos comunicándose a través de su propio manejador.
Los _workers_ compartidos se crean instanciando un objeto de la clase ```SharedWorker```:

```javascript
var worker = new SharedWorker("worker.js", "ejemplo");
```

El constructor admite dos parámetros. El primero es el nombre del fichero que contiene el código JavaScript a ejecutar, al que aplican las mismas reglas que para los _workers_ dedicados. El segundo parámetro es opcional, y es el nombre con el que se quiere identificar al _thread_ creado. Esto del nombre es importante, ya que permite crear distintos hilos a partir de un mismo fichero y que se puedan distinguir en tiempo de ejecución. Si no se indica nombre toma cadena vacía por defecto.

La comunicación con un _worker_ compartido se realiza a través de mensajes y manejadores de mensajes, de igual forma que con los _workers_ dedicados. La principal diferencia es que los _workers_ compartidos tienen un atributo más llamado ```port``` (puerto). Para cada cliente que instancia un _worker_ se le crea un puerto, y es a través de este puerto como se comunica con el _worker_. Además, los _workers_ tienen un manejador ```onconnect```, para saber cuando se les conecta un cliente y poder ejecutar un proceso de inicialización a medida para cada uno de ellos.

En la práctica el API no implica mucho cambio en el código con respecto a los _workers_ dedicados, simplemente añadir ```port``` en las llamadas a los métodos e implementar el manejador ```onconnect```.

El proceso básico constaría de los siguientes pasos:

1. Un cliente instancia el _worker_ compartido, establece su manejador de mensajes, y le envía un mensaje:

    ```javascript
    var worker = new SharedWorker("worker.js", "ejemplo");

    worker.port.onmessage = function(event){
      // Recibe "Información para el cliente" en event.data
    };

    worker.port.postMessage("Información para el worker");
    ```

2. El _worker_ recibe la nueva conexión en ```event.ports[0]```, y le asigna un manejador de mensajes que envía un mensaje al cliente conectado:

    ```javascript
    self.onconnect = function(event){

      var port = event.ports[0];

      port.onmessage = function(event){
        // Recibe "Información para el worker" en event.data

        port.postMessage("Información para el cliente");
      }
    }
    ```

3. Un segundo cliente instancia el _worker_ compartido, establece su manejador de mensajes, y le envía un mensaje:

    ```javascript
    var worker2 = new SharedWorker("worker.js", "ejemplo");

    worker2.port.onmessage = function(event){
      // Recibe "Información para el cliente" en event.data
    };

    worker2.port.postMessage("Información para el worker");
    ```

Como se observa, el código de los dos clientes es idéntico. La única diferencia es que cada uno de ellos accede a través de una referencia propia al _worker_ compartido, o dicho de otro modo, a través de su propio puerto.

Para comprobar que ambos clientes están accediendo al mismo _thread_ se puede escribir un ejemplo un poco más elaborado en el que se lleve la cuenta de los clientes conectados, y se asigne a cada cliente un número con el orden en que va conectándose:

```javascript
var count = 0;

self.onconnect = function(event){
  var port = event.ports[0];
  port.id = ++ count;

  port.onmessage = function(event){
    port.postMessage(port.id);
  }
}
```

Este código asignará el valor 1 al primer cliente en conectarse, 2 al segundo, y así sucesivamente. Si los clientes no se estuvieran conectando al mismo _thread_ entonces a todos se les asignaría siempre el valor 1. Esto se puede probar fácilmente quitando el segundo parámetro ```ejemplo``` en la llamada al constructor de uno de los dos clientes (no a los dos).
