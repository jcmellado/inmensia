# 11. JSP (4) - Expression Language

_16-05-2012_ _Juan Mellado_

Expression Language (EL) es un lenguaje que ofrece JSP para poder crear contenido de forma dinámica evitando el uso de Java embebido. Su principal función es permitir escribir expresiones dentro de una página JSP que pueden evaluar condiciones, acceder a objetos, e invocar métodos.

## 11.1. Sintaxis

Las expresiones en EL se escriben utilizando la siguiente sintaxis:

```jsp
${expresion}
```

O alternativamente de la siguiente forma:

```jsp
#{expresion}
```

Esta segunda sintaxis fue introducida a partir de la versión 2.1 de la especificación, con la incorporación de otra tecnología llamada JSF (JavaServer Faces), y denota una expresión que debe evaluarse de forma diferida.

## 11.2. Objetos Implícitos

De igual forma que existen algunos objetos definidos de forma implícita dentro de un _servlet_, para acceder a ellos desde dentro de los _scripts_ de un JSP, EL proporciona variables especializadas equivalentes, para acceder a ellas desde dentro de las expresiones.

A los atributos a nivel de página, petición, sesión y aplicación se accede con las variables ```pageScope```, ```requestScope```, ```sessionScope``` y ```applicationScope``` respectivamente.

Por ejemplo, la siguiente expresión:

```jsp
${sessionScope.cliente}
```

Es equivalente a ejecutar el siguiente código en un _servlet_ tradicional:

```jsp
request.getSession().getAttribute("cliente");
```

A través de las variables ```param``` y ```paramValues``` se puede acceder a los parámetros HTTP de una petición. La variable ```paramValues``` es similar a ```param```, pero devuelve todos los valores asociados al nombre del parámetro dado, ya que para un mismo nombre puede haber varios valores.

La mayoría de estas variables son de tipo ```Map```, por lo que opcionalmente se puede acceder a los valores utilizando la notación de corchetes:

```jsp
${param["idCliente"]}
```

Otras variables, como ```header```, ```headerValues```, ```cookie```, e ```initParam```, dan acceso a las cabeceras HTTP, _cookies_, y parámetros de inicialización del contexto, respectivamente. Otra variable es ```pageContext```, que da acceso a los objetos de la clase _servlet_ generada como resultado de la compilación del JSP.

## 11.3. Operadores

EL permite usar los siguientes operadores:

- Aritméticos: ```+```, ```-```, ```*```, ```/```, ```div```, ```%```, ```mod```

    ```jsp
    ${1 + 1}
    ```

- Lógicos: ```and```, ```&&```, ```or```, ```||```, ```not```, ```!```

    ```jsp
    ${true or false}
    ```

- Relacionales: ```==```, ```eq```, ```!=```, ```ne```, ```<```, ```lt```, ```>```, ```gt```, ```<=```, ```ge```, ```>=```, ```le```

    ```jsp
    ${1 != 2}
    ```

- Condicional: ```a ? b : c```

    ```jsp
    ${1 == 2 ? 3: 4}
    ```

- Vacío: ```empty```

    ```jsp
    ${empty param}
    ```

    Este último operador comprueba si una colección dada está vacía.
