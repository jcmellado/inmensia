# 13. Java: Reactive Streams (2)

_18-05-2018_ _Juan Mellado_

_Reactive Streams_ es una iniciativa para definir los requerimientos y las interfaces que deben cumplir los _reactive streams_, así como para proporcionar las herramientas necesarias para comprobar que las implementaciones cumplan todos los requerimientos. Posiblemente sea el esfuerzo más relevante llevado a cabo en la materia, y de hecho, Java 9 incorporó al JDK de forma nativa las interfaces propuestas por esta iniciativa.

Los requerimientos detallados que deben cumplir cada uno de los distintos componentes que intervienen en un _reactive stream_ están definidos en una especificación publicada por uno de los grupos de trabajo de _Reactive Streams_. Es interesante conocerlos, aunque sólo sea a través de una lectura rápida, para entender mejor el comportamiento esperado para este tipo de _streams_.

## 13.1. Publisher

1. Un publicador DEBE enviar a un suscriptor un número total de elementos igual o menor que el número total de elementos solicitados por dicho suscriptor.

2. Un publicador PODRÍA enviar menos elementos que los solicitados por un suscriptor, por ejemplo cuando ya no sea capaz de producir más elementos a partir de su fuente origen de información.

3. Un publicador DEBE invocar los métodos ```onSubscribe```, ```onNext```, ```onError``` y ```onComplete``` de un suscriptor de forma _thread-safe_ y utilizando algún mecanismo de sincronización en caso de utilizar múltiples _threads_.

4. Un publicador DEBE comunicar sus errores invocando el método ```onError``` de sus suscriptores.

5. Un publicador DEBE comunicar que ya no puede generar más elementos invocando el método ```onComplete``` de sus suscriptores.

6. Un publicador que invoque los métodos ```onError``` u ```onComplete``` de un suscriptor DEBE considerar cancelada dicha suscripción.

7. Un publicador no DEBERÍA invocar ningún otro método de un suscriptor después de haber invocado sus métodos ```onError``` u ```onComplete```.

8. Un publicador DEBE dejar de invocar eventualmente métodos de un suscriptor si su suscripción ha sido cancelada. No obstante, debido a la naturaleza asíncrona del proceso, y la demora entre la petición de cancelación y su cancelación efectiva, podría llegar a invocar alguno de sus métodos.

9. Un publicador DEBE, dentro la implementación de su método ```subscribe```, invocar el método ```onSubscribe``` del suscriptor pasado como parámetro antes de invocar cualquier otro de método de dicho suscriptor. El método debe terminar normalmente, excepto si el suscriptor pasado como parámetro es nulo, en cuyo caso debe elevar una excepción de tipo ```NullPointerException```. Cualquier otro tipo de error debe realizarse invocando el método ```onError``` del suscriptor.

10. Un publicador NO DEBE permitir la suscripción de un mismo suscriptor más de una vez.

11. Un publicador PUEDE decidir el número de distintos suscriptores que soporta a un mismo tiempo y si la implementación de las suscripciones la realiza de forma _unicast_ (transmisión de uno a uno) o _multicast_ (transmisión de uno a muchos).

## 13.2. Subscriber

1. Un suscriptor DEBE llamar al método ```request``` de ```Subscription``` para empezar a recibir elementos generados por la suscripción. Es decir, es responsabilidad del suscriptor indicar al publicador cuando quiere empezar a recibir y cuantos elementos está preparado para recibir.

2. Un suscriptor DEBERÍA procesar un elemento de forma asíncrona si estima que su tiempo de respuesta puede afectar la calidad de servicio del publicador.

3. Un suscriptor NO DEBE llamar a ningún método de ```Subscription``` ni ```Publisher``` dentro de sus métodos ```onComplete``` y ```onError``` para evitar ciclos y condiciones de carrera.

4. Un suscriptor DEBE considerar cancelada una suscripción cuando el publicador invoca su método ```onComplete``` u ```onError```.

5. Un suscriptor DEBE llamar al método ```cancel``` de ```Subscription``` cuando un publicador invoca su método ```onSubscribe``` teniendo el suscriptor ya una suscripción activa.

6. Un suscriptor DEBE llamar al método ```cancel``` de ```Subscription``` cuando la suscripción ya no sea necesaria con el objetivo de permitir liberar los recursos asociados a dicha suscripción.

7. Un suscriptor DEBE asegurar que todas las llamadas a los métodos de ```Subscription``` se realizan desde un mismo _thread_, o utilizando algún método de sincronización en caso de que la suscripción se esté usando concurrentemente desde distintos threads.

8. Un suscriptor DEBE estar preparado para recibir elementos a través de su método ```onNext```, aún después de haber llamado al método ```cancel``` de ```Subscription```, debido a la naturaleza asíncrona del proceso y la demora que se produce entre que el suscriptor solicita la cancelación y el publicador la cancela realmente.

