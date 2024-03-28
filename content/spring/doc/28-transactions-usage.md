# 28. Spring: Transacciones (2) - Definición y Uso

_25-04-2012_ _Juan Mellado_

Una vez presentados los temas generales acerca de la gestión de transacciones, es hora de escribir un poco de código para verlos en acción.

## 28.1. Definición

Como el resto de entidades dentro de Spring, un gestor de transacciones es un _bean_, con un identificador y una clase, normalmente declarado dentro del fichero XML de configuración:

```xml
<bean id="transactionManager"
      class="org.springframework.transaction.jta.JtaTransactionManager" />
```

En este ejemplo se utiliza la clase ```JtaTransactionManager```, que requiere una implementación de JTA. Si se intenta ejecutar Spring con esta configuración lanzando la aplicación desde un simple ```main``` se producirá un error, ya que no se encontrará ninguna implementación de JTA.

Normalmente la implementación de JTA la suministran los servidores de aplicación (Tomcat, Weblogic, ...) donde se despliegan las aplicaciones, pero en nuestro caso hay que suministrarla por nuestra propia cuenta. Para este ejemplo se utilizará la librería Atomikos, proporcionada por las siguientes dependencias en el fichero ```pom.xml``` de Maven:

```xml
<properties>
  <atomikos.version>3.7.1</atomikos.version>
</properties>

<dependency>
  <groupId>com.atomikos</groupId>
  <artifactId>transactions-jta</artifactId>
  <version>${atomikos.version}</version>
</dependency>

<dependency>
  <groupId>javax.transaction</groupId>
  <artifactId>jta</artifactId>
  <version>1.1</version>
</dependency>
```

Una vez descargadas las librerías, se debe volver a configurar el sistema indicando la implementación concreta de JTA que se quiere utilizar:

```xml
<bean id="atomikosTransactionManager" 
      class="com.atomikos.icatch.jta.UserTransactionManager" />

<bean id="transactionManager" 
      class="org.springframework.transaction.jta.JtaTransactionManager">
   <property name="transactionManager" ref="atomikosTransactionManager" />
</bean>
```

## 28.2. Uso

Para usar el gestor de transacciones de una forma directa se puede recuperar el _bean_ a través del contexto, para así poder llamar a los métodos de las interfaces que implementa:

```java
PlatformTransactionManager transactionManager =
  context.getBean(PlatformTransactionManager.class);
   
TransactionDefinition definition = new DefaultTransactionDefinition();

TransactionStatus status = transactionManager.getTransaction(definition);

try {
   
  // Lógica de negocio

} catch(Exception exception) {
  transactionManager.rollback(status);
  throw exception;
}

transactionManager.commit(status);
```

Spring proporciona una serie de utilidades para acceder a este tipo de recursos, evitando tener que escribir código que puede acabar siendo un tanto repetitivo, como el del ejemplo anterior. No obstante, en las aplicaciones convencionales no se necesita por lo general acceder de forma directa al gestor de transacciones. Normalmente se inyecta en los servicios que requieren de él.

## 28.3. TransactionTemplate

La clase ```TransactionTemplate``` permite ejecutar directamente el código de una función, por lo general anónima, dentro de una transacción:

```java
TransactionTemplate template = new TransactionTemplate(transactionManager);

template.execute(new TransactionCallbackWithoutResult() {

  @Override
  protected void doInTransactionWithoutResult(TransactionStatus status) {

    //Lógica de negocio

  }
});
```

El ejemplo anterior no retorna ningún objeto dentro de la función, pero si se necesita retornar algún objeto como resultado de la ejecución de la lógica de negocio entonces se debe instanciar como una función _callback_ de tipo ```TransactionCallback```.

Si se necesita hacer un _rollback_ dentro de la función _callback_, en respuesta a una excepción normalmente, se debe utilizar el método ```setRollbackOnly``` del parámetro ```status```:

```java
template.execute(new TransactionCallback<Object>() {

  @Override
  public Object doInTransaction(TransactionStatus status) {
    try {

        //Lógica de negocio

    } catch(Exception exception) {
      status.setRollbackOnly();
    }

    return new Object();
  }
});
```

Por último, para evitar instanciar el objeto ```TransactionTemplate``` por código, se puede declarar en el fichero XML de configuración, junto con las propiedades del mismo que se quieran inicializar, de la forma acostumbrada:

```xml
<bean id="transactionTemplate"
      class="org.springframework.transaction.support.TransactionTemplate">
  <property name="transactionManager" ref="transactionManager"/>
  <property name="timeout" value="1000"/>
</bean>
```

Este _bean_ se puede inyectar en las clases que lo requieran, compartiendo todas la misma instancia, siempre y cuando tengan las mismas necesidades de configuración.
