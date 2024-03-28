# 3. osCommerce: Configuración - Minimum Values

_21-03-2007_ _Juan Mellado_

Estos parámetros establecen valores de configuración mínimos para una serie de opciones de osCommerce, como por ejemplo el número mínimo de caracteres que los visitantes deben introducir en los campos "nombre" y "apellido" del formulario de alta de clientes para considerarlos válidos.

En cada uno de los apartados mostrados a continuación se indica el nombre de cada parámetro, su valor por defecto (entre paréntesis), y una breve explicación acerca de su significado, uso y consideraciones a tener en cuenta para su modificación.

## First Name (2)

Número mínimo de caracteres que debe tener un nombre de pila introducido en el formulario de alta de clientes para considerarse válido.

## Last Name (2)

Número mínimo de caracteres que debe tener un apellido introducido en el formulario de alta de clientes para considerarse válido.

## Date of Birth (10)

Número mínimo de caracteres que debe tener la fecha de nacimiento introducida en el formulario de alta de clientes para considerarse válida. El valor por defecto de ```10``` se corresponde con la longitud del típico formato "_DD/MM/YYYY_". Aparte de comprobar la longitud de la cadena de texto introducida se valida que efectivamente se corresponde con una fecha válida.

**Nota:** Existe otro parámetro de configuración que permite indicar si se debe preguntar o no la fecha de nacimiento a los clientes.

## E-Mail Address (6)

Número mínimo de caracteres que debe tener la dirección de correo introducida en el formulario de alta de clientes para considerarse válida. Aparte de comprobarse el tamaño mínimo, osCommerce también valida que la dirección de correo introducida es correcta, y que no existe otro cliente dado de alta con la misma cuenta.

**Nota:** Una cuenta de correo se considera válida si está bien formada (```a@b.c```), y el dominio al que pertenece es de primer nivel según el listado que se encuentra en el fichero ```catalog\includes\tld.txt``` (TLD = _Top Level Domain_).

## Street Address (5)

Número mínimo de caracteres que debe tener la dirección postal introducida en el formulario de alta de clientes para considerarse válida.

## Company (2)

Número mínimo de caracteres que debe tener el nombre de la empresa introducido en el formulario de alta de clientes para considerarse válido.

**Nota:** Existe otro parámetro de configuración que permite indicar si se debe preguntar o no el nombre de la empresa a los clientes.

## Post Code (4)

Número mínimo de caracteres que debe tener el código postal introducido en el formulario de alta de clientes para considerarse válido.

## City (3)

Número mínimo de caracteres que debe tener el nombre de la ciudad introducido en el formulario de alta de clientes para considerarse válido.

## State (2)

Número mínimo de caracteres que debe tener el nombre del estado introducido en el formulario de alta de clientes para considerarse válido. El nombre del estado introducido se comprueba que realmente exista en base de datos para el país seleccionado.

**Nota:** Existe otro parámetro de configuración que permite indicar si se debe preguntar o no el nombre del estado a los clientes.

## Telephone Number (3)

Número mínimo de caracteres que debe tener el número de teléfono introducido en el formulario de alta de clientes para considerarse válido.

## Password (5)

Número mínimo de caracteres que debe tener la contraseña introducida en el formulario de alta de clientes para considerarse válida. Personalmente recomiendo cambiar el valor por defecto y subirlo a 10 caracteres.

## Credit Card Owner Name (3)

Número mínimo de caracteres que debe tener el nombre del propietario de una tarjeta de crédito para considerarse válido. Incluye tanto el nombre como el apellido.

## Credit Card Number (10)

Número mínimo de caracteres que debe tener el número de una tarjeta de crédito para considerarse válido.

## Review Text (50)

Número mínimo de palabras que debe tener un comentario acerca de un producto de la tienda para considerarse válido.

## Best Sellers (1)

Número mínimo de productos a mostrar en el bloque de "_Mejor Vendidos_".

**Nota:** Existe otro parámetro de configuración que controla el número de productos máximos a mostrar en el bloque.

## Also Purchased (1)

Número mínimo de productos a mostrar en el bloque de "_Otros Productos Adquiridos por este Cliente_".

**Nota:** Existe otro parámetro de configuración que controla el número de productos máximos a mostrar en el bloque.
