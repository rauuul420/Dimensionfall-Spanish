# Lista de características

Esta es una lista extensa y detallada de las características del juego Dimensionfall. Estas características están implementadas a menos que una nota indique lo contrario.


### Combatir
Actualmente, se han implementado el combate a distancia y el combate cuerpo a cuerpo. 
- Las armas de una mano se pueden usar con ambas manos, mientras que las armas de dos manos ocuparán ambas ranuras.
- Cargar y descargar armas y cargadores.
- Se puede recoger munición y otros objetos de los cadáveres de los enemigos.
- Mejora tu precisión adquiriendo habilidades de combate a distancia y controlando el retroceso.


### Inventario
- Hay que controlar el peso y el volumen.
- Puedes arrastrar y soltar elementos desde diferentes contenedores.
- Interactúa con tus ranuras de equipo

![Inventario de Catax](Media/Dimensionfall_inventory.png)

### Mapa general

El juego creará un mapa infinito para explorar. El mundo está dividido en regiones, cada una con sus propias oportunidades y desafíos. Un marcador indicará tu ubicación actual.

![Mapa superpuesto de Catax](Media/Dimensionfall_overmap.png)

Se generan ciudades y carreteras. Explora toda una red de carreteras y conoce la zona.

![Mapa superpuesto de Catax](Media/Catax_overmap_large.png)

### Diario de búsqueda
Sigue las misiones para guiarte a través de este peligroso mundo. Una misión puede tener múltiples objetivos, que se mostrarán paso a paso.

![Mapa superpuesto de Catax](Media/Dimensionfall_quest_journal.png)

### Estaciones de fabricación
Las estaciones de creación te permiten fabricar objetos mientras realizas otras tareas. Amplía tu base con todas las estaciones de creación que necesites. Para crear un objeto, introduce los ingredientes necesarios y añádelo a la cola. Una vez finalizado, el resultado aparecerá en el inventario del mueble. Usa el editor de contenido para convertir cualquier mueble en una estación de creación. 

![Mapa superpuesto de Catax](Media/Dimensionfall_crafting_station.png)


### Menú principal
- Juega el juego
- Cargar un juego
- Gestionar opciones
- Lea la documentación
- Gestionar el contenido del juego
![Dimensionfall_main_menu](Media/Dimensionfall_main_menu.png)


### Menú para añadir/eliminar mods

Gestiona el contenido del juego de forma dinámica mediante el menú Añadir/Eliminar Mods. Desde este menú, puedes activar, desactivar, reordenar e inspeccionar los mods sin salir del juego. Al seleccionar un mod, sus detalles se muestran a la derecha.

![Dimensionfall_add_remove_mod_menu](Media/Dimensionfall_add_remove_mod_menu.png)


### Editor de contenido
El contenido de este juego se crea en el editor de contenido. A la izquierda, puedes seleccionar el contenido que deseas editar. Puedes editar mapas tácticos, mapas, objetos, casillas, muebles, criaturas, grupos de objetos, ranuras de equipamiento, habilidades y estadísticas.
- Todo el contenido se guarda en formato JSON, lo que permite editar los archivos manualmente o utilizando un editor externo si se desea.
- El contenido se carga como mods, incluso el contenido principal. Coloca todos tus archivos JSON y sprites en /mods/tumod/ y el juego podrá leerlos.
![Editor de contenido de Catax](Media/Catax_content_editor.png)


### editor de mapas tácticos
Un mapa táctico se compone de mapas y permite definir un área fija en un mundo infinito. Esto te permite crear un mapa más grande. Puedes especificar cualquier tamaño y empezar a rellenar la cuadrícula con mapas de la selección de la derecha. Las casillas de la cuadrícula se pueden rotar para conectar correctamente las carreteras y crear simetría. Los mapas tácticos aparecerán aleatoriamente en el mapa.
![Editor de mapas tácticos de Catax](Media/Catax_tacticalmap_editor.png)


### Editor de mapas
Un mapa tiene un tamaño fijo de 32x32 y un máximo de 21 niveles de altura, que van desde -10 hasta +10. 
- Coloca baldosas, criaturas y muebles en la cuadrícula y da rienda suelta a tu creatividad.
- Los controles te permiten moverte hacia arriba y hacia abajo, hacer zoom, rotar, copiar/pegar y mucho más.
- Crea pinceles personalizados con el Compositor de pinceles, lo que te ayudará a pintar terrenos de forma rápida (y aleatoria).

![Editor de mapas de Catax](Media/Catax_map_editor.png)

Defina áreas en el mapa para permitir una mayor aleatorización:
![Editor de mapas de Catax](Media/Catax_map_editor_areas.png)

Usa la vista previa del mapa para equilibrar las áreas y ajustar la aleatorización. Pulsa el botón "Generar" varias veces para visualizar la probabilidad de aparición y la cobertura del área.
![Editor de mapas de Catax](Media/Catax_map_editor_preview.png)

Con el editor de áreas, puedes ajustar el área a tu gusto. Establece las proporciones de las casillas, la probabilidad de aparición y excluye otras áreas de la generación. También puedes cambiar el nombre del área y habilitar la rotación aleatoria.
![Editor de mapas de Catax](Media/Catax_map_editor_area_editor.png)
	

