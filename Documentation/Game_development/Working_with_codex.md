# Trabajando con el códice

Esta guía se basa en https://github.com/knavillus1/codex_bootstrap/tree/main. He actualizado algunas partes para que se ajusten mejor al proyecto Dimensionfall.
Puedes usar esta guía si tienes acceso a https://chatgpt.com/codex

## Características

- Creación de archivos PRD y Task dentro de Codex
- Orquestación y gestión de tareas en Codex

## Configuración por primera vez
1. En el códice, ve a https://chatgpt.com/codex/settings/general y elimina cualquier instrucción personalizada que hayas utilizado anteriormente.
2. Ve a https://chatgpt.com/codex/settings/environments, selecciona o crea tu bifurcación del juego Dimensionfall conectada a GitHub.
3. Pulsa Editar -> copia y pega esto en el cuadro de texto del script de inicio y guarda:
```
# 1. Download the latest Godot 4.x Linux 64-bit binary
echo "⬇️  Fetching Godot 4.4.1 …"
wget --retry-connrefused --tries=5 -q --show-progress -O /tmp/godot.zip \
   https://github.com/godotengine/godot/releases/download/4.4.1-stable/Godot_v4.4.1-stable_linux.x86_64.zip

# 2. Unzip and install
unzip /tmp/godot.zip -d /usr/local/bin/
sudo mv /usr/local/bin/Godot_v4.4.1-stable_linux.x86_64 /usr/local/bin/godot
sudo chmod +x /usr/local/bin/godot

# 3. Cleanup
rm /tmp/godot.zip

# 4. Verify installation
godot --version

# 5. Check if project.godot exists
if [ -f "project.godot" ]; then
  echo "✅ project.godot found in current directory."
else
  echo "❌ project.godot NOT found in current directory."
fi

# 6. Install gdtoolkit we need to do formatting later
pip3 install "gdtoolkit==4.*"
```

## Preparación para su próximo reportaje
- Crea una nueva rama en tu bifurcación del juego Dimensionfall.
- Siempre que creamos la especificación de características o solicitamos Codex, utilizamos esta rama.

# Utiliza el Taskmaster para dejar que el códice haga el trabajo por ti.
Partiendo de cualquier descripción de una función, esta guía proporciona herramientas para automatizar la creación de archivos PRD y de lista de tareas, y luego el agente de Codex gestionará el trabajo a partir de esta lista, actualizándola según sea necesario. Hay cuatro fases:
- Creación de archivos PRD
- Creación de archivos de lista de tareas
- Completar las tareas
- Cierre de la función

Nota: cuando los scripts son **editados** por codex, también los **formateará** usando `gdformat path/to/modified_script.gd`Esto es necesario para asegurar que solo se utilicen tabulaciones para la indentación en lugar de espacios. Esto podría generar más cambios de los previstos para la función o tarea. El formato no altera el comportamiento, por lo que no debería haber problema.

## Creación de archivos PRD
- Agregar especificaciones de características y antecedentes a `.project-management/current-prd/prd-background/`:
    - `feature-specification.md` Contiene especificaciones de características con el nivel de detalle necesario. Obligatorio para la creación de PRD en Codex.
- Crear el PRD:
    - Codex: Inicie la tarea en modo *Código* con solo la frase **CreatePrd**.
- Habrá una sesión de preguntas y respuestas con el agente. En Codex, responda las preguntas y reanude en modo *Code* (el entorno se reiniciará).
- El resultado debería ser un archivo PRD en `.project-management/current-prd/`
- Fusionar la solicitud de extracción con la rama de destino

## Creación de archivos de lista de tareas
- Crear archivo de lista de tareas:
    - Codex: Inicia la tarea en modo *Código* con solo la frase **CreateTasks**.
- Preguntas y respuestas, contesta y haz clic en el código.
- El resultado debería ser un archivo de lista de tareas en `.project-management/current-prd/`
- Fusionar la solicitud de extracción con la rama de destino

## Capataz

- Una vez `.project-management\current-prd\tasks.*.md` Una vez creado, se puede utilizar el mensaje TaskMaster. Esto permitirá al agente comprometerse con una o más tareas en una sesión. El archivo de lista de tareas se actualizará como parte de la solicitud de extracción, marcando las tareas completadas y actualizando los archivos pertinentes según sea necesario.
- Inicie Codex en modo Código usando la frase *TaskMaster*. Esto hará que el agente haga referencia a `process-tasks-cloud.md`, que selecciona una o más tareas para completar en la sesión.

## Cierre de la función
- Realizar la revisión final de las características.
    - Codex: Inicie la tarea en modo *Código* con solo la frase **ClosePrd**.
- La revisión será una de las siguientes:
    - Resultado en cambios marcados: revise y vuelva a enviar el cierre.
    - Superar la revisión y cerrar el PRD: los archivos de características se mueven desde `current-prd` a `closed-prd`

## Tareas de mantenimiento
Además del flujo principal de PRD descrito anteriormente, también es posible que Codex encuentre y realice trabajos de mantenimiento. 
- Crea una rama en tu bifurcación (puedes llamarla `maintenance` Por ejemplo)
- Vaya a https://chatgpt.com/codex y en la barra de indicaciones seleccione su `maintenance` rama
- Introduzca la frase `MaintenanceTasks` y presione `code`Esto le indicará al Códice que siga las instrucciones en `.project-management/maintenance-tasks.md`
- Una vez terminado el códice, `.project-management\current-prd\tasks.*.md` se crearán. Probablemente habrá más tareas de mantenimiento de las que se pueden realizar con una sola solicitud de compra, por lo que tendrá que reducir la cantidad de tareas.
    - Crea una solicitud de extracción desde Codex para la lista de tareas.
    - Ve a las solicitudes de extracción en tu bifurcación y mira los archivos modificados.
    - Editar el creado `.project-management\current-prd\tasks.*.md` en GitHub para que puedas eliminar las tareas de mantenimiento que no necesitas
    - Confirma tus cambios y luego podrás fusionar la solicitud de extracción en tu `maintenance` rama en tu bifurcación.
- Ahora puede proceder como se indicó anteriormente ejecutando `TaskMaster` en el códice con tu `maintenance` Rama seleccionada.
- Una vez que TaskMaster haya terminado, cree una solicitud de extracción desde Codex, pruébela y fusione con su `maintenance` rama.
- Después, entrando `ClosePrd` en el códice con tu `maintenance` La rama seleccionada limpiará tus archivos y finalizará el trabajo.
- De la misma manera, crea la solicitud de extracción para closeprd en codex y únela a tu `maintenance` Ramificar y probarlo
- Crea una solicitud de extracción para la rama principal de dimensionfall desde el `maintenance` rama.
