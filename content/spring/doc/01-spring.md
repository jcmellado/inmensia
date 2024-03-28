# 1. Spring: Introducción

_19-04-2012_ _Juan Mellado_

Spring es uno de los _frameworks_ de desarrollo de aplicaciones para Java más reconocidos y utilizados en el ámbito empresarial. Abarca una gran cantidad de aspectos, tiene una filosofía propia, y precisamente esas virtudes se convierten a veces en sus mayores desventajas a la hora de empezar a utilizarlo. A mi siempre me ha parecido, como decía aquella canción, algo así como "feo, fuerte y formal".

Cuando se empieza a leer la documentación, lo primero que se encuentra son dos conceptos muy importantes que se repiten una y otra vez: "Inyección de Dependencias" (DI) e "Inversión de Control" (IoC). Son un poco sutiles y tienden a confundirse entre ellos.

La inyección de dependencias se refiere al hecho de que utilizando Spring no es necesario indicar de forma explícita las dependencias entre los componentes de una aplicación. Es decir, no hace falta grabar a fuego en el código fuente que el objeto A utiliza los objetos B y C que siguen una implementación concreta. Se puede dejar un poco en el aire e indicar las dependencias en una configuración aparte. Spring "inyectará" la dependencia apropiada en tiempo de ejecución. Así dicho suena un poco esotérico, por lo que recomiendo esperar hasta que se vean unos ejemplos más adelante.

La inversión de control por su parte sigue el conocido como "Principio de Hollywood", que es la famosa frase que le sueltan a los aspirantes a estrellas en la meca del cine: "No nos llame, ya lo llamaremos". En realidad no es nada nuevo, la programación orientada a eventos lleva bastante tiempo entre nosotros. No obstante, Spring lleva este concepto un poco más lejos e incluye la instanciación de objetos como parte de la inversión de control. Es decir, no hace falta que una clase A cree el típico método inicial que instancie los objetos que necesita de las clases B y C. Spring "llamará" a los constructores adecuados cuando sea necesario según la configuración. ¿Magia? Un poco, toca esperar a los ejemplos.

Notar que a estas alturas ya ha aparecido varias veces la palabra "configuración". Una de las críticas más habituales que siempre se hace cuando se empieza a utilizar Spring es que hay mucho que configurar. La configuración se realiza normalmente en ficheros ```.xml``` aparte, o dentro del propio código Java, ya que Spring proporciona un abanico de opciones muy grande a este respecto. La principal queja es que todo el tinglado a veces es difícil de entender por alguien ajeno al proyecto y requiere cierto bagaje. Lo normal es quejarse, todo el mundo lo hace, no se sienta incómodo haciéndolo.

Llegado este punto es normal preguntarse cómo diablos funciona en realidad Spring. ¿Qué servicios ofrece? ¿Por qué todo el mundo empieza explayándose acerca de la inyección de dependencias e inversión de control en vez de ir directos al grano?

Spring no es una librería corriente en ese aspecto. Un ejemplo, hoy en día el patrón _Singleton_ es muy conocido, y todo el mundo lo implementa de una u otra forma en su código, cada uno a su manera. Spring lo que hace es implementarlo por nosotros y ofrecerlo como un objeto de primera clase. Permite que trabajemos con clases POJO, es decir clases Java "planas", sin complicarnos la vida acerca de los detalles de implementación de ese u otro patrón de diseño. De hecho, uno de los aspectos clave de Spring es la implementación "con esteroides" que tiene del patrón _Factory_, que es el que le permite realizar la inyección de dependencias e inversión de control. El núcleo de Spring presta atención al ciclo de vida de una aplicación, y a partir de ahí empieza a ofrecer servicios más concretos.

Vale, eso está muy bien, pero sigue sonando muy abstracto, ¿qué servicios ofrece _Spring Framework_?

La funcionalidad se divide en varios grupos principales que a su vez se dividen en módulos:

- El grupo **Core Container** tiene toda la chicha del núcleo. Contiene los módulos "_Core and Beans_", "_Context_" y "_Expression Language_". Los _beans_ son los componentes básicos con los que trabaja Spring, básicamente todo son _beans_, lo que permite dar un tratamiento uniforme a todos los componentes de una aplicación. El contexto es a través del que se accede a los _beans_, y añade además características de internacionalización (_i18n_), carga de recursos, propagación de eventos, etc. El lenguaje de expresión de Spring (_SpEL_) permite utilizar expresiones elaboradas a la hora de definir los objetos, parámetros y demás.

- El grupo **Data Access/Integration** facilita el acceso a (base de) datos e integración. Contiene los módulos "_JDBC_", "_ORM_", "_OXM_", "_JMS_" y "_Transaction_". Mucha gente empieza a sentirse cómoda a partir de aquí. ¿JDBC? ¡Por fin algo que se entiende! ORM para el clásico mapeo de entidades en un modelo relacional utilizando JPA, JDO, Hibernate, o iBatis. OXM para utilizar XML con librerías como JAXB, Castor, XMLBeans, JiBX, o XStream. JMS para las colas de mensajes. Y transacciones sobre objetos POJO de una forma sencilla.

- El grupo **Web** lógicamente es para la web. Contiene los módulos "_Web_", "_Web-Servlet_", "_Web-Struts_", y "_Web-Portlet_". Los nombres son bastante descriptivos, sólo mencionar que Spring se adhiere al clásico patrón MVC (_Modelo-Vista-Controlador_).

- El grupo **AOP and Instrumentation** se utiliza para la programación orientada a aspectos (todo un mundo en si mismo) con AspectJ e instrumentación de aplicaciones.

- El grupo **Test** da soporte para la implementación de pruebas. Principalmente con JUnit, aunque se puede utilizar otra librería, ya que una constante de Spring es no forzar a las aplicaciones a utilizar una librería concreta.

¿Y ya está? Pues en realidad no, eso es sólo el principio, el "_Spring Framework_" a secas. Se ha hecho mucho trabajo alrededor de esta base y existen extensiones muy importantes como "_Spring Data_", "_Spring Web Services_", "_Spring Security_", "_Spring Batch_", "_Spring Mobile_", "_Spring Social_", ...

Como se observa, enumerar los servicios de _Spring Framework_ no sirve de mucho, por eso nadie empieza por ahí, sólo sirve para ver lo "grande" que es Spring. Lo importante es su filosofía de funcionamiento, que eso sí, a primera vista resulta un poco "fea" con tanta configuración. Lo importante es que cumple su cometido, es bastante "formal" en ese aspecto. Claro que para conseguirlo hay que remangarse, descargar algunos _jars_, y empezar a tirar líneas de código. Propósito del siguiente capítulo de esta serie.
