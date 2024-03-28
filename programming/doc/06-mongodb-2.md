# 6. MongoDB: Hands on&excl;

_24-08-2018_ _Juan Mellado_

MongoDB es una base de datos NoSQL distribuida y escalable que gestiona objetos almacenados en un formato llamado BSON. Un formato inspirado en el formato JSON, pero que almacena la información de forma binaria. Es habitual pensar en los objetos almacenados en MongoDB, llamados documentos, como objetos en formato JSON.

Una de las características más significativas de MongoDB es que es una base de datos de esquema dinámico, es decir, que no fuerza un esquema fijo predefinido como las base de datos relacionales. Cada documento (objeto JSON) puede tener los campos y tipos que se quieran. Aunque si realmente se necesita, es posible forzar unas reglas de validación siguiendo la especificación JSON Schema.

MongoDB permite almacenar los documentos agrupados en colecciones, equivalente a las tablas del modelo relacional, y realizar consultas sobre estas colecciones. Estas consultas pueden realizarse sobre cualquier campo de los documentos de una manera eficiente, ya que es posible crear índices, tanto sobre campos de primer nivel, como campos de objetos JSON anidados, e incluso _arrays_. Además, es posible realizar consultas agregadas sobre cualquier campo, a la manera de una cláusula ```GROUP BY``` en SQL. MongoDB da una gran importancia a las agregaciones, permitiendo incluso ejecutar varias agregaciones en paralelo con una única consulta.

## 6.1. Instalación

La instalación de MongoDB puede realizarse en local descargando un fichero comprimido, o alternativamente usando una imagen de Docker.

En Windows el ejecutable del servidor es ```mongod.exe```, y para arrancarlo hay que pasarle el parámetro ```-dbpath``` con el nombre de un directorio existente. El cliente de línea de comandos es ```mongo.exe```, y al arrancarlo se conectará automáticamente al servidor local por defecto.

El cliente GUI oficial de MongoDB se llama Compass, disponible a través de suscripción o de una versión _Community_ que requiere registro. Como alternativa pueden encontrarse distintos clientes no oficiales, desarrollados tanto por empresas como particulares en régimen de código abierto. Uno de los clientes más populares es Robomongo, que fue adquirido por una empresa llamada 3T Software Labs, y le ha cambiado el nombre a la herramienta por el de Robo 3T. Su versión gratuita portable para Windows no requiere instalación, basta con descomprimir el fichero descargado en un directorio y lanzar el ejecutable.

En lo que sigue se entenderá que los ejemplos se ejecutan directamente sobre el cliente de línea de comandos ```mongo.exe``` contra una base de datos local. En la práctica se puede utilizar cualquier cliente contra cualquier servidor, ya sea local o proporcionado por un tercero. La propia MongoDB proporciona instancias en la nube a través de su servicio Atlas.

## 6.2. Comandos

MongoDB ofrece una extensa lista de comandos organizados en los siguientes grupos:

- Conexiones

- Bases de Datos

- Gestión de Usuarios

- Gestión de Roles

- Colecciones

- Cursores

- Construcción de Objetos

- Actualización Masiva de Datos

- Planes de Ejecución de Consultas

- Réplica

- Balanceado de Carga

- Monitorización

- Invocación de Comandos Nativos

La lista de comandos es demasiado extensa como para examinarlos todos, por lo que es imprescindible revisar la documentación oficial que constituye la referencia más actualizada que se puede encontrar al respecto.

Como es habitual, los comandos que más se usan son los de acceso de datos, es decir, los de lectura y escritura sobre colecciones. En cualquier caso, la secuencia de creación de una base de datos y una colección es tan sencilla como la ejecución de los siguientes comandos:

```text
> use tienda
switched to db tienda

> db.createCollection('clientes')
{ "ok" : 1 }
```

La sintaxis del _shell_ por defecto es JavaScript y resulta sencilla de entender. En las dos líneas anteriores lo único que resulta conveniente aclarar es que el objeto global ```db``` del segundo comando referencia automáticamente a la base de datos seleccionada en el primer comando. Si la base de datos no existe se instancia en el momento que se crea una colección.

## 6.3. CRUD

Los documentos (objetos JSON) se insertan y recuperan en la colecciones siguiendo la sintaxis de JSON:

```text
> db.clientes.insertOne({nombre: 'Alberto', edad: 29})
{
"acknowledged" : true,
"insertedId" : ObjectId("5b55eda9251e77b7ec9bb363")
}

> db.clientes.find()
{ "_id" : ObjectId("5b55eda9251e77b7ec9bb363"), "nombre" : "Alberto", "edad" : 29 }

> db.clientes.find().pretty()
{
  "_id" : ObjectId("5b55eda9251e77b7ec9bb363"),
  "nombre" : "Alberto",
  "edad" : 29
}

> db.clientes.findOne({edad: 29})
{
  "_id" : ObjectId("5b55eda9251e77b7ec9bb363"),
  "nombre" : "Alberto",
  "edad" : 29
}
```

