# Generador de mapas basado en recetas

`Tools/map_generator.py` Convierte una receta JSON compacta en un archivo JSON de mapa de Dimensionfall completo. Los mapas de Dimensionfall tienen dimensiones fijas de 32 x 32 casillas y 21 posibles niveles lógicos. `z: -10` a través de `z: 10`Cada nivel poblado tiene exactamente 1024 entradas; los niveles no utilizados permanecen `[]`.

## Uso

Ejecutar desde la raíz del repositorio:

```sh
python3 Tools/map_generator.py \
  Tools/examples/map_recipe.json \
  /tmp/generated_meadow_prototype.json
python3 Tools/map_validator.py /tmp/generated_meadow_prototype.json
```

Las recetas utilizadas para los mapas de producción pueden estar bajo `Tools/recipes/`; los accesorios de esquema pequeños permanecen bajo `Tools/examples/`. Ambos utilizan el mismo contrato generador y validador. Los ejemplos de producción mantenidos incluyen Pine Hollow (`Tools/recipes/pine_hollow_outpost.json` → `Mods/Dimensionfall/Maps/pine_hollow_outpost.json`) y el Campamento de Investigación del Cráter (`Tools/recipes/crater_research_camp.json` → `Mods/Dimensionfall/Maps/crater_research_camp.json`). Los mapas publicados también necesitan un mismo ID `.png` Vista previa para la carga de Godot sin advertencias y miniaturas del navegador de mapas.

El nombre del archivo de salida debe ser `<id>.json`, que coincide con el ID derivado del nombre de archivo del cargador de mapas. La salida existente está protegida. Pasar `--overwrite` solo cuando se pretende un reemplazo. `--tiles PATH` para validar los ID de los mosaicos contra una base de datos de mosaicos distinta de `Mods/Dimensionfall/Tiles/Tiles.json`Las operaciones de mobiliario se validan según `Furniture/Furniture.json` junto a los seleccionados `Tiles` directorio por defecto; usar `--furniture PATH` cuando esa base de datos está en otro lugar.

Para la generación por lotes de variantes de semillas deterministas e instrucciones para inspeccionarlas en el editor de contenido de Godot, consulte [`map_example_generation.md`](map_example_generation.md).

## Formato de receta

La raíz debe ser un objeto JSON con los siguientes campos:

| Campo | Tipo | Significado |
| --- | --- | --- |
| `id` | cadena | ID del mapa usando solo letras, números, `_`, y `-`. |
| `name` | cadena no vacía | Nombre para mostrar. |
| `description` | cadena no vacía | Descripción del mapa. |
| `categories` | matriz de cadenas | Categorías de selección de mapa opcionales. Pine Hollow utiliza `["Field"]` Por lo tanto, participa como un mapa de campo genérico en lugar de un segmento de carretera.
| `seed` | entero | Semilla fija utilizada por las paletas de baldosas y muebles, rotaciones aleatorias, dispersión y celdas de patrones. Solo entrada de recetas; el formato del mapa no la almacena. |
| `base_tile` | objeto de mosaico | Mosaico de modo heredado colocado inicialmente en cada celda en `z: 0`. Obligatorio a menos que `levels` se utiliza. |
| `palette` | objeto | Conjuntos de mosaicos ponderados con nombre a los que pueden hacer referencia los objetos de mosaico. Opcional; por defecto es `{}`. |
| `furniture_palette` | objeto | Conjuntos de muebles ponderados con nombre utilizados por `furniture_scatter`. Opcional; por defecto es `{}`. |
| `areas` | matriz | Definiciones estrictas del área de tiempo de ejecución a las que hace referencia `area_rectangle`. Opcional; por defecto es `[]`. |
| `patterns` | objeto | Matrices con nombre de colocaciones relativas de mosaicos utilizadas por `pattern` operaciones. Opcional; por defecto es `{}`. |
| `regions` | matriz | Rectángulos rellenos heredados. Opcional; por defecto es `[]`. |
| `operations` | matriz | Operaciones de colocación ordenadas. Opcional; por defecto es `[]`. |
| `levels` | matriz no vacía | Definiciones de nivel agrupadas explícitas. No se puede combinar con la raíz `base_tile`, `regions`, o `operations`. |
| `connections` | objeto | Metadatos de conexión de borde de mapa creados por el autor. Opcional; por defecto, todos. `"ground"`. |
| `road_endpoints` | matriz | Anclajes de punto final de carretera creados en los bordes del mapa. Opcional; por defecto es `[]`. |
| `road_paths` | matriz | Rutas cardinales creadas entre puntos finales de carretera. Opcional; por defecto es `[]`; un camino puede pintar una ruta local del mapa con `tile`. |
| `templates` | objeto | Definiciones de plantillas reutilizables con relación `levels`, opcionalmente llamado `anchors`y definiciones de parámetros declaradas. Opcional; requiere `placements` cuando esté presente. |
| `placements` | matriz | Colocaciones de plantillas con un literal `origin` o alineación de anclaje a anclaje mediante `anchor_to` y `at_anchor`, además de la posibilidad de modificar parámetros opcionalmente. |
| `building_surfaces` | matriz | Semántica de superficie de piso, techo y cubierta definida por autor para cada edificio. Opcional; por defecto es `[]`. |
| `building_supports` | matriz | Rutas de soporte estructural multinivel creadas. Opcional; por defecto es `[]`. |

Una receta no define las dimensiones del mapa: todos los mapas generados son de 32 x 32. De arriba hacia abajo. `[x, y]` Las coordenadas comienzan en la esquina superior izquierda. Lógico `z` es elevación vertical, no un tercer elemento en `[x, y]`Cada forma debe encajar completamente dentro del mapa; las operaciones nunca se recortan silenciosamente.

Un objeto de baldosa tiene exactamente una de las siguientes características:

- `id`: un ID de mosaico sin procesar de la selección `Tiles.json`; o
- `palette`: el nombre de una paleta de nivel raíz.

También puede incluir una opción `rotation` al usar `id`La rotación es `0`, `90`, `180`, `270`, o `"random"`. Las fichas de operación pueden ser `null`, que escribe la representación de mosaico vacío del proyecto, `{}`.

Las entradas de la paleta son objetos con `id`, entero positivo opcional `weight` (por defecto `1`), y opcional `rotation`Los nombres de las paletas solo pueden contener letras, números, guiones bajos y guiones. La selección de paletas utiliza el mismo generador de números aleatorios con semilla que la dispersión, la dispersión de muebles y la rotación aleatoria, por lo que la misma receta completa y semilla producen resultados idénticos. Cambiar una receta de una ficha sin procesar a una referencia de paleta puede modificar las elecciones aleatorias posteriores, ya que la selección de paletas consume números aleatorios.

```json
{
  "palette": {
    "ground": [
      {"id": "grass_plain_01", "weight": 8, "rotation": "random"},
      {"id": "grass_flowers_00", "weight": 2, "rotation": "random"}
    ],
    "path": [
      {"id": "dirt_light_00", "weight": 3},
      {"id": "grass_dirt_00", "weight": 1, "rotation": "random"}
    ]
  },
  "base_tile": {"palette": "ground"}
}
```

Legado `regions` se aplican primero en orden de matriz, seguidos de `operations` en orden de matriz. Las celdas de patrón se expanden en su orden de definición en la posición de la operación invocadora. Las colocaciones de fichas posteriores en el mismo nivel lógico sobrescriben las celdas anteriores, incluyendo la completa `feature` de una baldosa amueblada anteriormente. Una operación de mobiliario rechaza en cambio una celda que ya tiene una característica. `regions` Se seguirán admitiendo las recetas de la versión uno y se utilizará la misma implementación de colocación de rectángulos rellenos que `rectangle` operaciones.

## conexiones de los bordes del mapa

El `connections` Los autores de campo definen los metadatos de conexión de los bordes del mapa: el tipo de terreno o carretera que se conecta en cada borde cardinal. El sistema utiliza estos valores para seleccionar las teselas de borde adecuadas cuando el mapa se instancia en el mundo principal.

```json
{
  "connections": {
    "north": "ground",
    "east": "road",
    "south": "ground",
    "west": "road"
  }
}
```

Cada llave es una de `north`, `east`, `south`, o `west`; cada valor es uno de `ground` o `road`Las instrucciones omitidas se establecen por defecto en `"ground"`, por lo que las recetas existentes sin una `connections` Los campos siguen siendo compatibles. El validador independiente rechaza las direcciones desconocidas y los tipos de conexión no compatibles.

Este campo solo genera metadatos: no crea teselas de carretera, rutas ni coloca las teselas de borde. Declara lo que el entorno de ejecución debe esperar en cada borde para que el generador del mapa general pueda conectar correctamente los mapas adyacentes.

Para una ubicación de campo como Pine Hollow, deje las cuatro conexiones de borde como `"ground"` y utilice un ordinario `line` Operación para un sendero peatonal/de servicio local. Ese sendero es geometría local del mapa y no implica que los vehículos puedan entrar al mapa desde un borde. La receta mantenida de Pine Hollow utiliza un corto `dirt_light_00` línea de `[8, 14]` a `[10, 14]`, finalizando en el enfoque exterior a nivel de conexión.

## Puntos finales de la carretera

El `road_endpoints` Los autores del mapa nombraron puntos de anclaje que indican dónde entran o salen las carreteras en sus bordes. Cada punto final es un punto de referencia que solo contiene datos; no genera teselas de carreteras, trazado de rutas ni geometría.

```json
{
  "connections": {"north": "ground", "east": "road", "south": "ground", "west": "road"},
  "road_endpoints": [
    {"id": "west_entrance", "direction": "west", "at": [0, 16], "z": 0},
    {"id": "east_exit", "direction": "east", "at": [31, 16], "z": 0}
  ]
}
```

Cada entrada tiene exactamente `id`, `direction`, `at`, y `z`. `id` Es único dentro de la receta y sigue el patrón de nomenclatura estándar. `direction` es uno de `north`, `east`, `south`, o `west` y debe coincidir con un `connections` entrada cuyo valor es `"road"` — un punto final de carretera no puede colocarse en un borde declarado como `"ground"`. `at` es un número entero de dos `[x, y]` coordenada que debe estar en el borde correcto del mapa para su `direction` (borde norte: `y = 0`borde sur: `y = 31`, borde oeste: `x = 0`, borde este: `x = 31`). `z` debe ser `0` — En esta primera sección, los extremos de las carreteras están únicamente a nivel del suelo.

El validador independiente comprueba de forma independiente el mismo contenido: rechaza direcciones desconocidas, ID duplicados, coordenadas que no están en el borde correcto y valores distintos de cero. `z` valores. `DMap` Conserva los puntos finales de carretera válidos mediante la función de guardar/cargar del editor y elimina las entradas con campos faltantes, direcciones no válidas, identificadores duplicados o coordenadas mal formadas.

