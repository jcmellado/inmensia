# 17. JSTL (4) - SQL

_17-05-2012_ _Juan Mellado_

La librería ```SQL``` permite abrir conexiones a una base de datos y ejecutar sentencias. Es decir, permite incluir dentro de un JSP el acceso a base de datos que normalmente se realiza en la capa de persistencia de una aplicación _web_ tradicional. Evidentemente, esta forma de acceso se considera una buena práctica, y no está recomendada.

## 17.1. Configuración

Para usar la librería es necesario hacer una configuración inicial, aunque se puede omitir y hacerla en tiempo de ejecución utilizando las acciones.

Para realizar el acceso a base de datos se utiliza un _DataSource_, que puede estar disponible a través de JNDI en el servidor, o definido de forma local en la aplicación. Como ejemplo se mostrará el segundo caso, con una conexión a una base de datos Oracle XE.

El _DataSource_ se debe declarar en el fichero ```web.xml```, añadiendo la variable de contexto ```javax.servlet.jsp.jstl.sql.dataSource``` con el detalle de la conexión (url, _driver_, usuario, _password_):

```xml
<context-param>
  <param-name>javax.servlet.jsp.jstl.sql.dataSource</param-name>
  <param-value>
    jdbc:oracle:thin:@localhost:1521:XE,oracle.jdbc.driver.OracleDriver,***,***
  </param-value>
</context-param>
```

Además, se puede definir la variable ```javax.servlet.jsp.jstl.sql.maxRows``` que limita el número de filas devueltas por las consultas.

## 17.2. Dependencias

El _driver_ indicado en el _DataSource_ debe estar disponible para la aplicación, por lo que debe añadirse la dependencia correspondiente en el fichero ```pom.xml``` de Maven:

```xml
<dependency>
  <groupId>com.oracle.jdbc</groupId>
  <artifactId>ojdbc14</artifactId>
  <version>10.2.0.3.0</version>
</dependency>
```

Este _driver_ no se encuentra distribuido en los repositorios oficiales de Maven, pero es fácil encontrarlo en otros.

## 17.3. Acciones

Todas las acciones permiten especificar el _DataSource_ al que deben atacar. Si no se especifica ninguno se ataca al configurado por defecto.

- ```sql:query```: Ejecuta una consulta y almacena el resultado en una variable de tipo ```javax.servlet.jsp.jstl.sql.Result```. Opcionalmente se puede especificar el número máximo de filas que puede retornar la consulta, y la primera fila que debe retornar, lo que resulta útil para mostrar de forma paginada los registros resultantes.

    ```xml
    <sql:query var="empresas">
      SELECT *
        FROM empresa
       ORDER BY nombre
    </sql:query>
    ```

    Una vez obtenida una colección de filas se puede iterar sobre ellas utilizando la acción ```c:forEach``` para procesarlas:

    ```xml
    <table>
      <c:forEach var="empresa" items="${empresas.rows}">
        <tr>
          <td><c:out value="${empresa.id_Empresa}"/></td>
          <td><c:out value="${empresa.nombre}"/></td>
        </tr>
      </c:forEach>
    </table>
    ```

- ```sql:update```: Ejecuta una sentencia SQL de actualización datos, o DDL, y almacena el número de filas afectadas en la variable indicada.

    ```xml
    <sql:update var="modificadas">
      UPDATE empresa
         SET nombre = 'Hard Ink'
       WHERE id_empresa = 5
    </sql:update>
    ```

- ```sql:param```: Establece el valor de un parámetro de una sentencia.

    ```xml
    <sql:query var="empresas">
      SELECT *
        FROM empresa
       WHERE id_empresa = ?
      <sql:param>5</sql:param>
    </sql:query>
    ```

- ```sql:dateParam```: Establece el valor de un parámetro de tipo fecha de una sentencia. Permite especificar un parámetro que indica el tipo concreto de fecha a establecer para satisfacer las necesidades concretas de algunos _drivers_. Admite los valores ```timestamp```, ```time``` y ```date```.

- ```sql:transaction```: Establece una transacción que se aplica al cuerpo de la acción.

    ```xml
    <sql:transaction>
    ...
    </sql:transaction>
    ```

- ```sql:setDataSource```: Almacena en la variable indicada un _DataSource_. Si no especifica una variable de destino, se almacenará en la variable ```javax.servlet.jsp.jstl.sql.dataSource```.

    ```xml
    <sql:setDataSource
      url="jdbc:oracle:thin:@localhost:1521:XE"
      driver="oracle.jdbc.driver.OracleDriver"
      user="***"
      password="***"
    />
    ```
