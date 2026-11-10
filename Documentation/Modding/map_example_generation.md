# Generación manual de mapas de ejemplo

`Tools/generate_map_examples.py` genera variantes deterministas de recetas de mapas mantenidas para inspección manual. Delega la construcción del mapa, validación, protección contra sobrescritura y publicación atómica a `Tools/map_generator.py`; no contiene una segunda implementación de generador.

Ejecute todos los comandos desde la raíz del repositorio. En el servidor Hermes estándar, realice la siguiente comprobación:

```bash
cd ~/local/dimensionfall/repository
```

## Genera una receta mantenida directamente en el mod.

Usar `Tools/map_generator.py` Cuando quieras seleccionar una receta, conservar su ID de mapa y publicar exactamente un mapa directamente en el mod Dimensionfall:

```bash
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_two_level_hill.json \
  Mods/Dimensionfall/Maps/generated_two_level_hill.json
```

Los ejemplos de recetas que se mantienen son:

| Receta | Mapa de resultados | Propósito |
|---|---|---|
| `Tools/examples/map_recipe.json` | `Mods/Dimensionfall/Maps/generated_meadow_prototype.json` | Paletas a nivel del suelo, operaciones de colocación, dispersión y patrones reutilizables. |
| `Tools/examples/map_recipe_furniture_outdoor.json` | `Mods/Dimensionfall/Maps/generated_furnished_clearing.json` | Decoración explícita más árboles, rocas y vegetación silvestre ponderados y conscientes del conflicto dispersos en terreno soportado en lógica `z: 0`. |
| `Tools/examples/map_recipe_area_meadow.json` | `Mods/Dimensionfall/Maps/generated_area_meadow.json` | Una definición de área de tiempo de ejecución y una `12×12` respaldado por el terreno `area_rectangle` límite de pertenencia en el límite lógico `z: 0`. |
| `Tools/examples/map_recipe_area_entity_clearing.json` | `Mods/Dimensionfall/Maps/generated_area_entity_clearing.json` | A `12×12` Área con respaldo de terreno cuya entidad de tocón ponderada es seleccionada de nuevo por el entorno de ejecución cada vez que se instancia el mapa.
| `Tools/examples/map_recipe_room_semantics.json` | `Mods/Dimensionfall/Maps/generated_room_semantics.json` | Autor `enclosed`, `covered_open`, y `ruin` Etiquetas de habitaciones en terrenos sin paredes, puertas, techos generados ni comportamiento de ejecución en interiores.
| `Tools/examples/map_recipe_room_connections.json` | `Mods/Dimensionfall/Maps/generated_room_connections.json` | Existente `door_wood` Características con enlaces semánticos explícitos de habitación a exterior y de habitación a habitación, sin topología inferida ni nuevo comportamiento de puerta en tiempo de ejecución.
| `Tools/examples/map_recipe_room_boundaries.json` | `Mods/Dimensionfall/Maps/generated_room_boundaries.json` | Una suscripción completa `enclosed` oficina con dirección existente `brick_wall_00`/`door_wood` evidencia, más parcialidad deliberada `covered_open` y `ruin` Referencias de la habitación. |
| `Tools/examples/map_recipe_single_level_building.json` | `Mods/Dimensionfall/Maps/generated_single_level_building.json` | Un rectángulo solo para datos `office_building` Huella que contiene la oficina completa diseñada y su evidencia de pared/puerta existente en un plano lógico. `z: 0`. |
| `Tools/examples/map_recipe_building_surfaces.json` | `Mods/Dimensionfall/Maps/generated_building_surfaces.json` | Un edificio validado con metadatos completos de habitaciones propias, techos superiores, contexto externo y acceso exterior: su oficina es interior, el garaje es espacio abierto, existen clasificaciones de techo/cielo en niveles lógicos `z: 1`, el terreno libre de habitaciones al oeste de la huella es el contexto externo creado, y `office_front_door` Asocia explícitamente ese contexto con el acceso exterior propio. Mantiene una composición de participación voluntaria sin geometría superior generada.
| `Tools/examples/map_recipe_furniture_anchors.json` | `Mods/Dimensionfall/Maps/generated_furniture_anchors.json` | Un edificio validado con metadatos de anclaje de mobiliario creados por el autor que apuntan al mobiliario existente dentro de su perímetro. |
| `Tools/examples/map_recipe_multi_entrance_building.json` | `Mods/Dimensionfall/Maps/generated_multi_entrance_building.json` | Un edificio validado con dos entradas autorizadas: la puerta principal de la oficina orientada al este y la entrada del garaje orientada al oeste. |
| `Tools/examples/map_recipe_multi_level_building_foundation.json` | `Mods/Dimensionfall/Maps/generated_multi_level_building_foundation.json` | Un edificio generado de dos plantas con pisos y paredes físicas, soportes creados, superficies por planta, escaleras, una planta baja en `z: 0`, una brecha abierta en `z: 1`y un primer piso en `z: 2`. |
| `Tools/examples/map_recipe_field_farmland.json` | `Mods/Dimensionfall/Maps/generated_field_farmland.json` | Un semántico representativo `field_farmland` casa de campo en la huella del legado x3…18/y19…29. Su declaración de cocina compacta utiliza `boundary_generation: "walls"` para generar un anillo de pared completo con esquinas rematadas mientras que su exterior e interior `room_connections` siguen siendo las únicas vacantes; también incluye propiedad por piso, anclas, `reachability_validation`y una escalera z1→z2 de autoría propia. |
| `Tools/examples/map_recipe_road_connections.json` | `Mods/Dimensionfall/Maps/generated_road_connections.json` | Un mapa sencillo para exteriores que muestra los metadatos de conexión de los bordes del mapa, con carreteras que entran desde el este y el oeste. |
| `Tools/examples/map_recipe_road_endpoints.json` | `Mods/Dimensionfall/Maps/generated_road_endpoints.json` | Un mapa exterior con puntos de referencia definidos en los extremos de las carreteras, en los bordes este y oeste, que indican dónde entran las carreteras al mapa. |
| `Tools/examples/map_recipe_two_level_hill.json` | `Mods/Dimensionfall/Maps/generated_two_level_hill.json` | Nivel del suelo `z: 0`, terreno elevado en `z: 1`y las cuatro rotaciones de pendiente. |
| `Tools/examples/map_recipe_two_level_depression.json` | `Mods/Dimensionfall/Maps/generated_two_level_depression.json` | Nivel del suelo `z: 0`, terreno rebajado en `z: -1`y las cuatro rotaciones de pendiente. |
| `Tools/examples/map_recipe_small_cabin_template.json` | `Mods/Dimensionfall/Maps/generated_small_cabin_template.json` | Una plantilla de cabaña anidada reutilizable con niveles relativos, rotaciones, anclajes y colocación compuesta. |
| `Tools/examples/map_recipe_village_square_composition.json` | `Mods/Dimensionfall/Maps/generated_village_square_composition.json` Una plaza de pueblo bien conservada, compuesta por plantillas reutilizables de plazas y cabañas.
| `Tools/examples/map_recipe_pine_hollow_outpost.json` | `Mods/Dimensionfall/Maps/generated_pine_hollow_outpost.json` | Un nuevo puesto de avanzada de supervivencia rural con un corto sendero peatonal/de servicio; su piso tipo loft y techo acolchado utilizan materiales derivados de habitaciones. `room_surface` operaciones. |
| `Tools/recipes/pine_hollow_outpost.json` | `Mods/Dimensionfall/Maps/pine_hollow_outpost.json` | El mapa jugable publicado de Pine Hollow Outpost generado a partir de la receta de producción. |