`road_paths` autores una polilínea cardinal con nombre entre dos puntos finales de la carretera. Para una ruta física local del mapa, agregue una `tile` campo; el generador rasteriza cada celda de la ruta y reemplaza su terreno con esa tesela. Esto mantiene la geometría de la ruta explícita y determinista, evitando un segundo sistema de búsqueda de rutas.

```json
{
  "road_paths": [
    {
      "id": "west_to_east_road",
      "from": "west_entrance",
      "to": "east_exit",
      "waypoints": [],
      "tile": {"id": "dirt_light_00"}
    }
  ]
}
```

Cada camino requiere `id`, `from`, `to`, y `waypoints`; `tile` Es opcional para rutas de metadatos compatibles con versiones anteriores. Ambos ID de punto final deben existir y ser diferentes. `waypoints` es una matriz opcional de elementos delimitados por el mapa `[x, y]` coordenadas; la secuencia completa desde el `from` punto final a través de los puntos de ruta hacia el `to` El punto final debe formar segmentos alineados cardinalmente. Cuando `tile` está presente, debe hacer referencia a un valor conocido no vacío `Ground` En este modelo de teselas cúbicas, cada celda de ruta debe contener terreno de apoyo, las celdas de ruta no pueden contener elementos del terreno y el generador pinta la ruta completa a nivel del suelo. Esta estricta protección de teselas hace que la ruta generada sea adecuada para la geometría de navegación en tiempo de ejecución; la prueba de navegación de Godot verifica la conectividad real. El generador aún no ejecuta la búsqueda de rutas, no modifica la detección de colisiones ni la navegación, ni integra múltiples mapas. `DMap` conserva rutas válidas con o sin `tile` y elimina rutas con puntos finales obsoletos, puntos mal formados, identificadores duplicados, objetos de mosaico no válidos o segmentos no cardinales.

`Tools/examples/map_recipe_road_endpoints.json` muestra el ejemplo mantenido con dos puntos finales de carretera: `west_entrance` en el borde oeste y `east_exit` en el extremo este.

## Expansión mínima de la plantilla

Las plantillas se expanden antes de la validación y generación de recetas ordinarias. Una plantilla define la relativa `levels` y puede definir anclas con nombre como `{ "at": [x, y], "z": dz, "facing": "north" }`Los anclajes son relativos al origen de la ubicación y no se serializan en el mapa generado. Una ubicación puede usar un literal. `origin`o alinear su `at_anchor` a un anclaje en una colocación previa usando `anchor_to`.

```json
{
  "templates": {
    "small_cabin": {
      "parameters": {
        "floor_tile": {"type": "tile_id", "default": "concrete_00"},
        "wall_material": {"type": "enum", "values": ["brick", "metal"], "default": "brick"},
        "style": {
          "type": "object",
          "properties": {
            "support_tile": {"type": "tile_id", "default": "dirt_light_00"},
            "wall_tile": {"type": "tile_id", "default": "brick_wall_00"},
            "include_roof": {"type": "boolean", "default": true}
          }
        },
        "width": {"type": "integer", "minimum": 3, "maximum": 6, "default": 4},
        "height": {"type": "integer", "minimum": 3, "maximum": 6, "default": 4},
        "features": {"type": "string_list", "values": ["roof"], "default": ["roof"]}
      },
      "anchors": {
        "entrance": {"at": [0, 1], "z": 0, "facing": "west"},
        "exit": {"at": [4, 1], "z": 0, "facing": "east"}
      },
      "levels": [
        {"dz": 0, "operations": [
          {"type": "rectangle", "x": 0, "y": 0, "width": "$width", "height": "$height", "tile": {"id": "concrete_00"}}
        ]}
      ]
    }
  },
  "placements": [
    {"template": "small_cabin", "origin": {"at": [10, 10], "z": 0}, "rotation": 0},
    {
      "template": "small_cabin",
      "anchor_to": {"placement": 0, "anchor": "exit"},
      "at_anchor": "entrance",
      "parameters": {"style": {"wall_tile": "metal_wall_00", "include_roof": false}},
      "rotation": 0
    }
  ]
}
```

Las plantillas también pueden exponer rectángulos con nombre `locations` Para una composición tridimensional más rica:

```json
"locations": {
  "roof_pad": {"at": [0, 0], "z": 2, "width": 4, "height": 4}
}
```

Una ubicación raíz puede alinear una ubicación de igual tamaño con una ubicación anterior:

```json
{
  "template": "roof_module",
  "location_to": {"placement": 0, "location": "roof_pad"},
  "at_location": "roof_pad",
  "rotation": 0
}
```

Las ubicaciones resuelven su rectángulo local mediante rotación de colocación y `origin.z + location.z`Las dimensiones rotadas colocadas y referenciadas deben coincidir exactamente. La alineación de ubicación permite intencionalmente la superposición dentro del volumen de ubicación referenciado; las colocaciones raíz ordinarias aún rechazan huellas tridimensionales conflictivas. Las colocaciones conectadas por anclaje también permiten superposición con su colocación referenciada explícitamente, preservando la composición intencional de anclaje coincidente. Cada huella de colocación se deriva de manera conservadora de su conjunto resuelto, rectángulo, contorno, línea, patrón, dispersión, mobiliario, área, habitación y operaciones anidadas. Las celdas x/y/z fuera de límites y los conflictos no conectados se rechazan antes de la generación ordinaria.

Las plantillas anidadas se declaran con una plantilla local. `placements` matriz. Una colocación anidada utiliza `{ "template", "origin", "rotation", "parameters" }`; es `origin.at` y `origin.z` son relativos al origen de la plantilla contenedora, su rotación se suma a la rotación contenedora y sus valores de parámetros pueden hacer referencia a parámetros contenedores. La plantilla hija se expande recursivamente en operaciones raíz ordinarias. Las colocaciones anidadas utilizan intencionalmente orígenes locales literales en esta porción: no pueden utilizar `anchor_to`, `at_anchor`, o `rotation: "auto"`y los anclajes secundarios no se reexportan a través del elemento principal. Se rechazan los ciclos, las plantillas secundarias desconocidas, los orígenes locales mal formados, las rotaciones no compatibles, los parámetros reenviados no resueltos y el desbordamiento lógico-z.

Para la colocación de anclaje a anclaje, la colocación de referencia debe aparecer antes en el `placements` formación. `rotation` acepta `0`, `90`, `180`, o `270` grados en el sentido de las agujas del reloj alrededor del origen de la plantilla. Gira todas las coordenadas de operación horizontales locales, huellas rectangulares, puntos finales de línea, regiones de dispersión, patrones, anclas, ancla `facing`y rotaciones explícitas de baldosas/muebles/patrones; nunca cambia de forma relativa. `dz`. Colocar `rotation` a `"auto"` solo con `anchor_to` y `at_anchor` para derivar el cuarto de vuelta que hace que el ancla colocada quede frente al ancla de referencia. La rotación automática requiere que ambas anclas conectadas declaren cardinal. `facing` valores; resuelve el colocado `at_anchor` después de la rotación antes de alinearlo con el ancla anterior. Para colocar estructuras separadas una al lado de la otra, cree anclas de conexión en el borde exterior inmediatamente más allá de cada huella (por ejemplo, una plantilla de 4×4 puede usar oeste `[0, 1]` y este `[4, 1]` anclas); las anclas interiores producen intencionalmente una geometría superpuesta cuando coinciden.

Las plantillas pueden declarar tipificado `parameters`. `tile_id` valores sustituir exacto `$parameter_name` cadenas antes de la validación de operación normal y puede proporcionar una cadena `default`; sin una configuración predeterminada, son obligatorios en cada colocación. `boolean` Los valores pueden proporcionar un valor booleano. `default` y controlar una operación de plantilla con `"when": "$parameter_name"`. `enum` Los valores requieren una cadena única y no vacía. `values` matriz y pueden proporcionar un valor predeterminado de esa matriz; seleccionan variantes de operaciones semánticas con `"when": {"parameter": "material", "equals": "metal"}`. `integer` Los valores requieren un número entero inclusivo. `minimum` y `maximum` límites y puede sustituir coordenadas de operación, dimensiones o recuentos. `string_list` Los valores requieren una cadena única y no vacía permitida. `values` matriz; cada lista resuelta es única y puede estar vacía, y `"when": {"parameter": "features", "contains": "roof"}` incluye condicionalmente una operación. `object` Los valores requieren una declaración no vacía `properties` objeto; cada propiedad utiliza cualquier esquema de parámetros admitido, resuelve su propio valor predeterminado o requerido y rechaza los campos desconocidos. Utilice una referencia exacta con puntos, como por ejemplo: `$style.wall_tile` o `$style.include_roof` para sustituir un campo de objeto declarado. Las ubicaciones anulan los valores declarados a través de `parameters`. Anulaciones desconocidas, valores obligatorios faltantes, declaraciones mal formadas, valores de enumeración/lista no válidos, enteros fuera de rango, sin resolver `$` Se rechazan las referencias y las condiciones no válidas. Los identificadores de mosaico sustituidos siguen la misma validación de la base de datos de mosaicos que los identificadores de mosaico literales.

La ampliación da soporte a los ya existentes. `set`Formas de operación de rectángulo, contorno, línea, patrón, dispersión, muebles, área-rectángulo, habitación-rectángulo y muebles-dispersión. Rechaza operaciones anidadas. `z` valores, rotaciones no compatibles, rotación automática sin un par de anclajes conectados, colocaciones anidadas no válidas, ciclos de plantillas, ubicaciones mal formadas, dimensiones de ubicación incompatibles, plantillas o anclajes desconocidos, duplicado relativo `dz` niveles, referencias de ubicación/anclaje raíz hacia adelante, huellas tridimensionales fuera de límites y huellas conflictivas no conectadas. Las plantillas aún no se pueden combinar con la raíz agrupada. `levels` diseño. El mapa generado contiene únicamente la salida de recetas ordinarias ampliadas; `templates`, `placements`Los anclajes y las ubicaciones no están serializados.

`Tools/examples/map_recipe_small_cabin_template.json` es el ejemplo de plantilla-característica mantenida. Sus pruebas verifican la expansión determinista, relativa `dz` Colocación, composición de paneles de techo anidados con parámetros de estilo y dimensión transmitidos, alineación de anclaje a anclaje, rotación automática compatible con la orientación, anulaciones de objetos de estilo estructurados, dimensiones de huella enteras limitadas y una huella rotada de 5×3.

`Tools/examples/map_recipe_village_square_composition.json` es el ejemplo de composición local del mapa de la Fase 9 mantenido. Compone una plaza central con bancos anidados, anclajes de carretera expuestos y dos variantes de cabañas multinivel orientadas automáticamente. Genérelo con `python3 Tools/map_generator.py Tools/examples/map_recipe_village_square_composition.json Mods/Dimensionfall/Maps/generated_village_square_composition.json --overwrite`, luego valide el mapa generado con `python3 Tools/map_validator.py Mods/Dimensionfall/Maps/generated_village_square_composition.json`.
## Niveles lógicos

