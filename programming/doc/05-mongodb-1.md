# 5. MongoDB (1)

_21-08-2018_ _Juan Mellado_

Estos últimos meses están siendo muy prolíficos en cuanto a _major releases_ se refiere. MongoDB recientemente publicó la versión 4 de su conocida base de datos NoSQL. La mayor novedad es que permite realizar operaciones sobre varios documentos dentro de una misma transacción de forma atómica. Una característica que al parecer llamó más la atención cuando se anunció que ahora que se ha liberado. No obstante, el cambio es importante, posiblemente el de mayor impacto desde que en la versión 3.2 se introdujo la posibilidad de realizar "_left outer joins_" entre documentos de colecciones distintas.

Decidir si MongoDB es la herramienta adecuada para un proyecto es un tema recurrente cuando se decide la arquitectura de una aplicación. A veces la tecnología viene impuesta por contrato, la mayoría de veces simplemente se utiliza lo ya que se conoce, y en algunas ocasiones se utiliza sólo para poder añadirla al portfolio de la empresa.

MongoDB es una base de datos, por lo que decidir utilizarla dependerá en buena medida del modelo de datos y los casos de uso fundamentales que se necesitan cubrir, ya que un modelo de datos no tiene porque ser visto sólo como un mero almacén de información. Las operaciones que se van a realizar sobre él son tan importantes como sus entidades y atributos. Y junto a ellos, las estimaciones de los volúmenes de información que se espera manejar de cara a decidir que gestor de base de datos utilizar.

## 5.1. Modelo Dinámico

MongoDB dice de si mismo que es adecuado cuando los modelos de datos no sean estáticos, cuando instancias de una misma entidad puedan tener un conjunto distinto de atributos. Un ejemplo: los productos de una gran superficie comercial. No es lo mismo una barra de pan que un calentador de agua, pero no por ello dejan de ser "productos". El calentador tendrá marca, modelo, altura, anchura, profundidad, capacidad, y otra serie de atributos propios de acuerdo a sus especificaciones técnicas. Una barra de pan tendrá peso, tamaño, variedad, y otros atributos de acuerdo a su composición química. Es bastante probable que sus datos provengan de fuentes muy distintas y se realice un tratamiento distinto para  cada uno de ellos, pero no por eso dejan de ser productos de un mismo catálogo que sería deseable tratar de una forma conjunta.

En la práctica la mayoría de este tipo de problemas se resuelven en tiempo de diseño adoptando soluciones fáciles de implementar. Siguiendo con el ejemplo del catálogo de productos, es bastante habitual que todos los atributos específicos de un producto acaben en un campo de texto de gran tamaño a modo de descripción. Lógicamente, el mayor inconveniente de esta solución es que la información no se puede explotar de una manera sencilla al no estar estructurada. Otra solución es utilizar MongoDB.

En MongoDB las entidades se llaman documentos y se guardan en colecciones mediante transacciones ACID. Cada documento dentro de una misma colección puede tener el número de campos que se quiera, y cada campo puede ser del tipo que se quiera. Lógicamente se tiende a guardar documentos del mismo tipo en una misma colección. Así, una colección de "productos" podría contener un documento con campos que describan una barra de pan y otro que describan un calentador de agua. Es un caso de uso que MongoDB cubre de una manera natural gracias a que trabaja con un modelo de datos dinámico, frente a una base de datos relacional que trabaja con un modelo de datos predefinido.

## 5.2. JSON

MongoDB capturó la atención de muchos desarrolladores presentando los documentos en formato JSON. Debido a que el uso de este formato estaba universalmente extendido con el auge de JavaScript, pensar que las colecciones de documentos de MongoDB eran _arrays_ de objetos en formato JSON facilitaba trabajar con ellas. No obstante, también llevaba a equívocos. Es un caso similar a lo que ocurría años atrás con el formato XML. En algunos modelos de datos se almacenaban datos en formato XML en columnas de tipo texto plano. El inconveniente es que todo el procesamiento de dichas columnas había que hacerlo manualmente, ya fuese en el servidor de aplicaciones tras recuperarlas de base de datos, o en procedimientos PL/SQL en el propio servidor de base datos. Algunas bases de datos relacionales reaccionaron llevando a cabo desarrollos específicos para que las columnas de las tablas pudieran almacenar y explotar información directamente en formato XML, pero al no ser parte de ningún estándar implicaba ligarse a un determinado proveedor. Con el formato JSON ocurre algo similar, algunas base de datos permiten almacenar información en formato JSON, pero en muchos casos no dejan de ser columnas en formato de texto plano optimizadas para eliminar espacios en blancos y claves duplicadas de los objetos JSON almacenados en ellas.

