# 23. Spring: AOP (4) - Ejemplo

_24-04-2012_ _Juan Mellado_

Este artículo contiene todos los ficheros de un sencillo proyecto de un ejemplo completo escrito a la par que se escribían estos artículos sobre programación orientada a aspectos.

- ```applicationContext.xml```:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">

      <aop:aspectj-autoproxy/>

      <bean id="secretaria" class="com.empresa.Secretaria"/>
      <bean id="espia" class="com.empresa.Espia"/>
      <bean id="grabadora" class="com.empresa.Grabadora"/>
    </beans>
    ```

- ```Secretaria.java```:

    ```java
    package com.empresa;

    public class Secretaria {

      public String conversacion(){
        return "bla, bla, bla";
      }

      public String pellizco(){
        throw new RuntimeException("¡pillín!");
      }
    }
    ```

- ```Espia.java```:

    ```java
    package com.empresa;

    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Pointcut;

    @Aspect
    public class Espia {

      @Pointcut("execution(public * conversacion(..))")
      public void anyConversacion() {
      }
    }
    ```

- ```Grabadora.java```:

    ```java
    package com.empresa;

    import org.aspectj.lang.annotation.AfterReturning;
    import org.aspectj.lang.annotation.AfterThrowing;
    import org.aspectj.lang.annotation.Aspect;

    @Aspect
    public class Grabadora {

      @AfterReturning(pointcut="com.empresa.Espia.anyConversacion()",
                      returning="cotilleo")
      public void grabarConversacion(Object cotilleo) {
        System.out.println(cotilleo);
      }

      @AfterThrowing(pointcut="execution(public * *(..))",
                    throwing="gritito")
      public void grabarGrititos(Exception gritito) {
        System.out.println(gritito);
      }
    }
    ```

- ```Main.java```:

    ```java
    package com.empresa;

    import org.springframework.context.support.AbstractApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;

    public class App {
      public static void main(String[] args){
        try{
          AbstractApplicationContext context = new ClassPathXmlApplicationContext(
                    "applicationContext.xml");
                context.registerShutdownHook();
        
          Secretaria secretaria = context.getBean(Secretaria.class);
          secretaria.conversacion();
          secretaria.pellizco();

        } catch(Exception exception) {
        }
      }
    }
    ```

- ```pom.xml```:

    ```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

      <groupId>com.empresa</groupId>
      <artifactId>ejemplo-aop</artifactId>
      <version>0.0.1</version>
      <packaging>jar</packaging>

      <name>ejemplo-aop</name>

      <properties>
        <spring.version>3.1.1.RELEASE</spring.version>
        <aspectj-version>1.5.4</aspectj-version>
        <cglib-version>2.2.2</cglib-version>
      </properties>

      <dependencies>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
          <version>${aspectj-version}</version>
        </dependency>
        <dependency>
          <groupId>cglib</groupId>
          <artifactId>cglib</artifactId>
          <version>${cglib-version}</version>
        </dependency>
      </dependencies>
    </project>
    ```
