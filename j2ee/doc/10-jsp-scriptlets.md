# 10. JSP (3) - Declaraciones, Expresiones y Scriptlets

_16-05-2012_ _Juan Mellado_

## 10.1. Declaraciones

Las declaraciones se utilizan para declarar variables o métodos dentro de un JSP, de igual forma que se haría dentro del código Java de un _servlet_ tradicional.

Por ejemplo, una variable de instancia se puede declarar de la siguiente forma:

```jsp
<% int i = 10; %>
```

Y un método así:

```jsp
<%! public int suma(int a, int b) { return a + b; } %>
```

Los objetos y métodos así declarados son añadidos al código del _servlet_ resultante al compilar el JSP, y son visibles por el resto de elementos de la página.

Embeber código Java dentro de un JSP está totalmente desaconsejado, ya que viola muchas de las buenas prácticas de programación. El código embebido no es reutilizable, no se puede probar de forma unitaria, y es imposible de depurar de forma aislada.

## 10.2. Objetos Implícitos

Además de los objetos declarados de forma explícita dentro del código de un JSP, existen una serie de objetos declarados de forma implícita. Estos objetos son a los que normalmente se tiene acceso dentro del código de un _servlet_ tradicional.

Lógicamente, entre estos objetos están las variables ```request``` y ```response```, que se corresponden con los objetos petición y respuesta respectivamente. Su tipo varía en función del protocolo de comunicación utilizado, pero normalmente se puede asumir que son de tipo ```HttpServletRequest``` y ```HttpServletResponse```.

Otra variable disponible es ```out```, que es el ```Writer``` sobre el objeto respuesta a través del que se va construyendo el resultado.

También se puede acceder a ```application```, ```session``` y ```config```, que representan el ```ServletContext```, la ```HttpSession``` y el ```ServletConfig``` respectivamente, con los se puede acceder a los atributos de la aplicación, la sesión, y los parámetros de configuración.

Y por último, están ```page``` (equivalente a ```this```), ```pageContext```, y ```exception```, aunque está última sólo disponible para las páginas configuradas para atender errores.

A todos estos objetos se puede acceder desde dentro de los elementos de _scripts_ de un JSP.

## 10.3. Expresiones

Una expresión es una sentencia Java que es evaluada y cuyo resultado es convertido a texto siguiendo las reglas habituales de conversión. La cadena de texto resultante es añadida directamente a la respuesta de la petición a través del ```Writer```, de igual forma que se haría por código dentro de un _servlet_ tradicional.

Las expresiones pueden servir para realizar operaciones aritméticas:

```jsp
<%= 1 + 1 %>
```

Para invocar métodos:

```jsp
<%= suma(1, 1) %>
```

O para acceder a objetos:

```jsp
Dirección IP: <%= request.getRemoteHost() %>
```

Al tratarse nuevamente de código Java embebido, su uso está desaconsejado.

## 10.4. Scriptlets

Los _scriptlets_ son sentencias Java que se ejecutan durante el procesado de una petición. Todos los _scriptlets_ presentes dentro de un JSP deben dar lugar a un bloque de código coherente en su conjunto.

Su principal objetivo es el de generar contenido de forma dinámica. Como el siguiente ejemplo, que produce un resultado distinto en función de la hora del día en que se ejecuta:

```jsp
<%@ page import="java.util.Calendar" %>
<html>
  <body>

    <% if (Calendar.getInstance().get(Calendar.AM_PM) == Calendar.AM) {%>
      Buenos días
    <% } else { %>
      Buenas tardes
    <% } %>

  </body>
</html>
```

Como se observa en el ejemplo anterior, si un _scriptlet_ necesita acceder a un tipo que no se encuentra dentro de la lista de tipos por defecto, es necesario importarlo como en una clase Java ordinaria. En cualquier caso, al tratarse nuevamente de código Java embebido, su uso está desaconsejado.

El hecho de utilizar JSP no implica que se tenga que renunciar a escribir _servlets_. La idea general es que se atienda la petición entrante en un _servlet_ tradicional, se aplique la lógica de negocio correspondiente, incluido el acceso a base de datos, y se utilice entonces un JSP para renderizar el resultado.