## Publicar la producción Pine Hollow Outpost

Las recetas de producción se encuentran bajo `Tools/recipes/`; los mapas generados en tiempo de ejecución viven bajo `Mods/Dimensionfall/Maps/`El juego carga los mapas desde este último directorio a través de `DMaps`/`RMaps`La receta de producción utiliza el ID de mapa estable `pine_hollow_outpost` y no reemplaza el legado `field_outpost` mapa.

```bash
python3 Tools/map_generator.py \
  Tools/recipes/pine_hollow_outpost.json \
  Mods/Dimensionfall/Maps/pine_hollow_outpost.json \
  --overwrite
python3 Tools/map_validator.py Mods/Dimensionfall/Maps/pine_hollow_outpost.json
```

`Tools/examples/map_recipe_pine_hollow_outpost.json` sigue siendo un accesorio de regresión mantenido. Conserva intencionalmente el `generated_pine_hollow_outpost` El ID permite que la salida del dispositivo y el mapa publicado se puedan probar de forma independiente.

Para ejemplos de varios niveles, las rotaciones de pendiente utilizan la convención del editor de mapas: `0` tiene su borde más alto al norte, `90` este, `180` sur y `270` oeste. El generador escribe esos valores directamente; Godot realiza la conversión en tiempo de ejecución específica para la pendiente al cargar un mapa recién generado.

