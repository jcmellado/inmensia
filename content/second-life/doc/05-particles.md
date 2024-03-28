# 5. Second Life: Sistemas de Partículas

_10-10-2007_ _Juan Mellado_

Los sistemas de partículas en Second Life son bastante fáciles de crear, ya que se construyen con una única llamada a una función del API. La función en cuestión se llama ```llParticleSystem```, y admite como parámetros una lista de reglas. Dicha lista de reglas incluye aspectos tales como la forma en que tienen que generarse las partículas desde el punto de emisión (explosión, cono, etc), el tamaño del punto de emisión, la vida de las partículas, el color y transparencia de las partículas en el punto de emisión, el color y transparencia de las partículas al morir, la textura a aplicar a las partículas, la velocidad de generación de las partículas, la aceleración a la que se ven sometidas las partículas, y así una serie bastante grande de parámetros, incluyendo por ejemplo la posibilidad de que las partículas emitan luz, o que se vean influidas por la fuerza del viento.

Los sistemas de partículas se crean a través de un _script_, siendo necesario asociar el _script_ a un objeto que actúe como contenedor. En todo momento sólo puede haber un sistema de partículas activo por objeto. Y una cosa curiosa de esto, es que una vez activado un sistema de partículas en un objeto ya no se detiene nunca, aunque se borre el _script_ del objeto, siendo necesario ejecutar otro _script_ con una llamada a ```llParticleSystem``` con una lista de argumentos vacía.

Desde un punto de vista más técnico, los sistemas de partículas en Second Life se construyen en la parte cliente mediante una textura en dos dimensiones que se orienta siempre hacia el usuario. Es decir, mediante la técnica de _billboarding_ habitual para estos casos. El límite de partículas renderizadas en el cliente es normalmente de 4096 por _frame_. El hecho de que se generen en el cliente quiere decir que normalmente no implican ningún tipo de "lag" (demora), al menos de red, aunque podrían llegar a saturar a los ordenadores más antiguos, o con tarjetas gráficas modestas.