Ahora has completado los hitos **fundamento, generador mínimo, primitivas de colocación general, evaluación manual, características/muebles y geometría de edificios**. El proyecto puede generar y probar por lotes mapas de Dimensionfall de 32x32 exteriores, estructuralmente válidos y visualmente reconocibles, a partir de recetas compactas, y ahora puede colocar elementos de mobiliario y edificios multinivel físicamente cerrados; aún no puede componer ciudades, carreteras como redes semánticas más allá de las rutas locales del mapa, ni ubicaciones de juego completas.

## Objetivo general

El objetivo a largo plazo es un sistema de creación de mapas en el que un agente describa un mapa a un nivel de diseño significativo, en lugar de escribir manualmente miles de entradas JSON.

```text
Map concept
    ↓
Compact recipe
    ↓
Deterministic generator
    ↓
Complete Dimensionfall map JSON
    ↓
Automated validation and tests
    ↓
Godot/editor smoke test
    ↓
Playable map
```

Por ejemplo, la receta final debería expresar conceptos como:

```text
Create a forest settlement with:
- a road entering from the west
- three small buildings around a square
- a pond to the southeast
- trees around the perimeter
- indoor and outdoor areas
```

El generador debe encargarse del trabajo mecánico:

* convertir coordenadas de arriba hacia abajo y valores lógicos z en índices de nivel y de matriz de mosaicos;
* generando las 1024 entradas por nivel poblado;
* insertar identificadores válidos para baldosas y muebles;
* generación de metadatos;
* creando áreas y conexiones;
* imponiendo límites horizontales y verticales;
* crear y validar transiciones entre niveles lógicos;
* produciendo resultados deterministas;
* rechazar diseños inválidos antes de que lleguen a Godot.

El agente sigue siendo responsable de la intención y la composición del mapa. El generador sigue siendo responsable de su corrección.

---

# Estado actual

## 1. Guía del agente: completa

`AGENTS.md` Se revisó para que funcionara bien con Hermes en lugar de depender de un comportamiento específico de Codex o TaskMaster.

Las directrices del repositorio ahora pretenden ser las siguientes:

* independiente de la herramienta;
* específico del repositorio;
* Adecuado para múltiples perfiles de Hermes;
* centrado en la estructura del proyecto, los estándares de codificación, la validación y la edición segura;
* Libre de detalles de configuración personal de Docker, worktree, Kanban y modelos.

Esto proporciona a los agentes cartográficos un conjunto estable de instrucciones de repositorio.

## 2. Saneamiento del mapa existente: completo

El proyecto ahora desinfecta los datos de mosaicos mal formados cuando se serializan los mapas.

Los diccionarios de teselas corruptos o incompletos, en particular las entradas que carecen de un ID de tesela válido, se convierten a la representación de tesela vacía establecida:

```json
{}
```

Se añadió una prueba de intuición en torno a este comportamiento.

Esto protege los mapas existentes y generados de las entradas de mosaicos mal formadas que puedan quedar.

## 3. Mejoras en el validador de mapas: completas

`Tools/map_validator.py` Ahora comprende las dimensiones fijas del mapa Dimensionfall.

Rechaza:

* `mapwidth` valores distintos de `32`;
* `mapheight` valores distintos de `32`;
* niveles poblados con menos de 1024 entradas;
* niveles poblados con más de 1024 entradas.

Continúa aceptando niveles no utilizados representados como:

```json
[]
```

El validador detectó correctamente el prototipo generado originalmente que no era válido:

```text
width: 12
height: 8
level 10 entries: 96
```

Se trata de un hito importante, ya que el validador ahora aplica de forma independiente los supuestos estructurales utilizados por el generador.

## 4. Prototipo de generador basado en recetas: completo

El trabajo fusionado añadió:

```text
Tools/map_generator.py
Tools/examples/map_recipe.json
Tools/tests/test_map_generator.py
Documentation/Modding/map_recipe_generator.md
```

El generador actualmente admite:

* ID del mapa;
* nombre para mostrar;
* descripción;
* semilla entera determinista;
* Generación de nivel de tierra compatible con versiones anteriores en el índice 10 de la matriz de niveles;
* niveles lógicos explícitos de `z: -10` a través de `z: 10`;
* Múltiples niveles poblados utilizando colocación por `z` o agrupados `levels` definiciones;
* una loseta base que cubre todo el nivel del suelo existente o un nivel agrupado explícito;
* regiones rectangulares ordenadas;
* ordenado, consciente del nivel `set`, `rectangle`, `rectangle_outline`, `line`, y `scatter` operaciones;
* rasterización de la línea de Bresenham inclusiva;
* dispersión determinista por conteo o densidad;
* definiciones de patrones celulares reutilizables a nivel raíz;
* Colocación de patrones ordenada y con nivelación en un punto de anclaje con rotación fija de un cuarto de vuelta;
* paletas de mosaicos ponderados de nivel raíz a las que hacen referencia los mosaicos base y las operaciones de colocación;
* rotaciones de baldosas fijas;
* rotaciones aleatorias deterministas;
* `null` baldosas de región que producen `{}`;
* Validación del ID de la baldosa con respecto a la base de datos de baldosas;
* rechazo estricto de campos de recetas desconocidos;
* validación de los límites de la región;
* validación del nombre del archivo de salida;
* protección contra sobrescritura;
* generación en un archivo temporal;
* validación antes de la publicación;
* reemplazo atómico del archivo de destino.

Siempre produce:

```text
mapwidth: 32
mapheight: 32
populated level size: 1024
unused levels: []
logical z range: -10 through +10
level-array index: logical z + 10
```

## 5. Pruebas del generador y del validador: completadas para la versión 1

El conjunto actual de pruebas de Python contiene 156 pruebas y las supera. Los conjuntos de pruebas GUT específicos verifican la rotación de pendientes y la conectividad precalculada a través de las rutas de geometría de ejecución de Godot.

La cobertura incluye:

* Dimensiones fijas de 32×32;
* exactamente 1024 entradas;
* generación determinista;
* superposiciones rectangulares;
* colocación de células individuales;
* rectángulos rellenos y delineados, incluidos los contornos finos;
* líneas horizontales, verticales y diagonales;
* dispersión determinista por conteo y densidad;
* Ordenación de operaciones y comportamiento de sobrescritura;
* Validación del campo de operación, tipo, coordenada, límites y argumento de dispersión;
* representación de casillas vacías;
* campos desconocidos;
* identificaciones no válidas;
* Identificadores de mosaico desconocidos;
* referencias de paleta, selección determinista ponderada y rechazo de paletas mal formadas;
* Expansión, rotación, ordenación, determinismo, compatibilidad y validación de patrones reutilizables;
* Conversión y límites de tipo lógico-z;
* Segmentación Z heredada por ubicación en todos los tipos de operaciones;
* Generación multinivel agrupada, orden de declaración, orden de sobrescritura local y orden del generador de números aleatorios;
* Rechazo de definiciones z duplicadas, ambiguas, mal formadas y anidadas;
* Salida exacta de 21 niveles y rechazo por parte del validador de recuentos de niveles incorrectos;
* Se mantuvieron recetas de colinas y depresiones de dos niveles con extremos de pendiente estructuralmente soportados;
* Conversión de rotación de pendiente orientada al editor y bordes altos de malla renderizada en Godot;
* Coincidencia de bordes altos de colisión para las cuatro rotaciones de pendiente;
* Coincidencia de los bordes altos de la fuente de navegación para las cuatro rotaciones de pendiente;
* Rutas de navegación bidireccionales generadas de forma asíncrona a través de las cuatro rotaciones de pendiente;
* metadatos no válidos;
* bases de datos de mosaicos mal formadas;
* colocación fuera de límites;
* El nombre del archivo de salida no coincide;
* protección contra sobrescritura;
* Rechazo del validador por dimensiones incorrectas;
* Rechazo del validador de niveles cortos y largos;
* aceptación de niveles vacíos no utilizados;
* aceptación de mapas generados correctamente.

Esta cobertura es suficiente para las capacidades actuales del prototipo.

## 6. Propiedad del repositorio y repositorios remotos: completo

Tu repositorio de trabajo ahora es tu bifurcación:

```text
origin:
https://github.com/snipercup/CataX.git

upstream:
https://github.com/Dimensionfall/Dimensionfall.git
```

Eso te da control sobre la hoja de ruta del generador sin depender de que cada etapa experimental sea aceptada inmediatamente por la dirección principal.

Su directorio de trabajo permanece:

```bash
~/hermes/dimensionfall/workspace/Dimensionfall-test-runner
```

## 7. Generación y evaluación de ejemplos manuales: completa

El proyecto ahora incluye:

```text
Tools/generate_map_examples.py
Tools/tests/test_generate_map_examples.py
Documentation/Modding/map_example_generation.md
```

El ejecutor genera una o más variantes de semillas deterministas a partir de recetas mantenidas, delega la validación y la publicación atómica al generador existente y puede colocar las salidas directamente en un mod. `Maps` Directorio para la inspección del editor de contenido. Valida el lote completo y rechaza las rutas de salida duplicadas antes de la publicación. Las salidas recién creadas y sus resúmenes SHA-256 se registran en un manifiesto que no es JSON, de modo que la limpieza elimina solo los archivos del ejecutor que no han sufrido cambios.

Diez pruebas específicas abarcan los identificadores de variantes, la forma completa del mapa, el determinismo, la generación y limpieza de la interfaz de línea de comandos, la sobrescritura de la propiedad, la preservación de archivos modificados, las recetas mal formadas, las salidas duplicadas y la seguridad ante fallos de publicación.

---

# Lo que el generador aún no puede hacer

El lenguaje de recetas actual es intencionadamente minimalista. Todavía no puede expresar la mayoría de los mapas significativos.

Actualmente no es compatible con:

* composición de patrones anidados, basados ​​en formas o de varios niveles;
* círculos o regiones irregulares;
* mezcla de terrenos;
* elementos o muebles;
* áreas o habitaciones;
* puertas;
* edificios;
* carreteras como objetos semánticos;
* ubicación de la conexión del borde del mapa;
* plantillas reutilizables;
* pueblos o asentamientos;
* reglas del bioma;
* comprobaciones generales de transitabilidad que van más allá de la prueba de navegación por carretera de la Fase 7;
* comprobaciones de accesibilidad o conectividad;
* encuentros, criaturas, objetos o puntos de aparición;
* Vistas previas de imágenes o HTML independientes fuera de Godot; el JSON del mapa generado ya se puede abrir con la vista previa del mapa del editor de contenido existente;
* importar un mapa existente a una receta;
* regeneración conservando los cambios realizados manualmente.

Actualmente, puede crear un diseño de terreno exterior reconocible con caminos, claros delimitados y variaciones dispersas. Sin embargo, aún no puede crear un escenario jugable creíble porque carece de mobiliario, áreas semánticas, edificios o contenido de juego.

---

# Seguimiento del progreso

Este documento es la fuente de información fidedigna de la hoja de ruta. Manténgalo actualizado en los hitos clave en lugar de registrar un historial cronológico de actividades:

* usar `planned`, `next`, `in progress`, o `complete` para cada fase;
* Una fase se considerará completada únicamente después de que cumpla con sus criterios de éxito y la validación indicada;
* Cuando una fase cambia, actualice su lista de entregas, capacidades y limitaciones actuales, recuento de pruebas, tarea siguiente recomendada y el resumen de vista única en conjunto;
* Mantenga las notas de implementación temporales y la salida de comandos fuera de este documento;
* conservar contratos operativos detallados en `Documentation/Modding/map_recipe_generator.md` y realizar pruebas en lugar de duplicarlas aquí.

---

# Hoja de ruta

## Fase 1 — Seguridad estructural

**Estado: completado**

Objetivos:

* desinfectar datos de mapas con formato incorrecto;
* comprender la estructura real del mapa;
* mejorar la cobertura de los validadores;
* establecer la orientación del agente.

Entregado:

* saneamiento en `DMap`;
* prueba de saneamiento intestinal;
* validador mejorado;
* Orientado a Hermes `AGENTS.md`.

## Fase 2 — Generador determinista mínimo

**Estado: completado**

Objetivos:

* definir una receta compacta;
* generar un nivel de suelo válido;
* validar antes de escribir;
* Determinismo de las pruebas y casos de fallo.

Entregado:

* Formato de receta versión 1;
* Generador de CLI;
* Salida fija de 32×32;
* Baldosa base más superposiciones rectangulares;
* validación de baldosas;
* 16 pruebas de Python;
* documentación y ejemplo.

## Fase 3 — Primitivas de colocación general

**Estado: completado**

Entregado:

* ordenado `operations` aplicado después del legado `regions`;
* `set`, completado `rectangle`, y `rectangle_outline` colocación;
* Líneas inclusivas de una sola célula de ancho utilizando el algoritmo de Bresenham entero;
* dispersión determinista usando exactamente uno de `count` o `density`;
* límites estrictos sin recorte silencioso;
* rechazo de campo desconocido y operación desconocida;
* Lógica de ubicación compartida para regiones heredadas y operaciones rectangulares;
* un ejemplo al aire libre con un sendero, un claro delimitado y flores dispersas;
* Pruebas específicas y documentación de recetas.

### Criterio de éxito

Una receta puede generar un diseño exterior visualmente reconocible que contenga:

* un terreno base;
* un camino;
* un claro delimitado;
* alguna variación dispersa determinista.

Aún no hay muebles ni edificios. Este criterio se cumple por `Tools/examples/map_recipe.json`.

## Fase 4A: Generador de ejemplos manuales y herramienta de evaluación

**Estado: completado**

Entregado:

* una interfaz de línea de comandos de Python que reutiliza el generador existente en lugar de duplicarlo;
* Generación con un solo comando de múltiples variantes de semillas deterministas;
* Identificadores de mapa únicos y nombres de archivo coincidentes;
* Receta configurable, base de datos de mosaicos, directorio de salida, número de variantes y semilla inicial;
* protección contra sobrescritura heredada del generador;
* un manifiesto de propiedad que no es JSON con comprobaciones de identidad SHA-256 y limpieza exacta de archivos;
* salida directa a un mod `Maps` directorio para su inspección en la vista previa del editor de contenido existente;
* Pruebas específicas y una guía para probadores manuales con ejemplos de comandos.

### Criterio de éxito

Un desarrollador puede ejecutar un comando documentado, iniciar Godot e inspeccionar al menos tres variantes generadas de forma determinista en el editor de contenido sin editar manualmente las recetas ni renombrar los archivos de salida. La limpieza no modifica los mapas no relacionados.

## Fase 4B: Paletas de azulejos y patrones reutilizables

**Estado: completado**

Los identificadores de mosaico sin procesar son engorrosos y fomentan que los agentes inventen identificadores no válidos. Introduzca paletas semánticas a nivel de receta.

Ejemplo:

```json
{
  "palette": {
    "ground": [
      {
        "id": "grass_plain_01",
        "weight": 8
      },
      {
        "id": "grass_flowers_00",
        "weight": 2
      }
    ],
    "path": [
      {
        "id": "dirt_light_00",
        "weight": 3
      },
      {
        "id": "grass_dirt_00",
        "weight": 1
      }
    ]
  }
}
```

Entregado:

* paletas de mosaicos semánticos de nivel raíz;
* selección determinista ponderada;
* referencias de paleta de `base_tile`, legado `regions`y cada operación de colocación;
* Validación estricta de la paleta de colores y comprobación de la identificación de los mosaicos conocidos;
* Ejemplo de receta actualizada que utiliza paletas de colores para el suelo, la limpieza, los caminos y las flores.
* patrones de celdas reutilizables con nombre y desplazamientos con signo;
* operaciones de patrón anclado con rotación fija de un cuarto de vuelta;
* validación estricta de definición de patrones, referencias, campos y límites expandidos;
* Expansión determinista del orden de las celdas utilizando la semántica de mosaicos y paletas existente;
* Receta de ejemplo actualizada que reutiliza un patrón de grupo de flores en tres orientaciones.

### Criterio de éxito

El generador puede crear terrenos con apariencia variada sin necesidad de introducir una receta para cada celda.

Este criterio se cumple con paletas ponderadas, dispersión determinista y patrones de celdas reutilizables. La composición más rica, anidada y basada en formas, corresponde a la Fase 8, en lugar de extender esta fase fundamental indefinidamente.

## Fase 4C — Niveles verticales y cimentaciones para la colocación en 3D

**Estado: completado**

Los mapas de Dimensionfall ya contienen 21 matrices de niveles. Índice de la matriz de niveles. `10` es una elevación lógica `0`, con elevaciones lógicas válidas desde `-10` a través de `+10`:

```text
level-array index = logical z + 10
logical z = level-array index - 10
```

Los mapas existentes utilizan esta estructura para colinas, agujeros, cráteres, ubicaciones subterráneas y edificios de varios pisos. El generador necesita un contrato de coordenadas verticales compartidas antes de que se puedan agregar elementos, habitaciones, edificios y plantillas.

Objetivos:

* agregar lógica a nivel de receta `z` coordenadas manteniendo el enfoque de arriba hacia abajo `[x, y]` coordenadas;
* valor predeterminado omitido `z` a `0` para que las recetas existentes sigan siendo compatibles;
* admite múltiples matrices de niveles pobladas;
* permitir que cada operación de colocación e invocación de patrón se dirija a un nivel lógico;
* investigar una estructura de recetas por nivel para mapas multinivel sustanciales;
* definir el comportamiento de los niveles base y vacíos por nivel;
* Mapa de elevaciones lógicas `-10` a través de `+10` a índices de matriz de nivel `0` a través de `20`;
* conservar exactamente 1024 entradas en cada nivel poblado y `[]` para niveles no utilizados;
* definir el funcionamiento determinista y el ordenamiento del generador de números aleatorios en todos los niveles;
* Validar los límites horizontales, los límites verticales, las definiciones de niveles duplicados y completar la expansión del patrón;
* Investigar la semántica de pendientes, escaleras, soportes, colisiones y recorridos antes de aplicar las reglas del juego.

Entregado:

* lógico `z` validación de `-10` a través de `+10` y conversión a los índices fijos de la matriz de niveles 21;
* ubicación raíz compatible con versiones anteriores donde se omitió `z` permanece a nivel lógico `0`;
* opcional `z` en regiones heredadas y en cada operación de ubicación raíz, incluida la invocación de patrones;
* agrupado estricto `levels` definiciones con características únicas `z`, losetas base, regiones y operaciones opcionales por nivel;
* rechazo de la mezcla ambigua entre niveles agrupados y raíz `base_tile`, `regions`, o `operations`;
* Consumo de RNG en orden de declaración a través de niveles agrupados y ordenación regional previa a la operación existente dentro de cada nivel;
* preservación automática de todos los niveles vacíos como `[]` y exactamente 1024 entradas por cada nivel poblado;
* Aplicación de validación independiente de matrices de exactamente 21 niveles;
* Mantuvieron recetas de colinas y depresiones de dos niveles utilizando recetas conocidas `shape: "slope"` baldosas de transición;
* Pruebas específicas para límites verticales, compatibilidad, todos los tipos de ubicación, ordenación determinista, esquemas mal formados, forma exacta del nivel y compatibilidad con puntos finales de pendiente;
* Pruebas GUT específicas para la conversión de pendientes del editor al tiempo de ejecución y la coincidencia de bordes altos de malla, colisionador y fuente de navegación;
* Construcción robusta de colisionador de pendiente convexa que inicializa la forma antes de asignarla a un nodo de colisión;
* un explícito `navigation_mesh_baked` señal de finalización para verificación asíncrona determinista;
* Manejo seguro de las devoluciones de llamada de horneado asíncronas obsoletas cuando un fragmento ya ha comenzado a descargarse;
* Cobertura de integración de Godot enfocada que genera un mapa de navegación de un fragmento y verifica las rutas de baja a alta y de alta a baja para cada orientación de pendiente;
* Verificación manual del controlador del jugador en los mapas de colinas y depresiones de dos niveles mantenidos, que cubren las cuatro orientaciones de la pendiente en ambas direcciones sin problemas de desplazamiento o colisiones invisibles.

Formulario de compatibilidad recomendado para la colocación individual:

```json
{
  "type": "pattern",
  "pattern": "wildflower_cluster",
  "at": [16, 12],
  "z": 1,
  "rotation": 90
}
```

Para recetas más largas, es preferible agrupar el contenido por nivel lógico en lugar de repetirlo. `z` en cada operación:

```json
{
  "levels": [
    {
      "z": 0,
      "base_tile": {"palette": "ground"},
      "operations": []
    },
    {
      "z": 1,
      "operations": []
    }
  ]
}
```

Nivel raíz heredado `base_tile`, `regions`, y `operations` no pueden coexistir con grupos `levels`El generador rechaza ese diseño ambiguo en lugar de inventar un ordenamiento de fusión o sobrescritura.

Validación estructural inicial:

* `z` es un número entero de `-10` a través de `+10`;
* Los índices de nivel convertidos permanecen dentro `0` a través de `20`;
* La salida siempre contiene exactamente matrices de 21 niveles;
* Cada nivel poblado contiene exactamente 1024 entradas de casillas con orden de fila principal;
* Los niveles no utilizados permanecen `[]`;
* Cada operación y patrón expandido permanece dentro de los límites tanto horizontal como verticalmente;
* Se rechazan las definiciones duplicadas o ambiguas por nivel;
* Las mismas recetas y semillas producen resultados idénticos en todos los niveles poblados.

Se ha establecido una investigación en tiempo de ejecución:

* `Chunk.create_block_position_dictionary_new_arraymesh()` índice de matriz de niveles de mapas `i` a la altura mundial `i - 10`;
* baldosas con `shape: "slope"` Generar malla inclinada, geometría de colisión y navegación dentro de un bloque vertical;
* La rotación de pendiente de la receta y del editor de mapas selecciona el borde más alto: `0` norte, `90` este, `180` sur y `270` Oeste;
* `Chunk.get_block_rotation()` convierte esos valores de pendiente recién cargados antes de que el código de malla, colisión y navegación utilice su orientación interna, por lo que las recetas deben conservar los valores visibles para el editor en lugar de convertirlos previamente;
* Las colinas, agujeros y edificios existentes colocan baldosas de pendiente en el nivel lógico superior de los dos niveles conectados;
* Por lo tanto, los ejemplos mantenidos pueden validar los puntos finales ocupados de alto nivel y de bajo nivel sin necesidad de inventar un modelo de soporte más amplio.
* La cobertura específica de GUT confirma que las cuatro rotaciones del editor producen bordes altos coincidentes en los vértices de la malla renderizada, la geometría de colisión convexa y las caras de origen de navegación.
* El proceso de navegación asíncrona real sincroniza las cuatro orientaciones de transición en un mapa de navegación por fragmentos, con rutas consultables en ambas direcciones entre las superficies inferior y superior.
* Las pruebas manuales realizadas en la colina y la depresión generadas confirman que el controlador del jugador puede subir y bajar por cualquier pendiente sin problemas.

Las fases posteriores deben determinar reglas más amplias específicas de la estructura a medida que se introducen sus esquemas:

* si las transiciones requieren baldosas correspondientes en ambos niveles;
* cómo los pisos, las paredes, los techos, los cielorrasos y los espacios de aire intencionales ocupan niveles apilados;
* ya sea que el soporte esté determinado por la presencia de baldosas, la forma de colisión u otra regla de tiempo de ejecución;
* Determina qué comprobaciones más amplias de transición, soporte y accesibilidad deben realizarse en Python, en las pruebas unitarias de Godot o en la validación posterior del juego.

Los elementos de referencia deben incluir colinas existentes, depresiones, cráteres profundos, edificios y mapas subterráneos como `field_grass_hill_00`, `field_grass_hole_00`, `crater_small`, `two_story_house`, y `underground_lab`.

### Criterio de éxito

Genera y valida una colina de dos niveles y una depresión de dos niveles a partir de recetas. Cada mapa tiene niveles poblados del tamaño correcto y conserva los niveles no utilizados como `[]`Utiliza teselas de transición válidas, se carga en Godot y permite el recorrido en tiempo de ejecución entre elevaciones generadas.

Se cumple este criterio de éxito. Las recetas mantenidas generan mapas válidos de dos niveles, las pruebas deterministas de Godot cubren la malla, la colisión, la geometría de la fuente de navegación y las rutas bidireccionales generadas asíncronamente, y las pruebas de juego manuales confirman que el controlador del jugador real recorre las cuatro orientaciones de la pendiente en ambas direcciones sin problemas.

## Fase 5: Características y mobiliario

**Estado: completado**

Aquí es donde los mapas generados comienzan a ser jugables en lugar de meramente visuales.

Investigación del esquema establecida:

* Los muebles recién generados se almacenan como uno `feature` Diccionario incrustado directamente en una casilla de terreno no vacía, no en una matriz de objetos separada a nivel de mapa;
* La función serializada utiliza `type: "furniture"`, un mueble `id`, un cuarto de vuelta frente al editor `rotation`y opcional `itemgroups`;
* No hay ningún campo de estado o modo presente en las características del mapa creado; plano `mode` aparece únicamente en los datos de mobiliario guardados durante la ejecución;
* La altura lógica se hereda del índice de la matriz de niveles que la contiene, y `Chunk.process_level_data()` convierte índice `i` a la altura mundial `i - 10`;
* Los datos de mobiliario en tiempo de ejecución eligen el generador estático o físico de la definición de mobiliario referenciada. `moveable` propiedad;
* Los identificadores de muebles provienen de las definiciones de muebles fusionadas; el generador valida contra la selección. `Furniture.json` base de datos para este flujo de trabajo inicial de modificación del núcleo;
* El editor de mapas permite solo un diccionario de características por celda de terreno y escribe un archivo vacío. `itemgroups` matriz cuando no se ha seleccionado ningún contenido de contenedor;
* Los datos cartográficos existentes no codifican una huella general de múltiples teselas ni un contrato de ocupación de niveles adyacentes, por lo que ninguno de los dos se infiere en esta primera sección.

Concepto de receta entregada:

```json
{
  "type": "furniture",
  "x": 15,
  "y": 13,
  "z": 0,
  "id": "bench_garden",
  "rotation": 90
}
```

Entregado:

* Operaciones de mobiliario de nivel raíz con lógica opcional `z`, por defecto `0` en consonancia con las operaciones existentes;
* operaciones de mobiliario de nivel agrupado que heredan el nivel lógico contenedor y rechazan anidamientos repetidos `z`;
* Campos de operación estrictos, límites horizontales y verticales, comprobaciones de identificación de muebles conocidas y validación de rotación fija de un cuarto de vuelta;
* Se requiere terreno de apoyo en la celda objetivo y rechazo cuando esa casilla ya tiene una característica;
* Comportamiento ordenado explícito: la colocación duplicada de muebles entra en conflicto, mientras que una operación de terreno posterior reemplaza por completo la loseta y la característica anteriores;
* serialización de mapa/editor establecida con `type`, `id`, `rotation`y vacío `itemgroups` cuando la receta omite una anulación;
* funcionamiento opcional con muebles no vacíos `itemgroups`, validado contra `Itemgroups.json` y serializado como una anulación de botín de contenedor creada por el autor;
* Carga diferida de la base de datos de muebles para que las recetas que solo utilizan terreno sigan siendo compatibles y no requieran datos de muebles;
* El validador estructural independiente comprueba los campos de características del mueble, la forma del ID, la rotación y las matrices de grupos de elementos;
* un mantenido `generated_furnished_clearing` receta en lógica `z: 0` utilizando un banco de jardín, un árbol y un pino;
* Cobertura de Python para los niveles lógicos raíz y agrupados, serialización exacta, conflictos de soporte y características, orden de operaciones, validación estricta, compatibilidad y salida de ejemplos mantenidos;
* La cobertura de Godot lo confirma `Chunk.process_level_data()` conserva la característica y deriva la altura mundial esperada a partir de su nivel lógico serializado.
* Paletas de muebles ponderadas con nombre, con estricta validación de rotación basada en ID conocido, peso positivo y visible para el editor;
* encerrado `furniture_scatter` operaciones que enumeran candidatos de terreno sin características en orden de fila principal, los muestrean sin reemplazo, seleccionan mobiliario ponderado de forma determinista y fallan antes de la colocación cuando el recuento excede la elegibilidad;
* un claro amueblado ampliado con un banco de jardín explícito, una fogata sin encender y una planta en maceta, además de 24 ubicaciones de árboles, pinos, sauces o tocones quemados que tienen en cuenta el conflicto;
* editor manual y `save and test` Verificación por parte del responsable del mantenimiento de que todas las funciones de limpieza se comportan correctamente, incluidas las ubicaciones predefinidas y las funciones explícitas;
* Dos nuevas definiciones básicas de mobiliario estático Nature: `rock_field_00` y `wild_vegetation_00`, utilizando sprites de 32×32 generados por IA específicos `ai_rock_32_32.png` y `ai_vegetation_32_32.png` en el directorio principal de sprites de muebles;
* entradas de referencia de ejemplo mantenidas y un pase de dispersión de cobertura del suelo ponderado que contiene los nuevos ID de rocas y vegetación, lo que proporciona al claro árboles, rocas, vegetación y objetos decorativos/interactivos simples sin cambiar el esquema de características.

Verificación final:

* El arte temporal fue reemplazado por sprites generados por IA. `ai_rock_32_32.png` y `ai_vegetation_32_32.png`;
* El mantenedor se regeneró `generated_furnished_clearing`, lo inspeccioné en el editor con `save and test`y confirmaron que tanto los muebles de roca como los de vegetación silvestre se reproducen sin problemas;
* Por lo tanto, el claro generado cumple con el criterio de éxito de composición al aire libre de esta fase, al contar con árboles, rocas, vegetación y objetos decorativos/interactivos sencillos.

### Criterio de éxito

Genera un mapa exterior con árboles, rocas, vegetación y objetos simples interactivos o decorativos. El primer ejemplo puede permanecer en `z: 0`, pero el esquema de características y la validación deben admitir niveles lógicos explícitos desde su primera versión.

## Fase 6: Áreas, habitaciones y edificios

**Estado: completado; se han completado los cimientos del área/habitación/edificio y una sección de edificio multinivel generada físicamente y accesible.**

El generador necesita áreas semánticas antes de poder crear edificios convincentes.

### Cimientos terminados

El contrato de mapas establecido fue inspeccionado antes de extender el generador:

* `DMap` serializa las definiciones de área en el nivel del mapa. `areas` formación;
* El editor de mapas almacena la pertenencia espacial en cada mosaico de terreno como un `areas` variedad de `{ "id", "rotation" }` referencias;
* `Helper.map_manager.process_areas_in_map()` selecciona una definición por `spawn_chance` y lo aplica a los grupos de pertenencia a mosaicos conectados en cada nivel poblado;
* Los identificadores de área distintos pueden superponerse en una misma casilla, lo que coincide con el comportamiento del editor/tiempo de ejecución; el generador de recetas rechaza una referencia duplicada al mismo identificador de área;
* Las membresías no introducen una nueva representación cartográfica paralela ni reemplazan el terreno ni las características del terreno.

El generador de recetas ahora admite raíz estricta. `areas` definiciones más un `area_rectangle` operación. La operación utiliza reglas de nivel lógico existentes: las operaciones raíz por defecto son `z: 0`, puede apuntar `-10` a través de `10`y las operaciones agrupadas heredan su nivel contenedor. Requiere terreno de soporte, valida su área nombrada, utiliza solo rotaciones de cuarto de vuelta orientadas hacia el editor y serializa el objeto de pertenencia de mosaico de tiempo de ejecución/editor existente.

`Tools/examples/map_recipe_area_meadow.json` es la evidencia mantenida: un determinista `12×12` `meadow_clearing` rectángulo de membresía de `[10, 10]` a través de `[21, 21]` en `z: 0`, con una regla de tiempo de ejecución del 100 por ciento que se aplica `grass_dirt_00`La cobertura de Python verifica la ubicación de niveles, las referencias mal formadas, las membresías duplicadas, las celdas no compatibles, los límites de los ejemplos mantenidos y las comprobaciones de forma/rotación del validador.

El contrato de entidad de área ahora es estricto y tiene en cuenta el catálogo. Las recetas pueden usar los tipos de entidad de tiempo de ejecución existentes. `furniture`, `mob`, `mobgroup`, y `itemgroup`Cada registro debe tener un ID de catálogo conocido y un peso entero positivo. La selección de entidades permanece deliberadamente bajo control del tiempo de ejecución: `map_manager` agrega su peso implícito de no aparición y selecciona un nuevo resultado para cada casilla de membresía cuando se instancia el mapa. Esto mantiene los campos de área únicos entre ejecuciones, a diferencia de los deterministas. `furniture_scatter`, que preescribe las mismas posiciones de características predefinidas en el JSON generado. `Tools/examples/map_recipe_area_entity_clearing.json` es la evidencia mantenida: su 12×12 `stump_clearing` El límite no tiene características prehorneadas, luego se enrolla de forma independiente. `burned_tree_stump` características en tiempo de ejecución. La cobertura GUT enfocada demuestra que la rotación de miembros se conserva para la estructura de muebles generada y el procesamiento en tiempo de ejecución alcanza un nivel serializado. `11` (lógico `z: +1`).

La semántica de la sala de autoría utiliza un nivel de mapa separado `rooms` conjunto más azulejos exclusivos locales `rooms: ["id"]` membresía. Cada definición tiene un ID de autor estable y exactamente un tipo semántico: `enclosed`, `covered_open`, o `ruin`Estas etiquetas son deliberadamente independientes del material del suelo y del tiempo de ejecución. `areas`: un piso de madera o concreto puede abarcar varias habitaciones, un garaje techado puede ser `covered_open`y una habitación en ruinas sigue siendo una habitación a pesar de los huecos previstos en las paredes. `Tools/examples/map_recipe_room_semantics.json` es la evidencia mantenida y coloca adyacente `office` y `garage_bay` etiquetas en el mismo `concrete_00` material.

`room_connections` agrega intención de puerta de autor explícita sin inferencia de topología. Cada conexión raíz nombra una única `id`, un `[x, y]` coordinar, lógica requerida `z`y dos puntos finales distintos: una habitación conocida o `exterior`. Una opción `target_at` puede nombrar la pieza física del herraje de la puerta por separado de la pieza de autor. `at` coordina, desacoplando el anclaje del borde de la habitación de la casilla de la puerta. El objetivo debe ser una casilla de terreno existente con muebles con capacidad de puerta reconocidos por el catálogo; la evidencia actual utiliza el nativo del tiempo de ejecución `door_wood` Esta función abarca tanto las puertas que dan al exterior como las que conectan las habitaciones entre sí, manteniendo el comportamiento existente de apertura, colisión y rotación de las puertas. `Tools/examples/map_recipe_room_connections.json` abarca ambos casos. El generador y el validador independiente rechazan los puntos finales mal formados, las habitaciones desconocidas, los objetivos de puerta duplicados o los objetivos que no son puertas; `DMap` Conserva las conexiones válidas y elimina los enlaces que hacen referencia a salas eliminadas.

`room_boundaries` Proporciona la ruta de compatibilidad explícita para la evidencia física creada sin generar un perímetro del edificio. Cada registro identifica una habitación, exacta `[x, y, z]`y o bien `wall_tile` o `door_furniture`; una dirección `side` puede identificar su borde cardinal. Opcional `target_at` y `room_at` Los campos pueden nombrar explícitamente la baldosa de pared/puerta física y la baldosa de pertenencia a la habitación, por lo que un límite creado puede hacer referencia a una baldosa de pared que no sea la baldosa de la habitación en sí. Los registros de pared usan la z lógica de la habitación mientras que su existente `Wall`-La casilla de categoría se materializa un nivel por encima de ella, con soporte de suciedad implícito en z0; los registros de puertas apuntan hacia afuera desde su casilla de muebles con capacidad de puerta existente y requieren una misma ubicación. `room_connections` El punto final nombra la habitación. Una habitación no puede duplicar un objetivo, pero una misma baldosa de pared puede declararse para diferentes habitaciones. `Tools/examples/map_recipe_room_boundaries.json` demuestra una aceptación total `office` recinto con once `brick_wall_00` segmentos y uno existente `door_wood` apertura, mientras se mantiene deliberadamente parcial `covered_open` garaje y `ruin` archivos.

Un `enclosed` La habitación puede añadir `boundary_validation: "complete"`Únicamente este modo de participación requiere evidencia direccional y valida cada borde cardinal expuesto de cada casilla de pertenencia a la habitación: exactamente un límite de pared o puerta conectada debe corresponder a cada borde. Se rechazan las declaraciones faltantes, mal orientadas, duplicadas o no expuestas. `covered_open` y `ruin` No pueden optar por participar, por lo que sus aperturas y daños intencionados siguen siendo válidos sin requisitos de cierre general.

El generador, validador de mapas independiente y `DMap` Ruta de guardado/carga, validar o conservar definiciones y referencias. Una primera `buildings` El campo raíz agrupa el contenido de la habitación existente bajo una huella rectangular explícita, solo de datos, en una z lógica. Cada registro nombra un conjunto no vacío de habitaciones conocidas, requiere que cada membresía de propiedad, objetivo de límite y objetivo de conexión de puerta nombrado se encuentre dentro de sus límites, no permite superposición del mismo nivel y requiere al menos una propiedad `enclosed` habitación con validación completa de los límites. `Tools/examples/map_recipe_single_level_building.json` proporciona una tracción 4x4 `office_building` Contiene la oficina ya probada, sus once azulejos de pared y la puerta exterior. La huella no genera ni modifica la geometría, el mobiliario, los techos, las colisiones, la navegación, la iluminación, el clima ni el estado interior; solo proporciona evidencia validada por el autor para una primera agrupación a nivel de edificio.

`building_surfaces` Ahora agrega una capa semántica de sobrecarga estrecha a esa huella validada. Cada registro raíz tiene un único `id`, nombra uno existente `building`, declara `kind` como `roof` o `ceiling`y declara el nivel lógico inmediatamente superior (`building.z + 1`). Un edificio puede contener un registro de autor de cada tipo; los registros heredan el rectángulo del edificio en lugar de duplicar la geometría por celda. `Tools/examples/map_recipe_building_surfaces.json` demuestra ambas clasificaciones sobre `office_building` en `z: 1`Estos registros no generan tejas ni mallas de techo/cielo, no marcan ninguna celda como ocupada o interior, no implican soporte/colisión ni alteran la iluminación, el clima o el comportamiento en tiempo de ejecución.

`building_compositions` agrega una restricción de registro cruzado opcional sobre los metadatos existentes de edificios y superficies. Cada registro tiene un identificador único. `id`, nombra uno existente `building`y enumera uno o ambos tipos de superficie requeridos (`roof`, `ceiling`); cada tipo nombrado debe tener ya un autor correspondiente `building_surfaces` registro. Un edificio tiene como máximo una composición, mientras que los edificios sin composiciones siguen siendo válidos. El ejemplo de superficies de edificios mantenidas afirma que `office_building` Requiere clasificaciones tanto para el techo como para el cielo raso. Esto solo verifica la coherencia del diseño: no crea geometría, cobertura, ocupación, soporte, estado interior ni comportamiento en tiempo de ejecución.

Ahora un edificio puede optar por `access_validation: "complete"`Cada habitación propia debe alcanzar `exterior` a través de same-z autorizado `room_connections`Los enlaces a puertas exteriores generan accesibilidad y los enlaces entre habitaciones propias la propagan. Los enlaces a habitaciones fuera del edificio no cumplen con la validación. `office_building` opta por participar a través de su existente `office_front_door` Enlace externo. La comprobación valida únicamente el acceso semántico autorizado: no infiere la adyacencia de puertas físicas, colisiones, navegación ni el recorrido del jugador.

Un edificio puede clasificar explícitamente un subconjunto no vacío de sus habitaciones de propiedad con `interior_rooms`Cada habitación interior debe seguir siendo un espacio de autor. `enclosed` habitación con validación completa de los límites; `covered_open` y `ruin` No puede ser interior. Estos metadatos nunca se infieren del acceso, los materiales del suelo, la ubicación de paredes/puertas, los registros de techos/cielos o la topología, y no añaden ningún comportamiento de ejecución en interiores. El edificio de oficinas mantenido clasifica su área cerrada ya completa. `office` como su habitación interior.

Un edificio puede clasificar explícitamente un subconjunto no vacío de sus habitaciones de propiedad con `open_space_rooms`Cada habitación nombrada debe ser `covered_open` o `ruin`y no puede superponerse `interior_rooms`El edificio bien conservado marca su apertura cubierta. `garage_bay` como espacio abierto. Estos metadatos nunca se infieren a partir de la falta de paredes, acceso exterior, materiales del suelo, registros de techos o cielorrasos, o topografía, y no añaden geometría ni comportamiento interior/exterior en tiempo de ejecución.

Un participante voluntario `room_partition_validation: "complete"` Ahora se requiere que cada habitación de un edificio se clasifique exactamente una vez como interior o espacio abierto. El edificio de oficinas/garaje en mantenimiento muestra esta división completa. La regla solo valida las etiquetas creadas por el usuario: no requiere que todas las celdas de la superficie estén etiquetadas como habitaciones ni añade comportamiento interior/exterior en tiempo de ejecución.

Un participante voluntario `overhead_validation: "complete"` requiere que un edificio de un solo nivel validado lleve ambos elementos de autoría `roof` y `ceiling` clasificaciones en su z lógica inmediatamente superior; para un edificio de varios niveles se requiere una `floor` o `ceiling` en cada nivel declarado ocupado sobre el suelo y un `roof` al nivel de ocupación más alto declarado. Esto solo verifica la coherencia de la clasificación: no genera superficies, no afirma la cobertura física, no marca celdas como ocupadas/interiores ni altera el soporte, la colisión, la iluminación, el clima o el comportamiento en tiempo de ejecución.

`exterior_context` Ahora nombra una casilla de terreno existente sin habitaciones en la coordenada z del edificio, directamente fuera y cardinalmente adyacente a su huella. Es un punto de referencia externo solo de datos: no genera ni reserva terreno, no infiere calles/patios, no altera las áreas de ejecución, no establece una entrada ni cambia el comportamiento de ejecución. Los nombres de edificios mantenidos `[6, 8, 0]` al oeste de su área de influencia.

`exterior_access_context` Ahora asocia explícitamente ese contexto con una conexión existente de habitación a exterior en la misma z lógica. Los nombres de edificios mantenidos `office_front_door`Esto valida únicamente la asociación semántica definida por el autor: no infiere rutas físicas ni adyacencia de puertas/contexto, no crea entradas, no altera la colisión/navegación ni cambia el comportamiento en tiempo de ejecución.

`entrance` ahora los autores crean una semántica de entrada a nivel de edificio solo de datos. Cada registro tiene exactamente `connection` y `facing`: `connection` nombra una conexión existente de habitación al exterior del mismo nivel Z propiedad del edificio (el mismo contrato que `exterior_access_context.connection`), y `facing` es uno de `north`, `east`, `south`, o `west`Requiere `exterior_context` y, cuando `exterior_access_context` también está presente, `entrance.connection` debe coincidir. El `facing` La dirección debe apuntar desde `exterior_context.at` hacia la huella del edificio: al avanzar una celda en esa dirección desde la coordenada de contexto, se debe aterrizar dentro de la huella. Los autores del edificio mantenido `office_front_door` frente a `east`, que coincide con su contexto exterior al oeste de la huella. Esto valida únicamente la orientación de entrada definida: no genera una puerta, no infiere una ruta física, no requiere adyacencia de coordenadas entre la casilla de contexto y la puerta referenciada, ni altera la colisión, la navegación, la iluminación, el clima o el comportamiento en tiempo de ejecución.

`entrance_validation: "complete"` Ahora opta por un edificio en la validación completa de la orientación y el acceso a la entrada. Requiere `entrance` y luego comprueba dos restricciones de autor. Primero, la `room_boundaries` registro con `element: "door_furniture"` en la conexión de entrada `at` y `z`, nombrando una habitación propiedad del edificio, debe tener una `side` igual a lo opuesto de `entrance.facing`—la puerta se abre hacia la dirección que se aproxima. Segundo, `exterior_context.at` y la conexión de entrada `at` ambos deben quedar dentro del rango perpendicular de la huella, asegurando que la ruta de acceso corra a lo largo del lado correcto del edificio. El edificio mantenido opta por participar; su `office_northwest_door` límite en `[8, 8]` con `side: "west"` partidos `entrance.facing: "east"`y tanto el contexto Y (8) como la puerta Y (8) se encuentran dentro de la altura de la huella (7–10). Esto valida únicamente la orientación y alineación definidas por el autor: no genera ni modifica la geometría, no infiere una ruta transitable, no comprueba colisiones ni navegación, ni altera el comportamiento en tiempo de ejecución.

`entrances` ahora crea múltiples semánticas de entrada de nivel de edificio solo de datos como alternativa a la singular `entrance`Cada registro es una matriz no vacía de `{ "id", "connection", "facing" }` Las entradas siguen el mismo contrato por entrada que la forma singular. `entrance` y `entrances` son mutuamente excluyentes. Cada `id` es único dentro del edificio, y cada `connection` Puede ser referenciado por como máximo una entrada. `entrances` requiere `exterior_context`; cuando `exterior_access_context` también está presente, su `connection` debe coincidir con una de las conexiones de entrada. La entrada principal es aquella cuya `connection` partidos `exterior_access_context.connection`, o la primera entrada cuando `exterior_access_context` está ausente. Solo se comprueba la entrada principal. `exterior_context.at` para la restricción de huella orientada hacia; las entradas no principales no se comprueban con respecto a la baldosa de contexto porque una sola coordenada de contexto no puede servir para múltiples direcciones de acceso. `entrance_validation: "complete"` Valida la orientación de las puertas para cada entrada y la alineación de acceso para la entrada principal. `Tools/examples/map_recipe_multi_entrance_building.json` Muestra el edificio de oficinas en buen estado con dos entradas: `front_entrance` (puerta principal de la oficina orientada al este) y `garage_entrance` (apertura del garaje orientada al oeste), cada una con su propio límite de puerta-mueble. Esto valida únicamente la orientación y alineación de las múltiples entradas definidas por el autor: no genera puertas, no infiere rutas físicas, no comprueba colisiones ni navegación, ni altera el comportamiento en tiempo de ejecución.