Índice de matriz de niveles del mapa `10` es lógico a nivel del suelo `z: 0`La conversión es:

```text
level-array index = logical z + 10
logical z = level-array index - 10
```

Los niveles lógicos válidos son números enteros de `-10` a través de `10`Los booleanos no son enteros para fines de recetas. Un nivel poblado contiene 1024 celdas de fila principal, mientras que un nivel que no contiene casillas no vacías se serializa como no utilizado. `[]`.

Existen dos modos de autoría mutuamente excluyentes.

### Diseño de raíz compatible con versiones anteriores

Las recetas existentes siguen utilizando raíz `base_tile`, `regions`, y `operations`. La baldosa base se rellena `z: 0`Cada región raíz y operación acepta opciones `z`; omisión significa `z: 0`Al seleccionar un nivel previamente no utilizado, ese nivel comienza como celdas vacías antes de aplicar la colocación.

```json
{
  "base_tile": {"id": "grass_plain_01"},
  "operations": [
    {
      "type": "pattern",
      "pattern": "wildflower_cluster",
      "at": [16, 12],
      "z": 1,
      "rotation": 90
    }
  ]
}
```

Raíz `regions` todavía se procesan antes de la raíz `operations`, incluso cuando las entradas apuntan a diferentes niveles lógicos. El consumo del generador de números aleatorios sigue este orden de procesamiento exacto.

### Diseño agrupado de varios niveles

Para mapas multinivel sustanciales, utilice una raíz `levels` matriz. Cada entrada requiere una lógica única. `z` y puede contener `base_tile`, `regions`, y `operations`La loseta base de la entrada, cuando está presente, llena ese nivel antes que sus regiones y operaciones. Sin una loseta base, el nivel comienza vacío.

```json
{
  "levels": [
    {
      "z": 0,
      "base_tile": {"id": "grass_plain_01"}
    },
    {
      "z": 1,
      "operations": [
        {
          "type": "rectangle",
          "x": 10,
          "y": 10,
          "width": 12,
          "height": 12,
          "tile": {"id": "grass_plain_01"}
        }
      ]
    }
  ]
}
```

Las entradas agrupadas se procesan en orden de declaración. Dentro de cada entrada, las regiones preceden a las operaciones. Este orden de declaración también define el consumo determinista de RNG en todos los niveles. Las regiones y operaciones anidadas heredan la entrada. `z` y no debe repetir un `z` campo. Raíz `base_tile`, `regions`, y `operations` no pueden coexistir con `levels`; esto rechaza el ordenamiento ambiguo de fusión y sobrescritura.

Las definiciones de patrones reutilizables siguen siendo bidimensionales. Una operación de patrón raíz selecciona su objetivo con `z`, mientras que una operación de patrón agrupado hereda el nivel que lo contiene. La composición de patrones y plantillas multinivel está fuera del esquema actual.

## Patrones celulares reutilizables

`patterns` Asigna un nombre a una matriz no vacía de posiciones relativas de mosaicos. Los nombres de los patrones siguen las mismas reglas de nomenclatura que las paletas. Cada celda tiene un número entero. `[x, y]` desplazamiento en `at` y un `tile`; los desplazamientos pueden ser negativos y los mosaicos pueden usar identificadores sin procesar, paletas, rotaciones o `null` Igual que las fichas de operación.

Las definiciones de patrones no consumen números aleatorios. Un patrón invocado consume aleatoriedad en el orden de la matriz de celdas solo cuando una celda utiliza una paleta o `"random"` Rotación de mosaicos. Por lo tanto, las definiciones no utilizadas no modifican la salida generada.

```json
{
  "patterns": {
    "wildflower_cluster": [
      {"at": [0, 0], "tile": {"palette": "flowers"}},
      {"at": [1, 0], "tile": {"palette": "flowers"}},
      {"at": [-1, 1], "tile": {"palette": "flowers"}}
    ]
  }
}
```

## Palets de muebles