Como se observa, MongoDB añade automáticamente a los documentos el atributo ```_id```  con un UUID autogenerado a modo de clave primaria. Opcionalmente se puede suministrar este valor desde el cliente, pero en la mayoría de casos esta opción no se utiliza.

Donde MongoDB se sale un poco del tiesto con la nomenclatura es en las modificaciones. Por defecto la base de datos trabaja con documentos completos, no parciales, por lo que para modificar o eliminar un atributo es necesario reescribir el documento completo. Para evitar dicho trasiego de información, MongoDB permite indicar lo que denomina operadores de actualización (update operators) en los comandos de modificación de documentos:

```text
> db.clientes.update(
... {_id: ObjectId('5b55eda9251e77b7ec9bb363')},
... {$set: {genero: 'masculino'},
... $inc: {edad: 5}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.clientes.find().pretty()
{
  "_id" : ObjectId("5b55eda9251e77b7ec9bb363"),
  "nombre" : "Alberto",
  "edad" : 34,
  "genero" : "masculino"
}
```

Como se observa, los operadores empiezan por el símbolo del dólar, e indican la operación concreta a realizar sobre los atributos. ```$set``` añade un nuevo atributo al documento, ```$incr``` incrementa el valor de un atributo, y existen otros como ```$unset``` que elimina un atributo. El mismo concepto se utiliza para realizar filtros con condiciones lógicas, aplicando operadores con nombres bastantes descriptivos como ```$and``` y ```$or```, por citar un par de ellos. Leer la documentación para hacerse una idea de las opciones disponibles es imprescindible.

El borrado de documentos es trivial utilizando un filtro:

```text
> db.clientes.remove({_id: ObjectId('5b55eda9251e77b7ec9bb363')})
WriteResult({ "nRemoved" : 1 })

> db.clientes.count()
0
```

## 6.4. Índices

MongoDB crea automáticamente un índice por el campo ```_id``` de todas las colecciones, y permite además definir índices sobre otros campos para mejorar el rendimiento de las consultas más habituales:

```text
> db.clientes.ensureIndex({nombre:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

El valor ```1``` pasado como parámetro en la operación indica que se quiere que el orden del índice sea ascendente. Un valor ```-1``` indicaría que se quiere que sea descendente.

En la práctica MongoDB permite crear una gran variedad de índices en función de los elementos sobre los que se definan:

- Sobre un único campo

- Sobre múltiples campos

- Sobre los elementos de un campo de tipo _array_

- Sobre campos de coordenadas geoespaciales

- Sobre campos de tipo string para búsquedas de texto

- Sobre colecciones distribuidas por varios servidores

- Sobre un conjunto filtrado de documentos de una colección

- Sobre documentos que tengan un determinado campo

- Sobre documentos que expiran transcurrido un tiempo

Interesante comentar en este punto que MongoDB permite obtener el plan de ejecución de una consulta, incluyendo los índices utilizados:

```text
> db.clientes.find({nombre:'Alberto'})
... .explain('executionStats')
{
  "queryPlanner" : {
...
    "winningPlan" : {
...
      "indexName" : "nombre_1",
...
```

## 6.5. Joins

Para recuperar dos documentos relacionados, que se encuentren en dos colecciones distintas, es necesario indicar el nombre de las dos colecciones y el nombre de los dos campos que las relacionan:

```text
> db.createCollection('paises')
{ "ok" : 1 }

> db.createCollection('capitales')
{ "ok" : 1 }

> db.paises.insertOne({nombre: 'Italia'})
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5b576be4251e77b7ec9bb364")
}

> db.captitales.insertOne(
... {nombre: 'Roma',
... pais_id: ObjectId("5b576be4251e77b7ec9bb364")})
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5b576c6f251e77b7ec9bb365")
}

> db.capitales.aggregate([
... {$lookup: {
... localField: 'pais_id',
... from: 'paises',
... foreignField: '_id',
... as: 'pais'}}]).pretty()
{
  "_id" : ObjectId("5b576c6f251e77b7ec9bb365"),
  "nombre" : "Roma",
  "pais_id" : ObjectId("5b576be4251e77b7ec9bb364"),
  "pais" : [
    {
      "_id" : ObjectId("5b576be4251e77b7ec9bb364"),
      "nombre" : "Italia"
    }
  ]
}
```

Como se observa, MongoDB trata esta operación como una agregación en vez de una consulta simple, y es en el parámetro ```$lookup``` donde se indican los nombres y colecciones sobre los que realizar el _join_, así como el nombre del objeto sobre el que proyectar el resultado.

MongoDB trata todos los _joins_ como "_left outer joins_", por lo que las consultas siempre retornan resultado incluso cuando no puede satisfacerse el _join_.

## 6.6. Agregaciones

Las agregaciones en MongoDB son un tipo de consultas que van más allá de la simple lectura de uno o varios documentos de una colección. De hecho, MongoDB las considera un _framework_ en si mismo, una alternativa al patrón _map-reduce_.

Las agregaciones se construyen como una secuencia de operaciones, denominada _pipeline_, que se ejecuta sobre una colección. Cada una de las operaciones, denominadas _stages_, generan un resultado o modifican el resultado de la operación anterior.

```text
> db.createCollection('nevera')
{ "ok" : 1 }

> db.nevera.insert([
... {nombre: 'ternera', tipo: 'carne', cantidad: 1},
... {nombre: 'pollo', tipo: 'carne', cantidad: 1},
... {nombre: 'queso', tipo: 'lacteos', cantidad: 4},
... {nombre: 'lomo', tipo: 'carne', cantidad: 2},
... {nombre: 'huevos', tipo: 'huevos', cantidad: 6}])
BulkWriteResult({
  "writeErrors" : [ ],
  "writeConcernErrors" : [ ],
  "nInserted" : 5,
  "nUpserted" : 0,
  "nMatched" : 0,
  "nModified" : 0,
  "nRemoved" : 0,
  "upserted" : [ ]
})

> db.nevera.aggregate([
... {
...   $group: {
...     _id: null,
...     count: {$sum: 1}
...   }
... }
... ])
{ "_id" : null, "count" : 5 }

> db.nevera.aggregate([
... {
...   $group: {
...     _id: '$tipo',
...     count: {$sum: 1}
...   }
... }
... ])
{ "_id" : "huevos", "count" : 1 }
{ "_id" : "lacteos", "count" : 1 }
{ "_id" : "carne", "count" : 3 }

> db.nevera.aggregate([
... {
...   $group: {
...     _id: '$tipo',
...     count: {$sum: '$cantidad'}
...   }
... }
... ])
{ "_id" : "huevos", "count" : 6 }
{ "_id" : "lacteos", "count" : 4 }
{ "_id" : "carne", "count" : 4 }

> db.nevera.aggregate([
... {
...   $group: {
...     _id: '$tipo',
...     count: {$sum: '$cantidad'}
...   }
... },
... {
...   $sort: {
...     _id: 1
...   }
... }
... ])
{ "_id" : "carne", "count" : 4 }
{ "_id" : "huevos", "count" : 6 }
{ "_id" : "lacteos", "count" : 4 }

> db.nevera.aggregate([
... {
...   $match: {
...     tipo: 'carne'
...   }
... },
... {
...   $group: {
...     _id: '$nombre',
...     count: {$sum: '$cantidad'}
...   }
... },
... {
...   $sort: {
...     count: -1
...   }
... }
... ])
{ "_id" : "lomo", "count" : 2 }
{ "_id" : "pollo", "count" : 1 }
{ "_id" : "ternera", "count" : 1 }
```

Como se observa, el parámetro de entrada es un _array_ de objetos (_pipeline_). Cada objeto es una operación (_stage_). Y MongoDB las ejecuta de forma secuencial, por lo que el orden es importante. Se puede empezar con una operación e ir añadiendo operaciones hasta que se obtenga el resultado deseado. El tipo de operaciones que se puede realizar es bastante extenso, por lo que es recomendable consultar la documentación de referencia para hacerse una idea de todas las opciones disponibles.

Las operaciones más comunes son ```$match``` para filtrar, ```$sort``` para ordenar, ```$group``` para agrupar, ```$count``` para contar ocurrencias, ```$skip``` para ignorar elementos, y ```$limit```  para limitar el tamaño del resultado. Otras operaciones interesantes son por ejemplo ```$unwind``` para extraer elementos de los _arrays_ embebidos en los documentos, ```$facet``` para calcular varias agregaciones en paralelo, y ```$graphLookup``` para hacer búsquedas recursivas.

## 6.7. Otros

Para terminar, comentar que MongoDB permite definir colecciones con un tamaño máximo prefijado que se caracterizan por ofrecer un rendimiento muy eficiente. Cuando el número de documentos insertados en dichas colecciones supera dicho máximo los documentos más antiguos se sobreescriben por los nuevos. Si se realiza una consulta sin especificar ningún orden se garantiza que los documentos se retornarán en el mismo orden en que se insertaron, pudiéndose indicar también que se retornen en el orden inverso. Este tipo de colecciones permiten además crear cursores sobre los últimos N elementos de los mismos, como el comando ```tail``` de UNIX, por lo que pueden utilizarse para almacenar documentos como si fueran entradas en un _log_.

Además, ofrece la posiblidad de definir un tiempo de expiración para los documentos, de forma que cuando se supere dicho tiempo el documento será automáticamente eliminado de la base de datos. Una característica muy popular en otra base de datos NoSQL como es Redis.

Y por supuesto, es una base de datos altamente escalable que posibilita implantarla utilizando distintas configuraciones de réplica y alta disponibilidad, incluyendo la posibilidad de distribuir colecciones sobre diversos servidores.

En definitiva, el servidor de base de datos orientado a documentos más popular hoy en día por el tiempo que lleva en el mercado y las posibilidades que ofrece.