`furniture_anchors` Ahora los autores nombraron metadatos de anclaje de muebles de solo datos para un edificio validado. Cada registro es una matriz no vacía de `{ "id", "at", "z", "kind" }` entradas. Cada `id` Es único dentro del edificio y sigue el patrón de nomenclatura estándar. `at` es un número entero de dos `[x, y]` Coordenadas dentro de los límites del mapa. `z` es un nivel lógico desde `-10` a través de `10` y debe coincidir con el propio edificio. `z`. `kind` es una etiqueta semántica de forma libre no vacía (por ejemplo `"door"`, `"storage"`, `"workstation"`) — no es un conjunto enumerado y no lleva comportamiento en tiempo de ejecución. Cada ancla debe hacer referencia a una baldosa dentro de la huella del edificio que tenga una característica de mobiliario existente en el autor. `[x, y, z]`; el ancla no genera muebles, no modifica el terreno, no infiere la transitabilidad, no comprueba la categoría o función de los muebles, ni altera la colisión, la navegación, la iluminación, el clima o el comportamiento en tiempo de ejecución. `Tools/examples/map_recipe_furniture_anchors.json` Muestra el edificio de oficinas mantenido con dos elementos de mobiliario principales: `office_door_anchor` y `garage_door_anchor`, cada uno apuntando a uno existente `door_wood` Función dentro de la huella. Este es un punto de referencia creado exclusivamente con datos que la futura composición de plantillas y la validación del juego pueden usar como ancla.

`building_levels` Ahora proporciona la primera base de datos de la huella de un edificio de varios niveles y metadatos de propiedad de habitaciones/muebles por piso. Un edificio permanece enraizado en la planta baja. `z: 0` y pueden declarar niveles de piso ocupados estrictamente ascendentes en valores z lógicos pares: `z: 0` suelo, `z: 1` brecha abierta intencional, `z: 2` techo/primer piso, `z: 3` brecha abierta, y así sucesivamente hasta `z: 10`Cada nivel puede asignar anclas de habitaciones y muebles de edificios agregados exactamente una vez. Las paredes siguen la regla derivada simple: la capa de pared es `floor_z + 1`No se almacena ninguna elevación de pared independiente. Un futuro `wall_height` El terreno puede soportar pisos realmente altos, pero esto se pospone intencionalmente.

`staircases` Ahora añade a esa base semántica de transición física definida por el autor. Nombres de cada escalera `lower_at` y `upper_at` coordenadas más rotaciones visibles para el editor, y la validación requiere teselas de pendiente coincidentes (cualquier base de datos de teselas) `shape: "slope"` baldosa) en la coordenada inferior en `z: 1` y coordenada superior en `z: 2` — exactamente los dos bloques de pendiente necesarios para el ascenso del jugador. Las escaleras rectas usan dos pendientes cardinalmente adyacentes con la misma rotación; las escaleras de esquina insertan un bloque de rellano plano en `z: 1` entre las pendientes de modo que la pendiente superior gira en la esquina (`upper_rotation`). Las pendientes nunca se apilan: la pendiente superior siempre está desplazada horizontalmente con respecto a la pendiente inferior. `Tools/examples/map_recipe_multi_level_building_foundation.json` Se muestra la formación en esquina con un bloque de aterrizaje; la formación en línea recta sigue cubierta por pruebas específicas.

`building_surfaces` ahora genera superficies de pisos y techos por piso para edificios de varios niveles, además de la semántica de techos/cielos de un solo nivel. Para un edificio con `building_levels`, `z` debe nombrar un nivel ocupado declarado y `kind` tal vez `floor`, `ceiling`, o `roof`; un multinivel `roof` Debe estar en el nivel de ocupación más alto declarado. El generador físico materializa la teja en la superficie del techo z definida. Si z ya corresponde a un nivel de piso ocupado, la capa de piso existente se reutiliza como techo y no se genera una segunda capa. El techo permanece como metadatos únicamente y se omite intencionalmente de la generación física. `building_supports` Añade vías de soporte estructural predefinidas desde los niveles inferiores a los superiores ocupados.

El generador, validador de mapas independiente y `DMap` Ruta de guardado/carga, validar o conservar estos contratos. Una opción de suscripción `building_geometry` El registro ahora genera pisos físicos en niveles ocupados declarados, baldosas de pared a partir de límites de habitaciones creados, baldosas de soporte a partir de rutas de soporte creadas y la baldosa de techo creada en la superficie del techo z cuando se crea una superficie de techo. Las paredes se materializan en `boundary_z + 1`. Solo las paredes cuyo límite está al nivel del suelo del edificio reciben `dirt_light_00` debajo de ellos; los muros del piso superior conservan lo declarado `floor_tile` debajo de ellos. Pared implícita `set` Las operaciones siguen la misma convención de tierra bajo la pared. Las puertas y las pendientes de las escaleras existentes siguen siendo operaciones predefinidas. Las celdas de la pendiente inferior de la escalera y del rellano reservan espacio libre en el piso z2 directamente encima para que el jugador no quede bloqueado por el piso superior. La geometría generada utiliza el sistema de detección de colisiones y navegación de Chunk existente; las pruebas específicas verifican la ruta multinivel y la geometría del techo del edificio.

Esta sección, intencionadamente, **aún** no define polígonos, límites de habitaciones derivados de la topología, comportamiento en tiempo de ejecución interior/exterior, generación automática de puertas o escaleras, ni plantillas generalizadas.

Capacidades:

* áreas rectangulares o poligonales con nombre y que tienen en cuenta el nivel;
* límites de la habitación;
* pisos y paredes;
* puertas y aberturas;
* designación interior/exterior;
* metadatos del área;
* Huellas de edificios reutilizables;
* anclajes para muebles;
* Geometría de edificios de varios niveles mediante coordenadas z lógicas;
* Semántica de piso ocupado, pared, techo y espacio aéreo intencional;
* anclajes de transición vertical.

Concepto de ejemplo:

```json
{
  "buildings": [
    {
      "template": "small_cabin",
      "origin": {
        "at": [10, 9],
        "z": 0
      },
      "entrance": "south",
      "area_id": "cabin_1"
    }
  ]
}
```

Comprobaciones importantes:

* Cada habitación tiene una entrada;
* Las puertas conectan celdas compatibles;
* Las paredes no bloquean todo el acceso;
* Las definiciones de área coinciden con los límites físicos;
* La geometría superior tiene soporte válido según las reglas de ejecución establecidas en la Fase 4C;
* Las plantas ocupadas cuentan con transiciones verticales válidas y sin obstáculos;
* Las escaleras y las rampas conectan destinos válidos en niveles lógicos adyacentes;
* Los techos, cielorrasos y paredes no ocupan incorrectamente celdas interiores transitables;
* Los muebles permanecen dentro de las habitaciones previstas.

### Criterio de éxito

Genera un edificio pequeño y transitable que se cargue correctamente y tenga un interior accesible. La estructura generada debe cumplir con la geometría física de pisos, paredes, soportes y techo, incluir al menos dos pisos habitables accesibles, soportes generados válidos, una transición vertical funcional y habitaciones y mobiliario con el alcance correcto. La receta de construcción de varios niveles mantenida proporciona la evidencia de este criterio.

## Fase 7: Carreteras y conexiones cartográficas

**Estado: completado; se han completado las conexiones de borde del mapa, los puntos finales de las carreteras, el trazado de carreteras locales en el mapa y la validación de la transitabilidad en tiempo de ejecución.**

El prototipo previamente tenía codificadas de forma fija las cuatro conexiones de borde como `"ground"`Ahora se trata de metadatos a nivel de receta, con anclas de punto final con nombre que identifican las celdas exactas del borde del mapa por donde entran o salen las carreteras.

### Cimientos terminados

El generador de recetas admite un nivel raíz `connections` campo. Acepta hasta cuatro claves. `north`, `east`, `south`, y `west` — cada uno con un valor de `ground` o `road`, que coincide con el formato de mapa de tiempo de ejecución existente. Las direcciones omitidas se establecen por defecto en `ground`De esta forma, las recetas existentes sin ese campo siguen siendo compatibles. El generador valida las direcciones desconocidas y los tipos de conexión no compatibles, y genera el diccionario completo de cuatro direcciones. El validador independiente comprueba el mismo contenido por sí mismo.

El generador de recetas ahora también admite el nivel raíz. `road_endpoints`Cada punto final tiene una identidad única. `id`, un cardenal `direction`, una coordenada de borde `at`y a nivel del suelo `z: 0`Su dirección debe ser declarada como `"road"` en `connections`y su coordenada debe estar en el borde correspondiente del mapa. El generador, el validador independiente y la ruta de guardado/carga de DMap validan o conservan estas referencias de autor. `Tools/examples/map_recipe_road_endpoints.json` La evidencia que se conserva es: un mapa sencillo al aire libre con los extremos de la carretera hacia el oeste y el este, y un camino de tierra que los conecta de forma determinista.

`road_paths` ahora los autores de una polilínea cardinal de nivel del suelo con nombre entre dos validadas `road_endpoints`El generador y el validador independiente requieren diferentes ID de punto final existentes, puntos de referencia delimitados por el mapa opcionales y continuidad cardinal a lo largo de toda la secuencia de punto final a punto final. Ahora, una ruta puede proporcionar un conocido `Ground` cubo `tile`; el generador rasteriza y pinta la ruta completa después de validar el terreno de soporte y rechazar las sobrescrituras de características. El mantenido `map_recipe_road_endpoints.json` Se muestra una ruta de tierra generada de oeste a este. Una prueba GUT específica genera una ruta terrestre correspondiente a través del sistema de navegación real de Chunk y verifica un camino entre puntos interiores cercanos a ambos bordes del mapa. Esta geometría sigue siendo local al mapa: no integra varios mapas.

Capacidades:

* entradas en los bordes del mapa;
* puntos finales de la carretera;
* Enrutamiento de rutas y trazado determinista de carreteras entre anclajes;
* metadatos de conexión norte/este/sur/oeste;
* Se garantiza la continuidad del terreno local en el mapa entre los puntos de entrada declarados;
* La gestión de puentes u obstáculos sigue siendo un trabajo futuro que va más allá de esta estrecha sección local del mapa.

La validación debe determinar:

* si cada conexión declarada tiene un borde de terreno correspondiente;
* si cada ruta generada tiene un terreno de apoyo continuo;
* si los caminos viales terminan en sus puntos finales declarados;
* si las rutas de cubos terrestres generadas están conectadas por navegación en tiempo de ejecución;
* si las teselas de borde coinciden con las expectativas del mapa adyacente; este contrato de compatibilidad entre mapas se pospone intencionadamente y puede reanudarse más adelante.

### Criterio de éxito

Genera un mapa independiente con una o más conexiones de borde de carretera y una ruta continua a nivel del suelo, pintada de forma determinista, entre los puntos finales declarados. La ruta debe utilizar un terreno transitable conservador y superar una comprobación de ruta de navegación real de Godot. La integración completa de rutas en múltiples mapas queda pendiente de validación posterior o se abordará como un tema de superposición de mapas.

## Fase 8: Plantillas y generación de composiciones

**Estado: Fase 8 completada; expansión de plantilla anidada determinista y con reconocimiento de rotación, huellas 3D validadas, ubicaciones compositivas, anclas compatibles con la orientación, variantes numéricas/de colección delimitadas y objetos de parámetros estructurados completos.**

La primera fase 8 incorpora definiciones de plantillas reutilizables sin crear una segunda generación de canalizaciones. Las plantillas se expanden a operaciones raíz ordinarias antes de las etapas existentes de validación de recetas y generación de mapas.

### Cimientos terminados

* Definiciones de plantillas con nombre con relación `dz` secciones de nivel;
* colocaciones sin rotación con orígenes horizontales trasladados y `origin.z + dz` resolución;
* Anclajes de plantilla con nombre y resolución de anclaje absoluta; los anclajes de conexión exterior colocan huellas distintas adyacentes, mientras que los anclajes interiores coincidentes se superponen intencionalmente;
* Comparación de la ubicación de anclajes con respecto a las ubicaciones previas;
* escrito `tile_id`, `boolean`y parámetros de plantilla de enumeración con anulaciones de resolución y nivel de ubicación requeridas/predeterminadas;
* operaciones de plantilla condicionales para la inclusión booleana y variantes semánticas seleccionadas por enumeración;
* Parámetros enteros acotados para coordenadas de operación, dimensiones y recuentos;
* Parámetros de lista de cadenas restringidos con inclusión de operación condicional;
* rotaciones de colocación de `0`, `90`, `180`, y `270` grados, geometría horizontal local giratoria, anclajes y caras de anclaje mientras se preserva `dz`;
* Selección automática de rotación de ancla a ancla a partir de orientaciones cardinales, con rechazo explícito cuando alguna de las anclas conectadas carece de metadatos de orientación;
* Colocaciones recursivas de plantillas anidadas con orígenes locales, rotación acumulativa y `dz`, anulaciones de parámetros reenviados y rechazo de ciclos;
* parámetros de objetos estructurados con esquemas de propiedades declarados, valores predeterminados/obligatorios por propiedad, anulaciones de ubicación y referencias con puntos;
* Cobertura de cabina rotada 5×3 y cobertura de conexión de cabina de ladrillo/metal orientada automáticamente;
* Validación completa de la huella de la plantilla tridimensional antes de la generación ordinaria, con cobertura de operaciones conservadora, límites de mapa/z lógico y rechazo de conflictos no conectados;
* Ubicaciones compositivas rectangulares con nombre, colocación que tiene en cuenta la rotación, comprobaciones de volumen coincidente y superposición intencional de volumen alineado;
* expansión en operaciones raíz ordinarias existentes antes de la validación normal;

### Nota de finalización de la fase 8

El siguiente trabajo de generación de mapas debería mantenerse separado de la composición generalizada de ciudades o mapas generales hasta que una necesidad concreta del juego lo requiera.

## Fase 9 — Ubicaciones compositivas locales del mapa

**Estado: completado; se mantuvo la composición village-square y el fixture de aceptación semántica field_farmland**

La primera sección de la Fase 9 demuestra la composición local del mapa con `Tools/examples/map_recipe_village_square_composition.json`. Expande un centro `village_square` Se utiliza una plantilla con elementos de plaza anidados y anclajes viales con nombre, y luego se emplea la colocación automática de anclajes compatibles con la orientación para fijar una cabina metálica abierta y una cabina con techo de ladrillo. `Tools/examples/map_recipe_field_farmland.json` es el complemento de aceptación semántica mantenido para el legado `field_farmland` Huella: su casa de campo representativa conserva x3…18/y19…29, pisos z0/z2, capas de pared ordinarias z1/z3 y un techo de hormigón específico en z4. Agrega una cocina completa, habitación conectada en la planta baja, habitación en la planta superior, entrada de autor y anclajes para muebles, `reachability_validation`y una escalera z1→z2 sin agregar lógica de generador específica para tierras de cultivo. Ambos mapas generados siguen siendo recursos de mapas tácticos independientes; ninguno introduce composición de ciudades, asentamientos o mapas generales.

