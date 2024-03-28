# 27. Spring: Transacciones (1) - Introducción

_25-04-2012_ _Juan Mellado_

Una de las características más elaboradas de Spring es el soporte que ofrece para la gestión de transacciones, gracias a su capa de abstracción de muy alto nivel que proporciona un modelo de programación muy consistente.

## 27.1. Transacciones

La mayoría de las aplicaciones que se desarrollan habitualmente no tienen grandes necesidades en cuanto a la gestión de las transacciones se refiere. El caso más habitual es una conexión a una base de datos vía JDBC que se abre y cierra de forma local dentro de un método. Si la operación falla se ejecuta un _rollback_ que deshace los cambios realizados de forma local.

Sin embargo, hay aplicaciones que tienen unos requerimientos mucho mayores. Por ejemplo, un sistema que consume mensajes de una cola JMS y realiza actualizaciones en una base de datos vía JDBC. Si durante la operación contra base de datos se produce algún error debería poder garantizarse que el mensaje JMS no se pierde. Es decir, la integridad de la transacción debe garantizarse de forma global.

Java ofrece un API estándar para la gestión de transacciones llamado JTA (Java Transaction API). Es una especificación de alto nivel que abstrae de los detalles de implementación definiendo sólo las interfaces públicas y las restricciones que deben de cumplir. Este estándar indica como se debe comunicar una aplicación con un gestor de transacciones. Diversas organizaciones y empresas proporcionan sus propias implementaciones.

De forma complementaria a JTA, existe el estándar XA (eXtended Architecture), que de forma similar al anterior, es una especificación de alto nivel. Este estándar especifica como se debe comunicar un gestor de transacciones con una fuente de recursos externa, como una base de datos por ejemplo, para comunicarle que es parte de una transacción.

De forma general, los programadores tenemos que trabajar contra JTA, y resolver mediante configuración si es necesario o no el uso de XA.

## 27.2. Dependencias

Para habilitar las capacidades de gestión de transacciones en Spring hay que añadir la dependencia correspondiente en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <spring.version>3.1.1.RELEASE</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${spring.version}</version>
  </dependency>
</dependencies>
```

## 27.3. Interfaces

Spring ofrece una serie de interfaces para dar acceso a toda la infraestructura que posibilita la gestión de transacciones. Estas interfaces definen la capa de abstracción de más alto nivel.

La interface principal es ```PlatformTransactionManager```:

```java
public interface PlatformTransactionManager {

  TransactionStatus getTransaction(TransactionDefinition definition)
    throws TransactionException;

  void commit(TransactionStatus status) throws TransactionException;

  void rollback(TransactionStatus status) throws TransactionException;
}
```

Esta interface la implementan los gestores de transacciones. El propósito de sus métodos es evidente. Obtener una transacción, terminar una transacción, y deshacer una transacción.

La interface ```TransactionDefinition``` permite especificar los detalles de la transacción:

```java
public interface TransactionDefinition {

  int getPropagationBehavior();

  int getIsolationLevel();

  int getTimeout();

  boolean isReadOnly();

  String getName();
}
```

Los detalles incluyen un identificador, un tiempo máximo de ejecución, un indicador de si es de sólo lectura, un nivel de aislamiento con respecto a otras transacciones, y un tipo de propagación que contempla aspectos tales como si debería continuarse la transacción en curso o crear una nueva.

Por último, la interface ```TransactionStatus``` permite consultar y controlar el estado de la transacción:

```java
public interface TransactionStatus extends SavepointManager {

  boolean isNewTransaction();

  boolean hasSavepoint();

  void setRollbackOnly();

  boolean isRollbackOnly();

  void flush();

  boolean isCompleted();
}
```

Estas interfaces por lo general no son nunca utilizadas directamente por las aplicaciones de uso corriente, que se limitan a utilizar métodos más adecuados ofrecidos por las implementaciones disponibles.

## 27.4. Implementaciones

Las implementaciones por defecto de la interface ```PlatformTransactionManager``` son ```JtaTransactionManager``` y ```DataSourceTransactionManager```.

```JtaTransactionManager``` es un gestor de transacciones que implementa JTA, ya sea utilizando la implementación de un servidor de aplicaciones J2EE, o utilizando una implementación local de JTA embebida dentro de la propia aplicación. Resulta apropiado para trabajar con transacciones distribuidas que acceden a distintos recursos, o para gestionar transacciones que actúan sobre los recursos controlados por un servidor de aplicaciones, como una origen de datos JDBC accedido a través de JNDI por ejemplo.

```DataSourceTransactionManager``` por su parte es más apropiado para un simple origen de datos JDBC. Esta clase es capaz de trabajar en cualquier entorno con virtualmente cualquier _driver_ JDBC estándar.

Otras implementaciones disponibles están especializadas en un tipo de recurso, como JMS o JPA, o un determinado proveedor, como Hibernate o Weblogic.
