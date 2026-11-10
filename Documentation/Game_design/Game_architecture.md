# Arquitectura de juegos
Esta página incluye información sobre la arquitectura general del juego.

En esta página se describirán varias partes clave.
| |

# Escena principal
La escena principal es scene_selector.tscn. Desde aquí, puedes navegar para jugar, modificar el contenido o abrir el menú de ayuda.


# Escena del juego
Cuando pulses el botón 'reproducir demostración':
1. Todos los mods activos de Gamedata.mods se fusionan con Runtimedata según su orden de carga.
2. El juego cambia a 'level_generation.tscn'. Cada entidad ejecutará su script en su función _ready:
3. El Generador de niveles comprobará si hay datos guardados para la ubicación y los cargará. Si no los hay, preguntará al `overmap_manager` para el terreno alrededor de la posición de aparición del jugador. Los datos de la `overmap_manager` Será una cuadrícula de mapas determinada por map_seed. Para cada fragmento de la cuadrícula que esté cerca del jugador, LevelGenerator llamará a Chunk.gd para crear un nuevo fragmento basado en los datos del mapa para esa posición. Chunk.gd procesará todos los bloques del mapa (máximo 32x32x21 bloques) y los instanciará, así como los muebles y los enemigos.
4. El jugador tendrá sus estadísticas, salud y equipo inicializados. Si se trata de una partida nueva, se aplicarán los valores predeterminados. De lo contrario, los datos se cargarán desde la memoria.
5. Si el jugador lleva algún arma, los controladores de armas llamados EquippedLeft y EquippedRight inicializarán el arma. Comprobarán si es un arma cuerpo a cuerpo o a distancia y ajustarán sus estadísticas en consecuencia.
6. La interfaz de usuario inicializará todos los menús, como el menú de creación, el inventario, el diario de misiones y el mapa general.
7. Al abrirse, el mapa general mostrará el área que rodea al jugador y que puede explorarse. Este mapa muestra un terreno infinito alrededor del jugador, basado en la semilla del mapa. El jugador puede usar este mapa general para determinar su próximo movimiento.


# Cargas automáticas
El juego tiene lo siguiente [autoloads](https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html)Estos se inicializarán cuando se inicie el juego, serán accesibles desde cualquier script y persistirán incluso después de cambiar de escena:

| Carga automática | Descripción |
| ------------- | ------------- |
| ItemManager | Aquí se realiza la manipulación de objetos en tiempo de ejecución. Gestiona aspectos como el inventario, la recarga, el equipamiento y la fabricación.
| Ayudante | Carga automática general con funciones auxiliares genéricas. También contiene las subfunciones auxiliares que se enumeran en la tabla siguiente |
| Datos del juego | Carga los datos de la carpeta /mods y permite que cualquier script acceda a ellos |
| Datos de ejecución | Carga los mods según el orden de carga de mods y los combina para su uso en el juego |
| Gloot | Un complemento que proporciona funcionalidad para el inventario. No accedemos a él directamente, solo a través de las clases proporcionadas por el complemento. |
| General | Un script de carga automática general para funciones que no encajan en ningún otro lugar |
| Gestor de recetas de elaboración | Gestiona la visibilidad y disponibilidad de las recetas de elaboración y realiza comprobaciones de los requisitos |
| QuestManager | Carga automática de la [QuestManager](https://github.com/Chevifier/QuestManager) Complemento. Gestiona el seguimiento de misiones en el backend.

## Nakama

Complemento multijugador para Godot. Consulte la documentación en [Github](https://github.com/heroiclabs/nakama-godot)


## Ayudante

Una función auxiliar general de carga automática que proporciona funciones genéricas utilizadas en muchos otros scripts. También incluye varias subfunciones auxiliares:
| Subcarga automática | Descripción |
| ------------- | ------------- |
| json_helper | Se puede acceder a través de `Helper.json_helper`Proporciona funciones para manipular archivos y datos JSON |
| map_manager | Se puede acceder a través de `Helper.map_manager`Proporciona funciones para acceder y manipular los datos del mapa. Por ejemplo, para descubrir qué hay alrededor del jugador y para construir y destruir bloques y muebles.
| overmap_manager | Se puede acceder a través de `Helper.overmap_manager`. Crea el mapa general basado en la semilla del mapa. Define regiones y llena las cuadrículas del mapa general con el terreno que el jugador explorará. Proporciona datos sobre el terreno al `levelgenerator` y la interfaz gráfica de usuario del mapa general. |
| ayudante de la misión | Se puede acceder a través de `Helper.quest_helper`Proporciona funciones para acceder y manipular misiones. Traducirá los datos JSON de las misiones en ScriptedQuests reales para que el jugador las complete. Responde a señales de todo el juego para realizar un seguimiento del progreso de la misión.
| time_helper | Se puede acceder a través de `Helper.time_helper`Proporciona funciones para acceder al tiempo transcurrido en el juego. Esto se puede utilizar para eventos basados ​​en el tiempo, deterioro de la comida, ciclo día/noche y otras cosas.
| save_helper | Se puede acceder a través de `Helper.save_helper`Proporciona funciones para guardar y cargar datos.
| signal_broker | Se puede acceder a través de `Helper.signal_broker`Este es un punto central a través del cual enviar señales para que otros scripts reaccionen. El caso de uso más importante es permitir señales entre las entidades del juego (bloques, enemigos, contenedores) y la interfaz de usuario del jugador.
| administrador_de_tareas | Se puede acceder a través de `Helper.task_manager`. Proporciona funciones para descargar tareas a un hilo separado. Un ejemplo sencillo sería: `my_data = await Helper.task_manager.create_task(process_my_file.bind(filecontents)).completed`. Solo necesitamos usar esto para funciones que muestren picos en el perfilador y caídas de FPS. Consulte Chunk.gd para la aplicación actual |


## datos del juego
Gestión centralizada de los datos del juego. Los datos se cargan desde el `/mods` carpeta. Esto incluye todos los datos de entidades y sprites. Se puede acceder a todos los datos a través de esta carga automática. Está estrechamente vinculado al Editor de contenido. Cuando se modifican los datos mediante el editor de contenido, la carga automática de Gamedata actualizará las entidades relacionadas si es necesario y guardará los datos. Los datos se almacenan en su propia clase, accesible a través de `Gamedata.mods.by_id("Dimensionfall").items` o `Gamedata.mods.by_id("Dimensionfall").furnitures` o `Gamedata.mods.by_id("Dimensionfall").maps`y así sucesivamente. Cada mod en Gamedata.mods tiene su propia lista de objetos, muebles, criaturas, etc.

## datos de tiempo de ejecución
Todos los mods activados se combinan según el orden de carga al iniciar el juego. Esto significa que los elementos añadidos mediante mods se agregan a Runtimedata.items. Los elementos con el mismo ID sobrescribirán los elementos cargados previamente. ¡Esto te permite aplicar cualquier cambio que desees mediante mods! Runtimedata no debe modificarse ni guardarse, y los mods se vuelven a combinar cada vez que se inicia el juego. No tiene concepto de mods, ya que se trata de un único mod.

## Gloot
Una carga automática proporcionada por el [Gloot addon](https://github.com/peter-kish/gloot)No accedemos a esto directamente, sino a través de las clases del complemento. Proporciona funcionalidad para el inventario.

## Administrador de elementos
Gestiona la creación, destrucción y movimiento de objetos en el inventario del jugador y en otros lugares.

## General
Un script de carga automática general para funciones que no encajan en ningún otro lugar.

## Gestor de recetas artesanales
Gestiona la visibilidad y disponibilidad de las recetas de elaboración y realiza comprobaciones de los requisitos.

## Administrador de misiones
Carga automática del [QuestManager](https://github.com/Chevifier/QuestManager) Complemento. Gestiona el seguimiento de misiones en el servidor.


# Generación de mapas
El mundo del juego es generado por el `overmap_manager` como `OvermapGrid`s lleno de `map_cell`s. Estos son solo datos para ser utilizados por el `level_generation` script, que instanciará el `maps` como `chunks` dentro del juego, de modo que el jugador tenga terreno sobre el que caminar.

## Mapa general
El `overmap_manager` Utiliza estos pasos para producir el mapa general final:
1. Se utiliza un mapa de ruido para generar la capa base de campos y bosques.
2. El `overmap areas` se colocan en el mapa. Un ejemplo de un `overmap area` es una ciudad
3. Si las ciudades se ubican en una cuadrícula, estarán conectadas por carreteras.
4. Los mapas tácticos se colocan en la cuadrícula de forma aleatoria, evitando las carreteras y áreas existentes.
5. El producto final será una cuadrícula de 100x100 rellena con celdas que tienen un ID de mapa.

La ventana del mapa general del juego es un elemento de la interfaz de usuario que genera una cuadrícula de casillas para representar el mapa general. Ayuda al jugador a explorar el mundo y puede señalar los objetivos de las misiones.

## Mapa táctico
Un mapa táctico es una cuadrícula de mapas creada manualmente. Esta cuadrícula puede tener cualquier dimensión, desde 1x1 hasta 64x64 o más grande. Está compuesta por mapas. En el juego, el `overmap_manager` El programa leerá un mapa táctico y, para cada una de sus celdas, colocará el mapa asociado en el mapa superpuesto. Esto permite crear áreas estáticas dentro del terreno generado proceduralmente. Al crear un mapa táctico, introduzca las dimensiones para establecer el tamaño deseado. Se mostrará una cuadrícula y podrá seleccionar uno de los mapas para dibujar sobre ella. Combinando mapas, puede crear una ciudad, un bosque, una montaña o cualquier otra cosa que desee.

## Edición de mapas
Los mapas se editan en el Editor de contenido y contienen datos para un máximo de 32x32x21 bloques. Al editar un mapa, se presenta una cuadrícula fija de 32 casillas. Puedes elegir casillas de la paleta en el editor para pintarlas en la cuadrícula y crear lo que quieras. También puedes añadir enemigos y muebles. Usa áreas para habilitar la generación aleatoria cuando el mapa se instancia en el juego. Por ejemplo, pisos aleatorios o posiciones de mobs. Para que un mapa aparezca en el juego, debe añadirse a un `tacticalmap` o un `overmap area`

## Áreas superpuestas
Se puede usar para crear áreas en el mapa general, como biomas. El área se puede modificar en el editor de contenido y está compuesta por regiones. Al generarse en el mapa general, definirá un punto central y dibujará un círculo a su alrededor, luego deformará la forma de manera aleatoria. Cada región del área cubrirá un porcentaje determinado. Esto permite crear un centro, un anillo intermedio y anillos exteriores. Por ejemplo, una ciudad está compuesta por un anillo interior. `urban` región, una zona media `suburban` región y un exterior `field` región. Puede especificar el tamaño del área en el editor.

Cada región contiene una lista de mapas. Puedes agregar mapas a las regiones arrastrándolos desde la lista de mapas. Luego puedes asignar un peso a cada mapa, que controlará las probabilidades de que aparezca. El editor también tiene una `generation` pestaña para facilitar la visualización del producto final. `overmap_manager` Se generarán 10 áreas en el mapa general, intentando mantenerlas separadas.


# Editor de contenido
El contenido del juego se crea y modifica en el editor de contenido. Se puede crear contenido en Godot, pero si el juego se exporta y el usuario final no tiene Godot, aún puede abrirlo y agregar contenido desde allí mediante mods. Los mods se pueden agregar a través de los menús: `content editor -> mod manager -> add/remove mods`.

El contenido del editor se presenta en listas, donde cada elemento tiene su propio ID. La interfaz permite añadir, duplicar, modificar y eliminar elementos de las listas. Los mapas y los mapas tácticos se almacenan en archivos JSON independientes. El resto se almacena en un archivo JSON por tipo. Por ejemplo, todos los objetos están en un archivo JSON, los muebles en otro, etc. Los archivos JSON son cargados por Gamedata, y estos son los que se manipulan mediante el editor. No se guarda nada en el disco hasta que el usuario pulsa el botón de guardar.

Cada tipo tiene su propio editor. La mayoría de los editores son simplemente formularios para que el usuario manipule el JSON subyacente. También se puede usar un editor externo para editar el JSON directamente. Sin embargo, el editor de contenido también gestiona las referencias entre entidades. Así, cuando un grupo de elementos tiene sus elementos configurados, estos obtienen una referencia al grupo. Cuando se elimina un elemento de ese grupo, también desaparece del mismo. Esto evita que se creen elementos inexistentes. Se establecen múltiples referencias en diferentes relaciones entre entidades.

Los sprites se almacenan en la misma carpeta que los datos JSON de cada tipo. A cada elemento se le puede asignar un sprite. Algunos elementos incluso pueden compartir el mismo sprite. Para añadir nuevos sprites, puedes colocarlos en la carpeta correspondiente dentro de la carpeta /mods. 



# Guardando y cargando
El Helper.save_helper guardará y cargará datos cuando cambie la escena o se cargue un juego. Los datos guardados se almacenan en la carpeta de datos del usuario. En Godot, ve a los botones del menú en la parte superior y haz clic en Proyecto -> Abrir carpeta de datos del usuario para abrir la carpeta de datos del usuario. Esto podría ser `C:\Users\User\AppData\Roaming\Godot\app_userdata\Dimensionfall` en ventanas.

Cuando el jugador presiona escape y hace clic en `save game` o `return to main menu`Todos los datos relevantes se almacenan. 
- La semilla del mapa general se genera una sola vez y se guarda al iniciar el juego. Después de eso, el estado del mapa general se guarda y se carga.
- El inventario del jugador se serializa mediante el complemento Gloot y se deserializa cuando se carga el juego.
- El juego realiza un seguimiento del equipo del jugador en ItemManager.player_equipment, que luego se guarda y se carga en json.
- Las estadísticas del jugador se leen directamente desde el nodo del jugador y se guardan como un diccionario en un archivo JSON que se carga posteriormente.
- El mapa en el que el jugador está jugando actualmente se guarda al regresar al menú principal. Cada fragmento se descarga y se guarda en un diccionario llamado Helper.loaded_chunk_data. Este diccionario se guarda luego en un archivo JSON para la coordenada actual en el mapa general, que se almacenará en un archivo como `C:\Users\User\AppData\Roaming\Godot\app_userdata\Dimensionfall\save\2024-05-25T213346\map_x1_y4\map.json`

Cuando el jugador presiona el botón de cargar juego en el menú principal, `_on_load_game_button_pressed` Se llama a la función en scene_selector y se cargan todos los datos guardados previamente. 