### Hitos de ejecución de la fase 9

La fase 9 se implementa en segmentos genéricos. `field_farmland` Es un componente de aceptación, no una implementación de caso especial.

#### 9.1 Perfil genérico de construcción semántica — completo

Los elementos canónicos ahora proporcionan una referencia de autoría genérica antes del enriquecimiento de tierras agrícolas:

* `Tools/examples/map_recipe_semantic_single_storey_building.json` y `Mods/Dimensionfall/Maps/generated_semantic_single_storey_building.json`;
* `Tools/examples/map_recipe_semantic_two_storey_building.json` y `Mods/Dimensionfall/Maps/generated_semantic_two_storey_building.json`;
* cobertura de generador enfocado en `Tools/tests/test_map_generator.py`.

El accesorio de una sola planta ejerce el perfil semántico existente completo del mismo nivel. Ambos accesorios canónicos pintan explícitamente todas las celdas de pared del perímetro de la huella requeridas, incluidas las cuatro esquinas. La planta baja omite la pared en su coordenada de puerta creada para que la puerta tenga un espacio libre z1; cada pared z1 restante, incluidas todas las esquinas, tiene una baldosa de soporte de tierra en z0. El accesorio de dos plantas agrega evidencia completa del límite de la habitación de la planta baja, propiedad de la habitación por piso y anclaje de muebles, geometría de pared z1/z3 ordinaria, un techo z4 explícito y `workroom_staircase` para demostrar la accesibilidad entre pisos y el recorrido en tiempo de ejecución; su loft está cerrado pero no tiene límites completos porque la abertura de la escalera rompe el perímetro. El techo explícito sigue siendo geometría de receta hasta que la generalizada diferida `wall_height` El contrato define los metadatos derivados del techo de la planta alta.

El perfil canónico utiliza los campos existentes en este orden:

1. Geometría física del suelo, las paredes, el techo y las zonas de transición;
2. raíz `rooms` más `room_rectangle` membresías;
3. completo `room_boundaries` y evidencia de puerta;
4. `room_connections` y la propiedad del edificio;
5. `building_levels` asignación de sala/anclaje;
6. `interior_rooms`, `open_space_rooms`, y `room_partition_validation`;
7. `exterior_context`, `exterior_access_context`, `entrance`, y `entrance_validation`;
8. `furniture_anchors` y mantuvo los objetos objetivo.

Un perfil válido debe demostrar que la pertenencia a la sala se encuentra dentro del perímetro, que las salas cerradas completas tienen límites definidos, que las puertas conectan los puntos finales declarados y que cada sala/ancla está asignada a un nivel previsto. El perfil de una sola planta también demuestra la validación actual del acceso semántico same-z. El acceso semántico entre plantas se pospone intencionadamente a la versión 9.3, donde las escaleras validadas se convierten en aristas verticales del grafo. `access_validation` Sigue siendo una validación de grafos semánticos; no es a prueba de colisiones ni de navegación.

#### 9.2 Validación independiente de los límites de los datos: completa

Ambas recetas canónicas se generan de forma determinista y se validan con `Tools/map_validator.py`. `Tests/Unit/test_dmap_sanitization.gd` Carga ambas fuentes de recetas a través del asistente JSON de producción y demuestra que los metadatos de sala, conexión, límite, edificio y soporte viajan de ida y vuelta a través de la sanitización de DMap.

Esto puso al descubierto y solucionó una brecha de compatibilidad real: el análisis JSON de Godot representa los literales numéricos como números de coma flotante, mientras que los validadores semánticos de DMap requieren números enteros para las coordenadas, los niveles z y las rotaciones. `Scripts/Helper/json_helper.gd` Ahora, normaliza recursivamente los números flotantes JSON enteros a enteros antes de devolver los diccionarios o matrices analizados; los números flotantes no enteros permanecen como tales. No duplique esta cobertura de límites genérica para tierras de cultivo.

#### 9.3 Contrato genérico de ruta obligatoria — completo

Los edificios pueden declarar una opción `reachability_validation` registro:

```json
{
  "required_entrances": ["front_entrance"],
  "required_furniture_anchors": ["workbench"],
  "required_building_levels": [0, 2]
}
```

Nombra únicamente las entradas existentes, los anclajes de muebles y los niveles ocupados declarados. No almacena coordenadas, identificadores de mosaico ni rutas precalculadas. La validación estática rechaza objetivos desconocidos, niveles no declarados, habitaciones semánticas inaccesibles y enlaces de grafos verticales faltantes. Los bordes del grafo vertical se derivan únicamente de escaleras ya válidas creadas; el enlace es granular a nivel de piso porque las pendientes de las escaleras ocupan el espacio abierto intencional sin pertenencia a ninguna habitación. El contrato se implementa de forma independiente en `Tools/map_generator.py`, `Tools/map_validator.py`, y `Scripts/Gamedata/DMap.gd`, con pruebas negativas y positivas focalizadas en `Tools/tests/test_map_generator.py` y cobertura de preservación de DMap en `Tests/Unit/test_dmap_sanitization.gd`Python no simula colisiones ni navegación en tiempo de ejecución; la función auxiliar reutilizable Phase 9.4 demuestra el recorrido en tiempo de ejecución. La base multinivel mantenida, el accesorio canónico de dos pisos y `field_farmland` Los dispositivos de aceptación demuestran registros completos entre pisos a través de sus escaleras diseñadas.

#### 9.4 Accesorio de navegación en tiempo de ejecución reutilizable — completo

`Tests/Unit/helpers/map_navigation_fixture.gd` proporciona el asistente GUT reutilizable en torno a la configuración de Chunk existente, horneado de navegación, conversión de cuadrícula a mundo y `NavigationServer3D.map_get_path()` aserciones. El asistente es genérico: recibe coordenadas de punto final, formas y etiquetas; no conoce tierras de cultivo, semántica de habitaciones ni ninguna receta mantenida.

Comprobado con:

* `Tests/Unit/test_semantic_single_storey_navigation.gd` — de exterior a interior y retorno sobre la geometría canónica de una sola planta;
* `Tests/Unit/test_semantic_two_storey_navigation.gd` — desde el exterior hasta la planta baja, desde la planta baja hasta el piso superior a través de la escalera diseñada, y ambos retornos;
* `Tests/Unit/test_chunk_road_navigation.gd` y `Tests/Unit/test_multi_level_building_navigation.gd`, refactorizado en el mismo auxiliar con cobertura sin cambios.

Las comprobaciones de tiempo de ejecución requeridas son de exterior a objetivo de suelo y, para accesorios de dos pisos, de suelo a objetivo superior. Las pruebas de tiempo de ejecución demuestran el comportamiento de colisión/navegación; las pruebas de Python no las reemplazan. Para hacer posible la prueba de dos pisos, el accesorio canónico de dos pisos ahora escribe `workroom_staircase` (pendiente inferior [10, 10] en z1, pendiente superior [10, 9] en z2) y mejora su `reachability_validation` requerir `loft_bench` y nivel z2; el loft es `enclosed` pero intencionalmente no `boundary_validation: complete` porque la abertura de la escalera rompe su perímetro, reflejando la cimentación multinivel que se ha mantenido.

#### 9.5 Aplicar el perfil genérico a las tierras de cultivo — completar

`Tools/examples/map_recipe_field_farmland.json` aplica el perfil genérico existente a un representante `field_farmland` casa de campo: una completa `farmhouse_kitchen`, conectado `farmhouse_ground_room`, `farmhouse_upper_room`, propiedad del edificio en z0/z2, una entrada de habitación al exterior, anclajes de puerta y estación de trabajo, `reachability_validation`, y `farmhouse_staircase`Conserva la planta original de la casa de campo (x3…18/y19…29), las elevaciones de piso y pared habituales (z0/z1 y z2/z3) y un techo en z4. La receta representa intencionadamente solo la sección de cocina, planta baja y planta alta necesaria para el dispositivo de aceptación; no reproduce todas las habitaciones del mapa original ni introduce un comportamiento de generador específico para terrenos agrícolas.

`Tests/Unit/test_field_farmland_navigation.gd` Utiliza el asistente de navegación genérico para demostrar el recorrido exterior→cocina, cocina→planta baja-habitación y recorrido bidireccional planta baja↔superior. `Mods/Dimensionfall/Maps/generated_field_farmland.json` es el artefacto generado determinista y pasa de forma independiente `Tools/map_validator.py` validación.

#### 9.6 Flujo de trabajo de ejecución y validación

Cada segmento de implementación sigue la misma secuencia de prueba primero:

1. agregar una prueba fallida centrada en el comportamiento;
2. Ejecute únicamente esa prueba y confirme que el fallo se debe a la falta del contrato, y no a un error de la prueba;
3. implementar el cambio más pequeño en el generador;
4. Reflejar el contrato en el validador independiente y DMap solo cuando ese límite de datos maneja el campo;
5. Vuelva a ejecutar la prueba específica y, a continuación, el conjunto completo de pruebas de Python;
6. regenerar el artefacto JSON mantenido;
7. correr `Tools/map_validator.py`, pruebas GUT relevantes y `git diff --check`.

Archivos principales por responsabilidad:

* Pruebas de generadores y recetas: `Tools/map_generator.py`, `Tools/tests/test_map_generator.py`;
* validación estática independiente: `Tools/map_validator.py`;
* Serialización/sanitización en tiempo de ejecución: `Scripts/Gamedata/DMap.gd`, `Tests/Unit/test_dmap_sanitization.gd`;
* navegación en tiempo de ejecución: `Tests/Unit/test_semantic_single_storey_navigation.gd`, `Tests/Unit/test_semantic_two_storey_navigation.gd`, y `Tests/Unit/test_field_farmland_navigation.gd`;
* recetas/artefactos mantenidos: `Tools/examples/` y `Mods/Dimensionfall/Maps/`;
* documentación: esta hoja de ruta y `Documentation/Modding/map_recipe_generator.md`.

Los comandos específicos son:

```bash
PYTHONPATH=. python3 -m unittest discover -s Tools/tests -p "test_*.py"
python3 -m py_compile Tools/map_generator.py Tools/map_validator.py Tools/tests/test_map_generator.py
python3 Tools/map_generator.py <recipe> <generated-map> --overwrite
python3 Tools/map_validator.py <generated-map>
/usr/local/bin/godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://Tests/Unit -gexit
git diff --check
```

#### 9.7 Criterio de éxito de la fase 9

Se puede crear una nueva ubicación en el mapa a partir de plantillas reutilizables, con un perfil semántico genérico, validación estática, generación determinista y comprobación mediante navegación en tiempo real. El mapa resultante sigue siendo un recurso individual de mapa táctico/mapa superpuesto. La generación generalizada de asentamientos y la compatibilidad de bordes entre mapas se posponen.

## Fase 10: Validación de la calidad y la jugabilidad

**Estado: en curso; la generación automática de límites de habitaciones de la fase 10.1.1 está completa.**

La fase 10 convierte la validez estructural en comprobaciones de aceptación explícitas. Implemente las comprobaciones en este orden:

### 10.1 Comprobaciones estáticas creadas por el usuario

Agregar comprobaciones específicas para:

* Cada entrada requerida indica una entrada/conexión existente y tiene un acceso exterior válido;
* Todos los anclajes necesarios para el mobiliario existen, pertenecen al edificio y están asignados a una planta determinada;
* Se declara cada piso ocupado requerido;
* Cada sala semántica tiene una ruta predefinida hacia una conexión exterior o una transición vertical válida;
* Cada extremo de la escalera se encuentra dentro del perímetro delimitado, cuenta con evidencia válida del terreno y la pendiente correspondiente, y alcanza un piso objetivo declarado;
* Las puertas, las escaleras y los objetivos importantes no quedan ocultos por geometrías de diseño incompatibles;
* todos los valores lógicos z permanecen dentro `-10` a través de `+10`;
* La regeneración determinista produce una salida equivalente a bytes.

Estos controles pertenecen a `Tools/map_generator.py` y `Tools/map_validator.py`; deben informar el edificio, la habitación/ID del objetivo, el piso y la relación faltante en caso de fallas.

#### 10.1.1 Fiabilidad de los límites de la habitación y redacción concisa: completa

La corriente `room_boundaries` El contrato establece una ruta de compatibilidad explícita: cada registro proporciona evidencia física documentada para un borde de pared o puerta. Debe seguir siendo compatible con las recetas existentes, la geometría irregular, las ruinas, las aberturas intencionadas y los materiales de pared inusuales. No debe ser la única forma de crear una habitación cerrada completa y convencional.

Por ejemplo, agregue un contrato opcional de generación de perímetro a nivel de habitación para habitaciones cerradas normales:

```json
{
  "id": "farmhouse_kitchen",
  "kind": "enclosed",
  "boundary_validation": "complete",
  "boundary_generation": "walls"
}
```

Cuando está habilitado, el generador deriva el perímetro final de la habitación a partir de la pertenencia a la habitación y utiliza el edificio propietario. `building_geometry.wall_tile` para crear el anillo de pared físico. `room_rectangle` sigue siendo una operación de membresía de sala; no coloca paredes por sí mismo. El perímetro derivado debe incluir tapas de esquina y debe dejar aberturas solo para válido `room_connections` Cruzando el borde de esa habitación. Una conexión entre habitaciones consume el borde frontal para ambas habitaciones generadas participantes. El resolvedor debe trabajar a partir de la máscara de pertenencia final para que las futuras operaciones de habitaciones no rectangulares, la traslación, el anidamiento y la rotación horizontal no requieran casos especiales específicos para rectángulos.

Implementa esta sección en el siguiente orden:

1. Agregue pruebas de caracterización antes de cambiar la generación. Cubra una habitación completa de 3×3, todas las esquinas del perímetro, aberturas exteriores y entre habitaciones, conexiones no válidas o no cardinales, herrajes de puerta faltantes, bordes compartidos de habitaciones generadas y preservación de elementos explícitos existentes. `room_boundaries` comportamiento.
2. Agregue y valide el nuevo campo a nivel de habitación. Inicialmente, restrinja su uso a: `enclosed` habitaciones con `boundary_validation: "complete"`; rechazar la mezcla de límites generados con explícitos `room_boundaries` para la misma habitación y nivel, por lo que la propiedad de la pared es inequívoca.
3. Implemente un resolvedor de perímetro reutilizable que derive los bordes semánticos expuestos, las aberturas de conexión, las coordenadas del anillo de muros y la propiedad determinista de muros compartidos a partir de la pertenencia final a la sala. No duplique esta lógica en el código específico de la receta.
4. Materializar las paredes resueltas en `floor_z + 1`, utilizando la baldosa de pared del edificio propietario, y rechazar la geometría de autor incompatible en lugar de sobrescribirla silenciosamente. Generar soportes de suelo solo en el piso propietario.
5. Conservar los metadatos semánticos cuando un soporte reemplaza una loseta de suelo. Reemplazar una loseta por un soporte no debe borrar su `rooms`, `areas`o cualquier otro metadato compatible.
6. Refleje el solucionador y los diagnósticos en `Tools/map_validator.py`; preservar y desinfectar el campo a nivel de habitación en `Scripts/Gamedata/DMap.gd` cuando ese límite de datos lo gestiona.
7. Migrar el `field_farmland` cocina a la declaración concisa a nivel de habitación y eliminar sus registros redundantes por pared y operaciones duplicadas de pared de cocina. Verificar que el perímetro generado incluya las celdas de esquina faltantes `[7,22]` y `[7,26]`, mientras las conexiones permanezcan abiertas.
8. Documentar la declaración compacta, la escotilla de escape de límites explícitos, las aperturas derivadas de la conexión y las reglas de conflicto en `Documentation/Modding/map_recipe_generator.md` y `Documentation/Modding/map_example_generation.md`.