9. Un suscriptor DEBE estar preparado para recibir la finalización de la suscripción a través de su método ```onComplete```, incluso antes de haber llamado al método ```request``` de ```Subscription```, debido a que el publicador tiene potestad para dar por terminada una suscripción.

10. Un suscriptor DEBE estar preparado para recibir la finalización de la suscripción a través de su método ```onError```, incluso antes de haber llamado al método ```request``` de ```Subscription```, debido a errores en el publicador.

11. Un suscriptor DEBE asegurar que el procesamiento asíncrono de sus métodos ```onSubscribe```, ```onNext```, ```onComplete``` y ```onError``` se realiza de manera _thread-safe_.

12. Un mismo suscriptor DEBE suscribirse a un mismo publicador una única vez.

13. Un suscriptor DEBE terminar las invocaciones a sus métodos ```onSubscribe```, ```onNext```, ```onComplete``` y ```onError``` sin elevar ninguna excepción. Excepto cuando algún parámetro recibido sea nulo, en cuyo caso deberá elevar una excepción de tipo ```NullPointerException```. Cualquier otro error debe resolverse cancelando la suscripción. El publicador no puede tratar los errores del suscriptor, por lo que dará por cancelada la subscripción en caso de producirse alguno.

## 13.3. Subscription

1. Una suscripción es el único componente a través del que se establece la comunicación entre un suscriptor y un publicador. Un objeto ```Subscription``` concreto pertenece de forma exclusiva a un ```Publisher``` y un ```Subscriber``` concretos. Sólo el publicador DEBE llamar a a los métodos ```request``` y ```cancel``` de ```Subscription```.

2. Una suscripción DEBE permitir que se llame a su método request desde los métodos ```onSuscribe``` y ```onNext``` de un suscriptor de manera síncrona. Es decir, la implementación del método ```request``` debe ser reentrante.

3. Una suscripción DEBE establecer un número máximo de llamadas recursivas entre los métodos ```request``` de ```Subscriber``` y ```onNext``` de ```Subscription```. La recomendación a este respecto es limitar a uno el número de llamadas recursivas evitando la secuencia ```request -> onNext -> request -> onNext -> ...```

4. Una suscripción DEBERÍA implementar su método ```request``` de forma que retorne lo antes posible para no afectar la calidad de servicio del suscriptor que lo invoque.

5. Una suscripción DEBE implementar su método ```cancel``` de forma idempotente, _thread-safe_, y retornar lo antes posible para no afectar la calidad del servicio del suscriptor que lo invoque.

6. Una suscripción NO DEBE realizar ninguna acción cuando se llame a su método ```request``` una vez que la suscripción haya sido cancelada.

7. Una suscripción NO DEBE realizar ninguna acción cuando se llame a su método ```cancel``` una vez que la suscripción haya sido cancelada.

8. Una suscripción DEBE almacenar la suma de todas las cantidades de elementos solicitadas por todas las llamadas realizadas por el suscriptor a su método ```request```.

9. Una suscripción DEBE invocar al método ```onError``` del suscriptor con una instancia de una excepción ```IllegalArgumentException``` si se solicita una cantidad de elementos igual o menor que cero en alguna llamada realizada por el suscriptor a su método request.

10. Un suscripción PODRÍA implementar en su método ```request``` invocaciones al método ```onNext``` del suscriptor de manera síncrona. Siempre y cuando la suscripción no haya sido ya cancelada.

11. Un suscripción PODRÍA implementar en su método ```request``` invocaciones al método ```onComplete``` del suscriptor de manera síncrona. Siempre y cuando la suscripción no haya sido ya cancelada.

12. Una suscripción DEBE requerir al ```Publisher``` que deje de invocar los métodos del ```Subscriber``` cuando el suscriptor llame a su método ```cancel``` . Siempre y cuando la suscripción no haya sido ya cancelada.

13. Una suscripción DEBE requerir del ```Publisher``` que libere cualquier referencia al ```Subscriber``` cuando el suscriptor llame a su método ```cancel```. Siempre y cuando la suscripción no haya sido cancelada.

14. Una suscripción PODRÍA causar que el ```Publisher``` pase a un estado de finalizado cuando el suscriptor llame a su método ```cancel```. Siempre y cuando la suscripción no haya sido ya cancelada.

15. Una suscripción NO DEBE elevar excepciones en la implementación de su método ```cancel```.

16. Una suscripción NO DEBE elevar excepciones en la implementación de su método ```request```.

17. Una suscripción DEBE permitir peticiones de una cantidad de elementos potencialmente infinita. Una petición de 2^63-1 (```java.lang.Long.MAX_VALUE```) o más elementos se considera como una petición de una cantidad infinita de elementos.

## 13.4. Processor

1. Un procesador DEBE cumplir tanto los requerimientos establecidos para los publicadores como para los suscriptores.

2. Un procesador PODRÍA elegir recuperarse de una invocación a su método ```onError``` por parte de un publicador, pero en ese caso DEBE considerar la suscripción cancelada, en caso contrario DEBE propagar el error inmediatamente invocando el método ```onError``` de sus suscriptores.
