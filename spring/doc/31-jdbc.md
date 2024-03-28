# 31. Spring: JDBC (1) - Introducción

_26-04-2012_ _Juan Mellado_

Spring proporciona una gran cantidad de formas de acceder a base de datos usando JDBC. Su objetivo es conseguir que los desarrolladores sólo tengan que concentrar sus esfuerzos en decidir a qué base de datos quieren conectarse, escribir sentencias SQL, y procesar el resultado, evitando así todas esas tediosas tareas de gestión de conexiones, así como de reserva y liberación de recursos.

## 31.1. DataSource

Para los ejemplos de toda esta parte, y en lo sucesivo, se utilizará la conexión contra la base de datos Oracle XE configurada en el ejemplo del artículo anterior.

## 31.2. Esquema

Como esquema se utilizarán dos sencillas tablas de empresa y empleado unidas por una _foreign key_, en donde un empleado sólo puede pertenecer a una empresa:

```sql
CREATE TABLE empresa(
  id_empresa NUMBER(15) NOT NULL PRIMARY KEY,
  nombre VARCHAR2(150) NOT NULL
);

CREATE TABLE empleado(
  id_empleado NUMBER(15) NOT NULL PRIMARY KEY,
  nombre VARCHAR2(150) NOT NULL,
  id_empresa NUMBER(15) NOT NULL REFERENCES empresa(id_empresa)
);

CREATE SEQUENCE seq_empresa;
CREATE SEQUENCE seq_empleado;
```

## 31.3. JDBC

Utilizando el API nativo de JDBC, para realizar una simple consulta que obtenga todos los registros de la tabla de empresas se debería ejecutar un código similar al siguiente:

```java
Connection connection = null;
Statement statement = null;
ResultSet result = null;

try {
  connection = DataSourceUtils.getConnection(dataSource);

  statement = connection.createStatement();
  statement.execute("SELECT id_empresa, nombre FROM empresa");

  result = statement.getResultSet();
  while(result.next()){
    System.out.println(result.getString("nombre"));
  }

} finally {
  if (result != null) {
    try { result.close(); } catch(SQLException e) { }
  }

  if (statement != null) {
    try { statement.close(); } catch(SQLException e) { }
  }

  if (connection != null) {
    try { connection.close(); } catch(SQLException e) { }
  }
}
```

Hace tiempo que no escribo código de esta forma, así que espero no haberme equivocado. Es muy fácil de leer y comprender, lo que lo hizo muy popular. Pero lo importante es fijarse en que es responsabilidad del desarrollador realizar todas las tareas: abrir una conexión, preparar una sentencia, iterar el resultado, y sobre todo, liberar de forma ordenada los recursos.

Los bloques finales de _try/catch_, aparte de ser difíciles de leer, siempre han sido una fuente de problemas, ya que resulta fácil olvidar liberar un recurso. En especial las conexiones, lo que puede hacer que se supere el número máximo que tenga configurada una base de datos y no acepte más, dejando el sistema inservible y algún DBA muy cabreado.

Con esto en mente es fácil imaginar porque _frameworks_ como Spring proporcionan alternativas en forma de _wrappers_ que evitan el código repetitivo, a la par que se integran con otras características de forma transparente.

## 31.4. Anotaciones

A la hora de trabajar contra base de datos hay dos anotaciones que nos pueden interesar. Por una parte la anotación ```@Repository```, para marcar las clases como componentes de tipo DAO, y por otra parte ```@Transactional```, para que las operaciones se realicen dentro de una única transacción:

```java
@Repository
public class ClaseDao {

  @Transactional
  public void metodoTransaccional() {
...
```

Notar que, como de costumbre, es necesario realizar la configuración adecuada para que Spring tenga en cuenta estas anotaciones.
