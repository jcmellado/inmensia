# 29. Spring: Transacciones (3) - Configuración y Anotaciones

_26-04-2012_ _Juan Mellado_

Spring permite definir como debe realizarse la gestión de las transacciones de forma declarativa, utilizando los conceptos de la programación orientada a aspectos, al tiempo que ofrece la anotación ```@Transactional``` que permite simplificar sobremanera la configuración.

## 29.1. Configuración

La configuración declarativa consiste en utilizar el fichero de configuración XML, y definir en él todos los detalles de funcionamiento del gestor de transacciones.

Este tipo de configuración es un poco más avanzada que las realizadas hasta el momento, ya que se basa en el uso de los conceptos de programación orientada a aspectos, aunque ya vistos en esta serie de artículos.

La configuración se explicará en base a un ejemplo de una interface muy sencilla que simula las operaciones más habituales que pueden realizarse sobre la cuenta de un banco:

```java
public interface CuentaService {

  void transferir();

  void comprobarSaldo();
}
```

Y su correspondiente implementación, notando que el método ```comprobarSaldo``` eleva una excepción:

```java
public class CuentaServiceImpl implements CuentaService {

  @Override
  public void transferir() {
  }

  @Override
  public void comprobarSaldo() {
    throw new RuntimeException();
  }
}
```

Para el fichero de configuración XML se supondrá que ya se encuentra configurado el gestor de transacciones Atomikos visto en un artículo anterior. Por lo que sólo se muestra a continuación la parte de configuración de la programación orientada a aspectos:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="...
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">
...
  <aop:config>
    <aop:advisor advice-ref="cuentaAdvice"
                 pointcut="execution(* com.empresa.CuentaService.*(..))"/>
  </aop:config>
 
  <tx:advice id="cuentaAdvice" transaction-manager="transactionManager">
    <tx:attributes>
      <tx:method name="comprobarSaldo" read-only="true"/>
      <tx:method name="*"/>
    </tx:attributes>
  </tx:advice>
...
</beans>
```

Llegado a este punto es tiempo de parar y analizar la configuración realizada. En primer lugar se define un _advisor_, con un _poincut_ cuya expresión regular casa con la ejecución de cualquier método de la interface ```CuentaService```. Y en segundo lugar se define el _advice_ correspondiente, que hace referencia al gestor de transacciones, e indica que el método ```comprobarSaldo``` debe ejecutarse con una transacción de sólo lectura.

Lo que quiere decir todo esto es que Spring detectará automáticamente la ejecución de cualquier método sobre la clase de cuentas, y hará que se ejecuten dentro de una transacción utilizando el gestor de transacciones indicado. Y en particular, el método ```comprobarSaldo``` lo forzará a que se ejecute sobre una transacción de sólo lectura.

El último paso de la configuración consiste es crear un _bean_ que implemente la interface de cuentas:

```xml
<bean id="cuenta" class="com.empresa.CuentaServiceImpl"/>
```

Y probar toda la configuración anterior con las siguientes líneas de código:

```java
AbstractApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

CuentaService cuenta = context.getBean(CuentaService.class);

cuenta.transferir();

cuenta.comprobarSaldo();
```

Para comprobar los detalles de la ejecución se debe activar y consultar el _log_ de la aplicación en modo _debug_.

En el _log_ puede verse como el primer método abre una transacción con los parámetros por defecto, que termina con un _commit_:

```text
Creating new transaction with name [com.empresa.CuentaServiceImpl.transferir]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
...
Initiating transaction commit
```

Y a continuación puede verse como el segundo método abre una nueva transacción, de sólo lectura (```readOnly```), que termina con un _rollback_ debido a que se eleva una excepción:

```text
Creating new transaction with name [com.empresa.CuentaServiceImpl.comprobarSaldo]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly
...
Initiating transaction rollback
```

La configuración declarativa de transacciones es un tema un tanto avanzado, por lo que es necesario revisar la documentación oficial de referencia para conocer los detalles más relevantes de la implementación que realiza Spring.

## 29.2. @Transactional

Esta anotación permite reducir la cantidad de configuración y código a escribir a la hora de tratar con transacciones. Para usarla hay que añadir la etiqueta ```tx:annotation-driven``` en el fichero de configuración, para que Spring active el procesamiento de anotaciones de este tipo:

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

El atributo ```transaction-manager``` se puede omitir si el _bean_ gestor de transacciones que se quiere utilizar se llama ```transactionManager```, ya que ese es el nombre por defecto.

Una vez activado el procesamiento de anotaciones se puede simplificar todo el código utilizando la anotación ```@Transactional``` sobre clases o método individuales:

```java
public class CuentaServiceImpl implements CuentaService {

  @Override
  @Transactional
  public void transferir() {
  }

  @Override
  @Transactional(readOnly = true)
  public void comprobarSaldo() {
    throw new RuntimeException();
  }
}
```

Como se observa en el último método de la implementación, se pueden añadir todos los parámetros que se necesiten a la anotación para un mayor control de la transacción.

Añadir la línea ```tx:annotation-driven``` y usar la anotación ```@Transactional``` es equivalente a toda la configuración del apartado anterior usando programación orientada a aspectos.
