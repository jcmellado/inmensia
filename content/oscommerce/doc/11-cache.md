# 11. osCommerce: Configuración - Cache

_24-03-2007_ _Juan Mellado_

Estos parámetros establecen si se deben activar las opciones de _cache_. Este término es un concepto informático que se utiliza para denotar un método genérico de optimización de rendimiento basado en el hecho de que normalmente los visitantes de una _web_ solicitan las mismas páginas una y otra vez. La optimización basada en _cache_ consiste en almacenar dichas páginas de forma estática, en vez de generarlas dinámicamente cada que vez que se solicitan, con el consiguiente ahorro de tiempo.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Use Cache (false)

Este parámetro indica si se tienen que activar o no las opciones de _cache_ de osCommerce. Por defecto tiene el valor ```false```, que hará que no se activen, cambiándolo a ```true``` se activarán. En lineas generales, creo que no necesitará activar esta opción a menos que el rendimiento de la _web_ sea muy pobre.

## Cache Directory (/tmp/)

Directorio del servidor _web_ en el que almacenarán físicamente las páginas "cacheadas".
