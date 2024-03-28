# 15. osCommerce: Configuración - Sessions

_24-03-2007_ _Juan Mellado_

Cuando un visitante navega por la _web_ de una tienda, osCommerce inicia lo que se conoce como "sesión". Una sesión guarda información de contexto del visitante que lo identifica de forma unívoca mientras navega por la _web_. Los parámetros de este apartado controlan algunos aspectos de la creación y seguimiento de estas sesiones. Evidentemente son parámetros técnicos que no deberían modificarse si no se está seguro de su significado e implicaciones.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Session Directory (/tmp)

Directorio en el que se guardan las sesiones si el control de las mismas se está realizando mediante la escritura en fichero.

## Force Cookie Use (False)

Fuerza el uso de _cookies_ para el control de sesiones siempre que estén activas en el navegador del cliente.

## Check SSL Session ID (False)

Fuerza la comprobación del identificador de sesión SSL cuando se está utilizando una conexión segura (HTTPS) cada vez que el cliente accede a una página, para comprobar que sigue siendo el mismo que se utilizó originalmente para abrir la sesión.

## Check User Agent (False)

Fuerza la comprobación de la versión del navegador cada vez que el cliente accede a una página, para comprobar que sigue siendo la misma que se utilizó originalmente para abrir la sesión.

## Check IP Address (False)

Fuerza la comprobación de la dirección IP cada vez el cliente accede a una página, para comprobar que sigue siendo la misma que se utilizó originalmente para abrir la sesión.

## Prevent Spider Sessions (False)

Evita que una serie de _robots_ puedan iniciar sesión. La lista de _robots_ conocidos por osCommerce se encuentra en el fichero de texto ```catalog/includes/spiders.txt```.

## Recreate Session (False)

Fuerza la creación de una sesión nueva cuando un visitante se registra en la _web_, o se conecta utilizando una cuenta previamente creada.