La fase 10.1.1 está completa. La cobertura del generador enfocado y del validador independiente demuestra que una habitación completa de 3×3 generada crea su anillo de pared exterior completo de 5×5, incluyendo las cuatro tapas de las esquinas, mientras que un exterior válido `room_connection` sigue siendo la única abertura. La cobertura adicional demuestra que los enlaces entre habitaciones se abren por ambos bordes enfrentados mientras se mantiene un objetivo de puerta física, se rechazan los cruces diagonales, las habitaciones generadas adyacentes comparten una pared divisoria determinista, explícita `room_boundaries` El comportamiento permanece intacto y la sustitución del soporte conserva los metadatos de los mosaicos. `field_farmland` ahora migra su cocina a la declaración concisa: ya no enumera límites por pared ni operaciones duplicadas de pared de cocina, y su anillo generado incluye `[7,22]` y `[7,26]`.

Esta sección pertenece a la Fase 10.1 porque mejora los diagnósticos genéricos y la fiabilidad estructural. No es específica de terrenos agrícolas y no debe introducir una condición para terrenos agrícolas en el generador ni en el validador.

### 10.2 Comprobaciones de jugabilidad en tiempo de ejecución: completas

La fase 10.2 está completa para la migración. `field_farmland` Recorrido de la sección en tiempo de ejecución de la granja. `Tests/Unit/test_field_farmland_navigation.gd` utiliza el genérico `MapNavigationFixture`, real `Chunk` navegación horneando y `NavigationServer3D` Consultas de ruta a verificar:

* Acceso exterior → cocina a través de la puerta oeste;
* cocina → planta baja a través de la conexión de habitaciones declarada;
* Recorrido bidireccional entre la planta baja y la planta superior a través de la escalera diseñada;
* El horneado de navegación se completa solo después de que `NavigationServer3D` La iteración del mapa avanza y dos marcos físicos adicionales estabilizan la región recién adjunta.

Esta sección no contempla el bloqueo de muros en tiempo de ejecución: el sistema actual aporta caras superiores de bloques a la geometría de la fuente de navegación, por lo que un muro de un bloque puede representarse como una superficie de navegación elevada. Se requiere un cambio posterior en el sistema de colisión/navegación antes de poder afirmar que las celdas de muro automáticas bloquean los cruces. El estado dinámico de las puertas, el combate y el recorrido completo mediante la entrada del jugador quedan fuera de esta fase.

Utilice el horneado de navegación de Chunk real y `NavigationServer3D`; no introduzca un buscador de rutas de Python.

### 10.3 Dispositivos de aceptación mantenidos — completos

El conjunto de pruebas de aceptación está completo. `MapGeneratorTests.test_maintained_recipe_suite_generates_validates_and_is_deterministic` enumera cada mantenido `Tools/examples/map_recipe*.json` receta, incluyendo los accesorios semánticos genéricos de una y dos plantas, la base de varios niveles, la composición de la plaza del pueblo y `field_farmland`Para cada receta:

* genera dos artefactos escritos de forma independiente;
* valida cada artefacto a través de `write_map`validación de la publicación y una nueva publicación independiente `MapValidator` instancia;
* requiere una salida idéntica en bytes de ambas regeneraciones; y
* Se produce un error cuando se agrega o elimina una receta sin actualizar el conjunto de recetas mantenido explícitamente.

La cobertura relevante del tiempo de ejecución de GUT sigue centrada en la clase de geometría: semántica de un solo piso, semántica de dos pisos, `field_farmland`y Pine Hollow Outpost funciona de verdad `Chunk` hornear y `NavigationServer3D` Comprobaciones de navegación. Mantenga este conjunto en verde siempre que cambien los ejemplos mantenidos o el comportamiento de generación de mapas; agregue una prueba de tiempo de ejecución enfocada equivalente solo cuando un fixture introduzca geometría de juego que aún no esté cubierta.

### 10.4 Criterio de éxito de la fase 10: completado

El criterio de éxito de la Fase 10 se ha completado para el conjunto de accesorios mantenido. El generador, el validador independiente, la persistencia de DMap y las pruebas de navegación en tiempo de ejecución ahora proporcionan una cobertura procesable para la estructura y el recorrido creados: las puertas de la casa de campo, las conexiones de las habitaciones y las escaleras previstas son accesibles, mientras que las comprobaciones limitadas en tiempo de ejecución rechazan los cruces no deseados a través de las paredes automáticas de la cocina y las tapas de esquina generadas. El genérico `MapNavigationFixture` Los modelos consideran las paredes como obstáculos de navegación verticales en lugar de superficies superiores transitables, por lo que estas comprobaciones realizan la misma consulta de ruta al servidor de navegación que utiliza el juego.

### 10.5 Nueva ubicación creada: Puesto de avanzada de Pine Hollow — implementado; inspección visual manual completada

**Estado: aceptación visual/compositiva automatizada y manual completada; conversión de la ubicación del mapa de campo completada**

La fase 10.5 cambia el objetivo principal de aceptación, pasando de reproducir un mapa existente ya creado a crear una nueva ubicación a partir de una solicitud de diseño concisa. `field_farmland` Sigue siendo una referencia visual y de comportamiento, pero el nuevo dispositivo no debe copiar sus coordenadas, secuencia de funcionamiento ni la disposición completa de los paneles o elementos.

La primera ubicación es un pequeño puesto maderero rural abandonado, adecuado para el entorno de supervivencia rural de Dimensionfall:

* una cabaña de cuidador desgastada por el tiempo, con una habitación cerrada y un altillo;
* un cobertizo cubierto o descubierto, o un refugio para almacenar leña, junto a la cabaña;
* un corto sendero de tierra para peatones y personal desde el patio hasta la entrada de la cabaña;
* un patio de trabajo cercado o parcialmente cubierto de maleza;
* un pozo, una bomba manual, una estación de trabajo o un mueble de almacenamiento representativo;
* Árboles, terreno accidentado y escasos detalles ambientales alrededor del sitio.

El objetivo es ejercitar el vocabulario genérico existente en una nueva composición: terreno base, clasificación del mapa de campo, senderos peatonales locales, habitaciones, muros automáticos, geometría de edificios, entradas, anclajes de mobiliario, escaleras, áreas y variación determinista. No introduzca un sistema de generación de asentamientos, una condición específica para la granja ni una nueva primitiva física de construcción, a menos que el puesto avanzado exponga un requisito concreto que falte.

#### 10.5 criterio de éxito

El puesto avanzado de Pine Hollow generado debe:

* ser elaborado a partir de una nueva receta concisa en lugar de una transcripción manual de un mapa fuente;
* Crear un puesto de avanzada rural de supervivencia visualmente reconocible con una cabaña, un cobertizo/área de almacenamiento, un patio y un corto sendero peatonal/de servicio;
* Proporcionar una cabaña accesible con entrada exterior y una conexión intencionada con el área exterior/cubierta-abierta;
* Proporcionar un altillo o un nivel de almacenamiento elevado con una escalera válida, altura suficiente y una superficie superior accesible;
* conservar los elementos/características representativas y la semántica del área que son importantes para la jugabilidad;
* Bloquear los cruces no deseados a través de muros cerrados, preservando al mismo tiempo las entradas y conexiones declaradas;
* validación del generador de pases, independiente `MapValidator` validación, regeneración determinista y la matriz de aceptación de recetas mantenida;
* pasar comprobaciones de tiempo de ejecución enfocadas usando reales `Chunk` hornear y `NavigationServer3D` para el recorrido desde el camino de servicio a la cabaña, desde la cabaña al área exterior y desde la cabaña al altillo.

Detalles decorativos, dispersión exacta del terreno, colocación menor de objetos y otras diferencias no críticas para la jugabilidad. `field_farmland` Se permiten intencionadamente. La fase 10.5 debe añadir comparaciones/informes solo cuando aclare la equivalencia estructural o visual; no debe convertir la nueva receta en una instantánea equivalente en bytes de la granja de código original.

La receta de producción es `Tools/recipes/pine_hollow_outpost.json`, con ID estable `pine_hollow_outpost`; su recurso de tiempo de ejecución publicado es `Mods/Dimensionfall/Maps/pine_hollow_outpost.json`Esta promoción no reemplaza el legado `field_outpost.json` mapa. La receta de ejemplo sigue siendo un accesorio de regresión mantenido por separado con el `generated_pine_hollow_outpost` IDENTIFICACIÓN.

`Tests/Unit/test_pine_hollow_outpost_navigation.gd` utiliza real `Chunk` hornear y `NavigationServer3D` Consultas de ruta para verificar las rutas de servicio a la cabaña y de la cabaña al cobertizo, el recorrido bidireccional de la planta baja al altillo y un hueco aislado en la pared norte que no se puede cruzar. La parte estructural/de ejecución automatizada y la revisión manual del editor de contenido de Godot están completas. La revisión confirmó el acceso peatonal, la alineación de la puerta, la silueta de la cabaña/cobertizo, el patio de trabajo, los muebles/accesorios, el altillo/escaleras/techo, el umbral de hormigón y la ausencia de cobertura de techo sobre el cobertizo.

El orden de implementación es:

1. ~~autor `Tools/examples/map_recipe_pine_hollow_outpost.json` a partir del concepto de ubicación;~~
2. ~~Añadir cobertura de generador enfocado y validador independiente para la nueva composición;~~
3. ~~Agregar o extender la cobertura de navegación en tiempo de ejecución de GUT enfocada para las rutas declaradas y los cruces de muros bloqueados;~~
4. ~~generar el mapa e inspeccionarlo a través del flujo de trabajo existente de Godot/editor;~~
5. ~~Marque la revisión visual como completada solo después de que dicha inspección confirme la composición prevista del puesto de avanzada rural.~~

### 10.6 Operaciones de superficie derivadas de la sala — implementadas

**Estado: implementado; aceptación automatizada completada**

La fase 10.6 cierra la brecha de geometría de propiedad de la habitación de Pine Hollow sin fusionar capas semánticas distintas. La raíz `rooms` El registro sigue siendo la fuente de la definición, `buildings[].rooms` sigue siendo la relación de propiedad retrocompatible, y `building_levels[].rooms` Se mantiene la relación de propiedad por planta.

`room_surface` Ahora obtiene la cobertura física de la membresía de la habitación final autorizada:

```json
{"type": "room_surface", "room": "outpost_loft", "z": 2, "tile": {"id": "concrete_00"}}
{"type": "room_surface", "room": "outpost_loft", "z": 4, "tile": {"id": "concrete_00"}, "outline_padding": 1}
```

Acepta una habitación conocida, z lógica objetivo, especificación de mosaico y no negativo opcional. `outline_padding`Las celdas derivadas utilizan la pertenencia final a una habitación que ocupa exactamente una coordenada z de origen; el relleno expande dicha pertenencia mediante el radio de Chebyshev y debe permanecer dentro de los límites del mapa. La materialización de la superficie se pospone hasta que las etiquetas de las habitaciones estén completas, conserva los metadatos/características existentes y mantiene las teselas de pendiente creadas en la misma coordenada z. `rectangle` Las operaciones siguen siendo válidas para terrenos, carreteras, patios y aleros intencionados. `room_connections` mantener la autoridad; esta fase no implica puertas ni aperturas.

`building_geometry` ahora limita la materialización del piso en declarado `building_levels` a las celdas miembros de las habitaciones de ese nivel, más los soportes de pared requeridos, a menos que ese nivel declare explícitamente un edificio en general `building_surfaces` piso. Ya no ocupa silenciosamente toda una superficie de varios niveles cuando solo una parte de la habitación tiene un nivel elevado.

Pine Hollow ahora expresa su `6×6` piso del altillo y acolchado `8×8` techo sin rectángulos de coordenadas duplicados:

```text
outpost_loft membership: x12..17, y11..16 (6×6)
loft wall envelope:       x11..18, y10..17 (8×8)
loft roof:                x11..18, y10..17 (8×8)
lean-to:                  outside the loft floor and roof coverage
```

La cobertura automatizada verifica la derivación de la membresía final independientemente del orden de operación, cobertura acolchada, habitaciones desconocidas/baldosas/acolchado no válido/membresía faltante/diagnósticos fuera de límites, caja de desván y escaleras preservadas, celdas adosadas excluidas, regeneración determinista de receta mantenida, independiente `MapValidator` Validación de la salida y aceptación de la navegación en tiempo de ejecución real existente. `room_surface` es una sintaxis exclusiva de recetas y no aparece en los datos del mapa generado, por lo que no se necesita ningún campo de persistencia de DMap ni una extensión de esquema de validador de mapa generado independiente.

La revisión visual y compositiva de la fase 10.5, realizada con el editor de contenido manual Godot, del acceso peatonal a Pine Hollow, la silueta de la cabaña/refugio, el patio de trabajo y la ubicación de los elementos de la escenografía, ya está completa.

### 10.7 Declaraciones canónicas de puertas y entradas — completas

**Estado: completado**

La fase 10.7 reduce las descripciones duplicadas de una única puerta física expuesta por Pine Hollow. Una puerta exterior actual puede describirse mediante una raíz. `room_connections` registro, un registro de entrada a nivel de edificio, `exterior_context`, `exterior_access_context`, un tipo de puerta `furniture_anchor`, una asignación de anclaje por piso y una operación independiente para puertas y herrajes. Estos registros no representan todos el mismo problema, pero sus identificadores y coordenadas repetidos dificultan la creación y el mantenimiento de recetas.

El modelo objetivo consiste en convertir la conexión entre habitaciones en la identidad canónica de una puerta o abertura. Una conexión entre una habitación y el exterior puede optar por incluir metadatos de entrada:

```json
{
  "id": "outpost_front_door",
  "at": [12, 14],
  "target_at": [11, 14],
  "z": 0,
  "from": {"kind": "room", "id": "outpost_cabin"},
  "to": {"kind": "exterior"},
  "entrance": {
    "exterior_at": [10, 14],
    "facing": "east"
  }
}
```

En este modelo:

* `room_connections[].id` es la identidad estable utilizada por los metadatos de construcción y accesibilidad;
* `at` sigue siendo la coordenada semántica del borde de la habitación;
* `target_at` sigue siendo la coordenada física de la puerta/apertura, preservando la distinción existente entre lo semántico y lo físico;
* `entrance.exterior_at` reemplaza el edificio separado `exterior_context` para esa entrada;
* `entrance.facing` reemplaza el edificio separado `entrance`/`entrances` declaración de confrontación;
* Una entrada se identifica mediante su ID de conexión, por lo que no se requiere un ID de entrada independiente;
* Los anclajes de muebles para cada tipo de puerta y las asignaciones de anclajes de puertas por piso se derivan del objetivo de conexión en lugar de crearse de nuevo;
* Los anclajes de mobiliario genéricos permanecen independientes para cajas, bancos, estaciones de trabajo, almacenamiento y otros objetivos de juego;
* Las conexiones internas entre habitaciones siguen siendo válidas sin metadatos de entrada;
* `room_connections` Siguen siendo declaraciones semánticas y no generan automáticamente una función de puerta en la primera sección.

