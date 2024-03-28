# 20. JSF (2) - Facelets

_02-06-2012_ _Juan Mellado_

Facelets es el lenguaje que se utiliza para escribir las páginas en JSF, también referidas normalmente como vistas. Es similar a JSP, pero tiene algunas ventajas sobre este:

- Utiliza el estándar XHTML
- No permite embeber código Java
- Permite utilizar Expression Language (EL)
- Permite utilizar librerías de acciones
- Permite definir plantillas
- Compila y renderiza más rápido

## 20.1. Tag Libraries

Facelets permite utilizar las librerías de acciones ya existentes, como las de JSTL por ejemplo, a la par que ofrece una serie de librerías propias:

Librería|Namespace|Prefijo
--------|---------|-------
core | <http://java.sun.com/jsf/core> | f
html | <http://java.sun.com/jsf/html> | h
composite | <http://java.sun.com/jsp/jstl/composite> | composite
facelets | <http://java.sun.com/jsf/facelets> | ui

El detalle completo de cada una de las acciones para cada librería se muestra en artículos posteriores de esta serie.

## 20.2. Vistas

En la práctica, una vista escrita con Facelets no difiere mucho de un documento JSP:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html">

  <h:head>
    <title>Hola JSF</title>
  </h:head>

  <h:body>
    <h:outputText value="${1 + 1}"/>
  </h:body>

</html>
```

Un tema al que hay que prestar atención son los _namespaces_, para no confundirlos con los de JSP. Por lo demás, las vistas siguen la misma convención y tienen una apariencia similar a la de los documentos JSP.

Una forma alternativa de escribir las vistas es utilizar el atributo ```jsfc``` sobre los elementos HTML originales:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html">

  <head jsfc="h:head">
    <title>Hola JSFC</title>
  </head>

  <body jsfc="h:body">
    <span jsfc="h:outputText" value="${1 + 1}"/>
  </body>

</html>
```

Cuando JSF analiza los nodos, detecta la presencia del atributo, y utiliza su valor como tipo en vez del tipo original del nodo.

La ventaja de esta forma de nomenclatura es que el fichero original puede renderizarse por cualquier navegador y manipularse con cualquier editor, aunque no se resuelvan las acciones. No obstante, esta forma de escribir las vistas no la utiliza prácticamente nadie.
