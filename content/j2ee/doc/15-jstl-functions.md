# 15. JSTL (2) - Functions

_17-05-2012_ _Juan Mellado_

La librería ```functions``` proporciona acciones de propósito general, principalmente para la manipulación de cadenas de texto. A diferencia de otras, las acciones de esta librería sólo pueden utilizarse dentro de expresiones EL.

Esta librería permite realizar dentro de los JSP las operaciones que normalmente se hacen por código en los _servlets_ utilizando los métodos de la clase ```java.lang.String```.

- ```fn:length```: Devuelve el número de elementos de una colección.

    ```xml
    fn:length(header)
    ```

- ```fn:trim```: Elimina espacios por el principio y final de una cadena dada.

    ```xml
    fn:trim("  espacioso   ")
    ```

- ```fn:escapeXml```: Codifica los caracteres especiales de una cadena de texto dada para convertirlos en un formato XML válido.

    ```xml
    fn:escapeXml("1 > 2")
    ```

- ```fn:replace```: Admite tres cadenas de texto, y reemplaza en la primera cadena todas las ocurrencias de la segunda por la tercera.

    ```xml
    fn:replace("uno,xxx,tres", "xxx", "dos")
    ```

- ```fn:toLowerCase```: Convierte las mayúsculas en minúsculas dentro de una cadena dada.

    ```xml
    fn:toLowerCase("MAYÚSCULAS")
    ```

- ```fn:toUpperCase```: Convierte las minúsculas en mayúsculas dentro de una cadena dada.

    ```xml
    fn:toUpperCase("minúsculas")
    ```

- ```fn:startsWith```: Admite dos cadenas de texto y comprueba si la primera cadena empieza con la segunda.

    ```xml
    fn:startsWith("uno,dos,tres", "uno")
    ```

- ```fn:endsWith```: Admite dos cadenas de texto, y comprueba si la primera cadena termina con el mismo texto que la segunda.

    ```xml
    fn:endsWith("uno,dos,tres", "tres")
    ```

- ```fn:contains```: Admite dos cadenas de texto, y comprueba si la primera cadena contiene a la segunda.

    ```xml
    fn:contains("uno,dos,tres", "dos")
    ```

- ```fn:containsIgnoreCase```: Admite dos cadenas de texto, y comprueba si la primera cadena contiene a la segunda sin atender a minúsculas ni mayúsculas.

    ```xml
    fn:containsIgnoreCase("uno,dos,tres", "DOS")
    ```

- ```fn:indexOf```: Admite dos cadenas de texto y devuelve la posición que ocupa la segunda en la primera.

    ```xml
    fn:indexOf("uno,dos,tres", "dos")
    ```

- ```fn:substring```: Admite una cadena de texto y dos posiciones dentro de ella, y devuelve la subcadena delimitida por las dos posiciones dadas.

    ```xml
    fn:substring("unodos", 0, 3)
    ```

- ```fn:substringAfter```: Admite dos cadenas de texto, y devuelve la subcadena de la primera cadena que se encuentra después de la segunda.

    ```xml
    fn:substringAfter("uno,dos","uno,")
    ```

- ```fn:substringBefore```: Admite dos cadenas de texto, y devuelve la subcadena de la primera cadena que se encuentra antes de la segunda.

    ```xml
    fn:substringBefore("uno,dos", ",dos")
    ```

- ```fn:split```: Admite una cadena de texto y un carácter separador, y devuelve un _array_ resultado de cortar la cadena por el separador dado.

    ```xml
    fn:split('uno,dos,tres', ',')[0]
    ```

- ```fn:join```: Admite una colección y un carácter separador, y devuelve una cadena resultado de unir todos los elementos de la colección utilizando el carácter separador.

    ```xml
    <c:set var="cadena" value="uno dos tres"/>

    <c:set var="coleccion" value="${fn:split(cadena, ' ')}"/>

    <c:out value="${fn:join(coleccion, ',')}"/>
    ```