`furniture_palette` asigna un nombre a una matriz ponderada no vacía de entradas de muebles conocidas. Las entradas tienen `id`, entero positivo opcional `weight` (por defecto `1`), y opcionalmente orientado al editor `rotation` (`0`, `90`, `180`, `270`, o `"random"`Los nombres de las paletas de muebles siguen la misma regla de letras, números, guiones bajos y guiones que las paletas de mosaicos. Cada entrada se valida con la base de datos de muebles seleccionada, incluidas las definiciones de paleta no utilizadas.

```json
{
  "furniture_palette": {
    "clearing_trees": [
      {"id": "Tree_00", "weight": 4, "rotation": "random"},
      {"id": "PineTree_00", "weight": 3, "rotation": "random"},
      {"id": "WillowTree_00", "weight": 1, "rotation": "random"},
      {"id": "burned_tree_stump", "weight": 2, "rotation": "random"}
    ]
  }
}
```

Las definiciones en sí mismas no consumen aleatoriedad. Una invocada `furniture_scatter` Primero, se muestrean las celdas elegibles, en orden de candidatos por fila, y luego se selecciona una entrada de la paleta y se resuelve cualquier rotación aleatoria para cada celda muestreada en el orden de muestreo. Esto forma parte del flujo de RNG de receta compartida: mover, agregar o eliminar una operación anterior que consume RNG puede modificar las selecciones posteriores.

## Operaciones de colocación

Cada operación requiere una `type`Los tipos de operación y campos desconocidos son errores. En la raíz de la receta, cada operación acepta valores lógicos opcionales. `z` y por defecto `0`Dentro de un nivel agrupado, la operación hereda `z` y debe omitir el campo.

## Áreas de tiempo de ejecución y membresía de mosaico

El sistema de áreas existente de Dimensionfall tiene dos formas serializadas vinculadas:

1. el nivel del mapa `areas` El array define una regla de generación/transformación en tiempo de ejecución; y
2. una loseta de terreno `areas` Los registros de la matriz son miembros de `{"id": "...", "rotation": 0}`.

En tiempo de ejecución, `Helper.map_manager.process_areas_in_map()` selecciona definiciones por `spawn_chance`Luego, aplica cada definición seleccionada a los grupos conectados de pertenencias a mosaicos coincidentes en cada nivel poblado. El editor muestra estas pertenencias y permite múltiples ID de área *diferentes* en un mismo mosaico, de modo que la generación de recetas conserva ese comportamiento de superposición compatible. Rechaza una pertenencia duplicada del mismo ID de área en la misma celda.

Las definiciones del área de recetas requieren exactamente `id`, `spawn_chance`, `rotate_random`, `pick_one`, `tiles`, y `entities`. Los identificadores de área utilizan letras, números, `_`, y `-`; `spawn_chance` es un número entero de `0` a través de `100`; `rotate_random` y `pick_one` son booleanos. `tiles` es una matriz no vacía de `{ "id", "count" }` entradas cuyos ID son mosaicos conocidos o el tiempo de ejecución `"null"` centinela. `entities` es una variedad de `{ "id", "type", "count" }` entradas. El generador solo acepta los tipos de entidades de tiempo de ejecución existentes.`furniture`, `mob`, `mobgroup`, y `itemgroup`—y valida cada ID con respecto a su catálogo principal antes de publicar un mapa.

```json
{
  "areas": [
    {
      "id": "meadow_clearing",
      "spawn_chance": 100,
      "rotate_random": false,
      "pick_one": false,
      "tiles": [{"id": "grass_dirt_00", "count": 1}],
      "entities": []
    }
  ]
}
```

### Variación de entidad en tiempo de ejecución

Una entidad de área es una regla de tiempo de ejecución, no una característica pregenerada. Para cada mosaico de membresía procesado, `map_manager` agrega una entrada implícita de no aparición ponderada por el total `tiles` peso, luego selecciona una entidad declarada por `count`Como resultado, el límite JSON generado es determinista, pero un campo que contiene una entidad puede diferir cada vez que se instancia el mapa. No reemplace esto con `furniture_scatter` Cuando se desea variación por instancia: scatter escribe intencionalmente las mismas características predefinidas en cada mapa generado.

Entidad `count` Los valores deben ser enteros positivos en las recetas. El generador aplica esta regla de autoría más estricta; el validador independiente acepta ponderaciones numéricas positivas heredadas para que los mapas creados existentes sigan coincidiendo con el contrato de selección ponderada del entorno de ejecución. Los tipos y catálogos de ID admitidos son:

| Tipo | Catálogo | Resultado de la función en tiempo de ejecución |
|---|---|---|
| `furniture` | `Furniture/Furniture.json` | `{ "type", "id", "rotation" }` |
| `mob` | `Mobs/Mobs.json` | `{ "type", "id", "rotation" }` |
| `mobgroup` | `Mobgroups/Mobgroups.json` | `{ "type", "id", "rotation" }` |
| `itemgroup` | `Itemgroups/Itemgroups.json` | `{ "type": "itemgroup", "itemgroups": ["id"], "rotation" }` |

Cuando `rotate_random` Si es falso, la entidad seleccionada utiliza la rotación visible en el editor de la membresía. Si es verdadero, el entorno de ejecución elige un cuarto de vuelta. La selección de entidades sigue siendo responsabilidad del entorno de ejecución y nunca es precalculada por el generador de Python.

```json
{
  "id": "stump_clearing",
  "spawn_chance": 100,
  "rotate_random": false,
  "pick_one": false,
  "tiles": [{"id": "grass_dirt_00", "count": 100}],
  "entities": [{"id": "burned_tree_stump", "type": "furniture", "count": 1}]
}
```

### `area_rectangle`

Agrega una membresía de área a cada casilla de terreno en un rectángulo relleno. Los campos son `type`, `area`, `x`, `y`, positivo `width`, positivo `height`, opcional a nivel raíz `z`y opcionalmente orientado al editor `rotation`. La rotación predeterminada es `0` y acepta `0`, `90`, `180`, o `270`.

```json
{
  "type": "area_rectangle",
  "area": "meadow_clearing",
  "x": 10,
  "y": 10,
  "width": 12,
  "height": 12,
  "z": 0,
  "rotation": 0
}
```

El área de nivel superior referenciada debe existir y cada celda objetivo debe contener terreno en el nivel lógico seleccionado. Una operación de pertenencia no reemplaza terreno, características u otras pertenencias de área distintas, y no consume RNG. Las coordenadas se verifican previamente contra el área completa. `32×32` mapa, por lo que la operación no puede recortarse ni aplicarse parcialmente.

## semántica de sala de autoría

Las habitaciones son etiquetas semánticas creadas; no se infieren del material del suelo, la topología de las paredes, la ubicación de las puertas o la cobertura del techo. Una receta puede definir una raíz. `rooms` matriz cuyas entradas contienen exactamente un `id` y `kind`:

```json
{
  "rooms": [
    {"id": "office", "kind": "enclosed"},
    {"id": "garage_bay", "kind": "covered_open"},
    {"id": "ruined_store", "kind": "ruin"}
  ]
}
```

Los tipos admitidos son:

| Tipo | Significado del autor |
|---|---|
| `enclosed` | Una habitación convencional destinada a ser cerrada físicamente una vez que se añade la generación de geometría. |
| `covered_open` Un espacio techado pero intencionalmente abierto, como un garaje sin puerta.
| `ruin` | Una habitación semántica que puede tener deliberadamente agujeros o secciones de pared abiertas. |

Usos de la membresía Tile `rooms` como una matriz de cadenas de un solo elemento, por ejemplo `{"id": "concrete_00", "rooms": ["garage_bay"]}`Una casilla de terreno puede tener exactamente una etiqueta de habitación, mientras que aún puede contener múltiples etiquetas de tiempo de ejecución. `areas` membresías. Esto hace que el comportamiento del material del piso y del área sea independiente de la semántica de la habitación: las habitaciones adyacentes pueden compartir un material de piso, y un área similar a un piso puede abarcar varias habitaciones.

### `room_rectangle`

Agrega una etiqueta de habitación creada por el autor a cada casilla de terreno en un rectángulo relleno. Los campos son `type`, `room`, `x`, `y`, positivo `width`, positivo `height`y nivel raíz opcional `z`No tiene rotación porque se trata de metadatos semánticos, no de una transformación espacial.

```json
{
  "type": "room_rectangle",
  "room": "garage_bay",
  "x": 12,
  "y": 7,
  "width": 10,
  "height": 8,
  "z": 0
}
```

La sala raíz referenciada debe existir. Cada celda objetivo debe contener terreno, y la pertenencia a la sala es exclusiva: una operación no puede superponerse a la etiqueta de otra sala ni aplicarse parcialmente. Una misma z `room_surface` (abajo) puede proporcionar ese terreno después de que se haya declarado la membresía. Las definiciones de sala y la membresía no consumen RNG.

### `room_surface`

Materializa una casilla a través de la membresía final creada de una habitación. Está deliberadamente separado de las operaciones de coordenadas genéricas: use `rectangle` para terrenos, caminos, patios y aleros intencionales; usar `room_surface` cuando un piso o techo pertenece a una habitación.

```json
{"type": "room_surface", "room": "outpost_loft", "z": 2, "tile": {"id": "concrete_00"}}
{"type": "room_surface", "room": "outpost_loft", "z": 4, "tile": {"id": "concrete_00"}, "outline_padding": 1}
```

Los campos son exactamente `type`, `room`, nivel raíz `z`, `tile`y opcional `outline_padding`. La sala debe haber creado la membresía en un único nivel lógico. `outline_padding` por defecto `0`; un valor no negativo expande cada celda miembro por ese radio de Chebyshev, produciendo un relleno de una celda `8×8` Envolvente del techo alrededor de Pine Hollow `6×6` loft sin nombrar un rectángulo duplicado. Las celdas derivadas deben permanecer dentro del mapa.

Las superficies de las habitaciones se posponen hasta que cada operación de diseño haya pintado la pertenencia a las habitaciones, por lo que su resultado es independiente del orden de `room_rectangle` Operaciones. Se conservan los metadatos de tiempo de ejecución existentes. Las características y las teselas de pendiente creadas siguen siendo válidas cuando se materializa una superficie de habitación con el mismo eje Z debajo de ellas. La operación valida habitaciones desconocidas, teselas mal formadas o desconocidas, relleno no válido, pertenencia a habitaciones faltante, pertenencia a habitaciones de varios pisos y celdas derivadas fuera de los límites.

Esta rebanada conserva `rooms` a través de `DMap`/content-editor guarda y carga rutas y valida la salida del mapa generado de forma independiente. `room_surface` es una sintaxis de autoría exclusiva de recetas y no se serializa en mapas generados, por lo que no requiere ningún campo de persistencia DMap nuevo.

### `room_connections`

`room_connections` acepta los campos existentes `id`, `at`, `target_at`, `z`, `from`, y `to`, además del nivel de conexión opcional `entrance` objeto. La implementación de la fase 10.7 y las pruebas específicas ahora cubren este objeto. Es válido solo en conexiones de habitación a exterior y debe definirse exactamente `exterior_at` y `facing`Ambas coordenadas deben estar dentro de los límites y la orientación debe ser cardinal. Los datos de conexión generados conservan el objeto validado. Los metadatos de entrada de edificios heredados que presenten conflictos se rechazan durante el período de compatibilidad.

El formulario admitido es:

```json
{
  "id": "office_front_door",
  "at": [8, 8],
  "target_at": [7, 8],
  "z": 0,
  "from": {"kind": "room", "id": "office"},
  "to": {"kind": "exterior"},
  "entrance": {"exterior_at": [6, 8], "facing": "east"}
}
```

El formulario de nivel de conexión está implementado y ahora se puede utilizar para nuevas recetas. Las recetas existentes pueden seguir utilizando el formulario de nivel de edificio establecido. `entrance`/`entrances`, `exterior_context`, y `exterior_access_context` campos durante la migración. No elimine los anclajes de puerta heredados ni los registros de entrada/contexto hasta que se haya establecido una cobertura de equivalencia exacta.


```json
{
  "room_connections": [
    {
      "id": "office_front_door",
      "at": [11, 10],
      "z": 0,
      "from": {"kind": "room", "id": "office"},
      "to": {"kind": "exterior"}
    },
    {
      "id": "office_to_garage",
      "at": [12, 10],
      "z": 0,
      "from": {"kind": "room", "id": "office"},
      "to": {"kind": "room", "id": "garage_bay"}
    }
  ]
}
```

`id` Debe ser único y utilizar los caracteres normales del nombre de la definición. `at` es exactamente `[x, y]` dentro del `32×32` mapa y `z` es un nivel lógico requerido desde `-10` a través de `10`Los puntos finales utilizan una de dos formas exactas:

```json
{"kind": "room", "id": "office"}
{"kind": "exterior"}
```

Los ID de las habitaciones deben existir en la raíz. `rooms`; los dos puntos finales deben ser distintos; y una coordenada de puerta puede tener solo una conexión. La casilla de destino ya debe existir en el nivel declarado y contener una característica de mobiliario con capacidad de puerta reconocida por el catálogo; por ejemplo, `door_wood`, cuya implementación en tiempo de ejecución existente posee el estado abierto/cerrado y el comportamiento de colisión.

Las conexiones son metadatos semánticos únicamente. Conservan su autoría. `from`/`to` Ordene, pero no cree una nueva función de puerta, altere el comportamiento de la puerta existente, infiera el cerramiento, genere geometría o introduzca un estado de ejecución interior/exterior. `DMap` Conserva los enlaces válidos mediante la función de guardar/cargar del editor de contenido y elimina los enlaces que hacen referencia a salas eliminadas.

### `room_boundaries`

`room_boundaries` adjunta explícitamente una baldosa de pared física existente o una abertura de puerta conectada a una habitación creada. No deriva un perímetro de las habitaciones, el material del suelo, la rotación de la pared o la geometría vecina. Un registro heredado/parcial contiene `id`, `room`, `at`, `z`, y `element`; puede añadir `side` cuando se necesita evidencia direccional:

```json
{
  "room_boundaries": [
    {
      "id": "office_north_wall",
      "room": "office",
      "at": [8, 7],
      "z": 0,
      "element": "wall_tile",
      "side": "south"
    },
    {
      "id": "office_front_opening",
      "room": "office",
      "at": [11, 10],
      "z": 0,
      "element": "door_furniture"
    }
  ]
}
```

`id` es único y utiliza caracteres normales de definición-nombre. `room` debe nombrar una sala raíz, `at` es un límite `[x, y]`, y `z` es un nivel lógico requerido desde `-10` a través de `10`. `element` es exactamente uno de:

| Elemento | Requisito objetivo existente |
|---|---|
| `wall_tile` | El ID del terreno objetivo está en el catálogo de mosaicos y tiene el `Wall` categoría; debe ser cardinalmente adyacente a una casilla etiquetada con la habitación nombrada. |
| `door_furniture` | La loseta de terreno objetivo contiene muebles con capacidad de puerta reconocidos por el catálogo y un `room_connections` entrada al mismo tiempo `[x, y, z]` Nombra la habitación.

Una habitación no puede tener la misma coordenada de destino dos veces, pero una misma baldosa de pared puede delimitar deliberadamente diferentes habitaciones mediante registros separados. `side`, cuando está presente, es `north`, `east`, `south`, o `west`. Para `wall_tile`, apunta desde el azulejo de la pared hacia su habitación; para `door_furniture`, apunta hacia afuera desde el azulejo de la habitación a través de esa puerta.

Un `enclosed` La habitación puede optar por la integridad estricta añadiendo `"boundary_validation": "complete"` a su definición raíz. El modo explícito heredado requiere que cada borde cardinal expuesto de cada baldosa de habitación etiquetada tenga exactamente una dirección `room_boundaries` registro. Por lo tanto, sus registros requieren `side`: una pared debe apuntar hacia el borde de la habitación correspondiente, y una puerta debe comenzar en la baldosa de la habitación correspondiente y conservar su posición requerida. `room_connections` Punto final. Se rechazan los registros que faltan, están mal orientados, duplicados o no expuestos. `covered_open` y `ruin` Las salas no pueden optar por participar y seguir permitiendo evidencia parcial de límites.

Para un recinto generado normal, agregue `"boundary_generation": "walls"` junto a `boundary_validation` en lugar de enumerar cada pared:

```json
{
  "id": "kitchen",
  "kind": "enclosed",
  "boundary_validation": "complete",
  "boundary_generation": "walls"
}
```

Este modo requiere un edificio propio con `building_geometry.wall_tile`. Después de pintar la membresía final de la habitación, el generador obtiene un anillo de pared de una baldosa en `floor_z + 1`, incluyendo tapas de esquina diagonales. Cada mismo z `room_connections` Un enlace que cruza el perímetro de una habitación generada se convierte en una abertura; los enlaces entre habitaciones abren el borde frontal para ambas habitaciones, manteniendo el objetivo de puerta-mueble creado. Los cruces de conexión deben ser cardinales. Las habitaciones adyacentes generadas sin conexión comparten exactamente una pared divisoria, colocada de forma determinista en la celda de pertenencia del ID de habitación lexicográficamente superior. Explícito `room_boundaries` Las solicitudes para una habitación generada se rechazan, por lo que una receta no puede mezclar modos de propiedad de pared en competencia. El validador independiente comprueba de forma independiente el anillo de pared y las aberturas resultantes con respecto a la baldosa de pared declarada del edificio. `room_rectangle` Sigue siendo solo para miembros; no coloca muros directamente.

Los soportes de pared de la planta baja generados utilizan `dirt_light_00` Conservando los metadatos existentes de habitaciones, áreas y baldosas compatibles. La generación de límites no crea herrajes para puertas, no infiere conexiones, no modifica la navegación ni añade efectos de interior/exterior.

### `buildings`

`buildings` Los grupos crearon evidencia de la habitación en una huella de edificio rectangular explícita. Por defecto, permanece solo como metadatos. Una opción de participación `building_geometry` El registro genera pisos físicos, baldosas de límite de pared y baldosas de soporte personalizadas, conservando al mismo tiempo las características existentes de las puertas y las pendientes de las escaleras:

```json
{
  "buildings": [
    {
      "id": "office_building",
      "rooms": ["office"],
      "footprint": {"x": 7, "y": 7, "width": 4, "height": 4},
      "z": 0
    }
  ]
}
```

Cada registro tiene requerido `id`, `rooms`, `footprint`, y `z`, más opcional `building_geometry`, `building_levels`, `staircases`, `access_validation`, `interior_rooms`, `open_space_rooms`, `room_partition_validation`, `overhead_validation`, `exterior_context`, `exterior_access_context`, `entrance`, `entrances`, `entrance_validation`, y `furniture_anchors`. `id` es único; `rooms` es una lista no vacía de identificadores de sala únicos conocidos; `footprint` tiene exactamente un número entero no negativo `x`/`y` más un número entero positivo `width`/`height`, completamente dentro del mapa de 32×32; y `z` es un nivel lógico requerido desde `-10` a través de `10`Las huellas en la misma coordenada lógica z no deben superponerse, aunque pueden tocarse.

Cada habitación propia debe tener membresía solo a nivel del edificio y completamente dentro de su área. Al menos una habitación propia debe ser una `enclosed` habitación con `"boundary_validation": "complete"`. Su mismo nivel `room_boundaries` y cualquier nivel del mismo `room_connections` Nombrarlo también debe estar dentro de la huella. Sin `building_geometry`, esto sigue siendo un contrato estricto de propiedad/contención; con él, los niveles declarados ocupados de la huella reciben un terreno de piso generado.

`building_geometry` tiene exactamente estos campos:

```json
{
  "building_geometry": {
    "floor_tile": {"id": "concrete_00"},
    "wall_tile": {"id": "brick_wall_00"},
    "support_tile": {"id": "brick_wall_00"},
    "roof_tile": {"id": "concrete_00"}
  }
}
```

`floor_tile` y `roof_tile` debe ser conocido `Ground`, `Floor`, o `Urban` baldosas; `wall_tile` y `support_tile` debe ser conocido `Wall` o `Ground` baldosas. El generador llena cada nivel de edificio declarado ocupado, se materializa `wall_tile` límites un nivel lógico por encima de su límite z, se materializa `support_tile` en `building_supports.from_z`y se materializa `roof_tile` en la superficie del techo creada z. Cada coincidencia creada `wall_tile` a nivel de la planta baja de un edificio recibe `dirt_light_00` debajo de ella; los límites elevados de la pared conservan su piso. `floor_tile` debajo de ellos. Se conservan las características existentes y las baldosas de pendiente, las coordenadas de pendiente de la escalera inferior reservan espacio libre en el piso z2 directamente encima, y ​​la geometría generada rechaza las sobrescrituras de características. `upper_clearance_at`Cuando se crea en una escalera, elimina esa coordenada en z2 para que un acceso excepcional a la escalera pueda mantener el espacio libre para el jugador. No hay una propiedad de elevación de pared independiente; una futura `wall_height` Esta función se pospone hasta que se requieran pisos realmente altos. Las puertas y las pendientes de las escaleras siguen siendo operaciones definidas por el usuario; la detección de colisiones y la navegación se verifican mediante pruebas de Godot en lugar de inferirse con Python.

Un edificio puede declarar una base de planta de varios niveles con `building_levels`:

```json
{
  "id": "office_building",
  "rooms": ["office"],
  "footprint": {"x": 7, "y": 7, "width": 4, "height": 4},
  "z": 0,
  "building_levels": [{"z": 0}, {"z": 2}]
}
```

`building_levels` es una matriz opcional no vacía y estrictamente ascendente de objetos con requisitos `z` y opcional `rooms` y `furniture_anchors` matrices. El registro del edificio en sí permanece arraigado en el nivel del suelo. `z: 0`Los niveles de piso declarados como ocupados deben ser niveles lógicos pares: `z: 0` es la planta baja, `z: 1` es una brecha abierta intencional, `z: 2` es el nivel del techo/primer piso, `z: 3` es otra brecha abierta, y así sucesivamente. El rango actualmente admitido es `0` a través de `10`. Los muros se derivan en `floor_z + 1`; no agregue un campo de elevación de pared independiente. Un futuro `wall_height` La propiedad se pospone hasta que se necesiten pisos realmente altos. La declaración debe comenzar con `{"z": 0}`; los niveles impares no son declaraciones de pisos ocupados. Una declaración como `[{"z": 0, "rooms": ["office"], "furniture_anchors": ["desk_anchor"]}, {"z": 2, "rooms": ["office_upper"], "furniture_anchors": ["upper_bench_anchor"]}]` asigna cada habitación y anclaje de muebles a un piso ocupado. Cuando se utiliza cualquiera de las listas de propiedad, debe asignar cada habitación o anclaje del edificio agregado exactamente una vez. Una declaración como `[{"z": 0}, {"z": 2}, {"z": 4}]` Sigue siendo válida como una base exclusivamente vertical sin listas de propiedad por piso.

Sin `building_geometry`, esta es una fundación que solo utiliza metadatos. Con `building_geometry`El generador crea losas físicas de piso, límites de pared, soporte y techo para los niveles de edificio declarados. Sin embargo, no genera techos, puertas, escaleras, transiciones verticales, estado de ejecución en interiores ni comportamiento en tiempo de ejecución más allá de la geometría Chunk existente para las losas emitidas. `DMap` Conserva las declaraciones válidas mediante guardar/cargar y elimina las declaraciones mal formadas. Su cargador JSON compartido normaliza los literales numéricos enteros a enteros antes de la sanitización, conservando las coordenadas, los niveles z, las rotaciones y otros contratos enteros creados con JSON; los valores no enteros permanecen como números de coma flotante.

Un edificio puede declarar escaleras con `staircases`Las escaleras son semánticas de transición física creadas para un edificio de dos plantas: se necesitan exactamente dos bloques de pendiente para ascender desde la planta baja hasta la primera planta. Las dos pendientes nunca se apilan; la pendiente superior debe estar desplazada horizontalmente con respecto a la pendiente inferior. Ambas baldosas de pendiente deben ser baldosas cuya base de datos de baldosas `shape` es `"slope"` (Por ejemplo `grass_ramp_00`, `wood_stairs`, o `dirt_light_ramp_00`), colocado en `z: 1` (inferior) y `z: 2` (superior). Las escaleras requieren niveles de construcción declarados `z: 0` y `z: 2`.

**Las escaleras rectas** utilizan dos pendientes cardinalmente adyacentes:

```json
{
  "building_levels": [{"z": 0}, {"z": 2}],
  "staircases": [
    {"id": "office_staircase", "lower_at": [9, 9], "upper_at": [10, 9], "rotation": 90}
  ]
}
```

La pendiente inferior en `lower_at` en `z: 1` y la pendiente superior en `upper_at` en `z: 2` son cardinalmente adyacentes y ambos utilizan el mismo editor. `rotation` por lo tanto, la pendiente continúa en la misma dirección.

**Escaleras de esquina** insertar un bloque de rellano plano en `z: 1` entre las dos pendientes, de modo que la pendiente superior está desplazada diagonalmente con respecto a la pendiente inferior y gira en la esquina:

```json
{
  "building_levels": [{"z": 0}, {"z": 2}],
  "staircases": [
    {"id": "corner_staircase", "lower_at": [10, 10], "upper_at": [11, 9], "landing_at": [11, 10], "rotation": 90, "upper_rotation": 0}
  ]
}
```

El bloque de aterrizaje en `landing_at` en `z: 1` debe ser una baldosa existente plana (sin pendiente), cardinalmente adyacente a ambos `lower_at` y `upper_at`. `upper_rotation` es opcional y por defecto es `rotation`; es la rotación orientada hacia el editor de la pendiente superior, que puede girar en la esquina. El mantenido `Tools/examples/map_recipe_multi_level_building_foundation.json` muestra ambas formaciones.

Cada registro de escalera tiene una característica única. `id` y requerido `lower_at`, `upper_at`y orientado al editor `rotation`Todas las coordenadas deben estar dentro del perímetro del edificio. `upper_clearance_at` es opcional y borra una coordenada de autor en z2 para un acceso de escalera excepcional; no agrega una pendiente ni un rellano. Los metadatos de la escalera aún no generan teselas de pendiente; valida la evidencia de pendiente existente. Cuando se combina con `building_geometry`Los pisos y soportes se generan alrededor de esas pendientes creadas, mientras que las pruebas de tiempo de ejecución de pendientes existentes siguen siendo responsables del comportamiento de la malla, la colisión, la navegación y el recorrido.

Un edificio puede optar por la integridad del acceso autorizado con `"access_validation": "complete"`:

```json
{"id": "office_building", "rooms": ["office"], "footprint": {"x": 7, "y": 7, "width": 4, "height": 4}, "z": 0, "access_validation": "complete"}
```

Para ese edificio, cada habitación propia debe tener un camino hacia `exterior` a través de same-z autorizado `room_connections`Una conexión entre una habitación y el exterior inicia la ruta; una conexión entre habitaciones la propaga solo cuando ambas habitaciones pertenecen al mismo edificio. Los enlaces a habitaciones fuera del edificio no cumplen esta regla. Esto valida el acceso semántico, no la transitabilidad física: no infiere la adyacencia de puertas, la rotación, la colisión, la navegación ni el recorrido del jugador.

Un edificio puede clasificar explícitamente un subconjunto no vacío de sus habitaciones de propiedad como interior con `interior_rooms`:

```json
{"id": "office_building", "rooms": ["office"], "footprint": {"x": 7, "y": 7, "width": 4, "height": 4}, "z": 0, "interior_rooms": ["office"]}
```

Cada habitación interior declarada debe ser una habitación única, propia y conocida cuyo tipo de autor es `enclosed` y cuyo `boundary_validation` es `complete`El contrato nunca infiere interiores a partir del material del suelo, paredes, puertas, enlaces de acceso, registros de techo/cielo o geometría. Excluye deliberadamente `covered_open` y `ruin`; aquellos conservan sus significados semánticos originales sin ser clasificados como interiores.

Un edificio puede clasificar explícitamente un subconjunto no vacío de sus habitaciones de propiedad como espacio abierto con `open_space_rooms`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "interior_rooms": ["office"], "open_space_rooms": ["garage_bay"]}
```

Cada espacio abierto declarado debe ser único, propiedad de, conocido y creado como `covered_open` o `ruin`No debe superponerse. `interior_rooms`Esto hace explícito un garaje techado o abierto, o un anexo dañado, sin tratarlo como un espacio interior. El contrato no infiere la existencia de espacio abierto a partir de la falta de paredes, techos, accesos exteriores, material del terreno o geometría, y tampoco genera ni modifica aberturas físicas.

Un edificio puede optar por una partición de habitación completamente personalizada con `room_partition_validation: "complete"`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "interior_rooms": ["office"], "open_space_rooms": ["garage_bay"], "room_partition_validation": "complete"}
```