### Editor de elementos
Te permite editar y añadir objetos al juego. Cada objeto funcionará según tu configuración. Puedes usarlo como arma o como comida, ya que ambas funciones no son excluyentes. Especifica las propiedades del objeto en el práctico editor, que incluye información emergente y controles para ayudarte a introducir los valores correctos.
![Editor de elementos de Catax](Media/Catax_item_editor.png)


Un objeto puede tener cualquier número de recetas de fabricación. Usa el editor de recetas para especificar los requisitos de este objeto. Para añadir objetos a la receta, arrástralos desde el lado izquierdo de la ventana hasta la receta.
![Editor de elementos de Catax](Media/Catax_crafting_editor.png)

### editor de mosaicos
Te permite especificar las propiedades de los mosaicos y añadir otros nuevos. Los sprites se cargan desde /mods/core/tiles, así que coloca tus sprites allí si quieres añadir nuevos.
![Editor de mosaicos de Catax](Media/Catax_tile_editor.png)


### Editor de la mafia
Te permite crear nuevas criaturas y configurarlas. Luego puedes usar el editor de mapas para colocarlas en el mapa.

![Editor de mobs de Catax](Media/Catax_mob_editor.png)


### editor de muebles
Crea muebles nuevos o edita los que ya tienes con este editor. Después de crearlos, puedes colocarlos en el mapa con el Editor de mapas. Pasa el cursor sobre los controles para obtener información sobre cada uno de ellos.

![Editor de muebles Catax](Media/Catax_furniture_editor.png)



### editor de grupos de elementos
Especifica grupos de objetos, que se utilizan para generar objetos en diversas ubicaciones, como contenedores y cadáveres. Al especificar la probabilidad y la cantidad de aparición, se ofrece una amplia variedad de posibilidades. Para añadir objetos a un grupo, arrástralos desde el lado izquierdo de la pantalla hasta el grupo.

![Editor de muebles Catax](Media/Catax_itemgroup_editor.png)



### editor de atributos del jugador
En línea con la posibilidad de modificación completa, puedes configurar los atributos del jugador mediante el editor. Esto permite una mayor inmersión y personalización. Una vez creado el atributo, puedes arrastrarlo desde el menú a los objetos para modificarlo. Por ejemplo, puedes hacer que una manzana aumente su atributo de "alimento" al usarse.

![Editor de muebles Catax](Media/Catax_playerattribute_editor.png)



### editor de ranuras para prendas de vestir
¿Alguna vez quisiste crear una armadura única, pero no tenías espacio para colocarla? Ahora puedes añadir tus propios espacios. Estos aparecerán en el inventario del jugador y le permitirán equipar armaduras en ellos.

![Editor de muebles Catax](Media/Catax_wearableslots_editor.png)



### Editor de estadísticas
Añade tus propias estadísticas al juego. Las estadísticas se utilizan para mejorar el uso de habilidades al fabricar objetos o usar un arma.

![Editor de muebles Catax](Media/Catax_stats_editor.png)


### editor de habilidades
¡Crea todas las habilidades que necesites! Algunas están incluidas en el mod principal, pero puedes añadir fácilmente las tuyas en el Editor de Habilidades. Una vez creadas, úsalas en la configuración de armas o recetas para otras entidades.

![Editor de muebles Catax](Media/Catax_skills_editor.png)


### editor de misiones
Crea una historia para el jugador con el editor de misiones. Estas aparecerán en el diario de misiones del juego. Te permite guiar al jugador a su próximo destino y recompensarlo por su esfuerzo. Hay varios tipos de pasos disponibles. Puedes configurar muchos detalles y añadir una pista para el jugador.

![Editor de muebles Catax](Media/Catax_quest_editor.png)


### editor de área de mapa superior
Este editor permite definir un área que se generará en el mapa general. Un área puede crearse a partir de un número ilimitado de regiones. En esta imagen, una ciudad está definida por las regiones urbana, suburbana y rural:

![Editor de muebles Catax](Media/Dimensionfall_overmaparea_editor.png)

Cada región del área abarcará un círculo diferente alrededor del centro, dependiendo del radio. Usando el generador, puedes ver rápidamente el resultado de tu configuración:

![Editor de muebles Catax](Media/Dimensionfall_overmaparea_generator.png)


Con esta herramienta puedes añadir nuevos mapas a las ciudades o crear una zona completamente nueva con los mapas que hayas creado. Esto permite, por ejemplo, la creación de biomas.

### Editor de grupos de la mafia
Con este editor, puedes crear tu propio grupo de criaturas añadiendo un sprite, nombre, descripción y monstruos (que puedes arrastrar directamente desde la sección del editor de criaturas dentro del gestor de contenido). Incluso puedes configurar la probabilidad de aparición de los monstruos individualmente modificando su peso, todo dentro del editor.

![Editor de grupos de mobs de Catax](Media/Dimensionfall_mobgroup_editor.png)

### editor de facciones de la mafia
(En desarrollo), puedes crear una nueva facción personalizada y asignarla a cualquier monstruo o grupo de monstruos que desees. Puedes configurar si esta facción es amistosa, neutral u hostil hacia otros monstruos, grupos de monstruos y facciones.

![Editor de acción de Catax_mobfaction](Media/Dimensionfall_mobfaction_editor.png)
