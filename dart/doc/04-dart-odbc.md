# 4. dart-odbc

_19-03-2013_ _Juan Mellado_

dart-odbc es un _binding_ a ODBC para Dart. Es decir, una librería que permite utilizar el API de ODBC directamente desde Dart. Mi objetivo inicial era conectarme a una base de datos Oracle y hacer un par de consultas, pero al final he acabado exponiendo todo el API completo.

- [https://github.com/jcmellado/dart-odbc](https://github.com/jcmellado/dart-odbc)

Con esta librería es posible conectarse desde un programa escrito en Dart a cualquier base de datos que tenga un driver ODBC. Y como hoy en día prácticamente todas las base de datos tienen un driver ODBC, pues equivale a decir que permite conectarse a cualquier base de datos. En mis pruebas he conseguido conectarme a servidores Oracle, MySql, PostgreSQL, y SQL Server.

En lo referente a la parte técnica, comentar que este proyecto tiene dos partes, por un lado una parte escrita en Dart, que expone las funciones del API para que se puedan llamar desde aplicaciones escritas en Dart, y por otro lado una parte escrita en C++, que actúa como puente entre el API de Dart y el API de ODBC, que está escrito en C.

El invento funciona gracias a que Dart permite llamar a funciones contenidas dentro de una librería externa escrita en C. Es decir, que permite llamar a funciones contenidas dentro de una ```.dll``` (Windows), un ```.so``` (Linux), o una ```.dylib``` (Mac). Es bastante similar al funcionamiento de la palabra reservada ```native``` de Java o los ```DllImport``` de C#. De hecho, en Dart también se utiliza la misma palabra reservada ```native``` para denotar funciones que están implementadas en librerías externas.

Además de eso, el equipo de desarrollo de Dart ha publicado el API de bajo nivel para que pueda ser llamado desde fuera de la VM. Es decir, existe un ```dart_api.h``` y un ```dart.lib``` que se pueden incluir en cualquier proyecto y hacer las mismas cosas que normalmente se hacen desde Dart, como crear objetos, acceder a atributos, o invocar métodos. Se puede crear un objeto Dart desde C y devolverlo como resultado de la función C que se está ejecutando, por ejemplo. O recibir un objeto Dart como argumento y acceder a sus atributos.

ODBC es un API de muy bajo nivel, así que la librería está más orientada a ser utilizada por desarrolladores de _frameworks_ que de _front-ends_. No obstante, con una buena capa de abstracción, a la manera de JDBC por ejemplo, podría utilizarse en ambos mundos.

De momento, sin _wrappers_ de por medio, el código Dart resultante es muy similar al que normalmente se escribe en C:

```dart
// Allocate an environment handle.
var hEnv = new SqlHandle();

sqlAllocHandle(SQL_HANDLE_ENV, null, hEnv);

// Set ODBC version to be used.
var version = new SqlPointer()..value = SQL_OV_ODBC3;

sqlSetEnvAttr(hEnv, SQL_ATTR_ODBC_VERSION, version, 0);

// Allocate a connection handle.
var hConn = new SqlHandle();

sqlAllocHandle(SQL_HANDLE_DBC, hEnv, hConn);

// Connect.
sqlConnect(hConn, "", SQL_NTS, "", SQL_NTS, "", SQL_NTS);

// Allocate a statement handle.
var hStmt = new SqlHandle();

sqlAllocHandle(SQL_HANDLE_STMT, hConn, hStmt);

// Prepare a statement.
sqlPrepare(hStmt, "SELECT name FROM customers WHERE customer_id = ?", SQL_NTS);

// Bind parameters.
var customerId = new SqlIntBuffer()..poke();

sqlBindParameter(hStmt, 1, SQL_PARAM_INPUT, customerId.ctype(),
SQL_INTEGER, 0, 0, customerId.address(), 0, null);

// Bind result set columns.
var name = new SqlStringBuffer(32);
var flags = new SqlIntBuffer();

sqlBindCol(hStmt, 1, name.ctype(), name.address(), name.length(), flags.address());

// Execute and fetch some data.
sqlExecute(hStmt);

sqlFetch(hStmt);

...

// Free statement handle.
sqlFreeHandle(SQL_HANDLE_STMT, hStmt);

// Disconnect.
sqlDisconnect(hConn);

// Free connection handle.
sqlFreeHandle(SQL_HANDLE_DBC, hConn);

// Free environment handle.
sqlFreeHandle(SQL_HANDLE_ENV, hEnv);
```

Estoy bastante satisfecho en términos generales con el funcionamiento de la librería, no obstante hay bastante margen aún para mejorar, particularmente en los siguientes puntos:

- Compilar la librería en otros entornos, además de en Windows. Actualmente sólo tengo la librería compilada en forma de DLL para Windows, para Win32 realmente, pero funcionando sin problemas en Win64. Me gustaría poder generar versiones para Linux o Mac. He echado la caña a ver si consigo que alguien pique y colabore. Añadir soporte para UNICODE también estaría bien.

- Mejorar el rendimiento. Aunque la librería no realiza ningún tipo de proceso pesado, es una mera intermediaria, hay un par de valores que pienso que podrían estar precalculados, en vez de calcularlos cada vez, como son el tamaño de las filas de los _buffers_ y los _offsets_ a las columnas. Actualmente se calculan por cada _peek_ o _poke_ que se realiza a los _buffers_.

- Revisar el uso de la memoria. Sobre todo a largo plazo, con un uso intenso y continuado, no sólo simples pruebas que se ejecutan y terminan de forma casi inmediata sin apenas consumir recursos. Le tengo echado el ojo a una utilidad que utiliza el equipo que está desarrollando Chrome para estos menesteres, y esta puede ser una buena excusa para probarla.

- Estudiar la posibilidad de exponer las llamadas al API de ODBC de forma asíncrona. Esta posibilidad la evalúe al principio, pero la descarte rápidamente porque la documentación del API de Dart especifica que algunas cosas no pueden hacerse cuando se está funcionando en modo asíncrono, al no existir un "_isolate_" (hilo de ejecución) concreto durante la ejecución. Como no sabía donde me metía lo dejé pasar, pero ahora que sé lo que hace falta, es un tema que podría revisar en el futuro.

- Añadir ejemplos de uso. Todo el proyecto lo he realizado en base a pruebas, y prácticamente existe un _test_ para cada función del API de ODBC. Lo que me gustaría incorporar es algún caso concreto de uso, como la típica aplicación de consola en la que escribes consultas SQL y te muestra los resultados por pantalla.

- Añadir más documentación. Una ventaja de este proyecto es que no tengo que explicar el API, hay mucha información disponible sobre ODBC. Pero quizás podría añadir alguna explicación más detallada acerca de cuando utilizar los objetos ```SqlBox``` y cuando los objetos ```SqlBuffer```. Los primeros son para llamadas al API que devuelven un valor de forma inmediata ("_dame el valor ahora_"), mientras que los segundos son para cuando los valores se devuelven de forma diferida ("_dame el valor cuando lo tengas_").