Al probar manualmente cualquiera de los ejemplos de varios niveles, verifique cada pendiente desde ambas direcciones:

1. acercarse desde el lado de nivel inferior;
2. Suba por la pendiente visible hasta llegar a la baldosa del lado más alto;
3. date la vuelta y baja caminando;
4. Confirme que no existe ninguna pared invisible ni plano de colisión con una orientación diferente;
5. Repita el procedimiento para los bordes altos norte, este, sur y oeste.

La cobertura GUT automatizada verifica que la conversión, los vértices de renderizado, la geometría de colisión y las caras de origen de navegación coincidan en cada arista alta. Una prueba de integración específica también realiza el horneado asíncrono y verifica las rutas de navegación bidireccionales en todas las orientaciones. Las pruebas de juego manuales de la colina y la depresión mantenidas han confirmado que el controlador del jugador real puede caminar hacia arriba y hacia abajo en las cuatro orientaciones sin problemas de recorrido o colisión invisible. Repita la lista de verificación después de cambiar la geometría de la pendiente, la colisión, la configuración de navegación o el comportamiento de movimiento del jugador.

Copia el comando correspondiente al mapa que deseas inspeccionar:

```bash
# Ground-level meadow
python3 Tools/map_generator.py \
  Tools/examples/map_recipe.json \
  Mods/Dimensionfall/Maps/generated_meadow_prototype.json

# Ground-level furnished clearing
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_furniture_outdoor.json \
  Mods/Dimensionfall/Maps/generated_furnished_clearing.json

# Ground-level runtime area meadow
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_area_meadow.json \
  Mods/Dimensionfall/Maps/generated_area_meadow.json

# Ground-level runtime-random area entity clearing
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_area_entity_clearing.json \
  Mods/Dimensionfall/Maps/generated_area_entity_clearing.json

# Ground-level authored room semantics
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_room_semantics.json \
  Mods/Dimensionfall/Maps/generated_room_semantics.json

# Ground-level authored room connections
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_room_connections.json \
  Mods/Dimensionfall/Maps/generated_room_connections.json

# Ground-level authored room boundaries
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_room_boundaries.json \
  Mods/Dimensionfall/Maps/generated_room_boundaries.json

# Ground-level single-level authored building footprint
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_single_level_building.json \
  Mods/Dimensionfall/Maps/generated_single_level_building.json

# Ground-level authored roof/ceiling semantics for a validated building
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_building_surfaces.json \
  Mods/Dimensionfall/Maps/generated_building_surfaces.json

# Ground-level authored furniture anchors for a validated building
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_furniture_anchors.json \
  Mods/Dimensionfall/Maps/generated_furniture_anchors.json

# Ground-level multi-entrance building with two authored doors
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_multi_entrance_building.json \
  Mods/Dimensionfall/Maps/generated_multi_entrance_building.json

# Two-floor building with floors, walls, supports, surfaces, and stairs
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_multi_level_building_foundation.json \
  Mods/Dimensionfall/Maps/generated_multi_level_building_foundation.json

# Semantic field_farmland farmhouse with entrance, anchors, and stairs
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_field_farmland.json \
  Mods/Dimensionfall/Maps/generated_field_farmland.json

# New authored Pine Hollow Outpost with a short pedestrian/service path, cabin, lean-to, and loft
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_pine_hollow_outpost.json \
  Mods/Dimensionfall/Maps/generated_pine_hollow_outpost.json

# Ground-level authored map-edge road connection metadata
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_road_connections.json \
  Mods/Dimensionfall/Maps/generated_road_connections.json

# Ground-level authored map-edge road endpoint anchors
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_road_endpoints.json \
  Mods/Dimensionfall/Maps/generated_road_endpoints.json

# Two-level hill
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_two_level_hill.json \
  Mods/Dimensionfall/Maps/generated_two_level_hill.json

# Two-level depression
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_two_level_depression.json \
  Mods/Dimensionfall/Maps/generated_two_level_depression.json
```