La primera implementación debe preservar la compatibilidad con versiones anteriores. Recetas que utilizan el nivel de construcción existente. `entrance`, `entrances`, `exterior_context`, `exterior_access_context`y los anclajes de herrajes de puerta siguen siendo válidos. Si están presentes tanto los metadatos de entrada heredados como los de nivel de conexión, el generador y el validador independiente deben exigir que describan la misma conexión, destino, contexto exterior y orientación. Los mapas generados deben continuar serializando los metadatos de construcción compatibles con el tiempo de ejecución establecidos hasta que los consumidores puedan migrar.

La función de accesibilidad debe utilizar todas las conexiones de habitación a exterior que tengan los metadatos de entrada como puntos de partida predeterminados. `required_entrances` Sigue siendo un requisito de restricción opcional para edificios con múltiples entradas; los edificios ordinarios de una sola entrada no deberían necesitar repetir el ID de conexión en una lista de requisitos separada.

La generación automática de características de puerta se pospone deliberadamente. Una versión posterior podría permitir que una conexión de entrada declare una característica de puerta y su rotación, pero esto requiere reglas explícitas para conflictos, puertas heredadas, puertas bloqueadas, ventanas y apertura de objetos que no sean puertas. La fase 10.7 inicialmente canoniza la identidad y la validación; no infiere topología, colisión, navegación ni estado interior.

#### 10.7 criterio de éxito

Una receta migrada al estilo Pine Hollow debe describir su puerta exterior con un ID de conexión de habitación canónico, conservando el mapa generado actual y el comportamiento en tiempo de ejecución. El generador y el independiente `MapValidator` Se debe validar la entrada de metadatos a nivel de conexión, detectar conflictos con declaraciones heredadas, derivar semillas de accesibilidad predeterminadas, preservar las conexiones internas y los anclajes genéricos de mobiliario, y mantener la salida generada compatible con los consumidores de DMap/tiempo de ejecución. La generación determinista, la validación de recetas mantenidas, el comportamiento de ida y vuelta de DMap y la aceptación de navegación real deben permanecer en verde.

#### 10.7 Orden de implementación

1. ~~Agregar cobertura de generador test-first y validador independiente para un nivel de conexión `entrance` bloque, incluyendo la inicialización de accesibilidad predeterminada y metadatos heredados no válidos/conflictivos. Mantenga la prueba inicial en rojo antes de agregar código de producción.~~ La cobertura enfocada ahora es VERDE en `Tools/tests/test_map_generator.py`.
2. ~~Agregue la validación de esquema compatible con versiones anteriores más pequeña y la representación interna normalizada en `Tools/map_generator.py` y `Tools/map_validator.py`.~~ Ambas herramientas ahora aceptan y validan la opción `entrance` objeto, conservarlo en los datos de conexión generados, validar el enlace externo, los límites, la orientación y rechazar las declaraciones mal formadas.
3. ~~Migrar la puerta exterior de Pine Hollow a la nueva declaración, conservando temporalmente la forma anterior como prueba de compatibilidad de equivalencia exacta.~~ Ambas copias de la receta de Pine Hollow ahora utilizan `room_connections[].entrance`; la entrada/contexto temporal del edificio heredado, los herrajes de las puertas y las declaraciones de anclaje de las puertas fueron posteriormente retirados tras las pruebas de equivalencia exacta.
4. ~~Refleje los metadatos de edificios persistentes normalizados en `Scripts/Gamedata/DMap.gd` y agregar cobertura de ida y vuelta de DMap si los campos serializados cambian. `DMap` Ahora conserva los metadatos de entrada válidos a nivel de conexión y elimina los datos de entrada anidados mal formados sin descartar una conexión que de otro modo sería válida; `Tests/Unit/test_dmap_sanitization.gd` abarca ambos comportamientos.
5. ~~Elimine el anclaje de puerta redundante de Pine Hollow, el anclaje de puerta por nivel y las declaraciones separadas de entrada/contexto solo después de que las pruebas de compatibilidad demuestren una salida equivalente.~~ La entrada canónica de nivel de conexión ahora posee la identidad de la puerta y los metadatos de aproximación exterior; las declaraciones redundantes se eliminaron de ambas recetas de Pine Hollow.
6. ~~Repita la generación mantenida, la comparación determinista, la validación independiente y la cobertura de navegación enfocada de Godot; documente el contrato de autoría final.~~ La matriz de aceptación mantenida permanece en verde después de la regeneración y el contrato de autoría final se documenta arriba.

### 10.8 Pase de detalle e inmersión del puesto de avanzada Pine Hollow — completo

**Estado: completado; variación de campo controlada, suelos de madera desgastados, mobiliario de cabaña, decoración exterior delimitada, grupos de artículos de contenedores y validación de aceptación completada**

La fase 10.7 convirtió a Pine Hollow en un mapa de campo jugable y correctamente clasificado. La fase 10.8 completó su revisión de detalles e inmersión utilizando los elementos genéricos existentes de terreno, paleta, área, mobiliario y habitaciones/edificios, sin añadir condiciones de generador específicas para Pine Hollow.

La primera sección implementada utiliza el campo compatible existente. `grass_plain_01`, `grass_dirt_00`–`02`, `grass_medium_dirt_00`, y `grass_flowers_00` entradas de paleta. Excluye deliberadamente `forest_underbrush_00`: Pine Hollow es un mapa de campo, no un mapa de bosque. También excluye `grass_dead_00`, lo cual no se ajusta al tratamiento de campo mantenido previsto. El `outpost_ground` El área de ejecución cubre cada celda del mapa fuera del perímetro de la cabaña y siempre reemplaza el suelo cubierto con una mezcla mayoritariamente de césped; no utiliza `null`. La cabaña y el altillo utilizan `floor_wood_shabby_00`, mientras que el umbral exterior y el cobertizo abierto utilizan explícitamente `concrete_00`La composición de la planta baja utiliza una cama de madera, estufa, armario, mesa, silla y banco de herramientas adosado; el revestimiento exterior delimitado de la naturaleza utiliza un `outpost_field_nature` Área de ejecución con vegetación silvestre, roca de campo, pino y tocones quemados en regiones de membresía solo para exteriores. `outpost_yard` El área de ejecución tiene tres variantes de terreno y excluye deliberadamente el camino, el edificio y el cobertizo. La selección del área de ejecución hace que el revestimiento exterior varíe según la instancia del mapa, en lugar de incorporarlo al JSON generado. Las aserciones del generador específico protegen el material, la ruta, el objeto requerido, la paleta que no sea de bosque, el área de ejecución y los contratos de autorización.

#### 10.8.1 Contrato de composición redactado por el autor

Mantenga estable la siguiente intención espacial mientras agrega detalles:

* el mapa sigue siendo un `Field` mapa con las cuatro conexiones de borde establecidas en `"ground"`;
* el mapa no tiene `road_endpoints` o `road_paths`; la ruta es un sendero peatonal/de servicio local, no una infraestructura vial transitable para vehículos;
* `room_connections[].id` sigue siendo la identidad de puerta de entrada canónica;
* `entrance.exterior_at` sigue siendo el punto de acceso al aire libre en `[10, 14]`;
* el umbral físico en `[11, 14]` permanece de hormigón y conserva el `door_wood` característica;
* El contrato relativo a la habitación de la cabina, el cobertizo, el altillo, las escaleras, la abertura en la pared y el paso a través de la pared bloqueada permanece sin cambios;
* La vía de servicio permanece continua desde el patio exterior hasta el acceso de entrada y no debe ser sustituida por variaciones aleatorias del terreno;
* La entrada a la cabina, las escaleras, el espacio libre para los muebles y las rutas de navegación necesarias se reservan antes de aplicar cualquier colocación decorativa o esparcida.

No utilice la decoración para alterar la pertenencia a una sala, crear aberturas, eludir la validación de edificios o implicar una nueva regla de colocación en el mapa general.

#### 10.8.2 Tratamiento del suelo y los caminos

Sustituir el tratamiento exterior actual, casi uniforme, por una variación determinista controlada:

* definir una paleta de terreno ponderada utilizando variantes compatibles de césped, césped/tierra, terreno accidentado y variantes estacionales o desgastadas ya presentes en el catálogo de baldosas;
* Mantener el terreno base legible como césped en lugar de convertir todo el mapa en tierra;
* Utilice un tratamiento de jardín con menor variación alrededor de la cabaña y el cobertizo, con parches desgastados cerca de la entrada, el área de trabajo, el almacenamiento, los restos de la cerca y el camino;
* conservar el corto `dirt_light_00` camino de servicio desde el patio hasta `[10, 14]` y mantener su anchura intencionadamente a escala peatonal;
* Evite colocar paletas o objetos dispersos en la cabina, cobertizo, altillo, techo/soporte, umbral, escalera, pared, muebles y celdas de navegación reservadas;
* utilice la semilla de receta fija y asegure que la generación repetida sea idéntica byte a byte;
* Prefiero las paletas y la dispersión delimitada a las grandes listas de mosaicos creadas manualmente, manteniendo explícitas las celdas focales compuestas deliberadamente.

La paleta de colores debe seleccionarse del catálogo de mosaicos real durante la implementación. No invente identificadores de mosaicos en la implementación del plan de desarrollo sin la validación del catálogo.

#### 10.8.3 Suelos de cabina y material estructural

Cambie el revestimiento del suelo predeterminado de la cabaña, pasando de hormigón genérico a una baldosa de madera transitable adecuada, siempre que el catálogo y el entorno de ejecución lo permitan:

* Utilice un suelo de madera para la habitación de la planta baja de la cabaña y, si procede, para el altillo;
* Conserve el hormigón en el umbral exterior si este comunica una base desgastada o una entrada protegida de la intemperie;
* conservar las escaleras, los azulejos/elementos de la puerta, los soportes de las paredes, el techo y la distribución de todas las habitaciones al cambiar el material del suelo;
* verificar que la baldosa de madera seleccionada sea transitable válida `Ground`/`Floor` mosaico y que no introduce una regresión de navegación o colisión;
* decidir explícitamente si el cobertizo sigue siendo de madera, tierra o terreno abierto, en lugar de permitir que la huella del edificio implique que hay un piso allí;
* Mantenga sin cambios el contrato de techo físico estable; no agregue techos físicos como parte de esta revisión visual.

Si no existe una baldosa de madera adecuada, deténgase y agregue el requisito previo de catálogo/contenido más pequeño en lugar de conservar el hormigón en silencio mientras documenta el piso como terminado.

#### 10.8.4 Plano de mobiliario de la planta baja

Agregue una composición de mobiliario interior deliberada a `outpost_cabin`, dejando una ruta despejada desde la puerta hasta las escaleras y preservando los puntos de referencia de navegación necesarios. El primer conjunto de muebles debe incluir:

* un aparato para dormir o un soporte similar a una cama;
* una mesa o superficie de trabajo con uno o más asientos, si están disponibles;
* una estufa, chimenea, linterna o accesorio de supervivencia equivalente si existe un artículo en el catálogo que lo admita;
* Almacenamiento de suministros, herramientas o leña, utilizando cajas o estantes cuando sea apropiado;
* una pequeña cantidad de objetos abandonados o daños en lugar de llenar cada celda;
* No se permite colocar muebles en el umbral, el rellano de la escalera, la celda con altura libre superior, la abertura de la pared o los puntos finales de la ruta requerida.

Cada colocación debe usar un ID de mueble existente, una rotación válida, terreno compatible y reglas de un elemento por celda. Los anclajes genéricos de muebles solo deben agregarse para objetivos relevantes para el juego, como almacenamiento o estaciones de trabajo; los muebles decorativos no deben recibir anclajes semánticos redundantes.

El altillo solo debe albergar una pequeña zona para dormir y guardar cosas. El cobertizo debe conservar su apariencia distintiva como espacio cubierto para almacenamiento y trabajo, con espacio para la caja de herramientas existente y un acceso despejado al lateral de la cabaña.

#### 10.8.5 Vegetación y narración exterior

Utilice el mobiliario natural existente y el vocabulario de dispersión para que el campo parezca recuperado sin convertirlo en un bosque impenetrable:

* Coloca vegetación silvestre dispersa y objetos a la altura del césped alrededor del perímetro del mapa y más allá del jardín mantenido;
* Añade algunas rocas, tocones, arbustos o árboles para recrear el ambiente rural o boscoso del puesto de avanzada;
* concentrar el crecimiento excesivo cerca de los restos de la cerca, el exterior del cobertizo, los bordes del patio y las esquinas menos utilizadas;
* Deje despejados el camino de servicio, el acceso exterior, el umbral de la puerta, el interior de la cabina, las escaleras, el acceso al cobertizo y los puntos de prueba de navegación;
* Evite colocar vegetación sobre elementos existentes, superficies derivadas de la habitación, soportes de pared, pendientes o celdas reservadas;
* utilizar un entorno de ejecución limitado y solo para el exterior. `areas[].entities` paleta para la decoración general de la naturaleza, de modo que cada instancia del mapa varíe; conservar las operaciones explícitas de mobiliario solo para los elementos principales que deben permanecer estables;
* Distinga los elementos decorativos estáticos de la naturaleza de las entidades de área: utilice elementos deterministas creados por el autor cuando la composición visual deba permanecer estable, y utilice entidades de área solo cuando la variación en tiempo de ejecución sea intencional.

La densidad debe ajustarse mediante la inspección del editor, en lugar de un recuento global arbitrario. Registre las regiones protegidas seleccionadas, los pesos del área de ejecución y las celdas reservadas en pruebas específicas.

#### 10.8.6 Detalles del patio de trabajo y del abandono

Refuerza la narrativa del puesto avanzado con una composición exterior sobria:

* conservar los marcadores de valla existentes que estén parcialmente destruidos;
* agregar un pequeño grupo de trabajo/almacenamiento cerca del cobertizo sin bloquear el paso;
* Utilice parches de suelo desgastado para conectar el camino, la puerta, el área de trabajo y el área de almacenamiento;
* Agregue uno o dos elementos de utilería abandonados o dañados solo si las definiciones de mobiliario existentes lo permiten;
* Mantenga el patio legible desde arriba y evite que todas las celdas exteriores estén igualmente recargadas;
* preservar el `outpost_yard` la semántica del área independientemente de la variación visual de las baldosas y la disposición de los muebles.

Se trata de narración ambiental creada por autores, no de un nuevo sistema genérico de asentamientos o de composición de mapas generales.

#### 10.8.7 Orden de implementación

Se han completado todos los pasos de implementación previstos. La receta de producción mantenida, la receta de ejemplo, el mapa publicado, las pruebas del generador, las comprobaciones del validador, la regeneración determinista y la aceptación de la navegación en tiempo de ejecución siguen alineados. La aceptación final del contenido/editor se completó como parte del cierre de esta fase.

