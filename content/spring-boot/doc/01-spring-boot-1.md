# 1. Spring Boot 2 (1)

_12-07-2018_ _Juan Mellado_

La última versión de Spring Boot supone la primera gran actualización que recibe desde que se publicó. Las principales novedades a mi juicio son la necesidad de utilizar la versión 8 o superior de Java y el uso de la versión 5 de Spring Framework.

La necesidad de utilizar como mínimo la versión 8 de Java supone poder explotar todas las novedades que se añadieron a dicha versión, como interfaces funcionales, funciones lambda o interfaces con métodos implementados por defecto, por citar algunas.

El uso de la versión 5 de Spring Framework supone poder implementar directamente el paradigma reactivo con Spring Boot. Lo que quiere decir que se puede utilizar Netty como servidor embebido, en vez de contenedores de servlets más tradicionales como Tomcat, Jetty o Undertow.

En este punto es curioso comentar como los distintos lenguajes de programación, y las herramientas construidas en torno a ellos, se están complementando. Por ejemplo, la forma de escribir una aplicación con Angular, el popular framework para JavaScript, se parece cada vez a más a la forma en que se escriben aplicaciones en Spring. Es decir, utilizando un estilo basado en anotaciones, llamados decoradores en JavaScript, para indicar si las clases son controladores o servicios, o para enriquecerlas con cualquier otra metainformación. Por su parte, Spring ha adoptado de forma nativa en Java el modelo de servidor con un único hilo de ejecución utilizado tradicionalmente en JavaScript, como en Express, el popular servidor HTTP de node.js.

Todos los cambios y novedades que incorpora Spring Boot 2 pueden encontrarse en la página con las release notes del framework, incluida una guía de migración que incluye una referencia a un módulo que han implementando de forma explícita para facilitar dicha migración. Lo que sigue de este artículo son unas breves notas sobre un conjunto pequeño de las novedades de Spring Boot 2 que me han resultado interesantes con algunos comentarios de mi parte.

## 1.1. Reactive

La adopción de la versión 5 de Spring Framework permite utilizar Spring Webflux como alternativa a Spring MVC, lo que se traduce en que ahora se pueden construir aplicaciones totalmente asíncronas y no bloqueantes con Spring Boot.

Este cambio es realmente importante dentro del uso de Spring y merece la pena dedicarle un tiempo a entenderlo bien. Lo primero que hay que comprender es que el paradigma tradicional dominante en Java es atender cada petición en un hilo de ejecución (thread) distinto. De forma que si una petición accede a un recurso bloqueante, como cuando realiza una consulta a una base de datos por ejemplo, el hilo detiene su ejecución hasta que obtiene una respuesta de dicho recurso. Mientras un hilo está bloqueado no puede utilizarse para otra petición, por lo que un sistema con muchas peticiones puede llegar a bloquearse si todos sus hilos de ejecución se encuentran bloqueados.

Una implementación alternativa a la expuesta en el párrafo anterior es utilizar un único hilo de ejecución. Es decir, con un servidor que tan sólo dispone de un hilo de ejecución a través del que atiende todas las peticiones. Cuando una petición entra es atendida completamente antes de atender la siguiente, excepto cuando la petición invoca un recurso, en cuyo caso se la deja esperando hasta que el recurso responde. La que espera es la petición, no el hilo de ejecución, que puede seguir atendiendo peticiones.

Lógicamente, la clave de esta implementación es que todos los recursos deben poder ser accedidos de forma no bloqueante. Una consulta de base de datos no debe bloquear la petición, debe retornar inmediatamente y señalizar de alguna forma su resultado cuando disponga del mismo.

En Java ya existían servidores que implementaban el patrón reactivo, Spring 5 simplemente lo ha incorporado a su caja de herramientas y Spring Boot ha facilitado el uso del mismo. No obstante, la adopción de Spring 5 no implica el uso obligatorio de WebFlux con Spring Boot 2, se pueden seguir desarrollando aplicaciones con Spring MVC con Spring Boot 2 como hasta ahora.

## 1.2. Reactive Data

Como se ha explicado en el apartado anterior, implementar una aplicación completamente asíncrona y no bloqueante implica que todos los recursos deben ser también asíncronos y no bloqueantes. En una aplicación tradicional esto supone que el acceso a una base de datos por ejemplo debe realizarse mediante un driver no bloqueante.

Spring Boot ha añadido módulos que facilitan el acceso al API asíncrono de las bases de datos NoSQL más populares como Redis, MongoDB o Cassandra. Sin embargo, las bases de datos relacionales más tradicionales accedidas a través de JDBC o JPA aún necesitan utilizar algún tipo de driver asíncrono, normalmente no oficial, o una librería de terceros que implemente un wrapper sobre el driver original.