El nombre del archivo de salida debe coincidir con el de la receta. `id`, seguido de `.json`El generador valida el mapa antes de publicarlo y se niega a reemplazar un archivo existente. Para regenerar deliberadamente el mismo mapa después de editar su receta, agregue `--overwrite`:

```bash
python3 Tools/map_generator.py \
  Tools/examples/map_recipe_two_level_hill.json \
  Mods/Dimensionfall/Maps/generated_two_level_hill.json \
  --overwrite
```

## Generar todos los ejemplos mantenidos con sobrescritura

`map_generator.py` acepta exactamente una receta de entrada y una ruta de salida por invocación. Para regenerar todo el conjunto de mapas de ejemplo mantenidos en un solo comando de terminal, recorra cada receta y llámela una vez por receta con `--overwrite`:

```bash
for recipe in \
  Tools/examples/map_recipe.json \
  Tools/examples/map_recipe_furniture_outdoor.json \
  Tools/examples/map_recipe_area_meadow.json \
  Tools/examples/map_recipe_area_entity_clearing.json \
  Tools/examples/map_recipe_room_semantics.json \
  Tools/examples/map_recipe_room_connections.json \
  Tools/examples/map_recipe_room_boundaries.json \
  Tools/examples/map_recipe_single_level_building.json \
  Tools/examples/map_recipe_building_surfaces.json \
  Tools/examples/map_recipe_furniture_anchors.json \
  Tools/examples/map_recipe_multi_entrance_building.json \
  Tools/examples/map_recipe_multi_level_building_foundation.json \
  Tools/examples/map_recipe_field_farmland.json \
  Tools/examples/map_recipe_pine_hollow_outpost.json \
  Tools/examples/map_recipe_road_connections.json \
  Tools/examples/map_recipe_road_endpoints.json \
  Tools/examples/map_recipe_two_level_hill.json \
  Tools/examples/map_recipe_two_level_depression.json \
  Tools/examples/map_recipe_small_cabin_template.json \
  Tools/examples/map_recipe_village_square_composition.json; do
  id=$(python3 -c "import json,sys;print(json.load(open('$recipe'))['id'])")
  python3 Tools/map_generator.py \
    "$recipe" \
    "Mods/Dimensionfall/Maps/$id.json" \
    --overwrite
done
```

El bucle lee cada receta `id` (que los ejemplos mantienen sincronizados con su nombre de archivo de salida canónico), luego regenera ese mapa en `Mods/Dimensionfall/Maps/` con `--overwrite`. Utilice esto para una prueba de humo rápida de extremo a extremo que cada receta mantenida aún genera y pasa la validación. Si se ha agregado o renombrado una receta, el bucle cubre todas las recetas que liste, y el nombre del archivo de salida siempre sigue el de la receta. `id`.

Después de generar un mapa, inicie o reinicie Godot y siga los pasos del editor de contenido que se indican a continuación. Los archivos creados directamente con `map_generator.py` No están registrados en el manifiesto de limpieza del ejecutor de ejemplo. Elimínelos explícitamente cuando finalicen las pruebas manuales:

```bash
rm Mods/Dimensionfall/Maps/generated_meadow_prototype.json
rm Mods/Dimensionfall/Maps/generated_furnished_clearing.json
rm Mods/Dimensionfall/Maps/generated_area_meadow.json
rm Mods/Dimensionfall/Maps/generated_area_entity_clearing.json
rm Mods/Dimensionfall/Maps/generated_room_semantics.json
rm Mods/Dimensionfall/Maps/generated_room_connections.json
rm Mods/Dimensionfall/Maps/generated_room_boundaries.json
rm Mods/Dimensionfall/Maps/generated_single_level_building.json
rm Mods/Dimensionfall/Maps/generated_building_surfaces.json
rm Mods/Dimensionfall/Maps/generated_furniture_anchors.json
rm Mods/Dimensionfall/Maps/generated_multi_entrance_building.json
rm Mods/Dimensionfall/Maps/generated_multi_level_building_foundation.json
rm Mods/Dimensionfall/Maps/generated_field_farmland.json
rm Mods/Dimensionfall/Maps/generated_pine_hollow_outpost.json
rm Mods/Dimensionfall/Maps/generated_road_connections.json
rm Mods/Dimensionfall/Maps/generated_road_endpoints.json
rm Mods/Dimensionfall/Maps/generated_two_level_hill.json
rm Mods/Dimensionfall/Maps/generated_two_level_depression.json
rm Mods/Dimensionfall/Maps/generated_small_cabin_template.json
rm Mods/Dimensionfall/Maps/generated_village_square_composition.json
```

