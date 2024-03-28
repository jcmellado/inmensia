# 26. JSF (8) - Templating

_02-06-2012_ _Juan Mellado_

JSF proporciona un sistema de construcción de vistas basado en el uso de plantillas reutilizables a través de la librería de acciones _facelets_.

## 26.1. Plantillas

Una plantilla es un fichero XTHML válido que establece como se disponen los elementos dentro de una vista de forma genérica. Actúa como un patrón donde se definen una serie de variables y su disposición. Las páginas que utilizan plantillas, llamadas páginas cliente, establecen el valor de las variables que al renderizarse se mostrarán según la disposición predefinida.

Sirva de ejemplo la siguiente plantilla que define una vista con una disposición clásica con una cabecera y un cuerpo:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
              "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:ui="http://java.sun.com/jsf/facelets">
  <head>
    <title><ui:insert name="titulo">Título por defecto</ui:insert></title>
  </head>
  <body>
    <div class="cabecera">
      <ui:insert name="cabecera"/>
    </div>
    <div class="contenido">
      <ui:insert name="contenido"/>
    </div>
  </body>
</html>
```

Cada acción ```ui:insert``` define una variable con un nombre. En la plantilla de ejemplo hay definidas tres variables. Una para el título de la página, con un valor por defecto por si la página cliente no lo establece. Y dos para el cuerpo, formado por una cabecera y un contenido.

## 26.2. Páginas Cliente

Una página cliente es un fichero XHTML válido que hace referencia a una plantilla y establece los valores de las variables definidas en dicha plantilla.

Sirva de ejemplo la siguiente página que hace referencia a la plantilla definida en el apartado anterior:

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:ui="http://java.sun.com/jsf/facelets">

  <body>
    <ui:composition template="plantilla.xhtml">

      <ui:define name="titulo">Título</ui:define>
      <ui:define name="cabecera">Cabecera</ui:define>
      <ui:define name="content">
        <ui:include src="contenido.xhtml"/>
      </ui:define>

    </ui:composition>
  </body>
</html>
```

La acción ```ui:composition``` indica la plantilla que debe utilizarse. La acción ```ui:define``` establece el valor de una variable de la plantilla. Y la acción ```ui:include``` permite incluir el contenido de otros ficheros.

Cuando JSF procesa la acción ```ui:composition``` ignora el resto del contenido de la página, lo que evita que la página HTML resultante final tenga varios bloques de la forma ```<html><body>...</body></html>```. Si se quiere incluir todo el contenido de la página se tiene que utilizar la acción ```ui:decorate```.

Todos los elementos presentes en una vista son organizados por JSF en forma de árbol de componentes. Si se quiere insertar un componente directamente en el árbol, sin utilizar una plantilla, se puede utilizar la acción ```ui:component```. Esta acción excluye el resto de contenido de la página, si se quiere incluir todo hay utilizar la acción equivalente ```ui:fragment```.

## 26.3. Parámetros

Una forma alternativa de definir plantillas es utilizar variables con expresiones EL:

```xml
<head>
  <title><h:ouputText value="#{titulo}"/></title>
</head>
```

Y en las páginas clientes pasar el valor utilizando la acción ```ui:param```:

```xml
<ui:composition template="plantilla.xhtml">
  <ui:param name="titulo" value="Título"/>
</ui:composition>
```

## 26.4. Depuración

Para poder hacer un análisis del proceso de resolución de plantillas y variables en tiempo de ejecución se puede añadir la acción ```ui:debug```. Esta acción abre una ventana en el navegador cliente cuando se pulsa ```shift+control+d``` con información detallada, aunque admite un atributo ```hotkey``` que permite cambiar la tecla por defecto.

```xml
<ui:debug/>
```