MongoDB no almacena los objetos en formato JSON, lo hace en un formato binario llamado BSON. Y gracias a esto MongoDB puede realizar operaciones de una forma eficiente sobre cualquier campo, de igual forma que una base de datos relacional opera con el valor de una columna de una tabla.

## 5.3. Joins

El gran caballo de batalla.

Tener la información totalmente estructurada pasa por tenerla totalmente normalizada. Cada cosa en su sitio, y un sitio para cada cosa. Por ejemplo, el registro de los clientes en una tienda _web_ puede requerir una serie de campos básicos como nombre y apellidos, pero también formas de contacto como teléfonos y direcciones, que pueden ser de varios tipos, como dirección de envío o dirección de facturación, y a su vez las direcciones pueden hacer referencia a países, o localidades más concretas dentro de un país. Es decir, es posible identificar un conjunto de entidades perfectamente delimitadas con los atributos que le son propios, así como sus relaciones con otras entidades.

Sin embargo, cuanto más estructurada esté la  información, más esparcida estará por el modelo. Insertar, recuperar, actualizar o borrar una entidad completa puede suponer tener que acceder a una gran cantidad de entidades relacionadas con ella. Lógicamente, hacer una consulta por cada entidad sería muy ineficiente, por lo que el procedimiento habitual es realizar una única consulta que recupere todas las entidades de una sola vez a través de sus relaciones. Es decir, mediante lo que normalmente se denomina "hacer un _join_". Cuantas más relaciones existan, más _joins_ será necesario realizar, más compleja y costosa será la consulta, y todo ello sin tener en cuenta la relaciones opcionales, es decir, lo que normalmente se denomina "hacer un _left outer join_". Otra solución es utilizar MongoDB.

En MongoDB las entidades se pueden almacenar de forma anidada unas dentro de otras. Es decir, un documento puede tener dentro otro documento, o conjunto de ellos. O dicho de otra forma, un objeto JSON puede tener un campo que a su vez sea otro objeto JSON, o un _array_ de objetos JSON. Volviendo al ejemplo anterior, un documento de la colección "clientes" puede tener un atributo "direcciones" que sea un _array_. De esta forma, en una única operación, ya sea de lectura o escritura, se tiene acceso a la entidad completa. Esta es la diferencia fundamental entre una base de datos relacional y una base de datos orientada a documentos como es MongoDB.

No obstante, llegados a este punto, surgen de manera natural muchas dudas acerca de las entidades embebidas. Continuando con el ejemplo de los clientes, ¿hay que repetir el nombre de un mismo país en todas las direcciones de todos los clientes de dicho país?, ¿no se puede tener una colección de países y que las direcciones hagan referencia a ella?, ¿cómo se garantiza la integridad referencial?, etc. Resumiendo: ¿cómo se crean "_foreign keys_" y se hacen "_joins_" en MongoDB?

Vayamos por partes.