Ejecute el comando de eliminación únicamente para los archivos que usted haya generado. Estos identificadores (ID) se utilizan como ejemplos de desarrollo; no sobrescriba un mapa con el mismo ID si ya se ha reutilizado como contenido del proyecto.

## Primera carrera segura

Genera tres variantes en un directorio temporal:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir /tmp/dimensionfall-map-examples
```

La receta predeterminada es `Tools/examples/map_recipe.json`Cada variante recibe:

- un ID único y un nombre de archivo coincidente que termina en `_example_001`, `_example_002`, etcétera;
- una semilla secuencial, comenzando con la semilla de la receta a menos que `--seed` se suministra;
- Los datos completos del mapa de 32 x 32 fueron validados antes de su publicación.

Si lo desea, puede inspeccionar los archivos generados con el validador:

```bash
python3 Tools/map_validator.py /tmp/dimensionfall-map-examples
```

## Elige el número de variantes y la semilla inicial.

Generar cinco variantes reproducibles utilizando semillas `1000` a través de `1004`:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir /tmp/dimensionfall-map-examples \
  --variants 5 \
  --seed 1000
```

Al ejecutar la misma receta completa con la misma semilla inicial, se obtiene el mismo archivo JSON del mapa. Cambiar la semilla modifica la distribución de las baldosas, la distribución de los muebles, la selección de la paleta de colores y las rotaciones aleatorias.

## Inspeccione los mapas generados en el editor de contenido.

Coloca los ejemplos en el directorio de mapas del mod principal:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir Mods/Dimensionfall/Maps \
  --variants 3 \
  --seed 1000
```

Inicia Godot después de la generación y ejecuta el proyecto Dimensionfall.
En el proyecto en ejecución, navegue a:

```text
Content Manager
  → Content Editor
  → select the Dimensionfall mod
  → Expand the Maps category if it is still collapsed
  → open a generated map by doubleclicking it in the map list
  → View Map in the editor, click `save and test` to test it manually
