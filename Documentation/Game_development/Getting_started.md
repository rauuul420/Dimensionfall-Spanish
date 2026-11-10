# Cómo empezar con el desarrollo de videojuegos

Esta guía está dirigida a nuevos colaboradores. Te enseñará los conceptos básicos para contribuir al proyecto. Usa tus habilidades para realizar cambios en el área que más te guste. Puede ser programación, modificación, gráficos, música, pruebas y muchas otras cosas. Si eres un jugador que lee esta página, puedes contribuir de la siguiente manera: [submitting an issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue) ¡Para informar de errores!


## Configuración básica
Estos pasos te ayudarán a configurar Godot y GitHub para que puedas empezar a contribuir a este repositorio:
1. Descargar [Godot](https://godotengine.org/download/) desde su sitio web o desde [Steam](https://store.steampowered.com/app/404790/Godot_Engine/).
2. Crear un [GitHub account](https://docs.github.com/en/get-started/start-your-journey/creating-an-account-on-github) Si aún no tienes uno.
3. Instalar [git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git) para una interfaz de línea de comandos o [GitHub Desktop](https://docs.github.com/en/desktop/overview/getting-started-with-github-desktop) para una interfaz gráfica (recomendado).
4. Haz un fork de este repositorio. Ver [this guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo?tool=webui) si usa git o [this guide](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-and-forking-repositories-from-github-desktop) Si utilizas GitHub Desktop (recomendado).
5. Clona tu bifurcación en tu ordenador local si aún no lo has hecho. Ahora puedes iniciar Godot y [open the project](https://docs.godotengine.org/en/stable/tutorials/editor/project_manager.html#opening-and-importing-projects)Una vez abierto, pulsa el botón *Reproducir* en la esquina superior derecha para probarlo. Si eres nuevo en Godot, lee su [getting started](https://docs.godotengine.org/en/stable/getting_started/step_by_step/index.html) guía.


## Haz tu primer cambio
Estos pasos te muestran cómo realizar un cambio sencillo en el juego sin necesidad de programar.
1. Antes de que puedas comenzar a hacer cambios, crea una rama usando [git](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository) o [GitHub Desktop](https://docs.github.com/en/desktop/overview/getting-started-with-github-desktop#making-changes-in-a-branch) (recomendado)
2. Abre el proyecto en Godot si aún no lo has hecho.
3. Pulsa el botón de reproducción en la esquina superior derecha para comenzar el juego.
4. Pulsa «Editor de contenido» en el menú principal y, a continuación, «Gestor de contenido».
5. Encuentra la lista de "grupos de elementos" en la parte izquierda del gestor de contenido. Puedes contraer las demás listas haciendo clic en su nombre.
6. Haz clic en el botón + junto a 'Grupos de artículos' para añadir un nuevo grupo de artículos. Aparecerá una ventana emergente que te pedirá un ID. Tienes que hacerlo único, por ejemplo `basic_mob_loot_00`.
7. El nuevo grupo de elementos se agrega a la lista. Desplácese hacia abajo en la lista y haga doble clic en el `basic_mob_loot_00` itemgroup para editarlo.
8. Se abre un nuevo editor en una pestaña que le permite cambiar las propiedades del grupo de elementos. Haga clic en la imagen que está al lado de `Sprite:` Para seleccionar un sprite para este grupo de objetos. Puede ser cualquier sprite, por ejemplo, el machete.
9. Introduzca un nombre y una descripción, por ejemplo `Mob loot` y `Some loot the player can find in mobs`.
10. No es necesario cambiar el tipo de grupo, pero puede pasar el cursor sobre el menú desplegable para leer lo que hace.
11. En la lista de la izquierda, expanda la lista de "elementos" si está contraída. Arrastre los elementos de la lista al área de "elementos" del editor de grupos de elementos. Esto le permite crear una lista de elementos que desea generar cuando se utilice este grupo de elementos.
12. No es necesario editar la probabilidad ni el recuento mínimo y máximo, pero puedes colocar el cursor sobre los controles para ver qué hacen.
13. Pulsa «Guardar» y «Cerrar» para finalizar los cambios. Ahora que hemos terminado, cierra el juego y confirma los cambios en la rama.
14. Para confirmar tus cambios en tu rama, lee [this guide](https://docs.github.com/en/desktop/overview/getting-started-with-github-desktop#making-changes-in-a-branch) para Github Desktop (recomendado) o [this guide](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository) para git
15. Por último, [create a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request#creating-the-pull-request)Alguien con permisos para fusionar tu solicitud la revisará y la fusionará si es aceptada. También aparecerá un mensaje sobre tu solicitud de extracción en el servidor de Discord, por lo que alguien la revisará pronto.


# Agregar/editar contenido
Cualquiera puede agregar o editar contenido para el juego. No necesitas saber nada sobre programación para empezar a trabajar en el contenido. Si no has trabajado antes en un proyecto de GitHub, asegúrate de hacer lo siguiente: [basic setup](basic-setup) y lee el [make your first change](make-your-first-change) guía.

## Trabajar con el editor de contenido
Para editar el contenido, primero inicia el juego. En el menú principal, haz clic en «Editor de contenido» y luego en «Gestor de contenido». Verás la ventana principal del gestor de contenido. En el lado izquierdo, podrás ver los tipos de contenido que puedes añadir o editar.

### Controles básicos
Los controles básicos son los siguientes:
- Haz clic en el nombre de la lista para contraerla/expandirla.
- Haz doble clic en un elemento de la lista para editarlo. Se abrirá un editor en una nueva pestaña.
- Haz clic en el `+` Haz clic en el signo junto al nombre de una lista para agregar un nuevo elemento a esa lista. Aparecerá una ventana emergente donde podrás ingresar el ID de este nuevo elemento. Debe ser único. Una vez que hayas ingresado el ID, haz clic en el botón correspondiente. `Ok` y el nuevo artículo estará al final de la lista.
- Haz clic en el `D` Para duplicar un elemento de la lista, aparecerá una ventana emergente con el ID del elemento duplicado. Edite el ID para que sea único y pulse `Ok` Para agregarlo a la lista, tendrá las mismas propiedades que el elemento original. Solo ha cambiado el ID.
- Haz clic en el `-` para eliminar un elemento de la lista.

### Agregar/actualizar sprites
Si eres artista y quieres actualizar o añadir sprites, tendrás que usar Godot o el explorador de archivos para navegar hasta `Mods/Dimensionfall/` y una de las subcarpetas. Desde allí, puedes editar, reemplazar o añadir un sprite. Las carpetas TacticalMaps y Maps están excluidas de esto porque TacticalMaps aún no tiene un sprite y el sprite para los mapas lo genera el editor de contenido.
- Si editaste o reemplazaste un sprite, se cargará en el juego en el próximo lanzamiento.
- Si agregaste un sprite, se cargará, pero debes asignarlo a una de las entidades en el editor de contenido. Por ejemplo, ve a la lista de elementos y haz doble clic en uno de los elementos para editarlo. El editor de elementos se abrirá en una nueva pestaña. Luego, haz clic en la imagen que está al lado de `Sprite:` y selecciona el sprite que añadiste. Por último, pulsa `Save` para guardar tus cambios


## Modificar el código del juego
Primero, haz el [basic setup](basic-setup). Lea también el [game architecture](https://github.com/Khaligufzel/CataX/blob/main/Documentation/Game_design/Game_architecture.md) Documento para obtener una comprensión básica de la estructura del código del juego.

### Navegando el proyecto
- La mayoría de los scripts del juego están en `/Scripts`Para el editor de contenido, estarán en `/Scenes/ContentManager`.
- Familiarícese con el [autoloads](https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html) como se describe en el [game architecture](https://github.com/Khaligufzel/CataX/blob/main/Documentation/Game_design/Game_architecture.md) documenta esto, ya que te proporcionará acceso centralizado a funciones que puedes usar en cualquier lugar.
- También puedes mirar qué [pull requests](https://github.com/Khaligufzel/CataX/pulls?q=is%3Apr) se han abierto previamente. Desde allí puedes abrir una de las solicitudes de extracción y ver qué elemento del juego se cambió y los archivos asociados con él. Si ves una solicitud de extracción que modificó parte de la generación del mapa, haz clic en ella y ve a `files changed` en la solicitud de extracción para ver los archivos que se modificaron.
- En Godot, usa `ctrl+shift+f` para encontrar texto en archivos. Por ejemplo, si quieres saber qué scripts modifican o leen el nodo del jugador, busca "player".

### Elige qué modificar
Quizás tengas algunas ideas sobre en qué trabajar, o tal vez quieras corregir un error. Aquí tienes algunos consejos para empezar:
- Mira qué [issues](https://github.com/Khaligufzel/CataX/issues) Están abiertos y asigna uno a tu nombre.
- En Discord, consulta el canal de la hoja de ruta y elige un tema en el que trabajar.
- También en Discord, explore los canales y vea qué ideas se han compartido, por ejemplo en el `your-ideas` canal.

### Datos del juego y jugabilidad
El editor de contenido y el juego están estrechamente relacionados. Los datos que necesitas en el juego deben definirse en el editor de contenido. Y viceversa: si defines algo en el editor de contenido, asegúrate de que también se implemente en el juego.

Aquí tienes un pequeño ejemplo de un cambio que puedes hacer. Añadiremos `weight` a muebles móviles.
1. Abierto `FurnitureEditor.tscn` En Godot, ve a la vista 2D. En el formulario, agrega dos controles en un lugar apropiado. Agregaremos una etiqueta y un control numérico.
2. Nombra la etiqueta `WeightLabel` y la caja giratoria `WeightSpinbox`.
3. Seleccione el `WeightSpinbox` en el editor y ve al inspector. Encuentra el `Tooltip` propiedad e ingrese la descripción para este control. Por ejemplo: `Specify the weight in kg for this furniture. A larger number means it will be heavier and harder to push. A smaller number means it will be lighter and easier to push. Only applies to moveable furniture`.
4. Ve al `FurnitureEditor.gd` script. Asigne el control spinbox a una variable de exportación de la siguiente manera: `@export var weight_spinbox: SpinBox = null`.
5. En el `load_furniture_data` función a la que añadimos las líneas:
   ```
       if weight_spinbox:
               weight_spinbox.value = dfurniture.weight
   ```
6. En el `_on_save_button_button_up` función agregamos la línea `dfurniture.weight = weight_spinbox.value`Ahora la propiedad se guardará y se cargará desde JSON.
7. En el `FurniturePhysics.gd` guion, en el `construct_self` función, agregamos la línea `mass = furnitureJSONData.get("weight", 0)` ajustar la masa según el peso.
8. Abre el editor de contenido, selecciona uno de los muebles móviles, establece la propiedad de peso y guarda los cambios. Ahora, el mueble utilizará la propiedad de peso que has implementado.
9. La misa se leerá en el `player.gd` script cuando el jugador comienza a empujar los muebles.
10. Siga los pasos 14 y 15 del [make your first change](make-your-first-change) párrafo para confirmar tus cambios.

Fue un cambio sencillo. Por supuesto, la aplicación práctica de los datos JSON variará en cada caso y algunos serán más complejos que otros. Para lograr lo que deseas, pide ayuda y consulta guías sobre el tema en el que quieres trabajar.

### desarrollo de interfaz de usuario
La interfaz de usuario es otra área que es algo accesible para los colaboradores que comienzan. Esto significa menús, ventanas e indicadores de estado. Las escenas principales están en `/Scenes/UI`Estas escenas se añaden a la `HUD` nodo en `level_generation.tscn` para que se muestren al jugador cuando se presione un botón.

Ten en cuenta que la interfaz de usuario se comunica con el mundo del juego a través de cargas automáticas. Por ejemplo, el menú de inventario funciona junto con el `ItemManager` (`/Scripts/item_manager.gd`). Algunos elementos del juego, como el jugador y algunas ventanas, se comunican a través de `Helper.signal_broker`Para saber qué se conecta con qué, pulse `ctrl+shift+f` en Godot y buscar `Helper.signal_broker` y aparecerá una lista.

Los iconos utilizados en la interfaz de usuario se pueden encontrar en `/Textures` y `/Images/Icons`. A veces no necesitas usar un icono, y un botón con un `x` o flecha `->` será suficiente. En el caso del menú de personajes, los iconos se cargan desde los datos del mod en `/Mods/Core/Stats` y `/Mods/Core/Stats`.
## Ejecutando pruebas
El proyecto utiliza el [GUT](https://github.com/bitwes/Gut) Complemento para pruebas unitarias. Las pruebas se gestionan dentro del editor de Godot.

Abre el proyecto en Godot y mira el panel inferior de la ventana. Junto a `Output` y `Debugger` encontrarás el `GUT` Pestaña. Haga clic en ella para abrir el panel de control, donde podrá ejecutar todas las pruebas o filtrarlas por directorio o archivo.


## Desarrollar utilizando el Codex de OpenAI:
Ver [Documentation/Game_development/Working_with_codex.md](https://github.com/Dimensionfall/Dimensionfall/blob/main/Documentation/Game_development/Working_with_codex.md).