Para un edificio que ha optado por participar, cada habitación de propiedad debe aparecer exactamente una vez en `interior_rooms` y `open_space_rooms`Las reglas de clasificación existentes proporcionan las garantías de coincidencia exacta: las listas no pueden superponerse, los interiores deben estar completos. `enclosed` Las habitaciones y los espacios abiertos deben ser `covered_open` o `ruin`Esto es opcional y solo valida las etiquetas creadas por el usuario; no requiere que cada celda de la huella pertenezca a una habitación y no infiere una clasificación a partir de la geometría o el comportamiento en tiempo de ejecución.

Un edificio puede optar por una clasificación completa de gastos generales con `overhead_validation: "complete"`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "overhead_validation": "complete"}
```

El edificio debe entonces tener ambos derechos de autor exactamente válidos `roof` y `ceiling` `building_surfaces` en lógica `z: 1` (`building.z + 1`La validación de superficie existente sigue requiriendo que esos registros nombren el edificio, tengan sus propios identificadores/tipos únicos y se ubiquen exactamente en ese nivel superior. Esta regla opcional valida únicamente las clasificaciones: no crea geometría de techo ni cielo raso, no requiere cobertura física, no marca celdas ocupadas ni interiores, no implica soporte/colisión ni altera el comportamiento de iluminación/clima/tiempo de ejecución.

Un edificio puede generar una coordenada de contexto de huella externa que solo contenga datos:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "exterior_context": {"at": [6, 8], "z": 0}}
```

