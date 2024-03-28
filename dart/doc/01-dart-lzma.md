# 1. dart-lzma

_22-02-2012_ _Juan Mellado_

Hoy he liberado los fuentes de dart-lzma, una versión para Dart del algoritmo de compresión LZMA. Es un nuevo proyecto que he empezado para ver en que estado está el lenguaje y las posibilidades que ofrece. Y la verdad es que me he encontrado que está todo en general en estado absolutamente _alpha_, aunque ya es posible editar, compilar y ejecutar programas.

- [https://github.com/jcmellado/dart-lzma](https://github.com/jcmellado/dart-lzma)

El uso de la librería es muy sencillo, basta con importarla y llamar a la función de descompresión:

```dart
#import("lzma.dart", prefix: "LZMA");

LZMA.decompress(inStream, outStream);
```

Aunque Dart viene con librerías que incluyen varios tipos de _streams_, he decidido aislar un poco mi librería de los posibles cambios en la implementación y definir dos interfaces propias muy sencillas:

```dart
interface InStream {
  int readByte();
  int get length();
}

interface OutStream {
  void writeByte(final int value);
  int get length();
}
```

Llama la atención que en una interface haya definido un método ```get```, pero parece que el lenguaje lo soporta.

He puesto un ejemplo completo en la página del proyecto, pero al final todo se reduce a instanciar dos objetos y llamar a una función:

```dart
final InStream inStream = new InStream(data);
final OutStream outStream = new OutStream();

LZMA.decompress(inStream, outStream);
```

La función de compresión está en desarrollo. He retrasado su desarrollo porque me ha desanimado bastante que el programa no funcione con la máquina virtual de Dart utilizando la línea de comandos. Eleva una excepción ```.\vm\object.cc:7453: error: unimplemented code``` por que hay código de la máquina virtual que aún no está desarrollado. La única forma de hacer las pruebas es realizando una compilación cruzada del código en Dart a JavaScript para poder llamarlo desde una página _web_ en vez de ejecutarlo de forma nativa.
