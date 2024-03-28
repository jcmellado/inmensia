# 12. osCommerce: Configuración - E-Mail Options

_24-03-2007_ _Juan Mellado_

Los parámetros de este apartado controlan como son enviados los correos desde la tienda por parte de osCommerce. Más concretamente se refieren al programa, protocolo, y formato con el que se envían. Son parámetros técnicos que no deben modificarse si se desconoce su significado. Una mala configuración de los mismos puede hacer que la tienda sea incapaz de enviar correos correctamente, con el consiguiente perjuicio.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso, y consideraciones a tener en cuenta para su modificación.

## E-Mail Transport Method (sendmail)

Este parámetro establece el mecanismo de transporte que debe utilizar osCommerce para el envío de correos. Si el servidor _web_ donde está alojada la tienda es una máquina corriendo algún tipo de Unix entonces hay que dejar el valor por defecto ```sendmail```, pero si es una máquina Windows entonces hay que cambiarlo a ```smtp```. Como nota técnica, indicar que ```sendmail``` es el nombre de un programa servidor de correos que se encuentra prácticamente en cualquier máquina Unix, ```smtp``` por su parte es el programa equivalente en máquinas Windows.

**Nota:** Si cambia el valor de este parámetro considere también cambiar el valor del siguiente parámetro.

## E-Mail Linefeeds (LF)

Este parámetro establece el separador que debe utilizar osCommerce para las cabeceras de los correos. En una máquina Unix hay que dejar el valor por defecto ```LF```, pero en una máquina Windows hay que cambiarlo a ```CRLF```. Como nota técnica, indicar que ```LF``` es una abreviatura de "_Line Feed_" (Salto de Línea), y ```CR``` de "_Carriage Return_" (Retorno de Carro).

## Use MIME HTML When Sending Emails (false)

Este parámetro permite indicar si queremos que los correos se envíen en formato HTML o en formato de texto plano. El valor por defecto de ```false``` indica que se envíen como texto plano, y es mejor no cambiar ese valor. ya que según la documentación oficial esta opción se encuentra aún en desarrollo.

## Verify E-Mail Addresses Through DNS (false)

Con este parámetro se pretende que osCommerce pueda verificar por si solo si una cuenta de correo es verdadera o falsa. La verificación se basa en realizar una comprobación acerca del dominio al que pertenece la cuenta a través de un servidor DNS (_Domain Name Solver_). Desafortunadamente este tipo de comprobación no es cien por cien fiable, y es más, en algunos casos ni tan siquiera es posible de realizar. Lo mejor es dejar esta opción deshabilitada con el valor por defecto ```false```.

## Send E-Mails (true)

Esta opción controla de forma global si osCommerce debe enviar correos o no automáticamente. Con su valor por defecto a ```true``` se enviará un correo a los clientes cuando se registren en la _web_, o después de realizar algún pedido, cambiándolo a ```false``` esos correos dejarán de enviarse.