```

Si Godot ya estaba en ejecución cuando se generaron los archivos, reinícielo para que se cargue de nuevo el contenido del mod. La vista previa del editor de mapas existente muestra el JSON del mapa generado; la herramienta de Python no crea una imagen ni una vista previa HTML por separado.

Para el claro amueblado mantenido, confirme que el editor muestra un banco de jardín en `[16, 16]`, una fogata sin encender en `[15, 16]`y una planta en maceta en `[17, 16]`. Dispersa de forma determinista 24 árboles o tocones quemados, luego 16 rocas o elementos de vegetación silvestre, sobre el claro de 16×16 mientras deja intactos esos tres elementos creados por el autor. El mantenedor inspeccionó el claro en el editor con `save and test` Una vez instalados los sprites de IA específicos, se confirmó que los muebles de roca y vegetación silvestre se generaban sin problemas. `rock_field_00` usos `ai_rock_32_32.png`, y `wild_vegetation_00` usos `ai_vegetation_32_32.png`La receta deja intencionalmente `itemgroups` Está vacío y no aplica reglas de contenido de contenedores, ocupación de múltiples celdas ni soporte entre niveles.

Para el área de pradera mantenida, confirme que la lista de áreas contiene `meadow_clearing` y el editor destaca exactamente el `12×12` rectángulo de `[10, 10]` a través de `[21, 21]` en lógica `z: 0`. La definición tiene una `100` porcentaje de probabilidad de aparición y reemplaza ese grupo de miembros conectados con `grass_dirt_00`; usar `save and test` Para confirmar que el entorno de ejecución aplica el área sin afectar el terreno exterior. El ejemplo no contiene deliberadamente entidades, habitaciones, paredes, puertas ni semántica de edificios.

Para la eliminación de entidades del área mantenida, confirme lo mismo. `[10, 10]` a través de `[21, 21]` límite de membresía para `stump_clearing`. El JSON deliberadamente no contiene características de muebles precalculadas: cada instancia utiliza la regla de área establecida para hacer una selección ponderada nueva entre sus `burned_tree_stump` entrada y el peso implícito de no aparición. Reinicia y crea instancias del mapa varias veces con **guardar y probar**; confirma que las posiciones de los tocones pueden variar entre ejecuciones, permanecen dentro del límite de membresía, conservan la rotación de membresía cuando aparecen y nunca alteran el terreno exterior. No confirmes el mapa de inspección generado a menos que se promueva explícitamente a contenido del proyecto.

Para el mapa de semántica de salas mantenido, inspeccione `generated_room_semantics` En el editor de contenido, guárdelo, ciérrelo y vuelva a abrirlo. Confirme que `office` (`enclosed`), `garage_bay` (`covered_open`), y `ruined_store` (`ruin`Las etiquetas persisten sin errores de carga. La oficina y el garaje comparten intencionalmente espacios adyacentes. `concrete_00` El terreno no permite deducir los límites de las habitaciones a partir del material del suelo. La vista previa del mapa y la opción **guardar y probar** solo deben usarse para confirmar la persistencia y la carga normal en esta versión: aún no crea paredes, techos, puertas, iluminación, clima ni comportamiento de juego en interiores.

Para ver el mapa de conexiones de habitaciones mantenidas, abra `generated_room_connections`, guárdalo, ciérralo y vuelve a abrirlo. Confirma que `office_front_door` sigue siendo un `office`→`exterior` enlace y `office_to_garage` sigue siendo un `office`→`garage_bay` enlace. En la vista previa del mapa y **guardar y probar**, verifique ambos existentes `door_wood` Las funciones se cargan y conservan la interacción normal de apertura/cierre. Estos metadatos no deben crear paredes, techos, iluminación, condiciones climáticas, cerramientos automáticos ni un comportamiento diferente de la puerta en tiempo de ejecución.

Para ver el mapa de límites de habitaciones mantenidos, abra `generated_room_boundaries`, guárdalo, ciérralo y vuelve a abrirlo. Confirma que `office` conserva `boundary_validation: "complete"`, sus ocho registros de límites dirigidos conservan su cardinalidad `side`y los once existentes de la oficina `brick_wall_00` baldosas más una existente `door_wood` La función permanece presente. Los registros parciales del garaje y las ruinas no tienen requisitos de completitud. **Guardar y probar** solo debe confirmar la carga normal y la interacción con la puerta existente: esta validación no agrega geometría, colisión, navegación, iluminación, clima ni comportamiento en interiores.

Para ver el mapa del edificio de una sola planta que se mantiene, abra `generated_single_level_building`Guárdalo, ciérralo y vuelve a abrirlo. Confirma la raíz. `buildings` El array retiene `office_building` con su `office` lista de habitaciones, `[7, 7]` `4×4` huella y lógica `z: 0`Confirmar los once existentes `brick_wall_00` baldosas y una `door_wood` La apertura aún carga y mantiene la interacción normal. La huella es solo metadatos: **guardar y probar** no debe generar terreno nuevo, paredes, techos, colisiones, navegación, iluminación, clima ni comportamiento en interiores.

Para ver el mapa de superficies de edificios mantenidas, abra `generated_building_surfaces`Guárdalo, ciérralo y vuelve a abrirlo. Confirmar `office_building` conserva `access_validation: "complete"`, `interior_rooms: ["office"]`, `open_space_rooms: ["garage_bay"]`, `room_partition_validation: "complete"`, `overhead_validation: "complete"`, `exterior_context: {"at": [6, 8], "z": 0}`, y `exterior_access_context: {"connection": "office_front_door"}`. El terreno exterior de baldosas permanece libre de habitaciones al oeste de la huella; el contexto de acceso nombra explícitamente la conexión existente de oficina propiedad al exterior sin afirmar una ruta física entre las dos coordenadas. La oficina y el garaje siguen siendo la partición exacta creada por el autor de las habitaciones propiedad del edificio, y el techo/cielo siguen siendo clasificaciones requeridas en `z: 1`. Su existencia `office_front_door` y `garage_opening` Los enlaces entre la habitación y el exterior satisfacen de forma independiente la integridad del acceso autorizado. Confirmar `office_roof` (`roof`) y `office_ceiling` (`ceiling`) conservan sus referencias de construcción y lógica `z: 1`, mientras `building_compositions` conserva `office_complete_overhead` que requieren ambos tipos. Los once `brick_wall_00` baldosas más las existentes `door_wood` Las características deben permanecer sin cambios. **Guardar y probar** no debe agregar terreno, tejas de techo/cielo, mallas, colisiones, soporte, iluminación, clima, estado interior, búsqueda de rutas, carreteras ni ningún comportamiento en tiempo de ejecución; esta sección verifica únicamente la persistencia de los metadatos creados y la coherencia entre registros.

Dimensionfall también busca un personaje con el mismo nombre. `.png` El sprite del mapa se genera durante el inicio. El ejecutor genera intencionadamente solo el JSON del mapa, por lo que Godot registra un error no crítico de recurso faltante para ese sprite. Esto no impide que el mapa JSON se cargue ni que la vista previa de la cuadrícula de mosaicos del editor de mapas lo muestre.

## Generar a partir de recetas específicas

Pase una o más rutas de recetas antes de las opciones. El ejecutor genera el número solicitado de variantes para cada receta:

```bash
python3 Tools/generate_map_examples.py \
  Tools/examples/map_recipe.json \
  --output-dir /tmp/dimensionfall-map-examples \
  --variants 2 \
  --seed 500