`exterior_context` tiene exactamente `at` y `z`Debe apuntar a una casilla de terreno existente en la propia z lógica del edificio, estar fuera de la huella, ser cardinalmente adyacente a una de sus celdas de huella y no pertenecer a ninguna habitación. Proporciona un punto de referencia externo creado explícitamente sin reutilizar el tiempo de ejecución. `areas`, infiriendo calles/patios o modificando el terreno. No genera una casilla, no requiere un material de terreno específico, no reserva la celda, no crea una entrada ni altera el comportamiento en tiempo de ejecución.

Un edificio puede asociar explícitamente ese contexto externo con una conexión de acceso semántico existente:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "exterior_context": {"at": [6, 8], "z": 0}, "exterior_access_context": {"connection": "office_front_door"}}
```

`exterior_access_context` tiene exactamente `connection`Requiere `exterior_context` y nombra un mismo-z existente `room_connections` registro de una de las habitaciones propiedad del edificio cuyo otro punto final es `exterior`La asociación es explícita; no infiere una ruta física, adyacencia de coordenadas, rotación de puerta, colisión, navegación o recorrido del jugador entre la casilla de contexto y la puerta referenciada.

Un edificio puede crear una entrada semántica de solo datos con `entrance`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "exterior_context": {"at": [6, 8], "z": 0}, "exterior_access_context": {"connection": "office_front_door"}, "entrance": {"connection": "office_front_door", "facing": "east"}}
```

`entrance` tiene exactamente `connection` y `facing`Requiere `exterior_context` y nombra un mismo-z existente `room_connections` registro de una de las habitaciones propiedad del edificio cuyo otro punto final es `exterior`, siguiendo el mismo contrato que `exterior_access_context.connection`. Cuando `exterior_access_context` también está presente, `entrance.connection` debe coincidir. `facing` es uno de `north`, `east`, `south`, o `west` y debe señalar desde `exterior_context.at` Hacia la huella del edificio: al avanzar una celda en la dirección de la vista desde la coordenada de contexto, se debe aterrizar dentro de la huella. Esto valida únicamente la orientación de entrada definida por el autor; no genera una puerta, no infiere una ruta física, no requiere adyacencia de coordenadas entre la celda de contexto y la puerta de referencia, ni altera la colisión, la navegación, la iluminación, el clima ni el comportamiento en tiempo de ejecución.

Un edificio puede crear múltiples semánticas de entrada solo de datos con `entrances`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "exterior_context": {"at": [6, 8], "z": 0}, "exterior_access_context": {"connection": "office_front_door"}, "entrances": [{"id": "front_entrance", "connection": "office_front_door", "facing": "east"}, {"id": "garage_entrance", "connection": "garage_opening", "facing": "west"}], "entrance_validation": "complete"}
```

`entrances` es una matriz no vacía de registros de entrada. `entrance` y `entrances` son mutuamente excluyentes. Cada entrada tiene exactamente `id`, `connection`, y `facing`. `id` Es único dentro del edificio y sigue el mismo patrón de nomenclatura que otros identificadores creados por el usuario. `connection` nombra un mismo-z existente `room_connections` registro de una de las habitaciones propiedad del edificio cuyo otro punto final es `exterior`, siguiendo el mismo contrato que el singular `entrance.connection`Cada conexión puede ser referenciada por un máximo de una entrada. `facing` es uno de `north`, `east`, `south`, o `west`. `entrances` requiere `exterior_context`. Cuando `exterior_access_context` también está presente, su `connection` debe coincidir con una de las conexiones de entrada.

La entrada principal es aquella cuya `connection` partidos `exterior_access_context.connection`, o la primera entrada cuando `exterior_access_context` está ausente. Solo se comprueba la entrada principal. `exterior_context.at`: dando un paso en una celda en su `facing` La dirección desde la coordenada de contexto debe aterrizar dentro de la huella. Las entradas no principales no se comprueban contra `exterior_context` porque una única ficha de contexto no puede servir para múltiples direcciones de acceso.

Un edificio puede optar por la validación completa de la orientación y el acceso a la entrada con `entrance_validation: "complete"`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "exterior_context": {"at": [6, 8], "z": 0}, "exterior_access_context": {"connection": "office_front_door"}, "entrance": {"connection": "office_front_door", "facing": "east"}, "entrance_validation": "complete"}
```

Para un edificio que ha optado por participar, `entrance_validation` requiere `entrance` o `entrances` y luego verifica dos restricciones de autor por entrada:

1. **Orientación de la puerta**: la `room_boundaries` registro con `element: "door_furniture"` en la conexión de entrada `at` y `z`, nombrando una habitación propiedad del edificio, debe tener una `side` igual a `OPPOSITE_SIDES[facing]`—la puerta se abre hacia la dirección de aproximación. Falta un límite de puerta, falta `side`o la puerta que no está orientada correctamente es rechazada.
2. **Alineación de acceso** (solo entrada principal): `exterior_context.at` y la conexión de la entrada principal `at` ambos deben caer dentro del rango perpendicular de la huella. Para `east`/`west` Enfrentando, ambos valores Y deben estar dentro de la altura de la huella; para `north`/`south` En esta dirección, ambos valores X deben estar dentro del ancho de la huella. Esto garantiza que la ruta de acceso desde la loseta de contexto hasta la puerta discurra por el lado correcto del edificio.

Esto valida únicamente la orientación y alineación definidas por el autor: no genera ni modifica la geometría, no infiere una ruta transitable, no comprueba colisiones ni navegación, no requiere que la casilla de contexto y la puerta sean cardinalmente adyacentes ni altera el comportamiento en tiempo de ejecución.

`DMap` conserva los registros de edificios a través de guardar/cargar del editor y elimina los registros cuya lista de habitaciones, habitaciones interiores declaradas, habitaciones de espacio abierto declaradas, contexto exterior mal formado, conexión de acceso exterior obsoleta, conexión de entrada obsoleta, matriz de entradas mal formada, conexión de entrada obsoleta en la matriz de entradas, validación de entrada mal formada, anclajes de muebles mal formados o mal formados `reachability_validation` Los requisitos se vuelven obsoletos. El validador independiente realiza comprobaciones estrictas de forma, contención, superposición del mismo nivel, habitación completamente cerrada, acceso optado y comprobaciones de interior/espacio abierto/partición/superior/contexto exterior/contexto de acceso exterior/entrada/entradas/validación de entrada/anclajes de muebles.

Un edificio puede tener un autor nombrado muebles ancla metadatos con `furniture_anchors`:

```json
{"id": "office_building", "rooms": ["office", "garage_bay"], "footprint": {"x": 7, "y": 7, "width": 5, "height": 4}, "z": 0, "furniture_anchors": [{"id": "office_door_anchor", "at": [8, 9], "z": 0, "kind": "door"}, {"id": "garage_door_anchor", "at": [11, 8], "z": 0, "kind": "door"}]}
```

`furniture_anchors` es una matriz no vacía de registros de anclaje. Cada entrada tiene exactamente `id`, `at`, `z`, y `kind`. `id` Es único dentro del edificio y sigue el mismo patrón de nomenclatura que otros identificadores creados por el usuario. `at` es un número entero de dos `[x, y]` Coordenadas dentro de los límites del mapa. `z` es un nivel lógico desde `-10` a través de `10` y debe coincidir con el propio edificio. `z` o uno de sus declarados `building_levels`. `kind` es una etiqueta semántica no vacía (por ejemplo `"door"`, `"storage"`, `"workstation"`) — es un texto de formato libre, no un conjunto enumerado, y no conlleva ningún comportamiento en tiempo de ejecución.

Cada anclaje debe hacer referencia a una baldosa dentro de la huella del edificio que tenga una característica de mobiliario existente en el autor. `[x, y, z]`El ancla no genera mobiliario, no modifica el terreno, no infiere la transitabilidad, no verifica la categoría ni la función del mobiliario, ni altera la colisión, la navegación, la iluminación, el clima ni el comportamiento en tiempo de ejecución. Es un punto de referencia creado exclusivamente a partir de datos: una etiqueta semántica con nombre para el mobiliario existente que la futura composición de plantillas y la validación del juego pueden usar como ancla.

`Tools/examples/map_recipe_furniture_anchors.json` Muestra el edificio de oficinas mantenido con dos elementos de mobiliario principales: `office_door_anchor` y `garage_door_anchor`, cada uno apuntando a uno existente `door_wood` característica dentro de la huella.

### `reachability_validation`

Un edificio puede optar por cumplir con los requisitos de accesibilidad semántica estática con `reachability_validation`:

```json
{
  "reachability_validation": {
    "required_entrances": ["front_entrance", "garage_entrance"],
    "required_furniture_anchors": ["office_door_anchor", "garage_door_anchor", "upper_bench_anchor"],
    "required_building_levels": [0, 2]
  }
}
```

El registro es opcional, debe ser un objeto y debe definir al menos una de exactamente tres claves:

- `required_entrances` nombres entradas del edificio `entrances` conjunto por su `id`. Un singular `entrance` No tiene ID y no se puede hacer referencia a él.
- `required_furniture_anchors` nombres entradas del edificio `furniture_anchors` por `id`.
- `required_building_levels` nombres declarados `building_levels` valores z (o los del propio edificio) `z` cuando no `building_levels` se declaran).

