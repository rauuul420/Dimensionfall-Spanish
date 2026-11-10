# Documentación del formato de mapa de Dimensionfall

Este documento define la estructura y el uso de los archivos JSON de mapas en Dimensionfall. Esta información está dirigida a agentes o desarrolladores que crean nuevos mapas o modifican los existentes mediante métodos automatizados.

## Descripción general
Los mapas de Dimensionfall se almacenan como archivos JSON. Un mapa consta de metadatos (nombre, peso, etc.), una lista de áreas que pueden generarse aleatoriamente en el mapa y una cuadrícula multicapa que representa la disposición real de las casillas.

## Estructura JSON del mapa

### Campos de nivel superior (metadatos)

| Campo | Tipo | Obligatorio | Descripción |
| :---0|:---|:---|:---|
| `id` | Cadena | Sí | Identificador único para el mapa (coincide con el nombre del archivo sin `.json`). |
| `name` | Cadena | Sí | Nombre fácil de usar. |
| `description` | Cadena | No | Texto descriptivo del mapa. |
| `categories` | Array[String] | No | Etiquetas utilizadas para agrupar mapas (por ejemplo, `["Field", "Plains"]`). |
| `weight` | Entero | No | Peso de probabilidad al seleccionar mapas en un grupo (predeterminado: `1000`). |
| `mapwidth` | Entero | No | Ancho de la cuadrícula del mapa en mosaicos (predeterminado: `32`). |
| `mapheight` | Entero | No | Altura de la cuadrícula del mapa en mosaicos (predeterminado: `32`). |
| `connections` | Diccionario | No | Tipos de conexión vial para direcciones cardinales (`north`, `south`, `east`, `west`). Los valores suelen ser `"road"` o `"ground"`. |
| `areas` | Array[Object] | Sí | Lista de áreas que se pueden instanciar en el mapa. |
| `levels` | Array[Array[Object]] | Sí | Los datos reales de los mosaicos para cada capa/nivel. |

### Estructura de áreas
Un área define una "plantilla" para los mosaicos y entidades que existen en una parte específica del mapa. Estos se utilizan para rellenar los niveles o definir zonas específicas.

**Campos del objeto de área:**
- `id` (Cadena): Identificador único para esta plantilla de área.
- `rotate_random` (Booleano): Si es verdadero, la rotación de la casilla del área se genera aleatoriamente.
- `spawn_chance` (Entero): Probabilidad (0-100) de que esta área aparezca en su nivel designado.
- `tiles` (Array[Object]): Una lista de casillas que pertenecen a esta área. 
	- Cada objeto de baldosa: `{ "id": "tile_id", "count": integer }`.
	- Se utiliza una casilla "nula" para alterar la probabilidad de aparición de entidades, por ejemplo, si { "id": "null", "count": 1000 } y la entidad es { "id": "Tree_00", "type": "furniture", "count": 1 }, la probabilidad de aparición del árbol es 1/1000.
- `entities` (Array[Object]): Una lista de entidades (muebles, criaturas, grupos de objetos) que pertenecen a esta área.
    - Cada objeto de entidad: `{ "id": "entity_id", "type": "furniture" | "mob" | "mobgroup" | "itemgroup", "count": integer }`.

### Estructura de niveles
El `levels` El campo es una matriz de matrices. Cada matriz interna representa una única profundidad/capa. 
- Un índice en el `levels` matriz que contiene una matriz vacía `[]` representa un nivel vacío.
- Un nivel poblado contiene objetos (baldosas) que representan la colocación real de las baldosas.

**Objeto de mosaico (en niveles):**
- `id` (Cadena): El ID de la ficha a colocar. Debe corresponder a una entrada válida en `Tiles.json`.
- `rotation` (Entero): Rotación en grados ($0, 90, 180, 270$).
- `areas` (Array[Object]): Una lista de referencias de área que están "activas" en la ubicación de esta casilla.
	- Cada referencia: `{ "id": "area_id", "rotation": integer }`.
	- La rotación del área se utilizará al colocar las baldosas y entidades del área, por lo que la rotación predeterminada de las baldosas/entidades puede alterarse al aplicar el área.

## Referencias de contenido válidas

Al crear mapas, asegúrese de que todos los identificadores utilizados existan en sus respectivos archivos fuente:

1.  **Baldosas**: Verificado a través de `Mods/Dimensionfall/Tiles/Tiles.json`.
	*   *Ejemplos*: `grass_plain_01`, `forest_underbrush_00`, `brick_wall_00`.
2.  **Muebles**: Verificado a través de `Mods/Dimensionfall/Furniture/Furniture.json`.
	*   *Ejemplos*: `table_round_wood`, `chair_wood`, `door_wood`.
3.  **Entidades**:
	*   `mobgroup`: p.ej, `basic_zombies`.
	*   `itemgroup`: es, `generic_field_finds`.

## Comportamiento del editor de mapas
El editor de mapas (`mapeditor.gd`) proporciona una interfaz de usuario para la edición manual.
- Permite explorar y seleccionar mosaicos de la base de datos de mosaicos.
- Permite configurar los metadatos del mapa (nombre, descripción, categorías).
- Proporciona casillas de verificación para alternar `"road"` vs `"ground"` conexiones.
- Guarda los cambios en el archivo JSON usando `DMap.save_data_to_disk()`.

## Ejemplos

### Mapa de campo simple (`field_grass_basic_00.json`)
Un mapa minimalista con una capa base de baldosas de césped y plantillas de área sencillas para árboles y objetos. Utiliza muchos niveles vacíos para representar capas de altura/profundidad que no se utilizan.

### Estructura compleja (`Generichouse.json`)
Incluye específico `pick_one: true` áreas que seleccionan de una lista de tipos de piso (por ejemplo, diferentes patrones de alfombra o madera) para asegurarse de que esa área se construya utilizando solo ese tipo de piso.

## Orientación para agentes

### ✅ Hacer
- **Crea nuevos archivos** para conceptos de mapas completamente nuevos para evitar dañar el contenido del mod existente.
- Utilice el `areas` Sistema para construir terrenos complejos y aleatorios sin necesidad de definir manualmente miles de casillas.
- Respeto `mapwidth` y `mapheight` límites al poblar `levels`.
- Verificar todo `id` cuerdas contra `Tiles.json` y `Furniture.json`.

### ❌ No lo hagas
- **Modificar mapas existentes** a menos que se le asigne específicamente un cambio pequeño (por ejemplo, cambiar un tipo de conexión).
- Codificar listas masivas de mosaicos en el `levels` matriz para áreas grandes; en su lugar, defina esas baldosas en una `area` objeto y referencia esa área en la entrada de la casilla del nivel.
- Utilice identificadores que no existan en las bases de datos de contenido del proyecto.
