# 35. Spring: ORM (2) - JPA

_28-04-2012_ _Juan Mellado_

Spring ofrece soporte para JPA (Java Persistence API), el API estándar de Java para la persistencia. Probablemente la solución más popular hoy en día.

## 35.1. Dependencias

JPA es sólo una especificación, para poder utilizarla se necesita una implementación. Los ejemplos de este artículo utilizarán la implementación que proporciona Hibernate, por lo que es necesario añadir la correspondiente dependencia en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <hibernate.version>4.1.2.Final</hibernate.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>${hibernate.version}</version>
  </dependency>
</dependencies>
```

## 35.2. JPA

De igual forma que en el artículo anterior, es conveniente realizar una pequeña introducción a JPA para ser capaz de diferenciar mejor que trata de solucionar Spring.

De la forma habitual, se necesita especificar de alguna forma los detalles y parámetros de configuración. En el caso de JPA, el nombre del fichero estándar es ```persistence.xml```, que se ubica en la ruta ```src/main/resources/META-INF```:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.0"
    xmlns="http://java.sun.com/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
        http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">

  <persistence-unit name="persistencia"
                    transaction-type="RESOURCE_LOCAL">
    <provider>org.hibernate.ejb.HibernatePersistence</provider>
    <properties>
      <property name="hibernate.connection.driver_classs"
                value="oracle.jdbc.driver.OracleDriver"/>
      <property name="hibernate.connection.url"
                value="jdbc:oracle:thin:@localhost:1521:XE"/>
      <property name="hibernate.connection.username"
                value="*****"/>
      <property name="hibernate.connection.password"
                value="*****"/>
    </properties>
  </persistence-unit>
</persistence>
```

Este fichero establece el nombre de la unidad de persistencia, el nombre de la clase proveedora del servicio, y los parámetros para realizar una conexión JDBC directa con una base de datos. No obstante, en la práctica lo más habitual será desplegar la aplicación en un servidor de aplicaciones, y especificar un _DataSource_ utilizando la etiqueta ```non-jta-data-source``` o ```jta-data-source``` según corresponda.

Importante notar que no se hace referencia a ningún fichero de mapeo, ni se especifica el nombre de clase o paquete de entidades. Esto es debido a que se usarán automáticamente las clases marcadas con la etiqueta estándar ```@Enttiy```. Aunque este comportamiento se puede modificar indicando las clases directamente con la etiqueta ```class```, ficheros con la etiqueta ```mapping-file```, e incluso evitando la detección automática con la etiqueta ```exclude-unlisted-classes```.

Por último, para probar la configuración, se puede utilizar el siguiente bloque de código:

```java
EntityManagerFactory factory =
    Persistence.createEntityManagerFactory("persistencia");

EntityManager manager = factory.createEntityManager();

EntityTransaction transaction = manager.getTransaction();

transaction.begin();

Query query = manager.createQuery("from Empresa");

List<Empresa> result = query.getResultList();
for (Empresa empresa : result) {
  System.out.println(empresa);
}

transaction.commit();

manager.close();

factory.close();
```

De igual forma que en el artículo anterior, cuando se utilizaba el API nativo de Hibernate para JPA, sin Spring, la responsabilidad de gestionar la transacción y liberar los recursos es responsabilidad del desarrollador.

## 35.3. Integración

Spring proporciona varias formas de integrar Hibernate dentro de una aplicación, de manera que se aplique de la forma más transparente posible toda la infraestructura de servicios que ofrece.

Una forma de realizar la configuración es utilizar la clase ```LocalContainerEntityManagerFactoryBean``` haciendo referencia al _DataSource_:

```xml
<bean id="entityManagerFactory"
      class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
  <property name="dataSource" ref="dataSource"/>
</bean>
```

Con esta configuración no hace falta especificar los detalles de conexión en el fichero ```persistence.xml```, ya que se toman del propio _DataSource_.

Esto permite inyectar el _bean_ directamente en las clases DAO que lo necesiten. Pero en vez de utilizar la etiqueta ```@Autowired``` se puede utilizar la estándar equivalente ```@PersistenceUnit```, que permite indicar el nombre de la unidad de persistencia si fuera necesario:

```xml
<bean id="empresaDao" class="com.empresa.dao.EmpresaDaoImpl"/>
```

```java
public class EmpresaDaoImpl implements EmpresaDao {

  private EntityManagerFactory factory;

  @PersistenceUnit
  public void setFactory(EntityManagerFactory factory) {
    this.factory = factory;
  }
...
```

El inconveniente de esta aproximación es que aún habría que crear un ```EntityManager``` en cada DAO. Algo que se puede evitar inyectándolo directamente usando la etiqueta estándar ```@PersistenceContext```:

```java
public class EmpresaDaoImpl implements EmpresaDao {

  private EntityManager manager;

  @PersistenceContext
  public void setManager(EntityManager manager) {
    this.manager = manager;
  }
...
```

No obstante, aún quedaría el problema de la gestión de transacciones. Algo que se puede solucionar de la forma habitual con Spring, creando un _bean_, esta vez de la clase especializada ```JpaTransactionManager```:

```xml
<bean id="transactionManager"
      class="org.springframework.orm.jpa.JpaTransactionManager">
  <property name="entityManagerFactory" ref="entityManagerFactory"/>
</bean>
```

A partir de aquí, los métodos que tuvieran aplicada la etiqueta ```@Transactional``` se ejecutarían dentro de una transacción:

```java
public class EmpresaDaoImpl implements EmpresaDao {

  @Transactional
  public void doIt() {
...
```

Además de todo esto, existen extensiones como Spring Data JPA específicas para facilitar el trabajo con JPA.