Cada requisito es un array no vacío sin entradas duplicadas; los nombres de las entradas y los anclajes son cadenas no vacías, y los niveles son números enteros. El registro no debe almacenar coordenadas, identificadores de mosaicos ni rutas precalculadas; solo debe indicar las entradas, los anclajes y los niveles declarados existentes.

La validación es una búsqueda estática en amplitud sobre hechos semánticos creados, nunca una simulación de colisión o navegación. Los nodos del grafo son `(room, level)` pares; el nivel de una habitación proviene de su `building_levels` asignación, o el edificio `z` cuando los niveles no están declarados. Los nodos semilla son las habitaciones detrás de las conexiones de habitación al exterior cuya conexión `z` iguala el nivel de la habitación, restringido a las entradas indicadas cuando `required_entrances` se declara, o todas las conexiones exteriores de propiedad de otro modo. Los bordes del mismo nivel provienen de habitación a habitación. `room_connections` donde ambas habitaciones comparten el nivel de la habitación y la conexión `z` coincidencias. Los bordes verticales se derivan únicamente de elementos ya válidos creados `staircases`: una escalera validada conecta lo declarado `z: 0` y `z: 2` Los niveles de piso se encuentran en ambas direcciones, y la conexión es granular porque las pendientes de las escaleras ocupan el espacio abierto intencional donde no existe pertenencia a ninguna habitación. La búsqueda debe llegar a la habitación detrás de cada entrada requerida, al nivel de cada anclaje de mueble requerido y al menos a un nodo de habitación en cada nivel de edificio requerido. Se rechazan los objetivos desconocidos, los niveles no declarados y los requisitos inalcanzables; Python nunca simula colisiones ni navegación en tiempo de ejecución: las pruebas de navegación en tiempo de ejecución de Godot son la prueba de ello.

El generador valida la estructura del registro, la reproduce en el registro del edificio y rechaza los requisitos inalcanzables durante el proceso de creación. `Tools/map_validator.py` Realiza las mismas comprobaciones de forma independiente en los mapas generados. `DMap` conserva registros válidos mediante la función de guardar/cargar del editor y elimina edificios cuyos registros hacen referencia a entradas obsoletas, anclajes o niveles no declarados, o cuyos requisitos están mal formados. El mantenimiento `map_recipe_multi_level_building_foundation.json` demuestra el registro completo en ambos pisos a través de su escalera de autor, y el accesorio semántico canónico de dos pisos ahora hace lo mismo a través de `workroom_staircase`.

### Elementos canónicos de construcción semántica

`Tools/examples/map_recipe_semantic_single_storey_building.json` es el perfil canónico completo de una sola planta. Combina características físicas. `building_geometry`, una habitación completamente cerrada, paredes perimetrales z1 requeridas que incluyen cada esquina, una celda de altura libre z1 por encima de su puerta de la habitación al exterior, soporte de tierra debajo de cada pared inferior restante, propiedad de habitación/anclaje por piso, `access_validation`, clasificación interior/partición, semántica exterior y de entrada, validación de entrada y un anclaje de herrajes de puerta. Su artefacto generado es `Mods/Dimensionfall/Maps/generated_semantic_single_storey_building.json`.

`Tools/examples/map_recipe_semantic_two_storey_building.json` es el perfil canónico ordinario de dos plantas. Asigna un perfil completo `workroom` en z0 y un completo `loft` En z2, otorga a cada uno la propiedad de anclaje de muebles, pinta explícitamente las paredes perimetrales z1/z3 requeridas de su huella de 6×6 y crea un techo de hormigón en z4. La celda z1 sobre su puerta inferior está deliberadamente vacía; todas las demás paredes z1, incluidas las esquinas, están sostenidas por `dirt_light_00` en z0. Su artefacto generado es `Mods/Dimensionfall/Maps/generated_semantic_two_storey_building.json`El accesorio también escribe `workroom_staircase` — una escalera recta con la pendiente inferior en [10, 10] en z1 y la pendiente superior en [10, 9] en z2 — por lo que su `reachability_validation` puede requerir `loft_bench` y nivel z2. El loft es `enclosed` pero intencionalmente no `boundary_validation: complete`: la abertura de la escalera rompe su perímetro, por lo que el accesorio refleja la base multinivel mantenida en lugar de forzar registros de límites completos no válidos. El techo z4 explícito sigue siendo geometría de receta hasta una futura generalizada `wall_height` El contrato puede derivar una superficie de techo a partir del tramo superior de la pared.

El `access_validation` El contrato sigue únicamente el mismo z `room_connections`Por lo tanto, el accesorio de dos pisos no opta por ello: las rutas semánticas genéricas entre pisos se expresan mediante `reachability_validation`, que agrega aristas al grafo solo a través de ya válidas `staircases`Ninguno de los contratos reemplaza las comprobaciones de navegación de Godot; la colisión y el recorrido real se demuestran en tiempo de ejecución mediante `Tests/Unit/test_semantic_single_storey_navigation.gd`, `Tests/Unit/test_semantic_two_storey_navigation.gd`, y `Tests/Unit/test_field_farmland_navigation.gd` a través del reutilizable `Tests/Unit/helpers/map_navigation_fixture.gd` Función auxiliar que proporciona configuración genérica de navegación de fragmentos, horneado, conversión de cuadrícula a mundo y aserciones de ruta para cualquier geometría de mapa sin conocer la semántica de la habitación.

`Tools/examples/map_recipe_field_farmland.json` aplica este perfil genérico al legado `field_farmland` Huella de aceptación x3…18/y19…29. Conserva deliberadamente solo una cocina completa representativa, una habitación conectada en la planta baja, una habitación en la planta superior, puertas, anclajes, requisitos de accesibilidad y una escalera z1→z2; no es una ruta de generador específica para terrenos agrícolas y no intenta reproducir todas las habitaciones heredadas.

### `building_surfaces`

`building_surfaces` Declara la semántica de la cubierta superior creada para una huella de edificio validada existente sin crear geometría de techo o cielo raso:

```json
{
  "building_surfaces": [
    {"id": "office_roof", "building": "office_building", "kind": "roof", "z": 1},
    {"id": "office_ceiling", "building": "office_building", "kind": "ceiling", "z": 1}
  ]
}
```

Cada registro tiene exactamente `id`, `building`, `kind`, y `z`. `id` es único; `building` debe nombrar una raíz `buildings` registro; y `kind` es exactamente `roof`, `ceiling`, o `floor`. Un edificio puede tener como máximo un registro de cada tipo en cada `z`.

Para un edificio de una sola planta (no `building_levels`), `z` es el nivel lógico inmediatamente por encima de la huella del edificio (`building.z + 1`), y solo `roof` y `ceiling` Se permiten tipos. `roof` y `ceiling` Son clasificaciones independientes, elaboradas por autores distintos, y ambas pueden estar presentes en un mismo edificio.

Para un edificio de varios niveles (con `building_levels`), `z` debe indicar el nivel del edificio declarado como ocupado. `floor` y `ceiling` pueden apuntar a niveles declarados ocupados, mientras que `roof` Está permitido únicamente en el nivel de ocupación más alto declarado. Esto modela directamente la estructura vertical descendente:

```text
z: 0  ground-floor surface      -> kind "floor"
z: 1  air gap (never a surface)
z: 2  first-floor surface       -> kind "floor"
z: 2  ground-floor ceiling      -> kind "ceiling"
```

En el juego de arriba hacia abajo, el jugador nunca ve el techo: `ceiling` en `z: 2` es la parte inferior de la losa del primer piso que cierra el espacio de aire sobre la planta baja, y son solo metadatos de autoría; nunca se renderizan. `ceiling` no puede declararse a nivel del terreno.

Las superficies no colocan baldosas sobre sus `z`, crear una malla de techo/cielo/suelo, implicar colisión/soporte, cambiar la iluminación/clima, marcar habitaciones interiores o hacer que las celdas superiores estén ocupadas. `DMap` conserva registros válidos y elimina superficies cuyo edificio referenciado se ha eliminado, cuyo tipo no es compatible o cuyo `z` Ya no coincide con los niveles declarados del edificio; el generador y la validación independiente imponen la estricta relación de referencia, unicidad y z.

### `building_supports`

`building_supports` Autores de la semántica de soporte estructural para un edificio de varios niveles sin generar columnas, muros, vigas, colisiones o geometría de soporte:

```json
{
  "building_supports": [
    {
      "id": "northwest_column",
      "building": "office_building",
      "at": [7, 7],
      "from_z": 0,
      "to_z": 2,
      "kind": "column"
    }
  ]
}
```

Cada registro tiene exactamente `id`, `building`, `at`, `from_z`, `to_z`, y `kind`. `kind` actualmente está `column` o `wall`. La coordenada de soporte debe estar dentro de la huella del edificio; `from_z` y `to_z` Debe ascender a los niveles declarados de edificios ocupados. Esta es una afirmación del autor que indica que la semántica de los pisos superiores/azoteas tiene una ruta de soporte estructural que regresa a un nivel inferior ocupado. No infiere soporte a partir de la presencia de baldosas o muebles y no genera ni modifica la geometría en tiempo de ejecución.

`DMap` Conserva los registros de soporte válidos y elimina los registros con edificios obsoletos, coordenadas no válidas, tipos no compatibles o niveles no declarados o no ascendentes.

### `building_compositions`

`building_compositions` es una afirmación voluntaria, solo de datos, que indica que un edificio existente contiene las clasificaciones de gastos generales nombradas:

```json
{
  "building_compositions": [
    {
      "id": "office_complete_overhead",
      "building": "office_building",
      "required_surfaces": ["roof", "ceiling"]
    }
  ]
}
```

Cada registro tiene exactamente `id`, `building`y no vacío `required_surfaces`. `id` es único; una composición puede estar dirigida a cada edificio; `building` debe nombrar una raíz existente `buildings` registro; y cada tipo de superficie requerida debe ser única y exactamente `roof` o `ceiling`Cada tipo nombrado debe tener un tipo válido coincidente `building_surfaces` Registro para el mismo edificio. Esto es opcional: un edificio sin composición sigue siendo válido, y la composición no requiere todos los tipos admitidos a menos que su autor los enumere.

Las composiciones no añaden geometría ni comportamiento en tiempo de ejecución. Solo validan la coherencia entre registros; no infieren el estado interior, la cobertura de la superficie, el soporte ni la presencia de tejas o mallas de techo. `DMap` Conserva los registros válidos y elimina los que hacen referencia a edificios eliminados.

No se introducen plantillas generalizadas, polígonos, registros de edificios multinivel, anclajes de muebles ni operaciones de generación.

### `set`

Coloca una ficha. Campos: `type`, `x`, `y`, `tile`y nivel raíz opcional `z`.

```json
{"type": "set", "x": 16, "y": 15, "tile": {"id": "grass_flowers_01"}}
```