1. Inventaría los catálogos reales de azulejos y muebles para pisos de madera, camas, mesas, asientos, estufas/utensilios, almacenamiento, rocas, vegetación, arbustos, árboles, tocones y accesorios dañados. Registra los conceptos no disponibles y elige los activos existentes más similares.
2. ~~Agregue aserciones de prueba primero para el contrato de receta previsto: `Field` Clasificación, bordes de terreno, sin metadatos de carretera, celdas de caminos de servicio preservadas, celdas de suelo de madera, hormigón de umbral y espacios libres reservados.
3. ~~Añade la paleta de colores de suelo más pequeña y la variación de terreno determinista. Ejecuta las pruebas del generador enfocado e inspecciona el mapa generado antes de añadir muebles.~~
4. Sustituya el material de hormigón de la planta baja donde corresponda, respetando las excepciones de umbrales y escaleras. Valide el mapa generado de forma independiente.
5. ~~Añade la composición de mobiliario de la cabaña de la planta baja y cualquier añadido de altillo/cobertizo. Añade solo los anclajes de mobiliario relevantes para el juego y actualiza las aserciones de objetivo necesarias.~~
6. ~~Añade vegetación exterior delimitada y pertenencia al área de ejecución de objetos naturales, luego ajusta la densidad y las regiones protegidas mediante la inspección del editor.~~
7. ~~Añade un desgaste moderado propio de un patio de trabajo, desorden, relaciones entre vallas y vegetación excesiva, y cualquier objeto abandonado disponible.~~
8. ~~Regenera tanto el ejemplo mantenido como el mapa de producción a partir de sus respectivas recetas, compara el artefacto publicado completo e inspecciona el resultado en Godot/editor de contenido.~~
9. Ejecute la matriz de aceptación de navegación estructural, determinista, de validación, DMap y real. No marque esta fase como completada si algún paso de detalle bloquea una ruta existente o modifica el contrato de puerta canónica.

#### 10.8.8 Criterios de aceptación

Todos los criterios de aceptación que se detallan a continuación se cumplen para el puesto avanzado Pine Hollow. La implementación final incluye variación controlada del terreno, espacios de cabañas amueblados y navegables, elementos naturales en tiempo de ejecución, grupos de objetos de contenedores definidos, semántica de rutas y puertas preservada, validación de mapas generados, salida determinista y aceptación final del contenido y del editor.

La revisión detallada se ha completado porque se han cumplido todas las siguientes condiciones de aceptación:

* El terreno presenta una variación determinista visible pero controlada, más allá de una sola loseta de césped uniforme;
* La planta baja de la cabaña se ve claramente como de madera o se resuelve una limitación del catálogo documentada explícitamente;
* La planta baja contiene una composición de mobiliario intencionada y funcional;
* La vegetación y los elementos naturales son visibles en el terreno y alrededor del puesto de avanzada sin obstruir las rutas;
* El sendero de servicio sigue siendo a escala peatonal, continuo y distinto de la infraestructura vial;
* La entrada a la cabaña, el umbral, las escaleras, el altillo, el cobertizo, la caja de trabajo y el mobiliario requerido siguen siendo válidos;
* No se introducen sobrescrituras de características, identificadores no compatibles, rotaciones no válidas ni ubicaciones fuera de los límites;
* Las recetas de producción y de accesorios siguen siendo semánticamente equivalentes cuando se pretende, al tiempo que conservan sus identificadores únicos;
* el independiente `MapValidator` No se reportan errores en el mapa publicado;
* La regeneración de la producción es idéntica byte a byte al mapa publicado comprometido;
* El conjunto completo de herramientas de mapeo de Python pasa las pruebas;
* Paquetes GUT de Godot completos y enfocados, incluyendo real `Chunk`/`NavigationServer3D` Pruebas de recorrido y de pared bloqueada en Pine Hollow;
* La revisión manual de Godot/editor de contenido confirma que el puesto de avanzada se percibe como un lugar rural, desgastado por el tiempo, habitado y luego abandonado, en lugar de un rectángulo de prueba decorado.

#### 10.8.9 No objetivos explícitos

No incluya lo siguiente en este pase de detalles:

* un nuevo algoritmo genérico de mobiliario automático para habitaciones;
* vegetación automática inferida a partir de `Field` clasificación;
* un nuevo sistema de ubicación o proximidad en la carretera;
* integración de carreteras de mapa a mapa;
* Composición que abarca una ciudad, un asentamiento o un mapa general;
* generación de techo físico;
* planta alta generalizada `wall_height`.

### 10.9 Mapa de contenido mantenido del campamento de investigación del cráter: implementación completada

**Estado: implementación y aceptación automatizada completadas; revisión visual manual en el juego/editor de contenido pendiente**

`crater_research_camp` demuestra que el flujo de trabajo de la receta genérica puede enviar una ubicación de campo terminada sin agregar comportamiento de generador específico del mapa. El mapa utiliza `Field` y `Plains` categorías con `ground` en los cuatro bordes. Su composición de autor incluye un cráter rocoso de cuatro niveles de profundidad con una cuenca redondeada asimétrica y un borde irregular inspirado en `crater_small`, terrazas anidadas con respaldo sólido, cobertura de escombros de superficie expuesta mantenida fuera de la ruta de acceso y cuatro pendientes reversibles dispersas que conectan la superficie con z=-4, una cabaña de investigación metálica cerrada con una entrada canónica en el lado este y un techo de hormigón agrietado que se puede sostener explícitamente, un camino de servicio conectado, una plataforma de equipo exterior de hormigón, muebles científicos e industriales con rotaciones deliberadas, anulaciones de grupos de elementos de contenedores, accesorios de materiales peligrosos y celdas de navegación reservadas.

Las áreas de ejecución ofrecen una amplia variedad de terreno fuera del perímetro protegido del campamento/cráter, escasa vegetación natural, escombros rocosos en el fondo del cráter y una posible patrulla de robots de seguridad. La paleta de campo sigue el método probado `crater_small` Proporciones de hierba, tierra y flores; el cráter, la cabaña, el sendero y la plataforma de equipamiento siguen definidos explícitamente para que la selección de área en tiempo de ejecución no sobrescriba la geometría principal. Los recursos del catálogo existentes fueron suficientes, por lo que no se introdujeron nuevas definiciones de mosaicos ni de mobiliario. Una vista previa del mapa táctico de 128 × 128 píxeles acompaña al JSON publicado.

El conjunto de origen/salida mantenido es:

* `Tools/recipes/crater_research_camp.json` — fuente de información fidedigna para la producción;
* `Tools/examples/map_recipe_crater_research_camp.json` — accesorio estructuralmente equivalente mantenido con una identidad de ejemplo distinta;
* `Mods/Dimensionfall/Maps/crater_research_camp.json` y `.png` — Mapa de ejecución y vista previa publicados;
* `Tests/Unit/test_crater_research_camp_navigation.gd` - real `Chunk`/`NavigationServer3D` Aceptación de descensos y ascensos desde refugios hasta el borde del cráter.

El contrato de mapas generados garantiza la clasificación y las conexiones del mapa, la profundidad del cráter y la rotación de la pendiente, la propiedad de los edificios y los metadatos de entrada canónicos, las rotaciones exactas de los muebles focales y los grupos de elementos, las celdas de ruta despejadas protegidas, los grupos de entidades del área de ejecución, la paridad de producción/accesorios, la publicación determinista y la validación independiente.

## Fase 11 — Altura generalizada de los muros de plantas altas

**Estado: aplazado a la espera de un requisito concreto de jugabilidad en altura; la aceptación de la Fase 10 es positiva**

No reintroducir `wall_z`. Los pisos ordinarios siempre derivan paredes como `floor_z + 1`.

Cuando exista un requisito real de espacio alto, agregue el genérico más pequeño. `wall_height` contrato:

```json
{"z": 0, "wall_height": 2, "rooms": ["tall_hall"]}
```

Esto significa:

```text
floor z0
wall layers z1 and z2
roof surface z3
```

Restricciones requeridas:

* omitido `wall_height` medio `1`;
* es un número entero positivo;
* Cada capa de pared derivada permanece dentro de los límites lógicos;
* un tramo de pared termina justo debajo del siguiente piso ocupado;
* La superficie del techo se deriva de la capa más alta de la pared;
* Los límites de las paredes se materializan en cada capa de pared derivada;
* Los soportes permanecen en la planta principal y no se duplican verticalmente;
* Las aberturas de puertas/ventanas de varias capas se rechazan inicialmente a menos que exista un diseño explícito del tramo de la abertura.

Implemente esto solo con cobertura de prueba primero en `Tools/tests/test_map_generator.py`, luego validación del espejo en `Tools/map_validator.py` y `Scripts/Gamedata/DMap.gd`. Agrega un mínimo `Tools/examples/map_recipe_tall_storey_building.json` y una cobertura de navegación/colisión GUT específica antes de aplicarla a tierras de cultivo o cualquier otro mapa mantenido.

---

# Tarea siguiente recomendada

**Tarea siguiente recomendada: revisar manualmente Crater Research Camp en el editor de contenido/juego, luego definir un requisito de juego concreto para habitaciones altas antes de implementar la Fase 11 (`wall_height`).** Pine Hollow está completo, y Crater Research Camp tiene la implementación completa y la aceptación automatizada, quedando pendiente únicamente su recorrido visual. Después de esa revisión, el plan no tiene trabajo adicional específico de ubicación para ninguno de los mapas. La fase 11 es el siguiente hito definido del generador, pero debería posponerse hasta que una ubicación real o una característica de juego requiera pisos más altos que la regla actual de una sola capa de pared. Una vez que exista ese requisito, comience con la cobertura de prueba primero para el genérico `wall_height` contrato, luego actualizar el generador, validador independiente, saneamiento de DMap, una receta de ejemplo mínima y pruebas de navegación/colisión de Godot enfocadas.

Hasta que exista ese requisito, el mejor siguiente paso es una breve fase de descubrimiento/diseño: elegir el primer caso de uso concreto para habitaciones altas, definir el comportamiento de apertura de puertas/ventanas y la geometría prevista del techo/soporte, y solo entonces comenzar la implementación de la Fase 11. No reabra Pine Hollow ni genere pueblos/composición a nivel de mapa como sustituto; conserve las áreas establecidas a nivel de mapa, la semántica de las habitaciones, el comportamiento de los grupos de elementos en tiempo de ejecución y los contratos de los bordes del mapa.

Ejecuta el conjunto completo de Python, las pruebas o comprobaciones de humo relevantes de Godot, todas las generaciones de ejemplos mantenidas hasta `Tools/map_validator.py`, y `git diff --check` después del trabajo de implementación.

No realices confirmaciones ni envíos a menos que se te solicite explícitamente.

---

# Posición actual en una vista

```text
[Complete] Hermes-compatible repository guidance
[Complete] Existing map sanitation
[Complete] Fixed-size map validation
[Complete] Minimal JSON recipe format
[Complete] Deterministic 32×32 generator
[Complete] Base tile and rectangle overlays
[Complete] General tile-placement primitives
[Complete] Manual example generation and evaluation harness
[Complete] Generator and validator test suite
[Complete] Documentation and recognizable outdoor example
[Complete] Development moved to snipercup/CataX

[Complete] Palettes, reusable cell patterns, and deterministic variation
[Complete] Vertical-level schema and 3D placement foundations
[Complete] Structural generation, slope geometry, baked paths, and player traversal
[Complete] Feature and furniture schema investigation
[Complete] Level-aware single-cell furniture placement
[Complete] Deterministic conflict-aware outdoor furniture distribution
[Complete] Furnished-clearing editor and runtime verification
[Complete] Outdoor rock and wild-vegetation content definitions
[Complete] Dedicated AI-generated outdoor sprite replacement
[Complete] Final Phase 5 outdoor-asset inspection
[Complete] Level-aware runtime area schema and maintained example
[Complete] Catalog-validated runtime area entity variation
[Complete] Authored room semantics (`enclosed`, `covered_open`, `ruin`)
[Complete] Explicit room-to-exterior and room-to-room door-link metadata
[Complete] Authored existing-wall and connected-door boundary references
[Complete] Opt-in directional completeness validation for `enclosed` rooms
[Complete] Small single-level authored building footprint
[Complete] Authored roof/ceiling semantics for validated footprints
[Complete] Validated building-level composition constraints
[Complete] Authored building access-completeness validation
[Complete] Authored building interior classification constraints
[Complete] Authored building exterior/open-space classification constraints
[Complete] Validated building-level room partition constraints
[Complete] Validated building overhead-classification constraints
[Complete] Authored building-level external footprint context constraints
[Complete] Validated building-level exterior-access context constraints
[Complete] Authored building-level entrance semantics constraints
[Complete] Validated building-level entrance orientation and approach alignment
[Complete] Authored multi-entrance building semantics with per-entrance validation
[Complete] Authored furniture anchor metadata
[Complete] Authored map-edge connection metadata
[Complete] Authored road endpoint anchoring
[Complete] Authored road path routing metadata
[Complete] Multi-level building footprint foundation with intentional open gaps
[Complete] Per-floor room and furniture ownership metadata
[Complete] Two-slope staircase physical semantics at z 1 and z 2
[Complete] Straight and corner staircase formations with no slope stacking
[Complete] Per-floor ceiling/floor surface semantics (top-down)
[Complete] Multi-level roof/support semantics
[Complete] Opt-in physical building floors, wall boundaries, and supports
[Complete] Multi-level building navigation test through authored slope geometry
[Complete] Enclosed maintained building geometry with walls one level up and dirt below
[Complete] Manual player traversal of the maintained enclosed generated building
[Complete] Physical standable roof generation for the maintained multi-level building
[Complete] Wall and door placement decoupling via `target_at`/`room_at`
[Complete] Map-local road route painting between declared endpoints
[Complete] Runtime navigation validation for a painted local road route
[Deferred] Explicit map-to-map edge compatibility (resume if needed)
[Complete] Rooms and buildings
[Complete] Roads and map connections
[Complete] Multi-level reusable templates and richer composition
[Complete] Phase 9.1 generic semantic single-storey and two-storey building profiles
[Complete] Phase 9.2 independent validator and DMap preservation for generic profiles
[Complete] Phase 9.3 generic required-route/static reachability contract
[Complete] Phase 9.4 reusable Godot runtime navigation fixture
[Complete] Phase 9.5 apply generic semantics and runtime checks to `field_farmland`
[Complete] Phase 10.2 focused `field_farmland` runtime gameplay checks
[Complete] Phase 10.3 maintained-fixture generation, standalone validation, and deterministic regeneration acceptance suite
[Complete] Phase 10.4 runtime structural-quality validation and Phase 10 success criterion
[Complete] Phase 10.5 Pine Hollow Outpost automated and manual acceptance plus field-map placement conversion
[Complete] Phase 10.6 room-derived surface operations and room-owned multi-level geometry
[Complete] Phase 10.7 canonical doorway and entrance declarations
[Complete] Phase 10.8 Pine Hollow detail and immersion pass
[In review] Phase 10.9 Crater Research Camp maintained content map — automated acceptance complete; manual visual review pending
[Next, pending concrete requirement] Phase 11 generalized tall-storey `wall_height`
[Target] Agent-generated playable maps
```

Ya no se está experimentando con la posibilidad de generar mapas de forma segura. Esa parte ya está resuelta. El proyecto se encuentra ahora en la fase de ampliar el vocabulario del generador hasta que pueda expresar una ubicación real.