Por lo tanto, es importante tener en cuenta que si una aplicación va a estar ligada a uno o más recursos accedidos de forma bloqueante, por ejemplo a través de un API como JDBC o JPA, entonces una mejor opción a WebFlux puede ser utilizar Spring MVC.

Oracle tiene una propuesta en marcha para el desarrollo de un API para el acceso no bloqueante a base de datos, pero de momento no ha llegado a cristalizar.

## 1.3. Netty

El principal cambio de Spring Boot 2 respecto a los servidores embebidos soportados es la incorporación de Netty, el servidor asíncrono no bloqueante por antonomasia, para la implementación de aplicaciones con WebFlux.

Los servidores web tradicionales como Tomcat, Jetty y Undertow siguen siendo opciones disponibles para desarrollar aplicaciones con Spring MVC, pero con el requerimiento de implementar como mínimo la versión 3.1 del API Servlet para utilizar WebFlux, ya que fue la primera versión que incorporó la gestión de operaciones de E/S de forma no bloqueante.

A este respecto, comentar que Spring Boot 2 ha actualizado su dependencia con Tomcat a la versión 8.5, y ahora utiliza HikariCP como pool de conexiones, en vez del pool por defecto de Tomcat.

## 1.4. HTTP/2

HTTP/2 introdujo muchas mejoras al protocolo HTTP, como un formato binario más eficiente de procesar que el formato anterior en texto plano, la capacidad de atender múltiples peticiones a través de una misma conexión en vez de tener que establecer una nueva por cada petición, una reducción en la cantidad de paquetes necesarios para intercambiar las cabeceras HTTP mediante el uso de compresión, e incluso la posibilidad de que los servidores tomen la iniciativa en el envío de información a los clientes.

Mejoras todas ellas que se han reflejado en un aumento del rendimiento a la hora de descargar recursos estáticos de una web, como hojas de estilos e imágenes. Su explotación por parte de los servicios REST de momento parece haberse restringido a una mejora del rendimiento motivado por las propias mejoras de rendimiento en el protocolo.

Spring Boot 2 incorpora la posibilidad de utilizar HTTP/2 con Tomcat, Jetty y Undertow, aunque requiere una configuración específica en cada caso y el uso de alguna librería nativa de terceros en según que caso. La combinación de Tomcat 9 con Java 9 o superior es particularmente interesante, ya que soporta el protocolo sin requerir de ninguna librería adicional.

## 1.5. Actuators

Los actuators son un conjunto de utilidades que pueden añadirse de forma automática a las aplicaciones desarrolladas con Spring Boot, y a las que pueden accederse mediante JMX o HTTP. Permiten comprobar si la aplicación está levantada, consultar sus parámetros de configuración, u obtener métricas en tiempo real, por citar sólo algunos casos de uso.

En Spring Boot 2 se han unificado las rutas HTTP para acceder a dichos servicios a través de un único endpoint ```\actuator```, se ha mejorado la calidad y estructura de la información retornada en los JSON, se ha adoptado HATEOAS para el autodescubrimiento de servicios mediante enlaces hipermedia en formato HAL, y se ha cambiado el sistema de métricas propio original configurado por defecto a favor de micrometer.

Esto último acerca de las métricas es más importante de lo que pueda parecer en un principio, sobre todo en el mundo actual donde priman los microservicios. Spring Boot recopila de forma automática una gran cantidad de métricas acerca del funcionamiento de las aplicaciones. Mide aspectos tales como el rendimiento de la máquina virtual de Java, la latencia de los servicios, el uso de la cache, el número de conexiones, el uso de aplicaciones específicas como Tomcat o RabbitMQ, y muchos otros parámetros. Métricas todas ellas que se pueden etiquetar y enviar a un servidor, o varios de ellos, para su procesamiento y monitorización, normalmente de forma agregada.

Para terminar, comentar que otra mejora importante es que se ha añadido a los actuators un API para exponer servicios propios de manera agnóstica, es decir, sin ligarlos a JMX o HTTP. Los beans de Spring anotados con ```@Endpoint```, y con métodos anotados con ```@ReadOperation```, ```@WriteOperation``` y ```@DeleteOperation``` son automáticamente expuestos por JMX, y por HTTP en el caso de encontrarse dentro de una aplicación web.

Resumiendo, en esencia Spring Boot sigue siendo el mismo que antes, la versión 2 incorpora las novedades de Spring 5 y refleja mejor el estado actual del desarrollo de aplicaciones web con Java.