Cada documento en MongoDB tiene un campo de nombre ```_id`` que contiene un valor que identifica dicho documento de manera unívoca. Es decir, es equivalente a una "_primary key_" en una base de datos relacional. Por tanto, para crear una "_foreign key_" entre un documento A (dirección) y un documento B (país) hay que añadir un campo en el documento A (dirección) que contenga el valor del identificador de B (país). Pero teniendo siempre presente que MongoDB no garantiza la integridad referencial. Es responsabilidad de los desarrolladores. MongoDB no es una base de datos relacional, es orientada a documentos.

Por otra parte, comentar que inicialmente MongoDB no permitía recuperar varios documentos relacionados de una sola vez. Si todos los documentos tenían todos sus atributos entonces no era necesario realizar más consultas, dicha opción no tenía sentido. Si se quería obtener un documento relacionado había que hacer una segunda consulta. No obstante, con el tiempo, MongoDB añadió esta característica. Es decir, a día de hoy, cuando se hace una consulta en MongoDB, se puede hacer un "_join_" entre dos colecciones indicando en una cláusula el nombre de las colecciones y campos sobre los que realizar el "_join_".

Pero a pesar de contar con todas estas opciones, los creadores de MongoDB defienden un modelo mixto. No ven mal el uso de "_foreign keys_" y "_joins_", pero recomiendan copiar los atributos más habitualmente usados de los documentos referenciados en los documentos que contienen la referencia. Lo que aplicado al ejemplo de los clientes de una tienda _web_ quiere decir que una dirección debería tener un campo con una referencia al país, pero también debería tener un campo con el nombre del país si el caso de uso más habitual es imprimir el nombre del país junto a la dirección. La redundancia de datos la justifican aduciendo que hoy en día el almacenamiento es económico y los campos habitualmente candidatos a ser redundados no suelen cambiar nunca con el tiempo.

## 5.4. Índices

Hubo un tiempo en que toda la información se presentaba en registros en un formato tabular. Cada valor en un columna distinta. Cada columna con un nombre. Y junto a cada nombre un botón para ordenar ascendente o descendentemente. Era un desastre.

Las base de datos son buenas almacenando información y recuperándola por clave primaria, pero penalizan mucho el rendimiento cuando se realizan consultas por cualquier campo de manera indiscriminada, sobre todo cuando el número de registros es elevado. El problema es que se les obliga a recorrer todos los registros para comprobar cuales cumplen los criterios de filtrado u ordenación de la consulta. Por ello dejar que cualquier tabla se ordenase por cualquier columna era una receta para el desastre.

La solución a este tipo de problema es obviamente cambiar la forma de presentar la información, restringir las opciones, y crear índices cuando sea preciso. Cuando se diseña un modelo de datos es importante identificar las consultas que se realizarán y planificar los índices adecuados sobre las entidades correspondientes.

MongoDB en este apartado no se diferencia del resto de base de datos. Permite crear índices sobre los campos de los documentos para agilizar las consultas. De hecho, hacer un "_join_" en MongoDB es inviable sin definir un índice en la colección a la que se referencia. Y penaliza de igual forma que cualquier otra base datos, con una mayor necesidad de espacio y con un mayor tiempo al insertar o actualizar.

## 5.5. Fragmentación

Los gestores de base de datos relacionales reservan un espacio determinado para las filas de una tabla. Espacio que normalmente se llama bloque y coincide en tamaño con el número de _bytes_ que pueden recuperar de disco en un sólo acceso de forma óptima. Si se inserta un registro con muchas columnas a nulo queda mucho espacio que pueden aprovechar otras filas. Si posteriormente se actualiza la fila cambiando sus nulos por valores no nulos entonces la fila reclama su espacio dentro del bloque. Si no hay espacio disponible dentro del bloque la fila se reparte entre dos bloques. Acceder a un bloque implica leer de disco duro, por lo que cuanto más bloques se requiera para una fila más lento será el acceso.

MongoDB gestiona el espacio de una forma similar. Insertar un documento dentro de una colección requiere tanto espacio como campos con valores no nulos tenga. Si posteriormente se actualiza el documento con nuevos campos no nulos reclamará su espacio. Si no hay espacio disponible el documento quedará fragmentado.

Por mi experiencia, este detalle del funcionamiento interno de los gestores de base de datos no se suele tener en cuenta, pero puede afectar al rendimiento de una aplicación. La situación ideal es tener entidades que no contengan valores nulos y que una vez insertadas no se modifican. Parece un escenario utópico, pero debería ser considerado parte de los objetivos de diseño más a menudo.

Si se da la circunstancia, es bueno saber que MongoDB ofrece comandos para desfragmentar las colecciones de igual forma que las base de datos relacionales, ya que puede afectar significativamente al rendimiento según el caso de uso.

## 5.6. Agregaciones

Hay una tendencia a evaluar una base de datos pensado en su rendimiento de cara a proporcionar información en tiempo real a una interface de usuario. No obstante, hay casos de uso que pueden resolverse mediante ejecuciones de procesos desatendidos donde el tiempo de respuesta no es un factor crítico. En esos casos la facilidad para manipular la información debería ser el factor más importante a tener en cuenta.

Utilizar sentencias SQL para generar informes puede que no sea la mejor solución, ya que si bien es cierto que permiten extraer registros de una forma sencilla, también lo es que carecen de capacidades avanzadas para agregar y manipular dicha información de una forma sencilla. Otra solución es utilizar MongoDB.

MongoDB proporciona un API exclusivo para realizar agregaciones. Es decir, para recolectar información existente en las colecciones y generar nueva información a partir de ellas. El API es bastante versátil y permite definir una secuencia de operaciones a realizar. Las operaciones generan resultados o modifican el resultado de las operaciones anteriores. En esencia es como programar pequeños fragmentos de código, añadirlos a una lista y pedir al servidor que los ejecute sobre una colección.

## 5.7. Conclusiones

Desde el punto de vista de un desarrollador puro y duro, hoy en día no hay muchas diferencias entre MongoDB y una base de datos relacional. Hay "_primary keys_", "_foreign keys_", "_joins_", índices, agrupaciones, …

Si el modelo de dominio del negocio se adecua a las características de un modelo orientado a documentos entonces se puede sacar partido del hecho de poder almacenar y recuperar toda una entidad de una sola vez. Y si no se adecua a dicho modelo entonces siempre se puede realizar un modelado de forma similar al que se haría en una base de datos relacional.

No obstante, si la única ventaja de MongoDB fuera que permite trabajar con las entidades de forma completa como objetos en formato JSON entonces habría que considerar otras soluciones. Por ejemplo, la exitosa versión 10 de PostgreSQL permite definir columnas de tipo JSONB, que es una representación de JSON en formato binario, para poder operar sobre los campos de un JSON de forma eficiente, e incluso definir índices sobre los mismos para agilizar las consultas como MongoDB.

Por su parte, la falta de integridad referencial no debería ser un factor crítico a tener en cuenta. No recuerdo haber tenido que resolver nunca un problema en producción por culpa de una "_foreign key_". Ese tipo de error se detecta normalmente de forma muy temprana durante el propio desarrollo.

Otro punto importante es el rendimiento, pero los _benchmarks_ son muy difíciles de hacer y muy fáciles de malinterpretar. Una batería de pruebas de una base de datos no puede servir para predecir como se comportarán todas las aplicaciones en producción que utilicen dicha base de datos. No importa que la base de datos sea extremadamente rápida realizando escrituras cuando el caso de uso típico de una aplicación son las lecturas. Y de igual forma, no importa que la base de datos sea capaz de realizar miles de lecturas por segundo cuando el caso de uso típico de la aplicación es de decenas de lecturas por minuto.

Hace unos pocos años hubo una presentación de un caso de éxito de MongoDB. El rendimiento era espectacular. Hasta que dijeron las especificaciones del CPD. Era tan impresionantes que es bastante posible que el sistema hubiera rendido igual de bien con una base de datos relacional.

¿Entonces?

En mi opinión, el factor clave no es el conjunto de características que MongoDB ofrece a los desarrolladores, ni que el _setup_ sea simple, ni que existan _drivers_ para los principales lenguajes de programación e integración con los _frameworks_ más populares. El factor clave es la escalabilidad horizontal. MongoDB fue diseñada desde el principio con ese objetivo específico en mente. Hoy funciona para cien, mañana funciona para un millón. Las bases de datos relaciones tradicionales no escalan de igual forma que MongoDB, ni que la mayoría de las bases de datos NoSQL en general. Estas últimas han sido creadas en un momento muy distinto, con unas coyunturas y necesidades totalmente distintas.

MongoDB es una alternativa a considerar cuando se quiere un desarrollo más dinámico, donde se pueda ir construyendo y ampliando. Y sobre todo cuando se prevea un crecimiento exponencial. Las bases de datos relaciones tradicionales no suelen tener mecanismos de escalado de esas dimensiones, y en caso de ofertar algo similar es normalmente mediante soluciones parciales, como el particionado de tablas, o con _software_ adicional con licencias propietarias o productos de terceros.
