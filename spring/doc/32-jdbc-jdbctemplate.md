# 32. Spring: JDBC (2) - JdbcTemplate

_26-04-2012_ _Juan Mellado_

La clase ```JdbcTemplate``` es la pieza central del paquete de facilidades que ofrece Spring a la hora de trabajar con JDBC. Encapsula todos los detalles de conexión y gestión de recursos, permitiendo a los desarrolladores centrarse en la ejecución de sentencias.

## 32.1. Configuración

Para crear una instancia de una clase ```JdbcTemplate``` hace falta pasarle como parámetro en el constructor una referencia al _DataSource_. Así que partiendo de esta base, hay dos formas tradicionales de configurar una aplicación.

La primera es inyectar el _DataSource_ y crear una instancia de ```JdbcTemplate```:

```java
private JdbcTemplate jdbcTemplate;
   
@Autowired
public void init(DataSource dataSource) {
  jdbcTemplate = new JdbcTemplate(dataSource);
}
```

La segunda es declarar un _bean_ de tipo ```JdbcTemplate``` e inyectarlo:

```xml
<bean name="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <constructor-arg><ref bean="dataSource"/></constructor-arg>
</bean>
```

```java
private JdbcTemplate jdbcTemplate;
   
@Autowired
public void init(JdbcTemplate jdbcTemplate) {
  this.jdbcTemplate = jdbcTemplate;
}
```

```JdbcTemplate``` es segura en entornos multihilos, y no debería representar ningún problema utilizar una única instancia.

Adicionalmente, se puede heredar de la clase ```JdbcDaoSupport```, que implementa directamente una propiedad de tipo ```JdbcTemplate```, aunque requiere de igual forma que se le inyecte un _DataSource_.

## 32.2. Operaciones

Se muestra a continuación algunos ejemplos de la forma de trabajar con ```JdbcTemplate``` para realizar las operaciones más habituales contra base de datos.

- Consulta sin parámetros que retorna un valor entero:

    ```java
    jdbcTemplate.queryForInt(
        "SELECT COUNT(1) FROM empresa");
    ```

- Consulta con parámetros que retorna un valor entero:

    ```java
    jdbcTemplate.queryForInt(
        "SELECT COUNT(1) FROM empleado WHERE id_empresa = ?",
        1L);
    ```

- Consulta con parámetros de una cadena de texto:

    ```java
    jdbcTemplate.queryForObject(
        "SELECT nombre FROM empleado WHERE id_empleado = ?",
        new Object[]{1L},
        String.class)
    ```

- Consulta con parámetros de una entidad:

    ```java
    Empresa empresa = jdbcTemplate.queryForObject(
      "SELECT id_empresa, nombre FROM empresa WHERE id_empresa = ?",
      new Object[]{1L},
      new RowMapper<Empresa>() {
        public Empresa mapRow(ResultSet result, int rowNum) throws SQLException {
          Empresa empresa = new Empresa();
          empresa.setIdEmpresa(result.getInt("id_empresa"));
          empresa.setNombre(result.getString("nombre"));
          return empresa;
        }
    });
    ```

- Consulta sin parámetros de un listado de entidades:

    ```java
    List<Empresa> empresas = jdbcTemplate.query(
      "SELECT id_empresa, nombre FROM empresa",
      new RowMapper<Empresa>() {
        public Empresa mapRow(ResultSet result, int rowNum) throws SQLException {
          Empresa empresa = new Empresa();
          empresa.setIdEmpresa(result.getInt("id_empresa"));
          empresa.setNombre(result.getString("nombre"));
          return empresa;
        }
    });
    ```

- Inserción de un registro:

    ```java
    jdbcTemplate.update(
      "INSERT INTO empresa(id_empresa, nombre) VALUES(?, ?)",
      1L, "UberShop");
    ```

- Modificación de un registro:

    ```java
    jdbcTemplate.update(
      "UPDATE empresa SET nombre = ? WHERE id_empresa = ?",
      "Easy Dinner", 1L);
    ```

- Borrado de un registro:

    ```java
    jdbcTemplate.update(
      "DELETE FROM empresa WHERE id_empresa = ?",
      1L);
    ```

- Ejecución de una sentencia arbitraria:

    ```java
    jdbcTemplate.execute("CREATE TABLE nomina(id_nomina NUMBER(15) ... )");
    ```

- Llamada a un procedimiento almacenado:

    ```java
    jdbcTemplate.update(
      "CALL contrata_empleado(?)",
      1L);
    ```

Como se observa, en ninguno de los casos hay que establecer conexiones ni gestionar recursos. No obstante, es importante tener en cuenta que aún pueden elevarse excepciones por sentencias mal escritas, parámetros incorrectos, o por que el objeto resultante de la ejecución no es el esperado según la nomenclatura del método.

## 32.3. NamedParameterJdbcTemplate

La clase ```NamedParameterJdbcTemplate``` es un _wrapper_ sobre la clase ```JdbcTemplate``` que se instancia pasándole como argumento el _DataSource_ en el constructor:

```java
namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
```

Lo que pretende es resolver los problemas de tener que trabajar con caracteres marcadores de posición (```?```) dentro de las sentencias SQL, permitiendo en su lugar utilizar nombres para los parámetros.

Los nombres de los parámetros se distinguen del resto de la sentencia antecediéndolos con un carácter de dos puntos (```:```):

```java
SqlParameterSource parameters = new MapSqlParameterSource("nombre", "Easy Dinner");

namedParameterJdbcTemplate.queryForInt(
  "SELECT id_empresa FROM empresa WHERE nombre = :nombre",
  parameters);
```

Una característica interesante es que se puede utilizar un objeto JavaBean con la clase ```BeanPropertySqlParameterSource```, que construye automáticamente un ```Map``` con los nombres de las propiedades y sus correspondientes valores:

```java
Empresa empresa = new Empresa();
empresa.setNombre("Easy Dinner");

SqlParameterSource parameters = new BeanPropertySqlParameterSource(empresa);

namedParameterJdbcTemplate.queryForInt(
  "SELECT id_empresa FROM empresa WHERE nombre = :nombre",
  parameters);
```

Otras clases similares ofrecidas por Spring permiten reducir la cantidad de código a escribir significativamente, como en las operaciones por _batch_ para las actualizaciones masivas de registros, por ejemplo:

```java
SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(empresas.toArray());

namedParameterJdbcTemplate.batchUpdate(
  "INSERT INTO empresa(id_empresa, nombre) VALUES(:idEmpresa, :nombre)",
  batch);
```

## 32.4. SimpleJdbcTemplate

La clase ```SimpleJdbcTemplate``` es otro _wrapper_ sobre ```JdbcTemplate```, aunque las diferencias entre una y otra son un poco sutiles. Básicamente está mejor adaptada para usar algunas de las características concretas de Java que se incorporaron en la versión 5, como los genéricos y los argumentos variables.

## 32.5. SimpleJdbc

Spring ofrece incluso una tercera forma de realizar las operaciones a través de las clases ```SimpleJdbcInsert``` y ```SimpleJdbcCall```. Son especializaciones que buscan reducir la configuración almacenando información previa, como el nombre de la tabla o procedimiento almacenado sobre el que aplica la operación.
