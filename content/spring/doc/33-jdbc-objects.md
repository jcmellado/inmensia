# 33. Spring: JDBC (3) - Objetos

_27-04-2012_ _Juan Mellado_

Aparte de todas las opciones disponibles comentadas en el artículo anterior, Spring ofrece incluso otra posibilidad más a la hora de realizar operaciones contra una base de datos utilizando JDBC. Esta opción permite proyectar de una manera conveniente los registros de base de datos en instancias de objetos.

## 33.1. MappingSqlQuery

Heredando de esta clase se puede definir de forma sencilla como se debe mapear cada columna de un registro en su correspondiente propiedad dentro de un objeto.

Las clases que hereden de ```MappingSqlQuery``` deben definir en su constructor la sentencia SQL de la que se obtendrán los registros, así como el número y tipos de parámetros que necesitan, y además, deben de implementar el método ```mapRow``` que convierta los registros en objetos:

```java
public class EmpresaQuery extends MappingSqlQuery<Empresa> {

  public EmpresaQuery(DataSource ds) {
    super(ds, "SELECT id_empresa, nombre FROM EMPRESA WHERE id_empresa = ?");
    super.declareParameter(new SqlParameter("id_empresa", Types.INTEGER));
    compile();
  }

  protected Empresa mapRow(ResultSet result, int rowNumber) throws SQLException {
    Empresa empresa = new Empresa();
    empresa.setIdEmpresa(result.getInt("id_empresa"));
    empresa.setNombre(result.getString("nombre"));
    return empresa;
  }
}
```

Una vez definida la clase se puede reutilizar llamándola tantas veces como sea necesario.

```java
Empresa empresa = empresaQuery.findObject(1);
```

La clase ejecuta la sentencia definida, inyectándole los parámetros dados, y por cada registro retornado invoca a ```mapRow``` para que se produzca la conversión personalizada de registro a objeto.

La clase ```MappingSqlQuery``` tiene más métodos convenientes definidos, como por ejemplo, para casos en los que la consulta definida retorne más de un registro.

## 33.2. SqlUpdate

Esta clase es una forma conveniente de encapsular las actualizaciones sobre base de datos. Las clases que extiendan ```SqlUpdate``` deben definir la sentencia SQL y los parámetros a utilizar en su constructor:

```java
public class EmpresaUpdate extends SqlUpdate {

  public EmpresaUpdate(DataSource ds) {
    setDataSource(ds);
    setSql("UPDATE empresa SET nombre = ? WHERE id_empresa = ?");
    declareParameter(new SqlParameter("nombre", Types.VARCHAR));
    declareParameter(new SqlParameter("id_empresa", Types.INTEGER));
    compile();
  }
}
```

Una vez definida la clase se puede utilizar para actualizar la tabla afectada por su sentencia SQL:

```java
empresaUpdate.update("Tomato Fashion", 1);
```

Es importante notar que el orden de los parámetros que se expone es de la sentencia, algo no muy conveniente, ya que viola el principio de encapsulación. En este caso es recomendable añadir un método público auxiliar que ofrezca una mejor interface, que además podría esperar un objeto como parámetro en vez de los valores sueltos:

```java
public int update(Empresa empresa) {
  return update(empresa.getNombre(), empresa.getIdEmpresa());
}
```

## 33.3. StoredProcedure

Esta clase permite realizar una abstracción del concepto de procedimiento almacenado en base de datos.

Para este caso me limitaré a copiar el ejemplo de la documentación oficial de referencia, que ejecuta una llamada a la función ```SYSDATE``` de Oracle que devuelve la fecha actual del servidor:

```java
public class GetSysdateProcedure extends StoredProcedure {

  public GetSysdateProcedure(DataSource dataSource) {
    setDataSource(dataSource);
    setFunction(true);
    setSql("SYSDATE");
    declareParameter(new SqlOutParameter("date", Types.DATE));
    compile();
  }

  public Date execute() {
    Map<String, Object> results = execute(new HashMap<String, Object>());
    Date sysdate = (Date)results.get("date");
    return sysdate;
  }
}
```

En definitiva, todas estas clases tratan de introducir el concepto de relación entre registro en base de datos y objeto de dominio en Java como solución intermedia a la implementación habitual más estándar con ORM que se estudiará en los siguientes capítulos.
