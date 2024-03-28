# 4. HTML5: Web Storage

_10-01-2011_ _Juan Mellado_

_Web Storage_ es un API de HTML5 para JavaScript que permite a las páginas _web_ guardar información en la parte cliente. Es el sustituto natural de las tradicionales _cookies_, aunque a diferencia de estas últimas, la información gestionada con el nuevo API sólo puede ser accedida desde el cliente, nunca desde el servidor.

Las _cookies_ que se utilizan actualmente siempre se mandan como una cabecera HTTP en cada conexión entre cliente y servidor, para que ambos vean la misma información. El problema es que esto implica un sobrecoste en la comunicación, motivando que el tamaño de los datos almacenados sea normalmente muy pequeño. Además de que permiten realizar ataques por parte de malintencionados que pueden cambiar "al vuelo" el contenido de las _cookies_ intercambiadas entre clientes y servidores. Y sobre todo, pueden dar lugar a inconsistencias entre el estado de un cliente _web_ y la información contenida en una _cookie_, por ejemplo cuando se abren dos ventanas de una misma _web_, o cuando en una _web_ de compras se añade un artículo nuevo a un carrito de la compra gestionado con una _cookie_ y luego se pulsa el botón de retroceso del navegador.

El nuevo API, al no intercambiar información con el servidor, permite que la cantidad de datos almacenados en el cliente pueda ser mucho mayor, del orden de 5 MB según la recomendación, y que en la práctica dependerá de la política de gestión de cuotas de cada navegador en particular.

El API define dos tipos de almacenamiento, uno de ellos temporal y a nivel de ventana (```sessionStorage```), y el otro persistente y a nivel de _web_ (```localStorage```).

```sessionStorage``` es por sesión y ventana, de forma que dos pestañas del navegador abiertas al mismo tiempo para una misma _web_ pueden tener información distinta según lo que se esté haciendo en cada momento en cada una de ellas. Al cerrar la sesión se pierde la información.

```localStorage``` es permanente para una misma _web_, permite almacenar datos que pueden ser accedidos cada que vez que se visita la _web_. Todas las sesiones abiertas sobre la misma _web_ ven la misma información.

Los datos se almacenan de forma estructurada en forma de pares (clave, valor). Donde la clave es una cadena de texto, y el valor puede ser de cualquier tipo, aunque en la práctica sólo se permite ```String``` para los valores. En la documentación oficial se indica además que los navegadores deberían elevar una excepción en caso de que se intentara almacenar un objeto de tipo ```ImageData```.

El API de JavaScript define una interface ```Storage``` bastante sencilla e intuitiva. Si se quiere utilizar almacenamiento temporal se utiliza ```sessionStorage```, y si se quiere almacenamiento permanente ```localStorage```. Estos atributos están directamente disponibles a nivel de ```window``` y no es necesario instanciarlos. Para los ejemplos utilizaré ```localStorage```.

Con el método ```setItem(key, value)``` se almacena un valor para una clave dada. Si ya existe un valor previo almacenado para la clave dada se sobreescribe con el nuevo.

```javascript
localStorage.setItem("user", "Holmes");
```

Con el método ```geItem(key)``` se recupera el valor almacenado para una clave dada. Si no existe valor para la clave dada el método retorna ```null```.

```javascript
localStorage.getItem("user");
```

Con el método ```removeItem(key)``` se borra una clave dada y su valor asociado del almacén. Si la clave dada no existe no se eleva ningún error.

```javascript
localStorage.removeItem("user");
```

Con el método ```clear()``` se borran todas las claves actuales almacenadas.

```javascript
localStorage.clear();
```

Con el atributo ```length``` se obtiene el número de claves actuales almacenadas.

```javascript
localStorage.length;
```

Con el método ```key(index)``` se obtiene el nombre de la clave para un índice dado. Este método puede resultar un poco extraño de entrada, pero es útil para recorrer todas las claves que actualmente se encuentran almacenadas. Si en el almacén hay ```N``` claves, entonces se pueden recuperar sus nombres iterando entre ```0``` y ```N-1```.

```javascript
for (var i= 0; i < localStorage.length; ++ i){
  hacerAlgoCon(localStorage.key(i));
}
```

Finalmente, comentar que si lo que se quiere almacenar son objetos más complejos de JavaScript, siempre se puede utilizar las utilidades de conversión entre objetos JSON y String.

```javascript
var user = {
  name: "Holmes",
  friend: "Watson"
};
...
localStorage.setItem("user", JSON.stringify(user));
...
user = JSON.parse(localStorage.getItem("user"));
```

En la especificación oficial se indica además que cualquier tipo de cambio en el almacén debe disparar un evento de tipo ```StorageEvent```, de forma que cualquier ventana con acceso al almacén pueda responder al mismo.

```javascript
window.addEventListener("storage", onStorage, false);

function onStorage(event){
  // Evento recibido de tipo StorageEvent
}
```

El evento tiene los atributos ```key```, ```oldValue```, ```newValue```, ```storageArea``` y ```url```, para indicar la clave afectada, el valor anterior, el nuevo valor, el almacén afectado, y la URL desde la que se realizó el cambio. Si la clave es nueva el valor anterior llega como ```null```. Si la clave está siendo borrada el nuevo valor llega como ```null```. Y si el almacén entero está siendo borrado la clave y los dos valores llegan a ```null```.

En la práctica, en Chrome, que es el navegador con el que he estado haciendo las pruebas, y aunque no parece que se diga nada concreto al respecto en la documentación oficial, el evento no se dispara sobre la ventana que realiza los cambios, pero si sobre las demás.
