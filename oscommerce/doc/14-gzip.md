# 14. osCommerce: Configuración - GZip Compression

_24-03-2007_ _Juan Mellado_

Estos parámetros permiten activar la compresión en el envío de las páginas _webs_ desde el servidor al navegador por parte de osCommerce. La compresión se realiza a través de GZip (abreviatura de _GNU Zip_), y no debe confundirse con el popular ZIP, ya que de hecho son incompatibles.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## Enable GZip Compression (false)

Este parámetro activa el uso de la compresión. Por defecto está deshabilitada, se puede cambiar a ```true``` para habilitarla.

## Compression Level (5)

Nivel de compresión a utilizar. Es un número que abarca de ```1``` a ```9```, siendo ```1``` el nivel que menos comprime, y ```9``` el que más.
