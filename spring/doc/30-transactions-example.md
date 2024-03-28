# 30. Spring: Transacciones (4) - Ejemplo

_26-04-2012_ _Juan Mellado_

En este artículo se desarrolla un ejemplo completo, más real que los anteriores, para la configuración típica de un gestor de transacciones sobre un recurso JDBC atacando una base de datos Oracle XE.

## 30.1. Dependencias

Debido a que se quiere acceder a una base de datos a través de JDBC, se debe incluir por una parte el paquete JDBC correspondiente de Spring, y por otra parte el _driver_ de Oracle en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
  <oracle.jdbc.version>10.2.0.3.0</oracle.jdbc.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>com.oracle.jdbc</groupId>
    <artifactId>ojdbc14</artifactId>
    <version>${oracle.jdbc.version}</version>
  </dependency>
</dependencies>
```

El _driver_ de Oracle no está disponible normalmente en los repositorios públicos de Maven, pero se puede localizar fácilmente en otros.

## 30.2. DataSource

En el fichero de configuración XML se tiene que definir el recurso JDBC sobre el que se quiere aplicar el gestor de transacciones. En este caso un _bean_ de la clase ```DriverManagerDataSource```, sobre el que se definen las propiedades de configuración que permiten establecer la conexión con la base de datos:

```xml
<bean id="dataSource"
      class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
  <property name="url" value="jdbc:oracle:thin:@localhost:1521:XE"/>
  <property name="username" value="*****"/>
  <property name="password" value="*****"/>
</bean>
```

```DriverManagerDataSource``` sólo debería utilizarse para realizar pruebas, ya que no implementa un pool de conexiones.

Los detalles de configuración de la conexión no son muy importantes en este momento, aunque tampoco deberían resultar difíciles de entender. Es un recurso que se necesita introducir en este momento para poder continuar con la explicación.

## 30.3. Transaction Manager

A continuación se declara un _bean_ de la clase ```DataSourceTransactionManager```, que será el que actúe como gestor de transacciones. Es una implementación especializada preparada para gestionar el recurso JDBC establecido en su propiedad ```dataSource```:

```xml
<bean id="transactionManager"
       class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>
```

¡Y ya está! Sólo se necesitan esas sencillas líneas de configuración para que Spring vuelva a obrar su magia.

## 30.4. JNDI

Alternativamente, si se va realizar un despliegue sobre un servidor de aplicaciones J2EE con soporte para JTA, y utilizar un recurso JDBC a través de JNDI, se podrían sustituir las dos definiciones de _beans_ anteriores por las dos siguientes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
     xmlns:jee="http://www.springframework.org/schema/jee"
     xsi:schemaLocation="
     ...
     http://www.springframework.org/schema/jee
     http://www.springframework.org/schema/jee/spring-jee-3.0.xsd">

  <jee:jndi-lookup id="dataSource" jndi-name="jdbc/pruebas"/>

  <bean id="transactionManager"
        class="org.springframework.transaction.jta.JtaTransactionManager" />
</beans>
```

En este caso al gestor de transacciones no le hace falta introducirle una dependencia con el recurso, porque será gestionado por el servidor de aplicaciones junto con el resto de recursos disponibles.

Lo que es interesante notar es que no es necesario modificar el código fuente Java si se quiere cambiar la política de gestión de las transacciones, o su implementación. Bastará con cambiar la configuración.
