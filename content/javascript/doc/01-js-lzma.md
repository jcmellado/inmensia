# 1. js-lzma: Una implementación de LZMA escrita en JavaScript

_01-04-2011_ _Juan Mellado_

Como parte de un proyecto personal en el que estoy trabajando me ha surgido la necesidad de descomprimir desde JavaScript un fichero utilizando LZMA, el algoritmo de compresión utilizado por el popular 7-Zip. Al final he acabado produciendo mi propia versión y publicándola como código abierto bajo el nombre de _js-lzma_:

- [https://github.com/jcmellado/js-lzma](https://github.com/jcmellado/js-lzma)

El código es una traducción directa, prácticamente línea por línea, de la versión Java original que se encuentra disponible como parte del LZMA SDK liberado al dominio público por su autor, Igor Pavlov.

Lo cierto y verdad es que sólo he codificado la parte correspondiente al algoritmo de descompresión, que es la que me interesaba para mi proyecto. Puede que en un futuro me anime a implementar la compresión, pero a día de hoy no he tomado ninguna decisión al respecto. Dependerá de si continúa mi actual interés por las posibilidades que ofrece JavaScript para el tratamiento de grandes bloques de información binaria, uno de sus muchos caballos de batalla.

Bueno, y ahora la parte técnica. Para descomprimir hay que llamar a la función ```decompress``` de la librería, que he bautizado con el poco imaginativo nombre de ```LZMA```:

```javascript
LZMA.decompress(properties, inStream, outStream, outSize);
```

Aunque por cierto, lo de los _namespaces_ en JavaScript es una auténtica locura. Cometí el error de mirarme unos cuantos blogs aquí y allá al azar, en vez de ir directo a la especificación, y al final he acabado más liado de lo que empecé. Al final he utilizado un poco de todo, pero predominando el _Revealing Module Pattern_, que hace que el aspecto del código final sea bastante parecido al concepto de "clases" de otros lenguajes como Java, lo que era perfecto para mis propósitos.

Los parámetros de entrada de la función son:

- ```properties```: Un _stream_ de _bytes_ de entrada con las propiedades LZMA a utilizar en la descompresión. Estas propiedades están descritas en el SDK y básicamente son cinco _bytes_, el primero con unos parámetros llamados ```lc```, ```lp``` y ```pb```, y los restantes cuatro _bytes_ con el tamaño del diccionario (formato _little endian_). Los ficheros comprimidos con LZMA tienen estos datos en su interior como cabecera, así que realmente no hay que preocuparse mucho por entender que significa todo esto.

- ```inStream```: Un _stream_ de _bytes_ de entrada con los datos a descomprimir.

- ```outStream```: Un _stream_ de _bytes_ de salida con el resultado de la descompresión.

- ```outSize```: El tamaño esperado de los datos de salida.

Los _streams_ de entrada deben ser objetos JavaScript que expongan un función pública llamada ```readByte```, como en el siguiente ejemplo:

```javascript
var inStream = {
  data: [ /* Poner los datos comprimidos aquí */ ],
  offset: 0,
  readByte: function(){
    return this.data[this.offset ++];
  }
};
```

Los _streams_ de salida deben ser objetos JavaScript que expongan un función pública llamada ```writeByte```, como en el siguiente ejemplo:

```javascript
var outStream = {
  data: [ /* Los datos descomprimidos acabarán aquí */ ],
  offset: 0,
  writeByte: function(value){
    this.data[this.offset ++] = value;
  }
};
```

Debe quedar claro que en vez de un ```Array``` para contener los datos se puede utilizar un ```String```, algún tipo de ```Typed Array```, o lo que se quiera.

Por último, notar que a diferencia de la versión original, que admite un tamaño de salida de 64 bits, esta versión en JavaScript limita la salida a 32 bits. Pero no es una limitación de JavaScript verdaderamente, sino que para lo que yo uso la librería me basta y sobra con ese tamaño.
