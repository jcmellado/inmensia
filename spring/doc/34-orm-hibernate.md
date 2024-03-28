# 34. Spring: ORM (1) - Hibernate

_27-04-2012_ _Juan Mellado_

ORM (Object Relational Mapping) es el nombre con el que se conoce la técnica de mapear registros de una base de datos relacional a entidades de un lenguaje de programación orientado a objetos como es Java. Spring ofrece la posibilidad de aplicar esta técnica utilizando todas las soluciones más reconocidas actualmente, como por ejemplo Hibernate.

Spring ofrece integración con especificaciones como JPA y JDO, que son las tecnologías que implementan esta técnica, la primera orientada a base de datos relacionales, y la segunda a cualquier tipo de recursos.

## 34.1. Dependencias

Para realizar los ejemplos de esta parte se requiere incluir la librería ORM de Spring y la de Hibernate como dependencias en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
  <hibernate.version>4.1.2.Final</hibernate.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>${hibernate.version}</version>
  </dependency>
...
```

Por lo demás, se seguirá utilizando la base de datos Oracle XE de los ejemplos anteriores, por lo que es necesario mantener las correspondientes dependencias, en especial del _driver_.

## 34.2. Hibernate

Spring recomienda atacar directamente el API nativo de Hibernate, por lo que es recomendable hacer un repaso rápido de los conceptos básicos de funcionamiento de esta librería.

Hibernate soporta varias formas de configuración, entre ellas una que consiste en utilizar un fichero llamado ```hibernate.cfg.xml``` donde se le indica, entre otros, los parámetros de conexión a base de datos:

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
  <session-factory>
    <property
      name="connection.driver_class">oracle.jdbc.driver.OracleDriver</property>
    <property
      name="connection.url">jdbc:oracle:thin:@localhost:1521:XE</property>
    <property name="connection.username">*****</property>
    <property name="connection.password">*****</property>
    <property name="connection.pool_size">1</property>
    <property name="dialect">org.hibernate.dialect.OracleDialect</property>
    <property name="show_sql">true</property>
    <property name="hbm2ddl.auto">validate</property>
    <mapping resource="Empresa.hbm.xml"/>
  </session-factory>
</hibernate-configuration>
```

El último parámetro de la configuración hace referencia al fichero ```Empresa.hbm.xml```, donde se encuentra definido el mapeo entre la tabla de empresas y su correspondiente objeto POJO:

```xml
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.empresa.entity">
  <class name="Empresa" table="EMPRESA">
    <id name="idEmpresa" column="ID_EMPRESA" type="java.lang.Long">
      <generator class="sequence">
        <param name="sequence">SEQ_EMPRESA</param>
      </generator>
    </id>
    <property name="nombre" column="NOMBRE" type="java.lang.String" />
  </class>
</hibernate-mapping>
```

No obstante, Hibernate soporta también el uso del paquete de anotaciones de persistencia estándar, por lo que un ejemplo más actual eliminaría el fichero ```Empresa.hbm.xml``` de la configuración por una referencia a la clase directamente:

```xml
<mapping class="com.empresa.entity.Empresa"/>
```

Y toda la configuración del mapeo se especificaría en la propia clase utilizando las anotaciones del paquete estándar ```javax.persistence```:

```java
@Entity
@Table(name="EMPRESA")
@SequenceGenerator(name="seq_empresa", sequenceName="SEQ_EMPRESA")
public class Empresa {

  private Long idEmpresa;

  private String nombre;

  @Id
  @Column(name="ID_EMPRESA", unique=true, nullable=false, precision=15, scale=0)
  @GeneratedValue(generator="seq_empresa", strategy=GenerationType.SEQUENCE)
  public Long getIdEmpresa() {
    return idEmpresa;
  }

  public void setIdEmpresa(Long idEmpresa) {
    this.idEmpresa = idEmpresa;
  }

  @Column(name="NOMBRE", nullable=false, length=150)
  public String getNombre() {
    return nombre;
  }

  public void setNombre(String nombre) {
    this.nombre = nombre;
  }
}
```

Con esta configuración básica ya se podría acceder a la tabla de empresas para hacer una consulta por HQL de todos los registros, que Hibernate automáticamente mapeará en entidades:

```java
Session session = sessionFactory.openSession();
session.beginTransaction();

Query query = session.createQuery("from Empresa");

List<Empresa> result = query.list();
for (Empresa empresa : result) {
  System.out.println(empresa);
}

session.getTransaction().rollback();
session.close();
```

Un último detalle es pensar que en vez de una conexión JDBC directa, en un servidor de aplicaciones lo normal será utilizar un _DataSource_.

En cualquier caso, lo importante es observar como toda la gestión de sesiones y transacciones vuelve a estar en manos del desarrollador.

## 34.3. Integración

Toda la configuración y código del apartado anterior es exclusivamente de Hibernate. ¿Donde interviene Spring entonces? Pues hay algunas pistas evidentes, por ejemplo ```sessionFactory``` es un buen candidato a ser un _bean_ de tipo _Singleton_:

```xml
<bean id="sessionFactory"
      class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
  <property name="dataSource" ref="dataSource"/>
  <property name="packagesToScan">
    <list>
      <value>com.empresa.entity</value>
    </list>
  </property>
</bean>
```

La clase ```LocalSessionFactoryBean``` pertenece al paquete de integración de Spring para Hibernate, y permite referenciar a un _DataSource_, especificar opciones de configuración, e incluso auto-escanear un paquete de clases en busca de entidades.

El siguiente paso lógico es inyectar el _bean_ en un DAO, usando anotaciones por ejemplo:

```java
@Repository
public class EmpresaDaoImpl implements EmpresaDao {

  private SessionFactory sessionFactory;

  @Autowired
  public void setSessionFactory(SessionFactory sessionFactory) {
    this.sessionFactory = sessionFactory;
  }
...
```

Sin embargo, llegado este punto, aún sería responsabilidad del desarrollador la apertura de sesión y la gestión de transacciones. Lo que evidentemente es un tarea adecuada para dejarla en manos de la infraestructura de Spring:

```xml
<bean id="transactionManager"
      class="org.springframework.orm.hibernate4.HibernateTransactionManager">
  <property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

La clase ```HibernateTransactionManager``` forma parte también del paquete de integración de Spring para Hibernate. Gracias a ella se abrirá automáticamente una sesión y se realizará la gestión de transacciones sin intervención del desarrollador:

```java
@Repository
public class EmpresaDaoImpl implements EmpresaDao {

  @Transactional
  public List getAll() {
    return sessionFactory.getCurrentSession()
      .createQuery("FROM Empresa")
      .list();
  }

  @Transactional
  public Empresa getEmpresa(Long idEmpresa) {
    return (Empresa)sessionFactory.getCurrentSession()
      .createQuery("FROM Empresa WHERE idEmpresa = :idEmpresa")
      .setLong("idEmpresa", idEmpresa)
      .uniqueResult();
    }
...
```

Spring ofrece también una clase ```HibernateTemplate```, con una filosofía de funcionamiento similar a las que se vióS para JDBC, pero su uso está desaconsejado por los propios desarrolladores de Spring, que prefieren que se realicen llamadas al API nativo de Hibernate directamente.