```

Por ejemplo, genere una colina con limpieza gestionada y una depresión con limpieza gestionada directamente en la carpeta del mod:

```bash
python3 Tools/generate_map_examples.py \
  Tools/examples/map_recipe_two_level_hill.json \
  Tools/examples/map_recipe_two_level_depression.json \
  --output-dir Mods/Dimensionfall/Maps \
  --variants 1
```

Esto produce:

```text
Mods/Dimensionfall/Maps/generated_two_level_hill_example_001.json
Mods/Dimensionfall/Maps/generated_two_level_depression_example_001.json
```

Usar `map_generator.py` para obtener un resultado de receta exacto. Utilice `generate_map_examples.py` cuando se necesitan varias semillas, varias recetas en un solo comando o una limpieza basada en el manifiesto.

Utiliza diferentes bases de datos de baldosas y muebles al probar la receta de muebles de otro mod:

```bash
python3 Tools/generate_map_examples.py \
  path/to/recipe.json \
  --output-dir /tmp/dimensionfall-map-examples \
  --tiles path/to/Tiles.json
```

El ejemplo del corredor infiere `Furniture/Furniture.json` junto a los seleccionados `Tiles` directorio. Para una invocación de un solo mapa cuya base de datos de muebles se encuentra en otro lugar, utilice `Tools/map_generator.py --furniture path/to/Furniture.json`.

## Protección contra sobrescritura

Los archivos de salida existentes están protegidos de forma predeterminada. Si se repite un comando con los mismos identificadores, se generará un error en lugar de reemplazarlos.

Para regenerar deliberadamente esos archivos, agregue `--overwrite`:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir /tmp/dimensionfall-map-examples \
  --variants 3 \
  --seed 1000 \
  --overwrite
```

Un archivo sobrescrito que no fue creado originalmente por este ejecutor no es reclamado por su manifiesto de limpieza y no será eliminado por `--clean`.

## Limpieza segura

El ejecutor registra los nombres de archivo de salida recién creados y los resúmenes SHA-256 en:

```text
.dimensionfall-map-examples-manifest
```

El manifiesto no tiene `.json` extensión, por lo que no se trata como un mapa. La limpieza elimina solo los archivos de mapa registrados cuyo contenido aún coincide con su resumen SHA-256 generado:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir /tmp/dimensionfall-map-examples \
  --clean