### `rectangle`

Rellena un rectángulo. Campos: `type`, `x`, `y`, positivo `width`, positivo `height`, `tile`y nivel raíz opcional `z`.

```json
{"type": "rectangle", "x": 10, "y": 9, "width": 12, "height": 14, "tile": {"id": "grass_plain_01"}}
```

### `rectangle_outline`

Coloca solo el borde rectangular y utiliza los mismos campos que `rectangle`. Si el ancho o la altura es `1`, cada celda en la forma resultante de una sola celda de ancho es un borde.

```json
{"type": "rectangle_outline", "x": 10, "y": 9, "width": 12, "height": 14, "tile": {"id": "grass_dirt_00"}}
```

### `line`

Coloca una línea inclusiva de un cuadro de ancho entre números enteros `[x, y]` puntos finales. Campos: `type`, `from`, `to`, `tile`y nivel raíz opcional `z`Las líneas utilizan el algoritmo de Bresenham para números enteros, por lo que las líneas horizontales, verticales, diagonales, inclinadas y en sentido contrario son deterministas.

```json
{"type": "line", "from": [0, 16], "to": [31, 16], "tile": {"id": "dirt_light_00"}}
```

### `scatter`

Selecciona celdas únicas en un rectángulo `region` utilizando el generador de números aleatorios con semilla de la receta. Campos: `type`, `region`, exactamente uno de `count` o `density`, `tile`y nivel raíz opcional `z`.

- `count` es un número entero desde cero hasta el número de celdas en la región.
- `density` es un número de `0` a través de `1`; el recuento de ubicaciones es `floor(region area × density)`.
- Las celdas seleccionadas reemplazan a sus mosaicos existentes.
- La misma receta completa y la misma semilla producen la misma selección y el mismo resultado.

```json
{
  "type": "scatter",
  "region": {"x": 0, "y": 0, "width": 32, "height": 32},
  "density": 0.1,
  "tile": {"id": "grass_flowers_00", "rotation": "random"}
}
```

### `pattern`

Coloca cada celda de un patrón con nombre en relación con un ancla. Campos: `type`, `pattern`, `at`, opcional a nivel raíz `z`y opcional `rotation`. La rotación se establece por defecto en `0` y acepta giros de un cuarto de vuelta fijos en el sentido de las agujas del reloj: `0`, `90`, `180`, o `270`.

Los desplazamientos giran alrededor del punto de anclaje: `90` mapas `[x, y]` a `[-y, x]`El generador realiza una comprobación previa de la expansión completa y rechaza la operación si alguna celda resultante está fuera del mapa; los patrones nunca se recortan ni se aplican parcialmente.

```json
{
  "type": "pattern",
  "pattern": "wildflower_cluster",
  "at": [16, 12],
  "rotation": 90
}
```

### `furniture`

Inserta una característica de mobiliario de una sola celda conocida en una loseta de terreno existente. Campos: `type`, `x`, `y`, `id`, opcional a nivel raíz `z`, opcional `rotation`y opcional `itemgroups`. La rotación se establece por defecto en `0` y acepta giros de un cuarto de vuelta fijos orientados hacia el editor: `0`, `90`, `180`, o `270`Cuando está presente, `itemgroups` debe ser una matriz no vacía de identificadores conocidos de la selección `Itemgroups/Itemgroups.json` catalogar.

```json
{
  "type": "furniture",
  "x": 16,
  "y": 16,
  "z": 0,
  "id": "cabinet_wood_00",
  "rotation": 270,
  "itemgroups": ["cabinet_general"]
}
```

La celda objetivo debe contener ya una casilla de terreno en el nivel lógico seleccionado y no debe contener ya una característica. El generador valida `id` compara con la base de datos de muebles seleccionada y escribe la representación del mapa/editor existente:

```json
{
  "id": "grass_dirt_00",
  "feature": {
    "type": "furniture",
    "id": "bench_garden",
    "rotation": 90,
    "itemgroups": []
  }
}
```

La operación no consume aleatoriedad. Repetir una operación de muebles en la misma celda es un error de conflicto. Una operación de baldosa posterior reemplaza deliberadamente el diccionario de baldosas completo y, por lo tanto, elimina una característica anterior, siguiendo el comportamiento de sobrescritura de baldosas ordenado establecido. Para muebles de contenedor, una receta no vacía `itemgroups` Esta lista sobrescribe el grupo de contenedores predeterminado de la definición de muebles; el entorno de ejecución elige aleatoriamente uno de los grupos de la lista cuando la instancia del mapa crea botín por primera vez. Omita este campo para conservar el grupo predeterminado de la definición de muebles.

La función serializada no tiene campo de nivel lógico, estático/móvil, estado o modo. `Chunk.process_level_data()` deriva la altura del mundo del índice de la matriz de nivel contenedora, y los datos de mobiliario en tiempo de ejecución seleccionan el generador estático o físico de la definición de mobiliario referenciada. `moveable` Propiedad. Plano `mode` pertenece al estado de mobiliario guardado en tiempo de ejecución y no forma parte de una característica de mapa generada recientemente.

### `furniture_scatter`

Coloca un número limitado de elementos de mobiliario de una sola celda seleccionados de una lista nombrada. `furniture_palette`. Campos: `type`, `region`, `palette`, exactamente uno de `count` o `density`y nivel raíz opcional `z`.

- `count` es un número entero desde cero a través del área de la región geométrica.
- `density` es un número de `0` a través de `1`; el número de ubicaciones solicitadas es `floor(region area × density)`.
- Las celdas elegibles deben contener terreno en el nivel lógico de destino y no deben tener ya ninguna característica.
- Si el número solicitado supera el número de células elegibles restantes, la generación falla antes de mutar las células objetivo de la operación de dispersión.
- Las coordenadas candidatas se enumeran en orden de filas, y luego se muestrean sin reemplazo con el generador de números aleatorios de la receta. Cada celda muestreada selecciona una entrada de mueble ponderada y resuelve su rotación en el orden de muestreo.
- La dispersión incrusta lo mismo `feature` objeto como explícito `furniture`; no infiere huellas, grupos de elementos ni ocupación de niveles adyacentes.

```json
{
  "type": "furniture_scatter",
  "region": {"x": 8, "y": 8, "width": 16, "height": 16},
  "z": 0,
  "count": 24,
  "palette": "clearing_trees"
}
```

## Validación y limitaciones

El generador rechaza campos desconocidos en cada nivel de receta, campos de dimensión de nivel raíz, ubicaciones mal formadas o fuera de límites, niveles lógicos fuera de límites `-10` a través de `10`, niveles agrupados duplicados, diseños raíz/agrupados ambiguos, bases de datos de mosaicos o muebles mal formadas, ID de mosaicos y muebles desconocidos, definiciones o membresías de área mal formadas, ID de área duplicados o membresías de mosaicos con el mismo ID, celdas de área no compatibles, conflictos de características, solicitudes de dispersión que exceden las celdas elegibles, rotaciones no válidas, Unicode no válido y recetas que no son de objetos. Valida los datos generados con `Tools/map_validator.py` antes de publicar el resultado. El validador requiere de forma independiente exactamente 21 matrices de nivel y exactamente 1024 entradas en cada nivel poblado, y valida estructuralmente los campos de muebles incrustados, los ID, las rotaciones, las matrices de grupos de elementos y las matrices de referencia de área.

La salida utiliza 21 niveles; lógico `z: 0` es un índice de cuadrícula por filas `10`. Establece `categories` a `[]`, `weight` a `1000`y las cuatro conexiones a `"ground"`Omite `areas`, coincidiendo `DMap.get_data()` cuando la lista de áreas está vacía.

El generador actualmente crea características de mobiliario explícitas, conocidas y de una sola celda, dispersión determinista ponderada de mobiliario sobre terreno elegible, membresías de área rectangular estrictas compatibles con el tiempo de ejecución en niveles lógicos explícitos y etiquetas de habitaciones creadas a través de terreno respaldado. `room_rectangle` operaciones. Las áreas pueden declarar adicionalmente un tiempo de ejecución ponderado validado por catálogo. `furniture`, `mob`, `mobgroup`, y `itemgroup` entidades; la selección ocurre cuando se instancia el mapa en lugar de durante la generación de Python. Las etiquetas de las habitaciones son solo semánticas y aún no crean ni infieren geometría. El mod principal define `rock_field_00` y `wild_vegetation_00` para composición en exteriores, utilizando los sprites generados por IA específicos `ai_rock_32_32.png` y `ai_vegetation_32_32.png`Todavía no admite contenido de grupos de elementos de muebles, múltiples características por celda, ocupación de objetos altos o de múltiples casillas, inferencia de soporte automático, áreas poligonales, paredes, puertas, edificios, carreteras como objetos semánticos, ciudades, patrones anidados o de varios niveles, o plantillas basadas en formas.

Los ejemplos de áreas mantenidas son: `Tools/examples/map_recipe_area_meadow.json` y `Tools/examples/map_recipe_area_entity_clearing.json`; el ejemplo de habitación mantenida es `Tools/examples/map_recipe_room_semantics.json`. El ejemplo de mobiliario mantenido es `Tools/examples/map_recipe_furniture_outdoor.json`Ejemplos estructurales mantenidos están disponibles en `Tools/examples/map_recipe_two_level_hill.json` y `Tools/examples/map_recipe_two_level_depression.json`Ellos usan `grass_ramp_00`, cuya definición de baldosa tiene `shape: "slope"`.

Las rotaciones de pendiente en las recetas utilizan los mismos valores que muestra el editor de mapas:

| Rotación | Borde alto |
|---:|---|
| `0` | norte |
| `90` | este |
| `180` | sur |
| `270` | oeste |

El generador conserva estos valores visibles para el editor en el JSON del mapa. Cuando se carga un mapa recién generado, `Chunk.get_block_rotation()` Aplica la conversión específica de la forma de la pendiente antes de que la malla renderizada, la colisión y la geometría de navegación utilicen la orientación interna. No convierta previamente las rotaciones de la receta a la interna. `calculate_slope_vertices()` cartografía.

`Tests/Unit/test_chunk_slope_rotation.gd` Verifica las cuatro rotaciones del editor con respecto a la rotación de tiempo de ejecución convertida y confirma que los bordes altos coinciden en los vértices de la malla renderizada, la geometría de colisión convexa y las caras de origen de navegación. `Tests/Unit/test_chunk_slope_navigation.gd` Realiza el horneado asíncrono real y confirma que cada orientación tiene una ruta consultable de baja a alta y de alta a baja. La prueba de ejemplo en Python confirma por separado que la casilla de lado alto creada y la casilla de lado bajo de nivel inferior están ocupadas. Las pruebas de juego manuales de ambos mapas mantenidos también han confirmado que el controlador del jugador real camina hacia arriba y hacia abajo por las cuatro orientaciones de pendiente sin problemas de recorrido ni colisiones invisibles.