```

Elimine los ejemplos instalados para la inspección del editor con:

```bash
python3 Tools/generate_map_examples.py \
  --output-dir Mods/Dimensionfall/Maps \
  --clean
```

Los mapas no relacionados en el directorio de salida permanecen intactos. También se conserva una ruta generada que haya sido modificada o reemplazada posteriormente. Si falta el manifiesto, la limpieza no elimina nada. Si está mal formado, la limpieza se detiene en lugar de intentar adivinar qué archivos se pueden eliminar sin riesgo.

## Comprobaciones útiles

Mostrar todas las opciones de comando:

```bash
python3 Tools/generate_map_examples.py --help
```

Ejecute el conjunto de pruebas de regresión de navegación en tiempo de ejecución de Godot farmhouse desde la raíz del repositorio:

```bash
godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://Tests/Unit -gprefix=test_field_farmland_navigation -gexit
```

Esto verifica el anillo de pared de cocina automático migrado con horneado real de Chunk y `NavigationServer3D`: las rutas declaradas de la granja siguen siendo transitables a través de la puerta oeste, la conexión interior y la escalera creadas, mientras que las comprobaciones delimitadas rechazan la pared norte/sur, la partición este que no es una puerta y ambos cruces de la tapa de la esquina generados.

Ejecuta la aceptación de navegación en tiempo de ejecución de Pine Hollow Outpost enfocada desde la raíz del repositorio:

```bash
godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://Tests/Unit -gprefix=test_pine_hollow_outpost_navigation -gexit
```

Esto hornea la nueva geometría del puesto avanzado a través de datos reales. `Chunk` y `NavigationServer3D` código. Verifica el camino de servicio a la cabaña, de la cabaña al cobertizo abierto y el recorrido bidireccional de la cabaña al desván, y rechaza un cruce aislado de la pared norte. Generar `generated_pine_hollow_outpost.json` con el comando anterior y revíselo en el editor de contenido para la aceptación de la composición visual restante.

Ejecuta los conjuntos de regresión de geometría de pendiente y navegación precalculada de Godot enfocados:

```bash
godot --headless --path . \
  -s addons/gut/gut_cmdln.gd \
  -gtest=Tests/Unit/test_chunk_slope_rotation.gd \
  -gtest=Tests/Unit/test_chunk_slope_navigation.gd \
  -gexit
```

Validar todos los mapas generados en un directorio de salida temporal:

```bash
python3 Tools/map_validator.py /tmp/dimensionfall-map-examples
```

Confirme las dimensiones del mapa generado y los niveles poblados:

```bash
python3 - <<'PY'
import json
from pathlib import Path

for path in sorted(Path("/tmp/dimensionfall-map-examples").glob("*.json")):
    data = json.loads(path.read_text(encoding="utf-8"))
    populated = [index for index, level in enumerate(data["levels"]) if level]
    print(
        path.name,
        f"{data['mapwidth']}x{data['mapheight']}",
        f"ground entries={len(data['levels'][10])}",
        f"populated levels={populated}",
    )
PY
```

## Limitaciones actuales

- Las variantes modifican la base de la receta; no sintetizan nuevas operaciones de receta.
- Los mapas generados admiten terreno, mobiliario explícito conocido de una sola celda, dispersión determinista ponderada del mobiliario y pertenencias de áreas rectangulares compatibles con el tiempo de ejecución en niveles lógicos explícitos. Las áreas también pueden declarar tiempo de ejecución ponderado `furniture`, `mob`, `mobgroup`, o `itemgroup` entidades, por lo que un campo de área puede variar cada vez que se instancia su mapa. El claro mantenido utiliza sprites de roca y vegetación silvestre generados por IA. El contenido de grupos de elementos, las características de múltiples casillas o altas, la inferencia de soporte automático, las áreas poligonales, las habitaciones, los edificios, las carreteras semánticas y las plantillas multinivel aún no son compatibles.
- Los mapas se pueden inspeccionar con la vista previa del editor de contenido existente, pero el ejecutor no inicia Godot ni inyecta mapas en una sesión de editor que ya esté en ejecución.
- Instalando ejemplos en `Mods/Dimensionfall/Maps` Las pone a disposición del editor de contenido, pero no las referencia automáticamente desde la generación del área del mapa general.
